# Table of content

1. **Predstavenie projektu**
		- priblíženie zadania projektu
		- časová náročnosť
		- implementačná náročnosť
		- zdroje
2. **Implementačné detaily a rady**
		- predstavenie SFTP
		- požiadavky na klienta
		- požiadavky na server
		- minimálne vs maximálne požiadavky
3. **Na čo si dať pozor - chyby z minulého roku**
		- upozornenie a rozobratie častých chýb z minulého roku
4. **Ako bol projekt testovaný minulý rok**
		- rozobratie minuloročných testov
		- bodové zrážky z minulého roku
5. **Dokumentácia projektu**
		- ako na dokumentáciu
		- čo treba spomenúť
		- rozsah
		- časté chyby a nedostatky z minulého roku

****
# Predstavenie projektu
- *~20 hodín*
- implementácia vs networking
- podpora IPv4 aj IPv6

## Zdroje
- SFTP https://datatracker.ietf.org/doc/html/rfc913
- TCP https://datatracker.ietf.org/doc/html/rfc793
- TELNET https://datatracker.ietf.org/doc/html/rfc2946
- NETINET https://www.mutekh.org/doc/Network_library_module_reference.html

## V skratke
- server-client komunikácia prebieha cez **TCP**
- client sa pripojí na server pomocou **TELNET**-u
- client odosiela príkazy serveru, ten ich spracuje a odošle odpoveď
- prihlasovanie na server
- **CTRL + C** kedykolvek ukončí server aj klienta
- operácie
		- premenovávanie súborov
		- zmena adresára
		- vylistovanie obsahu adresára
		- stiahnutie súboru zo serveru
		- nahratie súboru na server
		- vypísanie velkosti súboru
		- mazanie súboru

****
# Implementačné detaily a rady
- networking cez `sys/socket.h`
- TCP

![[Pasted image 20220404133452.png]]

## SFTP

### Príkazy
```XML
<command> : = <cmd> [<SPACE> <args>] <NULL>

<cmd> : =  USER ! ACCT ! PASS ! TYPE ! LIST ! CDIR ! KILL ! NAME ! DONE ! RETR ! STOR

<response> : = <response-code> [<message>] <NULL>

<response-code> : =  + | - |   | !

<message> can contain <CRLF>
```

- odpovede zo serveru:
`+  Success.`
`-  Error.`

`Number.`
`!  Logged in.`

- pripojenie na server
		- *negative greeting* -> zrušenie pripojenia
		- *positive greeting* -> pokračovanie na prihlasovanie

- príklad:
```
(the first word should be the host name)

+MIT-XX SFTP Service

-MIT-XX Out to Lunch
```

**USER** *user-id*

**ACCT** *account*

**PASS** *password*

**TYPE** { *A | B | C* }

**LIST**  { *F | V* } *directory-path*

**CDIR** *new-directory*

**KILL** *file-spec*

**NAME** *old-file-spec*

**TOBE** *new-file-spec*

**DONE**

**RETR** *file-spec*

**STOR** { *NEW | OLD | APP* } *file-spec*

- bližšia špecifikácia príkazov: [[IPK project 2#SFTP#THE PROTOCOL]]



## Client vs Server
- Kontrola príkazu (client vs server)
- Vytvorenie použivatela na servery
- Vlastník súborov/adresárov

## Client
- vytvorenie pripojenia na server
- vo `while(true)` cykle načítavať od usera príkazy (STDIN) a odosialať ich na server

## Server
- vedieť verifikovať používateľa na základe súboru *passwords.txt* 
	(*user-id*:*password*)
- overenie príkazu a následné vykonanie
- vykonanie príkazu pomocou unixového príkazu (**LIST** -> **ls**)
- vrátenie odpovede

## MIN vs MAX
*Menej je niekedy viac.*

- implementácia príkazov
- prihlasovanie používateľa
- práva zo súbormi
- korektná komunikácia
- podpora IPv4 a IPv6

- vytváranie použivateľov/skupiny použivateľov na servery
- ukladanie si vlastníka súboru/adresára do špeciálneho súboru
- limitovanie používateľa na špecifikovaný pracovný adresár
- šifrovanie, bezpečnosť
- viac pripojených používateľov na server, zároveň

- *keep it simple*

Funkčnosť > Features

****
# Na čo si dať pozor
- neodovzdalo 29/49
- správny Makefile -> musí preložiť server al client naraz
- nepodarilo sa spustiť klienta alebo server
- server po vypnutí nenabinduje port
- nepreložitelnosť 
	(`(make: *** No rule to make target 'tcplib.c', needed by 'TCPLIB'.`)
- SEGFAULT
- nepodarilo sa príhlásiť
- nepodporovaná IPv6

****
# Ako bol projekt testovaný (rok 2021)
## Testovacie príkazy
1) T0:**user** [1b] 
2) T1:**list** [2b] 
3) T2:**cdir** [2b] 
4) T3:**retr** [2b] 
5) T4:**stor** [2b] 
6) T5:**kill** [2b] 
7) T6:**IPv6** [2b]

## Bodové zrážky
- odovzdaný iný archív ako **.tar** -> [-1b]
- archív neobsahoval `README.md` -> [-1b]
- dokumentácia pomenovaná inak ako `manual.pdf` -> [-1b]
		**POZOR neodovzdajte to ako dokumentace.pdf**, častá chyba
- neodovzávajte aj preložený, spustitelný súbor
****
# Dokumentácia
- dôležitá časť projektu
- je kontrolovaná a to podrobne
- rozsah aspoň tých ~10 strán
- **správna bibliografia !!!**

## Ako na to ?
- popísať aj teóriu ktorá sa schováva za riešením projektu (networking)
- vysvetliť ako vaše riešenie funguje
- čo sa podarilo/nepodarilo implementovať

**Dostatočne opísať tetovanie projektu !!!**
- vyvarovať sa "*iba som tam pisal príkazy*", "*použil som WireShark*", ...
- viac to rozviť a opísať ako ste testovali (samozrejme sa predpokladá že ste testovali na virtuálnej sieti (IPv4 aj IPv6) pomocou *Virtualbox* alebo *VMWare*)

## Bodovanie
1) Forma [2b] 
2) Popis teorie [1b] 
3) Popis implementace [2b]
4) Testování [2b]

## Časté chyby
- Bibliografie ve špatném formátu
- Slabá teorie a testování
- Bez čísel stránek
- Minimalistická dokumetace
- Slabé popisy

