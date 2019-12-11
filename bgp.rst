BGP (Border Gateway Protocol)
================================================================

Premessa
--------

In questo spazio, nato da un'idea di `ITNOG <https://www.itnog.it/>`__ (*the ITalian Network Operators Group*), intendiamo fornire una breve guida ai principali concetti e meccanismi sottesi al funzionamento di Internet illustrando alcune delle architetture di instradamento basate sul protocollo *BGP - Border Gateway Protocol*.
L'obiettivo dunque non è quello di tradurre e parafrasare le RFC (*Request for comments*), né quello di somministrare un ricettario; piuttosto vorremmo divulgare (con linguaggio semplice) i capisaldi della letteratura conditi con esempi intuitivi così da facilitare la strada a quanti vorranno poi approfondire.

La struttura del testo si rifà a un approccio sempre verde, scandito secondo questi punti:

- protocolli di instradamento: fondamenti e concetti;
- distance vector vs link-state;
- come funziona BGP;
- sessioni BGP;
- processo di instradamento;
- controllo degli instradamenti;
- filtri e manipolazioni;
- ridondanza e bilanciamento.

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

Distance vector vs link-state
--------

Come calcolare la distanza più breve per raggiungere una destinazione? Questo è il campo dove si gioca la partita dei protocolli di instradamento sulla rete.

Quello basato sul vettore delle distanze prevede una lista (il vettore appunto) di distanze associate al prefisso di ogni destinazione appreso dai messaggi provenienti da altri *router* sulla rete. Cioè praticamente ciascun *router* calcola autonomamente il percorso migliore verso ogni destinazione e, subito dopo, invia il proprio vettore di distanze agli altri *router* in rete.

Così facendo, tutti i *router* coinvolti nel processo contribuiscono a influenzare le proprie tabelle di instradamento fino a convergere su una lista di *best path* condivisa.

Proprio l'aspetto della convergenza è stato riconosciuto nel tempo come momento critico per un protocollo. Cioè, quanto tempo occorre affinché l'intera rete (o meglio i *router* che la costituiscono) sia al corrente della comparsa, scomparsa o cambiamento di un particolare instradamento?

Le argomentazioni necessarie a dare una risposta a quella domanda costituiscono una base critica all'adozione su larga scala di protocolli *distance vector*: dunque lentezza nella convergenza e difficoltà di gestire un gran numero di destinazioni.

Alcuni esempi di protocolli di instradamento basati sull'algoritmo del *distance vector* sono: *RIP - Routing Information Protocol* `[RFC2453] <https://www.rfc-editor.org/rfc/rfc2453.txt>`__, *EIGRP - Enhanced Interior Gateway Routing Protocol*  `[RFC7868] <https://www.rfc-editor.org/rfc/rfc7868.txt>`__.

_____

Diverso protocollo di instradamento, e più complesso del precedente, è quello basato sull'algoritmo *link-state*. In questo caso i *router* si scambiano informazioni proprio sullo stato del collegamento e non si scambiano quindi tabelle di instradamento.

L'oggetto delle comunicazioni tra dispositivi che ne fanno uso risiede nelle informazioni su dispositivi e reti adiacenti incluse delle grandezze associate alla connessione. In altre parole ciascun *router* produce un messaggio che contiene una descrizione del dispositivo stesso e di dove si connette agli altri; messaggio che viene distribuito a tutti gli altri *router* della rete i quali lo archiviano in una base di dati interna. Così ciascun dispositivo sarà in grado di ricostruire autonomamente una topologia della rete che sarà uguale per tutti.

Dopodiché, tutti i *router* saranno in grado di calcolare e tratteggiare un albero (dove ciascun dispositivo pone sé stesso alla radice) di cosiddetti *best path* verso ciascuna destinazione applicando l'algoritmo *SPF, Shortest Path First*.

Queste caratteristiche rendono il *link-state* un algoritmo più adatto a essere impiegato in scenari grandi e complessi, tuttavia sempre interni a un sistema autonomo. Infatti su grandi reti, come Internet, l'instabilità di alcuni collegamenti renderebbe le ritrasmissioni e i conseguenti calcoli un lavoro troppo oneroso (e di conseguenza inefficiente) per i singoli *router*.

