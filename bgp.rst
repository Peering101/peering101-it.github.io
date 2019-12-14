BGP (Border Gateway Protocol)
================================================================

Premessa
--------

In questo spazio, nato da un'idea di `ITNOG <https://www.itnog.it/>`__ (*the ITalian Network Operators Group*), intendiamo fornire una breve guida ai principali concetti e meccanismi sottesi al funzionamento di Internet illustrando alcune delle architetture di instradamento basate sul protocollo *BGP - Border Gateway Protocol*.
L'obiettivo dunque non è quello di tradurre e parafrasare le RFC (*Request for comments*), né quello di somministrare un ricettario; piuttosto vorremmo divulgare (con linguaggio semplice) i capisaldi della letteratura conditi con esempi intuitivi così da facilitare la strada a quanti vorranno poi approfondire.

La struttura del testo si rifà a un approccio sempreverde, scandito secondo questi punti:

- `protocolli di instradamento: fondamenti e concetti`_;
- `distance vector vs link-state`_;
- `instradamento interno o esterno, statico o dinamico`_;
- `come funziona BGP`_;
- `sessioni BGP`_;
- `processo di instradamento`_;
- `controllo degli instradamenti`_;
- `filtri e manipolazioni`_;
- `ridondanza e bilanciamento`_.

Per quanto possibile saranno indicate le fonti, le migliori pratiche e le recenti tecnologie così come pubblicate da `RFC Editor <https://rfc-editor.org>`__ e dalle altre organizzazioni coinvolte nella definizione degli standard.

Protocolli di instradamento: fondamenti e concetti
--------

L'attore protagonista in tutte le attività di instradamento si chiama *router*: quell'apparato capace di instradare il traffico dei dispositivi secondo delle tabelle contenenti indispensabili informazioni sui migliori percorsi (*best path*) verso tutte le destinazioni che conosce.

Per consentire l'instradamento, il *router* segue una procedura che si articola in diversi punti:

1. scelta di un cosiddetto protocollo di instradamento (*routing protocol*) utile a scambiare con altri *router* della rete informazioni sui percorsi e sulle destinazioni;

2. inserimento di tali informazioni in tabelle di instradamento (*routing table*), una per ciascun protocollo;

3. analisi di tutte le tabelle popolate e scelta dei migliori percorsi verso ciascuna destinazione;

4. associazione di ciascuna destinazione con l'indirizzo del dispositivo collegato (*next-hop device*) all'interfaccia di uscita dei pacchetti verso quella destinazione;

5. popolamento di una tabella di inoltro (*forwarding table*) con queste ultime informazioni;

6. individuazione dell'indirizzo di destinazione di un pacchetto ricevuto attraverso la lettura dell'intestazione di quel pacchetto;

7. consultazione della tabella di inoltro per ottenere informazioni sull'interfaccia di uscita e l'indirizzo del dispositivo utile al raggiungimento della destinazione;

8. compimento di eventuali altre azioni prima di inoltrare il pacchetto al successivo dispositivo;

9. ripetizione della procedura fin quando la destinazione viene raggiunta secondo lo schema di un salto dopo l'altro (*hop-by-hop*), tipico delle reti a commutazione di pacchetto.

Alla base dei protocolli di instradamento ci sono generalmente due algoritmi: vettore delle distanze (*distance vector*) e stato del collegamento (*link-state*).

*Distance vector vs link-state*
--------

Come calcolare la distanza più breve per raggiungere una destinazione? Questo è il campo dove si gioca la partita dei protocolli di instradamento sulla rete.

Quello basato sul vettore delle distanze prevede una lista (il vettore appunto) di distanze associate al prefisso di ogni destinazione appreso dai messaggi provenienti da altri *router* sulla rete. Cioè praticamente ciascun *router* calcola autonomamente il percorso migliore verso ogni destinazione e, sùbito dopo, invia il proprio vettore di distanze agli altri *router* in rete.

Così facendo, tutti i *router* coinvolti nel processo contribuiscono a influenzare le proprie tabelle di instradamento fino a convergere su una lista di *best path* condivisa.

Proprio l'aspetto della convergenza è stato riconosciuto nel tempo come momento critico per un protocollo. Cioè, quanto tempo occorre affinché l'intera rete (o meglio i *router* che la costituiscono) sia al corrente della comparsa, scomparsa o cambiamento di un particolare instradamento?

