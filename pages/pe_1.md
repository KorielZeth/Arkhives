# Le format de fichier PE

Le PE, pour "Portable Executable" en anglais est un format de fichier utilisé pour les exécutables
(.exe) et librairies (.dll) du système d'exploitation Windows, et les exécutables de l'UEFI (.efi) et créé en 1993.

Celui-ci est composé de deux parties principales ; les en-têtes ("headers" en anglais) et les sections.


Je traiterai dans la prochaine partie des en-têtes, avant de couvrir ensuite les différentes sections, et enfin le processus de loading.

Ce post est sponsorisé (pas littéralement) par la documentation Microsoft (https://docs.microsoft.com/fr-fr/), et l'inspiration principale de ce post reste le blog de "0xrick" (https://0xrick.github.io/).