# Les sections du format P-E

Après avoir vu ensemble tout ce qui touchait aux en-têtes du format Portable Executable dans le précédent post, nous pouvons désormais examiner ensemble les sections, brièvement définies dans le post précédent. Pour rappel : 

> Les sections, situées après tout nos headers et constituant la seconde partie de notre fichier PE, sont des containers pour du code ou des données dans un fichier PE/COFF, similaires aux segments dans l'architecture Intel8086. Les quatre sections les plus communes dans les fichiers PE sont ;

>*   La section .text : qui contient le code de notre exécutable.
>*   La section .data : contient les "données initialisées", comme des variables globales.
>*   La section .rsrc : contient des ressources utilisées par l'exécutable (favicons, autres exécutables).
>*   La section .idata: contient les repertoire des données liés aux imports de fonctions de notre exécutable

>Il en existe bien d'autres (.bss, .rdata, .tls), détaillées dans la documentation officielle Microsoft (https://docs.microsoft.com/fr-fr/windows/win32/debug/pe-format#optional-header-data-directories-image-only)

## La section .text

La plus simple et explicite, cette section contient tout simplement le code allant être exécuté (sous la forme d'instructions Assembleur). Quelque chose comme :

```assembly
push 0
push 0x403000
push 0x403017
push 0
call [0x402070]
push 0
call [0x402068]
```

Pas grand chose à ajouter dessus, pour être parfaitement honnête.

## La section .data

[à compléter]

## La section .idata

La section .idata est pour moi la plus intéressante ; elle contient les répertoire de données (souvenez vous, les "datas directories") en rapport avec les imports de notre programme (dll importées et les fonctions allant avec). Jetons y un oeil, dans l'ordre.

### L'Import Directory Table

Premier répertoire de données de notre section .idata, l'Import Directory Table est un tableau de structures IMAGE_IMPORT_DESCRIPTOR ; chacune d'entre elles pointant vers une DLL en particulier.

IMAGE_IMPORT_DESCRIPTOR est définie comme ceci :

```cpp
typedef struct _IMAGE_IMPORT_DESCRIPTOR {
    union {
        DWORD   Characteristics;            // 0 for terminating null import descriptor
        DWORD   OriginalFirstThunk;         // RVA to original unbound IAT (PIMAGE_THUNK_DATA)
    } DUMMYUNIONNAME;
    DWORD   TimeDateStamp;                  // 0 if not bound,
                                            // -1 if bound, and real date\time stamp
                                            //     in IMAGE_DIRECTORY_ENTRY_BOUND_IMPORT (new BIND)
                                            // O.W. date/time stamp of DLL bound to (Old BIND)

    DWORD   ForwarderChain;                 // -1 if no forwarders
    DWORD   Name;
    DWORD   FirstThunk;                     // RVA to IAT (if bound this IAT has actual addresses)
} IMAGE_IMPORT_DESCRIPTOR;
typedef IMAGE_IMPORT_DESCRIPTOR UNALIGNED *PIMAGE_IMPORT_DESCRIPTOR;
```

*	Le DWORD Characteristics