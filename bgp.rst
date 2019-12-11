BGP (Border Gateway Protocol)
================================================================

Premessa
--------

In questo spazio intendiamo fornire una breve guida ai principali concetti e meccanismi sottesi al funzionamento di Internet illustrando alcune delle architetture di instradamento basate sul protocollo *BGP*, Border Gateway Protocol.
L'intento dunque non è quello di tradurre e parafrasare le RFC, né quello di somministrare un ricettario; piuttosto vorremmo divulgare (con linguaggio semplice) i capisaldi della letteratura conditi con esempi intuitivi così da facilitare la strada a quanti vorranno poi approfondire.

La struttura del testo si rifà a un approccio sempre verde, scandito secondo questi punti:

- protocolli di routing: fondamenti e concetti;
- distance vector vs link-state;
- come funziona BGP;
- sessioni BGP;
- processo di routing;
- controllo degli instradamenti;
- filtri e manipolazioni;
- ridondanza e bilanciamento.

Per quanto possibile saranno indicate le fonti, le migliori pratiche e le recenti tecnologie così come pubblicate da `RFC EDITOR <https://rfc-editor.org>`__.