Le argomentazioni necessarie a dare una risposta a quella domanda costituiscono una base critica all'adozione su larga scala di protocolli *distance vector*: dunque lentezza nella convergenza e difficoltà di gestire un gran numero di destinazioni.

Alcuni esempi di protocolli di instradamento basati sull'algoritmo del *distance vector* sono: *RIP - Routing Information Protocol* `[RFC2453] <https://www.rfc-editor.org/rfc/rfc2453.txt>`__, *EIGRP - Enhanced Interior Gateway Routing Protocol*  `[RFC7868] <https://www.rfc-editor.org/rfc/rfc7868.txt>`__.

_____

Diverso protocollo di instradamento, e più complesso del precedente, è quello basato sull'algoritmo *link-state*. In questo caso i *router* si scambiano informazioni proprio sullo stato del collegamento e quindi non tabelle di instradamento.

L'oggetto delle comunicazioni tra dispositivi che ne fanno uso risiede nelle informazioni su dispositivi e reti adiacenti incluse delle grandezze (*metric*) associate alla connessione. In altre parole ciascun *router* produce un messaggio che contiene una descrizione del dispositivo stesso e di dove si connette agli altri; messaggio che viene distribuito a tutti gli altri *router* della rete i quali lo archiviano in una base di dati interna. Così ciascun dispositivo sarà in grado di ricostruire autonomamente una topologia della rete che sarà uguale per tutti.

Dopodiché, tutti i *router* saranno in grado di calcolare e tratteggiare un albero (dove ciascun dispositivo pone sé stesso alla radice) di cosiddetti *best path* verso ciascuna destinazione applicando l'algoritmo `SPF - Shortest Path First <http://www-m3.ma.tum.de/foswiki/pub/MN0506/WebHome/dijkstra.pdf>`_ concepito nel 1959 dal matematico olandese Edsger W. Dijkstra.

Queste caratteristiche rendono il *link-state* un algoritmo più adatto a essere impiegato in scenari grandi e complessi, tuttavia sempre interni a un sistema autonomo. Infatti su grandi reti, come Internet, l'instabilità di alcuni collegamenti renderebbe le ritrasmissioni e i conseguenti calcoli un lavoro troppo oneroso (e di conseguenza inefficiente) per i singoli *router*.

I due più importanti esempi di protocolli di instradamento basati sull'algoritmo *link-state* sono *OSPF - Open Shortest Path First* (versione 2 `[RFC2328] <http://www.rfc-editor.org/rfc/rfc2328.txt>`__ e versione 3 `[RFC5340] <https://www.rfc-editor.org/rfc/rfc5340.txt>`__ che supporta IPv6) e *IS-IS - Intermediate System to Intermediate System* `[ISO/IEC 10589:2002] <http://standards.iso.org/ittf/PubliclyAvailableStandards/c030932_ISO_IEC_10589_2002(E).zip>`__.

Instradamento interno o esterno, statico o dinamico
--------

Abbiamo visto come diversi siano i metodi per rendere le risorse di rete raggiungibili, ma occorre aggiungere ancora un tassello determinante per la prosecuzione dell'illustrazione, e cioè il loro àmbito di applicazione. Per questo è necessario introdurre la nozione di sistema autonomo, fin qui solo velocemente menzionata.

Si tratta della cellula più piccola che dà vita all'organismo di Internet e dobbiamo immaginarla come la tessera di un mosaico la quale può trovarsi nel centro o ai bordi del quadro ma sempre con almeno un lato (preferibilmente almeno due) connesso ad altre tessere.

Da un punto di vista tecnico la definizione può essere rintracciata nella `[RFC1930] Guidelines for creation, selection, and registration of an Autonomous System (AS) <http://www.rfc-editor.org/rfc/rfc1930.txt>`__ dove si legge:

   *"Un sistema autonomo è un gruppo di uno o più prefissi IP gestito da uno o più operatori di rete con una politica di instradamento UNICA e BEN DEFINITA."*
   
Fino al 2007 la rappresentazione di un *AS* avveniva per mezzo di un numero a 16 bit, dopodiché per mezzo di un numero a 32 bit, come oggi regolata dalla `[RFC6793] BGP Support for Four-Octet Autonomous System (AS) Number Space <https://www.rfc-editor.org/rfc/rfc6793.txt>`__.

