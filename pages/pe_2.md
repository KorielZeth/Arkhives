# Les en-têtes du format PE

## L'en-tête MZ-DOS

L'en-tête MZ-DOS permet d'indiquer au système d'exploitation que le fichier est
bien un exécutable valide, dans l'éventualité où celui-ci serait exécuté sur MS-DOS (le système d'exploitation
à kernel monolithe ayant précédé Windows), à des fins de rétrocompatibilité.

Sa structure, IMAGE_DOS_HEADER, visible dans la librairie winnt.h (comme toute les structures liées au format PE), ressemble à ceci :

```cpp
typedef struct _IMAGE_DOS_HEADER {      // DOS .EXE header
    WORD   e_magic;                     // Magic number
    WORD   e_cblp;                      // Bytes on last page of file
    WORD   e_cp;                        // Pages in file
    WORD   e_crlc;                      // Relocations
    WORD   e_cparhdr;                   // Size of header in paragraphs
    WORD   e_minalloc;                  // Minimum extra paragraphs needed
    WORD   e_maxalloc;                  // Maximum extra paragraphs needed
    WORD   e_ss;                        // Initial (relative) SS value
    WORD   e_sp;                        // Initial SP value
    WORD   e_csum;                      // Checksum
    WORD   e_ip;                        // Initial IP value
    WORD   e_cs;                        // Initial (relative) CS value
    WORD   e_lfarlc;                    // File address of relocation table
    WORD   e_ovno;                      // Overlay number
    WORD   e_res[4];                    // Reserved words
    WORD   e_oemid;                     // OEM identifier (for e_oeminfo)
    WORD   e_oeminfo;                   // OEM information; e_oemid specific
    WORD   e_res2[10];                  // Reserved words
    LONG   e_lfanew;                    // File address of new exe header
  } IMAGE_DOS_HEADER, *PIMAGE_DOS_HEADER;
```
  
Il n'existe en général que deux entrées pertinentes dans cette structure : le WORD "e_magic" et le LONG "e_lfanew". 

L'on peut observer en regardant le premier que les deux premiers octets de cette structure, et donc les deux premiers octets de tout fichier PE sont 4D et 5A, formant ainsi 'MZ' en ASCII. Pour la petite histoire ceux-ci correspondent aux initiales de Mark Zbikowski, l'un des lead developers de MS-DOS (mentionné plus haut).

Le second est simplement un pointeur vers l'en-tête PE (que nous allons voir bientôt).
 
### Le segment DOS
 
 Un segment "bonus" logé entre les deux premiers en-têtes et considéré comme faisant partie du premier, le segment DOS est exécuté uniquement si le fichier n'a pu être reconnu comme un exécutable valide, ou si il est exécuté sous MS-DOS, et affiche le message "Ce programme ne peut pas être exécuté en mode DOS"
 
## L'en-tête PE
 
 L'en-tête PE est composé d'une signature et de deux structures, toutes les trois regroupées au sein de la structure _IMAGE_NT_HEADERS.
 ```cpp
 typedef struct _IMAGE_NT_HEADERS {
    DWORD Signature;
    IMAGE_FILE_HEADER FileHeader;
    IMAGE_OPTIONAL_HEADER32 OptionalHeader;
} IMAGE_NT_HEADERS32, *PIMAGE_NT_HEADERS32;
```

Pour commencer, le DWORD "Signature" doit être égal à 0x00004550, dont l'équivalent en ASCII est "PE\0\0" (les nullbytes \0 arrivant les premiers, endianness oblige.

### IMAGE_FILE_HEADER

La structure IMAGE_FILE_HEADER ressemble à ceci :

```cpp
typedef struct _IMAGE_FILE_HEADER {
    WORD    Machine;
    WORD    NumberOfSections;
    DWORD   TimeDateStamp;
    DWORD   PointerToSymbolTable;
    DWORD   NumberOfSymbols;
    WORD    SizeOfOptionalHeader;
    WORD    Characteristics;
} IMAGE_FILE_HEADER, *PIMAGE_FILE_HEADER;
```

*   Le WORD "Machine" indique l'architecture du processeur visée par notre exécutable. Les valeurs les plus communes sont donc 0x8864 (x64) et 0x14c (Intel 386 et affiliés) ; d'autres valeurs peuvent également y figurer, comme 0x0 pour des architectures inconnues, mais je ne m'y attarderai pas. 
*   Le WORD "NumberOfSections" indique la taille de notre table de sections, située immédiatement après nos en-têtes.
*   Le WORD "SizeOfOptionalHeader" indique la taille de l'en-tête Optionel (optionel dans le sens où certain types d'objets ne l'ont pas )

Et pour finir, le WORD "Characteristics" indique l'attribut associé au fichier ; par exemple 0x102 pour indiquer que le fichier est une exécutable 32 bits, ou 0x2000 pour indiquer que c'est une DLL


### IMAGE_OPTIONAL_HEADER32

