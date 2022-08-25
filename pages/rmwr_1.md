# Projet de développement de ransomware de démonstration à des fins éducationelles (non obfusqué)

Projet de fin de cursus oblige, j'ai envie de créer quelque chose d'intéressant, et quoi de mieux qu'une simulation de compromission par ransomware développée par votre humble serviteur (avec la réponse à incident fictive qui va avec) ! 

Je compte d'abord produire un échantillon purement fonctionnel (et très très peu optimisé, que ça soit niveau performances ou même good practices), avant de l'améliorer peu à peu et de le reverse-engineer à chaque étape, afin de me servir de ce projet comme "témoin" de mes progrès en malware dev et reverse-engineering au cours de cette année. L'échantillon sera d'abord disponible sur Github, étant inoffensif pour les raisons suivantes ;

*	Clé de chiffrement hardcodée
*	Mécanisme de persistence très aisément désactivable
*	Dossiers à chiffrer hardcodés et fictifs
*	Et absolument aucun mécanisme d'obfuscation, la durée de vie du sample face à VirusTotal étant donc de 5 milisecondes avant que le dashboard dudit site s'illumine comme un sapin de Noel.

A terme, j'aimerai implémenter, de manière plus propre, pour la deuxième partie du projet ; 

*	Une logique de chiffrement améliorée, avec une blacklist d'extensions/dossiers spécifiques à ne pas toucher, une liste d'extensions de fichiers pour qui seuls les premiers bytes seront chiffrés, et une liste d'extension de fichiers plus gros (ex : .vdmk) pour lequel plus de butes seront chiffrés.
*	Un service qui garde un oeil sur la progression du service de chiffrement et qui empêche tout restart et reboot pendant la progression.
*	Une GUI plus propre avec un message de démo.
*	Un mécanisme de propagation (via SMB, probablement ?)
*	Un service qui shutdown certains process spécifiques. Inspiré de la brêche d'Okta par le groupe LAPSUS. Ces derniers avaient tout simplement utilisé Process Hacker, un utilitaire de monitoring et de gestion des processus Windows pour ... click-droit + shutdown l'agent d'EDR de FireEye.

Mais en attendant et pour des besoins de démonstration, une simple PoC suffira bien amplement. 

Petit rappel légal issu tout droit de l'article 323 du code pénal à destination des potentiels script kiddies ayant atteri par erreur ici ;

>Le fait, sans motif légitime, notamment de recherche ou de sécurité informatique, d'importer, de détenir, d'offrir, de céder ou de mettre à disposition un équipement, un instrument, un programme informatique ou toute donnée conçus ou spécialement adaptés pour commettre une ou plusieurs des infractions prévues par les articles 323-1 à 323-3 est puni des peines prévues respectivement pour l'infraction elle-même ou pour l'infraction la plus sévèrement réprimée.