Più dettagliatamente possiamo considerare un "dentro" e un "fuori" dal punto di vista di un *AS* e cioè rispettivamente instradamenti *intra-AS* e instradamenti *inter-AS*.

Ora, gli instradamenti possono essere classificati anche per la modalità con la quale vengono appresi dai *router*: quando inseriamo manualmente un percorso verso una destinazione, allora si chiamerà "instradamento statico" (*static routing*); quando invece i dispositivi apprendono gli instradamenti grazie a un protocollo, allora si parlerà di "instradamento dinamico" (*dynamic routing*).

All'interno di questa ultima categoria distinguiamo: per il cosiddetto instradamento interno al sistema autonomo, *IGP - Interior Gateway Protocol* (come *RIP, EIGRP, OSPF, IS-IS*); per l'instradamento esterno tra sistemi autonomi diversi, *EGP - Exterior Gateway Protocol* (come *BGP*).


Come funziona BGP
--------

Nato nel 1989, quando *IETF* (*Internet Engineering Task Force*) partorì la `[RFC1105] A Border Gateway Protocol (BGP) <https://www.rfc-editor.org/rfc/rfc1105.txt>`__ recante la versione 1 del protocollo, BGP subì nel tempo alcuni profondi cambiamenti e, nel 1995, RFC Editor pubblicò le specifiche della versione 4, oggi raccolte nella `[RFC4271] A Border Gateway Protocol 4 (BGP-4) <https://www.rfc-editor.org/rfc/rfc4271.txt>`__.

Il BGP si basa su un algoritmo di instradamento chiamato "vettore di percorsi" (*path vector*), cioè i messaggi che produce contengono una lista di percorsi dati dai sistemi autonomi che occorre attraversare per raggiungere una certa destinazione (identificata da un prefisso di rete).

**Esempio di AS path:**

============== ============  ==========================
     Rete         Vicino              Percorso
============== ============  ==========================
203.0.113.0/24 198.51.100.1  64496_65551_64511_65536
============== ============  ==========================

Questo esempio può essere così letto:

è possibile raggiungere la rete 203.0.113.0/24 (*network*) attraverso il dispositivo 198.51.100.1 (*next hop*) il quale propone un percorso (*path*) che consiste nel transitare, oltre che per sé stesso ovviamente, per gli *AS* 64496, 65551, 64511, così da approdare infine all'*AS* 65536 dove la risorsa di destinazione risiede.

Il dispositivo chiamato "vicino" (*neighbor*) è un *router* capace di parlare la lingua del BGP (*BGP speaking*) che viene trasportata dal protocollo *TCP* (*Transport Control Protocol*) sulla porta 179, registrata presso `IANA - Internet Assigned Numbers Authority <https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.txt>`__.

L'intestazione del messaggio BGP che viene scambiato tra due *router* ha il seguente aspetto::

      0                   1                   2                   3
      0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |                                                               |
      +                                                               +
      |                                                               |
      +                                                               +
      |                           Marker                              |
      +                                                               +
      |                                                               |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |          Length               |      Type     |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

A seconda del tipo di messaggio, dopo l'intestazione potrebbero seguire dei dati.

A ogni modo il campo *marker*, oggi presente ma non più usato, ha una consistenza fissa di 16 byte e veniva usato per determinare se il messaggio BGP contenesse informazioni di autenticazione oppure no.

Il campo denominato *length* serve a dichiarare la lunghezza dell'intero messaggio BGP, intestazione compresa; per questo è semplice calcolarne il valore minimo: 19 byte (cioè 16 il *marker*, 2 il *length*, 1 il *type*). Il valore massimo, per RFC, è di 4096 byte.

Il campo *type* definisce invece il tipo di messaggio trasmesso e può recare dei valori che possono indicare i seguenti codici:

- *OPEN*;
- *UPDATE*;
- *NOTIFICATION*;
- *KEEPALIVE*;
- *ROUTE REFRESH*.

Per una migliore comprensione dell'intero processo di instradamento gestito dal protocollo BGP, è utile a questo punto esaminare e comprendere la funzione degli ingranaggi in movimento sotto il cofano.

Partiamo dal messaggio *OPEN* che costituisce il primo passo affinché due *router* possano stabilire una connessione secondo il protocollo BGP.

Gli elementi del messaggio *OPEN* sono:

