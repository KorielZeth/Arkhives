# Le format de fichier PE

Le PE, pour "Portable Executable" en anglais est un format de fichier utilisé pour les exécutables
(.exe) et librairies (.dll) du système d'exploitation Windows, et les exécutables de l'UEFI (.efi) et créé en 1993.

Celui-ci est composé de deux parties principales ; les en-têtes ("headers" en anglais) et les sections.


Je traiterai dans la prochaine partie des en-têtes, avant de couvrir ensuite les différentes sections, et enfin le processus de loading.

Ce post est sponsorisé (pas littéralement) par la documentation Microsoft (https://docs.microsoft.com/fr-fr/), et l'inspiration principale de ce post reste le blog de "0xrick" (https://0xrick.github.io/), suppléé par cet article de Satyajit Daulaguphu (https://tech-zealots.com/malware-analysis/journey-towards-import-address-table-of-an-executable-file/) et, tout bêtement, Wikipédia.

## Rapide rappel sur les concepts de VA et RVA

### Virtual Address

Lorsqu'un processus Windows est crée pour une exécutable, celui-ci n'accède pas directement à la mémoire physique du système, mais se voit assigner son propre espace mémoire virtuel. Ainsi, les addresses virtuelles (Virtual Addresses) sont les addresses mémoires référencées dans notre application.

### Image Base

L'Image Base désigne l'addresse de base à laquelle est l'exécutable est initialement chargé en mémoire. La valeur par défaut était originellement située à 0x0010000 sous Windows NT. A l'arrivée de Windows 95 Microsoft changea celle-ci pour 0x0040000, la précédente étant désormais utilisée par une plage d'addressage partagée par tous les processus.

### Relative Virtual Address

Une fois dans le cadre de l'espace mémoire virtuel de notre application, une Virtual Address est l'addresse originale en mémoire, alors que la RVA est l'addresse relative à l'Image Base.

Imaginons une instruction située à l'addresse virtuelle 0x0045000. Si l'exécutable a bien été chargé en mémoire depuis 0x0040000, alors la RVA de cette instruction est 0x0005000.