# Projet de PoC SIEM

Me voilà de retour après une saison entière sans articles écrits, wouh !

Toujours dans la continuité de mes projet de fin de cursus et personnels, je compte cette fois ~~finir mes séries d'articles~~ déployer une stack SIEM sur mon hyperviseur, en utilisant divers produits Open Source déjà disponibles, afin de me familiariser avec le déploiement de ladite stack et de son pipeline de log (étant déjà familier avec leur exploitation au quotidien dans le cadre de mont travail)

En suivant l'exemple de la chaîne Youtube de Taylor Walton (https://www.youtube.com/@taylorwalton_socfortress), il me faut a minima installer des composants permettant de:

*	Gérer les logs remontés par les différents produits
*	Analyser lesdits logs
*	Visualiser lesdits logs 

Pour le moment, rien de compliqué. J'aimerai cependant ajouter  :

*	L'obtention automatisée d'information supplémentaires concernant les IoCs collectés, notamment via des plateformes collaboratives open source
*	La possibilité d'ouvrir et gérer des incidents de sécurité en utilisant les logs remontés
*	Un méchanisme permettant de lancer des actions de réponse à incident ou de forensique directement sur les machines impactées
*	Et enfin, si il reste assez de place dans mon Hyperviseur, un système de surveillance et de gestion des métriques de toute l'infrastructure


J'aimerai également en parallèle utiliser les technologies Terraform et Ansible afin de pouvoir mettre à disposition des lecteurs un repo clé en main permettant de déployer toute cette infrastructure en quelques commandes sur une plateforme cloud (Azure, AWS, GCP, etc ...). La vitesse de ce projet, que je considère comme secondaire en général, sera majoritairement tributaire du taux de cafféine présent dans mon organisme.