# Le développement de malwares sous Windows à des fins éducationelles

Ce post servira d'introduction à une série d'articles centrée sur le développement, à des **fins éducationelles** et dans le cadre de mes **études**, de programmes émulant des comportements malicieux sous Windows. Toute démonstration sera faite sur des systèmes m'appartenant explicitement et avec des charges désarmées. Je décline toute responsabilité concernant le mésusage par des individus malintentionés des snippets de code présents dans ces articles.


N'ayant pas un background de développeur, j'ai commencé à m'intéresser au sujet après avoir tripatouillé de mon côté avec l'API Windows et découvert que celui-ci était très facile et satisfaisant à utiliser, étant relativement simple et bien documenté. Dans le cadre de CTFs et de mes futures aspirations de reverser, j'ai donc commencé à chercher des ressources concernant son utilisation dans le cadre de la sécurité informatique, et n'ai pas été déçu. Lesdites ressources incluent, entre autres ;

*	Le cours de maldev de Sektor7 (https://www.udemy.com/course/ehf-maldev-in-windows/) : une excellente ressource que je ne peux que conseiller. Les premiers posts de cette série seront d'ailleurs principalement composés des connaissances acquises pendant ce cours.
*	Le blog de cocomelonc : https://cocomelonc.github.io/
*	Le blog de M. Patryk Czecsko aka 0xpat : https://0xpat.github.io/Malware_development_part_1/
*	Le gitbook d'IredTeam : https://www.ired.team/
*	Le blog de slashbinbash (pour une vue très concise du couple C++/Assembleur) : https://slashbinbash.de/index.html

Dans un premier article, je parlerai de la création d'un binaire ayant pour vocation d'exécuter une charge (on va dire payload à partir de maintenant par contre) directement sur le système via l'appel d'un _certain_ tryptique de fonctions que les plus expérimentés d'entre vous ne connaîtrons que trop bien.

D'ici là, bonne journée à vous !