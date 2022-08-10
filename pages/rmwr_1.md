# Projet de développement de ransomware de démonstration à des fins éducationelles (non obfusqué)

Projet de fin de cursus oblige, j'ai envie de créer quelque chose d'intéressant, et quoi de mieux qu'une simulation de compromission par ransomware développée par votre humble serviteur (avec la réponse à incident qui va avec) !

Liste des services à implémenter (l'AV evasion a la priorité minimale, Defender sera désactivé par GPO sur notre lab dans le cadre du scénario);

> Un service qui se charge du chiffrement. Je pense faire ça comme le fait Conti, avec une blacklist d'extensions/dossiers spécifiques à ne pas toucher, une liste d'extensions de fichiers pour qui seuls les premiers bytes seront chiffrés, et une liste d'extension de fichiers plus gros (ex : .vdmk) pour lequel plus de butes seront chiffrés.
> Un service qui garde un oeil sur la progression du service d'encryption et qui empêche tout restart et reboot
> Une GUI (ou cli) avec un message de démo, probablement un truc pas très inspiré à la WannaCry
> Le service de decryption des fichiers (ici pour le principe, dans le scénario fictif notre entreprise décidera de ne pas payer la rançon et d'essayer de reconstruire et sécuriser son SI)
> Un mécanisme de propagation (si j'ai le temps, sinon déploiement via GPO, à la barbare).
> Un service qui shutdown certains process spécifiques. Inspiré de la compromission de 