La structure IMAGE_OPTIONAL_HEADER32 ressemble à ceci :

```cpp
typedef struct _IMAGE_OPTIONAL_HEADER {
    //
    // Standard fields.
    //

    WORD    Magic;
    BYTE    MajorLinkerVersion;
    BYTE    MinorLinkerVersion;
    DWORD   SizeOfCode;
    DWORD   SizeOfInitializedData;
    DWORD   SizeOfUninitializedData;
    DWORD   AddressOfEntryPoint;
    DWORD   BaseOfCode;
    DWORD   BaseOfData;

    //
    // NT additional fields.
    //

    DWORD   ImageBase;
    DWORD   SectionAlignment;
    DWORD   FileAlignment;
    WORD    MajorOperatingSystemVersion;
    WORD    MinorOperatingSystemVersion;
    WORD    MajorImageVersion;
    WORD    MinorImageVersion;
    WORD    MajorSubsystemVersion;
    WORD    MinorSubsystemVersion;
    DWORD   Win32VersionValue;
    DWORD   SizeOfImage;
    DWORD   SizeOfHeaders;
    DWORD   CheckSum;
    WORD    Subsystem;
    WORD    DllCharacteristics;
    DWORD   SizeOfStackReserve;
    DWORD   SizeOfStackCommit;
    DWORD   SizeOfHeapReserve;
    DWORD   SizeOfHeapCommit;
    DWORD   LoaderFlags;
    DWORD   NumberOfRvaAndSizes;
    IMAGE_DATA_DIRECTORY DataDirectory[IMAGE_NUMBEROF_DIRECTORY_ENTRIES];
} IMAGE_OPTIONAL_HEADER32, *PIMAGE_OPTIONAL_HEADER32;
```

Elle est globalement divisée en deux parties. Les 8 premières entrées sont caractéristiques du format COFF (Common Object File Format) duquel dérive le format PE. Les autres 
sont des extensions utilisées par le loader et linker (durant le processus de compilation) sous Windows.

Dans la première partie, les entrées notables sont 
*	Le WORD "Magic" qui indique si l'exécutable est optimisé pour des architectures 32 ou 64 bits.
*	Le DWORD "AddressOfEntryPoint", un pointeur vers l'entrypoint, aka l'endroit ou l'exécution commencera, de l'exécutable.

Dans la seconde partie, les entrées notables sont 

*	Le DWORD "ImageBase", aka l'adresse dans la mémoire virtuelle où notre premier byte (suivi ensuite par tout le reste du PE) sera placée. A cause des diverses protections mémoire modernes, comme l'ASLR (Adress Space Layout Randomization), cette adresse n'est quasiment jamais respectée (sa valeur par défaut étant 0x40000000)
*	Les DWORD "FileAlignment" et "SectionAlignment", relatifs à l'alignement en mémoire des différentes sections.
*	Le DWORD "SizeOfImage", indiquant la taille totale de notre image (en-têtes compris), vu que celle-ci doit être chargée en mémoire. Naturellement un multiple de "SectionAlignment" (alignement des blocks mémoires oblige)
*	Le DWORD "SizeOfHeaders", plutôt explicite, et indiquant la taille totale des en-têtes MS-DOS, PE, et de la table de sections.
*	Le WORD "Subsystem", indiquant si l'application est un driver, une application graphique, ou en ligne de commande ...



Je vais m'attarder également sur les deux dernières entrées de cette structure, le DWORD "NumberOfRvaAndSizes", indiquant selon la documentation le nombre de répertoire de données ("Data Directories" en anglais), et le tableau de structures "IMAGE_DATA_DIRECTORY". Mais que sont donc ces "répertoire de données" ?

Ce terme se réfère à des répertoires situés dans les sections de notre fichier PE, et contenant des données utiles pour le loader Windows. Un exemple concret serait l'Import Directory (répertoire des imports), situés dans la section .idata (nous y reviendrons plus tard).

De retour à notre en-tête NT, le tableau "IMAGE_DATA_DIRECTORY" est défini comme ceci :

```cpp
IMAGE_DATA_DIRECTORY DataDirectory[IMAGE_NUMBEROF_DIRECTORY_ENTRIES]
```


 Où "IMAGE_NUMBEROF_DIRECTORY_ENTRIES" est définie (toujours dans winnt.h) comme une constante de 16 (indiquant que ce tableau peut avoir un maximum de 16 entrées): 
 
```cpp
#define IMAGE_NUMBEROF_DIRECTORY_ENTRIES    16
```
 
 Mais qu'est donc la structure "IMAGE_DATA_DIRECTORY"? Voici sa définition ci-dessous :

```cpp
typedef struct _IMAGE_DATA_DIRECTORY {
    DWORD   VirtualAddress;
    DWORD   Size;
} IMAGE_DATA_DIRECTORY, *PIMAGE_DATA_DIRECTORY;
```