- **[version]** la versione del protocollo (oggi si usa sempre la versione 4);
- **[my autonomous system]** il numero di sistema autonomo al quale il *router* appartiene;
- **[hold timer]** il numero di secondi che può trascorre tra i successivi messaggi di *UPDATE* o *KEEPALIVE*;
- **[bgp identifier]** l'identificativo del *bgp speaking router* (spesso il più alto indirizzo IP assegnato al dispositivo);
- **[optional parameter length]** la lunghezza in byte del seguente parametro opzionale;
- **[optional parameters]** una lista di parametri opzionali come a esempio quelli per l'autenticazione.

Affinché la connessione BGP tra due *router* venga stabilita correttamente è necessario che l'iter superi alcuni passaggi.

Innanzitutto partiamo dallo stato di riposo (**idle**) nel quale si trova un *router* prima di ricevere il via alla connessione che possiamo dare noi stessi intervenendo sulla configurazione del dispositivo. Ricevuto il via (*start*), il primo *router* tenta una connessione TCP sulla porta 179 del secondo e poi si mette in ascolto di risposte provenienti dal secondo *router*.

Ecco che entriamo nel passaggio di connessione (**connect**) durante il quale si attende che la connessione TCP avvenga con successo. In quest'ultimo caso si procede verso un ulteriore passaggio chiamato *opensent*. Se invece la connessione TCP non viene stabilita, allora si va verso il passaggio *active*. E ancora, nel caso in cui si esaurisca il tempo per l'operazione, si azzera il *timer* e viene ritentata una connessione TCP, mentre lo stato rimane *connect*. Altri eventi innescati dal sistema o manualmente da noi, producono il ritorno allo stato di riposo.

Segue lo stato attivo (**active**) che semplicemente indica un momento di transizione o verso il successo della connessione TCP o verso il suo fallimento con successivo innesco di un ulteriore tentativo.

Se la connessione TCP va a buon fine, allora siamo nel passaggio **opensent** dove scende in campo il protocollo BGP che si mette in attesa di un messaggio di tipo *OPEN* da parte del secondo *router*. Se arriva, il messaggio viene controllato e in caso di errore il *router* risponde con un messaggio di notifica (*NOTIFICATION*), dopodiché torna in stato di riposo.

Ma se il messaggio *OPEN* è corretto, allora il motore del BGP si mette in moto e il primo *router* comincia a inviare al secondo messaggi di tipo *KEEPALIVE* per mantenere viva la connessione.

Siamo ancora nel passaggio *opensent* quando il *router* confronta il campo *my autonomous system* inviatogli dal secondo *router* con il proprio numero di *AS* così da capire se entrambi appartengano o no allo stesso sistema autonomo. Nel primo caso saremmo nel contesto di BGP interno (*internal BGP*), nell'altro invece ci troveremmo nel contesto di BGP esterno (*external BGP*), una notizia importantissima che influenza molti comportamenti del protocollo.

A questo punto ci troviamo in un passaggio denominato **openconfirm** che conduce verso due distinte situazioni: il primo *router* attende un messaggio *KEEPALIVE* dal secondo; se arriva, la negoziazione si completa e dunque la connessione si considera stabilita (*established*). Altrimenti se il primo *router* riceve un messaggio di *NOTIFICATION*, si torna allo stato di riposo.

Infine, se è andato tutto a buon fine ci si ritrova all'ultimo passaggio, **established**, dove i *router* cominciano a scambiarsi messaggi di tipo *UPDATE* che devono essere privi di errori, poiché, se rinvenuti, viene generato un messaggio di *NOTIFICATION* e inevitabilmente si va dritti allo stato di riposo.

Qualora la connessione TCP dovesse interrompersi, il *router* tornerebbe allo stato *active*.

Nominato più volte, ispezioniamo il contenuto del messaggio *NOTIFICATION* precisando che viene generato in caso di errore e 
infatti contiene: un codice di errore, un altro codice subordinato al primo e un campo di dati a lunghezza variabile.

Il messaggio *KEEPALIVE* ha invece una diversa funzione, ma altrettanto importante perché, inviato a intervalli di tempo prestabiliti, serve a capire se i *router* sono ancora disponibili. Ha una lunghezza fissa di 19 byte e non reca contenuti.