I due più importanti esempi di protocolli di instradamento basati sull'algoritmo *link-state* sono *OSPF - Open Shortest Path First* `[RFC2328] <http://www.rfc-editor.org/rfc/rfc2328.txt>`__ e *IS-IS - Intermediate System to Intermediate System* `[ISO/IEC 10589:2002] <http://standards.iso.org/ittf/PubliclyAvailableStandards/c030932_ISO_IEC_10589_2002(E).zip>`__.

Instradamento interno o esterno, statico o dinamico
--------

Abbiamo visto come diversi siano i metodi per rendere le risorse di rete raggiungibili, ma occorre aggiungere ancora un tassello determinante per la prosecuzione dell'illustrazione, e cioè il loro àmbito di applicazione. Per questo è necessario introdurre la nozione di sistema autonomo, fin qui solo velocemente menzionata.

Si tratta della cellula più piccola che dà vita all'organismo di Internet e dobbiamo immaginarla come la tessera di un puzzle la quale può trovarsi nel centro o ai bordi del quadro ma sempre con almeno due lati connessi ad altre tessere.

Da un punto di vista tecnico la definizione può essere rintracciata nella `[RFC1930] Guidelines for creation, selection, and registration of an Autonomous System (AS) <http://www.rfc-editor.org/rfc/rfc1930.txt>`__ dove si legge:

   *"Un sistema autonomo è un gruppo di uno o più prefissi IP gestito da uno o più operatori di rete con una politica di instradamento UNICA e BEN DEFINITA."*
   
Fino al 2007 la rappresentazione di un *AS* avveniva per mezzo di un numero a 16 bit, dopodiché per mezzo di un numero a 32 bit, come oggi regolata dalla `[RFC6793] BGP Support for Four-Octet Autonomous System (AS) Number Space <https://www.rfc-editor.org/rfc/rfc6793.txt>`__.

Più dettagliatamente possiamo considerare un "dentro" e un "fuori" dal punto di vista di un *AS* e cioè rispettivamente instradamenti *intra-AS* e instradamenti *inter-AS*.

Ora, gli instradamenti possono essere classificati anche per la modalità con la quale vengono appresi dai *router*: quando inseriamo manualmente un percorso verso una destinazione, allora si chiamerà "instradamento statico" (*static routing*); quando invece i dispositivi apprendono gli instradamenti grazie a un protocollo, allora si parlerà di "instradamento dinamico" (*dynamic routing*).

All'interno di questa ultima categoria distinguiamo: per il cosiddetto instradamento interno al sistema autonomo, *IGP - Interior Gateway Protocol* (come *RIP, EGRP, OSPF, IS-IS*); per l'instradamento esterno tra sistemi autonomi diversi, *EGP - Exterior Gateway Protocol* (come *BGP*).


Come funziona BGP
--------

Nato nel 1989, quando *IETF* (*Internet Engineering Task Force*) partorì la `[RFC1105] A Border Gateway Protocol (BGP) <https://www.rfc-editor.org/rfc/rfc1105.txt>`__ recante la versione 1 del protocollo, BGP subì nel tempo alcuni profondi cambiamenti e, nel 1995, RFC Editor pubblicò le specifiche della versione 4, oggi raccolte nella `[RFC4271] A Border Gateway Protocol 4 (BGP-4) <https://www.rfc-editor.org/rfc/rfc4271.txt>`__.

Il BGP si basa su un algoritmo di instradamento chiamato "vettore di percorsi" (*path vector*), cioè i messaggi che produce contengono una lista di percorsi dati dai sistemi autonomi che occorre attraversare per raggiungere una certa destinazione (identificata da un prefisso di rete).
   
=========== ============  ==========================
  Rete       Vicino               Percorso
=========== ============  ==========================
1.0.0.0/24  202.93.8.242  54728 20130 6939 13335
=========== ============  ==========================

Questo esempio si legge così:

è possibile raggiungere la rete 1.0.0.0/24 (*network*) attraverso il dispositivo 202.93.8.242 (*next hop*) il quale propone un percorso (*path*) che consiste nell'attraversare, oltre sé stesso ovviamente, gli *AS* 54728, 20130, 6939 per approdare infine all'*AS* 13335 dove la risorsa di destinazione risiede.

Sessioni BGP
--------

Processo di instradamento
--------

Controllo degli instradamenti
--------

Filtri e manipolazioni
--------

Ridondanza e bilanciamento
--------
