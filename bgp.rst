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

Dopodiché, tutti i *router* saranno in grado di calcolare e tratteggiare un albero (dove ciascun dispositivo pone sé stesso alla radice) di cosiddetti *best path* verso ciascuna destinazione applicando l'algoritmo `SPF - Shortest Path First <http://www-m3.ma.tum.de/foswiki/pub/MN0506/WebHome/dijkstra.pdf>`_ concepito nel 1959 dal matematico olandese Edsger W. Dijkstra.

Queste caratteristiche rendono il *link-state* un algoritmo più adatto a essere impiegato in scenari grandi e complessi, tuttavia sempre interni a un sistema autonomo. Infatti su grandi reti, come Internet, l'instabilità di alcuni collegamenti renderebbe le ritrasmissioni e i conseguenti calcoli un lavoro troppo oneroso (e di conseguenza inefficiente) per i singoli *router*.

I due più importanti esempi di protocolli di instradamento basati sull'algoritmo *link-state* sono *OSPF - Open Shortest Path First* `[RFC2328] <http://www.rfc-editor.org/rfc/rfc2328.txt>`__ e *IS-IS - Intermediate System to Intermediate System* `[ISO/IEC 10589:2002] <http://standards.iso.org/ittf/PubliclyAvailableStandards/c030932_ISO_IEC_10589_2002(E).zip>`__.

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

A ogni modo il campo *marker* ha una consistenza fissa di 16 byte e viene usato per determinare se il messaggio BGP contiene informazioni di autenticazione oppure no.

Il campo denominato *length* serve a dichiarare la lunghezza dell'intero messaggio BGP, intestazione compresa; per questo è semplice calcolarne il valore minimo: 19 byte (cioè 16 il *marker*, 2 il *length*, 1 il *type*). Il valore massimo, per RFC, è di 4096 byte.

Il campo *type* definisce invece il tipo di messaggio trasmesso e può recare dei valori che possono indicare i seguenti codici:

- *OPEN*;
- *UPDATE*;
- *NOTIFICATION*;
- *KEEPALIVE*.

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

Innanzitutto partiamo dallo stato di riposo (*idle*) nel quale si trova un *router* prima di ricevere il via alla connessione che possiamo dare noi stessi intervenendo sulla configurazione del dispositivo. Ricevuto il via (*start*), il primo *router* tenta una connessione *TCP* sulla porta 179 del secondo e poi si mette in ascolto di risposte provenienti dal secondo *router*.

Ecco che entriamo nel passaggio di connessione (*connect*) durante il quale si attende che la connessione TCP avvenga con successo. In quest'ultimo caso si procede verso un ulteriore passaggio chiamato *opensent*. Se invece la connessione TCP non viene stabilita, allora si va verso il passaggio *active*. E ancora, nel caso in cui si esaurisca il tempo per l'operazione, si azzera il *timer* e viene ritentata una connessione TCP, mentre lo stato rimane *connect*. Altri eventi innescati dal sistema o manualmente da noi, producono il ritorno allo stati di riposo.

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