Arriviamo finalmente al carburante del protocollo BGP: il messaggio *UPDATE* che veicola i contenuti senza i quali nulla della nostra trattazione avrebbe senso e che si presenta così::

      +-----------------------------------------------------+
      |   Withdrawn Routes Length (2 octets)                |
      +-----------------------------------------------------+
      |   Withdrawn Routes (variable)                       |
      +-----------------------------------------------------+
      |   Total Path Attribute Length (2 octets)            |
      +-----------------------------------------------------+
      |   Path Attributes (variable)                        |
      +-----------------------------------------------------+
      |   Network Layer Reachability Information (variable) |
      +-----------------------------------------------------+

Partiamo col dire che uno stesso messaggio *UPDATE* può contenere contemporaneamente informazioni sia relative a instradamenti da eliminare (*withdrawn route*) sia a instradamenti da aggiungere (*NLRI - Network Layer Reachability Information*) alla tabella interna al *router*.

In più, ciascun campo citato può contenere multipli valori.

Riprendiamo un esempio esposto precedentemente:

============== ============  ==========================
     Rete         Vicino              Percorso
============== ============  ==========================
203.0.113.0/24 198.51.100.1  64496_65551_64511_65536
============== ============  ==========================

Proviamo a popolare il messaggio *UPDATE* con questo contenuto::

      +-----------------------------------------------------+
      |                                                     | Withdrawn Routes Length
      +-----------------------------------------------------+
      |                                                     | Withdrawn Routes
      +-----------------------------------------------------+
      |                                                     | Total Path Attribute Length
      +-----------------------------------------------------+
      |   AS_PATH  64496 65551 64511 65536                  | Path Attributes
      |   NEXT_HOP 198.51.100.1                             |
      +-----------------------------------------------------+
      |            203.0.113.0/24                           | NLRI
      +-----------------------------------------------------+

Altra ipotesi potrebbe essere la seguente::

      +-----------------------------------------------------+
      |                                                     | Withdrawn Routes Length
      +-----------------------------------------------------+
      |           203.0.114.0/24                            | Withdrawn Routes
      +-----------------------------------------------------+
      |                                                     | Total Path Attribute Length
      +-----------------------------------------------------+
      |                                                     | Path Attributes
      +-----------------------------------------------------+
      |                                                     | NLRI
      +-----------------------------------------------------+

Oppure una combinazione delle due precedenti::

      +-----------------------------------------------------+
      |                                                     | Withdrawn Routes Length
      +-----------------------------------------------------+
      |           203.0.114.0/24                            | Withdrawn Routes
      +-----------------------------------------------------+
      |                                                     | Total Path Attribute Length
      +-----------------------------------------------------+
      |   AS_PATH  64496 65551 64511 65536                  | Path Attributes
      |   NEXT_HOP 198.51.100.1                             |
      +-----------------------------------------------------+
      |            203.0.113.0/24                           | NLRI
      +-----------------------------------------------------+
      
Una speciale considerazione va rivolta agli attributi del percorso (**path attributes**) i quali si articolano in quattro diverse categorie:

- **[well-known mandatory]** attributo imprescindibile che deve essere conosciuto da qualunque *bgp speaking router*;
- **[well-known discretionary]** attributo che può essere omesso ma che deve essere conosciuto da qualunque *bgp speaking router*;
- **[optional transitive]** attributo opzionale che, se presente ma non riconosciuto, deve ugualmente essere trasmesso agli altri *bgp speaking router*;
- **[optional non-transitive]** attributo opzionale che, se presente ma non riconosciuto, può essere tranquillamente ignorato e non deve essere trasmesso agli altri *bgp speaking router*.

Facciamo alcuni esempi:

AS_PATH rientra fra gli attributi *well-known mandatory*, come pure NEXT_HOP e ORIGIN (in tutto sono tre);
gli unici due *well-known discretionary* sono LOCAL_PREF e ATOMIC_AGGREGATE;
gli attributi *optional transitive* sono AGGREGATOR, COMMUNITY, EXTENDED_COMMUNITY, AS4_PATH, AS4_AGGRGATOR, mentre gli *optional non-transitive* sono MULTI_EXIT_DISC, ORIGINATOR_ID, CLUSTER_LIST, Multiprotocol Reachable NLRI e Multiprotocol Unreachable NLRI.

