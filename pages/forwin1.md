# Introduction à la forensique sous Windows

## Définition
Avant de commencer cette série d'articles sur la forensique sous Windows, définissons les termes utilisés : la forensique, c'est quoi exactement ?


La réponse est plutôt simple : la forensique informatique est une discipline visant à collecter, analyser et interpréter des artéfacts numériques dans le cadre d'un incident de sécurité (ou d'une enquête légale). Une sorte de version digitale (et bien moins morbide, surtout) des enquêteurs forensiques de "Faites entrer l'accuser".


## Le cycle forensique

Comme la plupart des domaines liés à la cybersécurité, la forensique informatique a ses propres standards d'industrie. L'un d'entre eux, nommé PICERL (Preservation, Identification, Collection, Examination, Analysis, Reporting, and Learning) et développé par l'institut SANS, est l'une de ces approches méthodiques et structurées pour mener des investigations numériques:

*	Préservation:  la première étape cruciale, l'enjeu étant de garantir l'intégrité des preuves numériques afin d'éviter toute altération. Cela implique la mise en place de mesures de préservation telles que la sécurisation de la "scène de crime" numérique, ainsi que la création de copies exactes des médias numériques pertinents (clés usb, disques durs).
*	Identification: ici, les éléments clés de l'enquête sont identifiés, notamment les systèmes, utilisateurs, et autre actifs pertinents, en plus des sources potentielles de preuves. On en profite pour créer une liste d'éléments à examiner.
*	Collection :  la collecte des preuves numériques identifiées de manière légale et appropriée. Copie des fichiers, extraction des données à partir de dispositifs, acquisition des images disques ou encore utilisation d'outils de collecte spécialisés. L'objectif est de rassembler toutes les informations nécessaires à l'analyse ultérieure.
*	Examen : découverte des faits pertinents à l'enquête. Identification et extraction d'informations importantes, récupération de fichiers supprimés, recherche de traces d'activité, analyse des journaux, etc.
*	Analyse : les preuves collectées sont analysées afin de déterminer les relations entre elles, établir/reconstituer une chronologie, et en tirer des conclusions.
*	Rapport : les résultats de l'analyse sont documentés dans un rapport détaillé. Le rapport doit être clair, précis et objectif, décrivant les actions entreprises, les méthodes utilisées et les conclusions tirées. Il doit être compréhensible par un public non technique et pouvoir servir de preuve dans un contexte juridique.
*	Leçon tirées (traduit de "Learning" en anglais) : l'étape finale, qui consiste tout simplement à tirer des enseignements de l'enquête. Cela peut inclure l'évaluation des procédures utilisées, l'identification des améliorations possibles, la mise à jour des politiques de sécurité, etc.

![PICERL](../docs/assets/images/forwin1_picerl.png)


Dans le cadre des articles suivants, je me concentrerai donc sur des environnements Windows. L'analyse forensique sur ces derniers implique (entre autres) l'examen des données stockées sur des disques durs, des clés USB, des registres système, des journaux d'événements, des fichiers de configuration, etc. Ces informations servent à réveler des détails précieux sur (entre autres, toujours) les activités des utilisateurs, les programmes exécutés, les connexions réseau, les modifications de configuration, ou encore des violations de sécurité et d'autres événements pertinents.

Les premiers articles, dont le prochain serviront à introduir certains concepts clés utilisés dans la forensique Windows, comme le registre Windows et ses clés/valeurs. Pour y accéder, [je vous laisse cliquer ici](./forwin2.md) !