Le DWORD "VirtualAddress" est une adresse de type RVA pointant vers le début d'un répertoire de données, et le DWORD "Size" indique évidemment la taille dudit répertoire.

>RVA correspond à "Relative Virtuelle Adress" en anglais, appellée ainsi car relative à l'adresse spécifiée dans "ImageBase", vu plus haut dans le même en-tête).

Il existe plusieurs types de répertoires de données, définis, vous l'aurez devinés, dans winnt.h :

```cpp
#define IMAGE_DIRECTORY_ENTRY_EXPORT          0   // Export Directory
#define IMAGE_DIRECTORY_ENTRY_IMPORT          1   // Import Directory
#define IMAGE_DIRECTORY_ENTRY_RESOURCE        2   // Resource Directory
#define IMAGE_DIRECTORY_ENTRY_EXCEPTION       3   // Exception Directory
#define IMAGE_DIRECTORY_ENTRY_SECURITY        4   // Security Directory
#define IMAGE_DIRECTORY_ENTRY_BASERELOC       5   // Base Relocation Table
#define IMAGE_DIRECTORY_ENTRY_DEBUG           6   // Debug Directory
//      IMAGE_DIRECTORY_ENTRY_COPYRIGHT       7   // (X86 usage)
#define IMAGE_DIRECTORY_ENTRY_ARCHITECTURE    7   // Architecture Specific Data
#define IMAGE_DIRECTORY_ENTRY_GLOBALPTR       8   // RVA of GP
#define IMAGE_DIRECTORY_ENTRY_TLS             9   // TLS Directory
#define IMAGE_DIRECTORY_ENTRY_LOAD_CONFIG    10   // Load Configuration Directory
#define IMAGE_DIRECTORY_ENTRY_BOUND_IMPORT   11   // Bound Import Directory in headers
#define IMAGE_DIRECTORY_ENTRY_IAT            12   // Import Address Table
#define IMAGE_DIRECTORY_ENTRY_DELAY_IMPORT   13   // Delay Load Import Descriptors
#define IMAGE_DIRECTORY_ENTRY_COM_DESCRIPTOR 14   // COM Runtime descriptor
```

## Les en-têtes de sections

Les dernier en-têtes, juste après l'en-tête optionel, sont les en-têtes de sections. Avant de s'y intéresser, définir ce qu'est une section est nécessaire.

Les sections, situées après tout nos headers et constituant la seconde partie de notre fichier PE, sont des containers pour du code ou des données dans un fichier PE/COFF, similaires aux segments dans l'architecture Intel8086. Les quatre sections les plus communes dans les fichiers PE sont ;

*   La section .text : qui contient le code de notre exécutable.
*   La section .data : contient les "données initialisées", comme des variables globales.
*   La section .rsrc : contient des ressources utilisées par l'exécutable (favicons, autres exécutables).
*   La section .idata: contient les repertoire des données liés aux imports de fonctions de notre exécutable

Il en existe bien d'autres (.bss, .rdata, .tls), détaillées dans la documentation officielle Microsoft (https://docs.microsoft.com/fr-fr/windows/win32/debug/pe-format#optional-header-data-directories-image-only). Nous y reviendrons plus en détail prochainement.


De retour à nos en-têtes de sections. Ceux-ci sont définis comme-ci ; 

```cpp
typedef struct _IMAGE_SECTION_HEADER {
  BYTE  Name[IMAGE_SIZEOF_SHORT_NAME];
  union {
      DWORD PhysicalAddress;
      DWORD VirtualSize;
  } Misc;
  DWORD VirtualAddress;
  DWORD SizeOfRawData;
  DWORD PointerToRawData;
  DWORD PointerToRelocations;
  DWORD PointerToLinenumbers;
  WORD  NumberOfRelocations;
  WORD  NumberOfLinenumbers;
  DWORD Characteristics;
} IMAGE_SECTION_HEADER, *PIMAGE_SECTION_HEADER;
```

*   Le DWORD name est plutôt explicite, et indique le nom de la section (limité à 8 caractères). Exemple : ".$tls"
*   Les DWORDs PhysicalAddress et VirtualSize, deux mêmes noms pour une seule notion ; la taille de notre section une fois chargée en mémoire.
*   Le DWORD VirtualAddress est un pointeur vers le début de notre section relative à la base de l'image.
*   Le DWORD SizeOfRawData correspond à la taille de la section sur disque (doit être un multiple de FileAlignement, vu plus-haut dans l'en-tête optionel)
*   Le DWORD PointerToRelocations est un pointeur vers les relocalisations (que nous verrons plus tard)
*   Le DWORD Characteristics est une valeur indiquant les caractéristiques de la section tel que définies par la documentation Microsoft (https://docs.microsoft.com/fr-fr/windows/win32/debug/pe-format#section-flags). Par exemple, 0x00000020 indique que la sectio contient du code exécutable, comme la section .code.




Et c'est tout pour aujourd'hui ! Dans le prochain post, nous examinerons les principales sections du format de fichier PE : code, imports, data.