Quindi alla luce di quanto appena documentato ripetiamo il completo schema di messaggio *UPDATE*::

      +-----------------------------------------------------+
      |           14 byte                                   | Withdrawn Routes Length
      +-----------------------------------------------------+
      |           203.0.114.0/24                            | Withdrawn Routes
      +-----------------------------------------------------+
      |           67 byte                                   | Total Path Attribute Length
      +-----------------------------------------------------+
      |   ORIGIN   IGP                                      |
      |   AS_PATH  64496 65551 64511 65536                  | Path Attributes
      |   NEXT_HOP 198.51.100.1                             |
      +-----------------------------------------------------+
      |            203.0.113.0/24                           | NLRI
      +-----------------------------------------------------+
      
Sessioni BGP
--------
è arrivato il momento di sporcarsi le mani e testare alcune configurazioni utili a stabilire sessioni BGP con altri *bgp speaking router*. A seconda di chi ha implementato il protocollo BGP, è possibile trovare scostamenti nella sintassi e nelle opzioni usate nei dispositivi. Per questo qui vorremmo coprire almeno tre grandi categorie di software: il classico Cisco IOS, l'alternativo Juniper Junos e l'open-source OpenBGPD di OpenBSD.

**CISCO IOS**

Innanzitutto comunichiamo al *router* quale sia il suo sistema autonomo di appartenenza:

**router bgp 64500**

Indichiamo poi quale sia il prefisso che dovrà annunciare:

**network 203.0.113.0 mask 255.255.255.0**

è la volta del nostro dirimpettaio: quale è il suo indirizzo e a quale sistema autonomo appartiene?

**neighbor 198.51.100.1 remote-as 64496**

Inseriamo anche una descrizione per chiarezza:

**neighbor 198.51.100.1 description PEER v4 CON AS64496**

Ora, per far sì che la nostra rete 203.0.113.0/24 venga installata nella tabella BGP è necessario che appaia anche nella tabella degli instradamenti. Per questo la instradiamo verso l'interfaccia virtuale Null numero 0.

**ip route 203.0.113.0 255.255.255.0 Null0**

Vale ovviamente lo stesso ragionamento per IPv6. Di sguito tutto insieme::

  router bgp 64500
  network 203.0.113.0 mask 255.255.255.0
  network 2001:db8\::\/32
  neighbor 198.51.100.1 remote-as 64496
  neighbor 198.51.100.1 description PEER v4 CON AS64496
  neighbor fd66:32:48:64::1 remote-as 64496
  ip route 203.0.113.0 255.255.255.0 Null0
  ipv6 route 2001:db8\::\/32 Null0


Processo di instradamento
--------

BGP è un protocollo molto flssibile, per questo gode di ottima salute nonostante il peso degli anni e le mutanti esigenze dell'industria di Internet. La sua grande abilità è di rendere note le posizioni di tutte le risorse numeriche che si affacciano in Rete originanti dagli oltre 66mila sistemi autonomi a oggi attivi nel mondo.

Se in molti casi il processo per scegliere il miglior percorso (*best path*) verso una destinazione è assai lineare perché si può preferire semplicemente il percorso più breve (cioè l'*AS_PATH* più corto), a volte si deve applicare un chiaro algoritmo che i *router* devono osservare tutte le volte che per la stessa destinazione hanno a disposizione più percorsi diversi:


1. Preferire l'instradamento con il valore di *LOCAL_PREF* più alto.
2. Preferire l'instradamento con l'*AS_PATH* più corto.
3. Preferire l'instradamento con il tipo di *ORIGIN* più basso *( {[0] - IGP} < {[1] - EGP} < {[2] - INCOMPLETE})*.
4. Preferire l'instradamento con il valore di *MULTI_EXIT_DISC* più basso.
5. Preferire i percorsi appresi da *external BGP* a quelli appresi da *internal BGP*.
6. Preferire l'instradamento che può essere raggiunto attraverso il percorso più breve verso il *NEXT_HOP*.
7. Preferire l'instradamento appreso dal dispositivo con il *ROUTER_ID* più basso.
8. Preferire l'instradamento appreso dal dispositivo con il *NEIGHBOR_ID* più basso.

Alcune implementazioni presenti sul mercato aggiungono altri criteri selettivi come a esempio:

9. Preferire l'instradamento appreso (e installato nella tabella degli instradamenti) per primo.

Ovviamente se il *NEXT_HOP* non è raggiungibile allora l'instradamento viene ignorato, come pure se vengono implementate delle regole per filtrare via alcuni annunci.

Controllo degli instradamenti
--------
[todo]

Filtri e manipolazioni
--------
[todo]

Ridondanza e bilanciamento
--------
[todo]
