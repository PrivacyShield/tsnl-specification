# TSNL protocol
### **Time-based Spatial Network Layers**

Il protocollo TSNL ambisce a fornire un'astrazione del sistema di routing base fornito dagli ISP, dove ogni utente serve da nodo e relay per gli altri utenti. Offre anche l'aliasing degli indirizzi IP permettendo la comunicazione con un indirizzo senza mai avere la necessità nè possibilità di conoscere con certezza il suo indirizzo IP autentico. Il TSNL usa anche un sistema di crittografia, principalmente al fine di alterare il [wire image](https://en.wikipedia.org/wiki/Wire_data) di ogni pacchetto ogni qualvolta venga ridirezionato da un nodo, oltre che per un'essenziale protezione dei dati Point to Point. 

TSNL è essenzialmente progettato al fine di proteggere il più possibile il contenuto del traffico dati dei suoi utenti permettendo al contempo un'ampiezza di banda sufficiente per la trasmissione di dati multimediali in tempo reale come video streaming. 

Si può riassumere il TSNL come un abile mazziere in grado di mescolare le carte facendo perdere i punti di riferimento a chiunque le osservi.

## Concetti base
L'acronimo di TSNL inizia con le parole **Time-based Spatial** *Network Layers* proprio per il suo modo di localizzare spazialmente gli indirizzi nella mappa di routing, mentre il plurale di layer è dovuta alla natura multi dimensionale della rete. La TSNL non ha sotto reti e il raggruppamento degli indirizzi non avviene associando l'IP del nodo al suo blocco IP, ma bensì creando un punto di riferimento pseudo-spaziale: ogni indirizzo alias (rappresenta da intero da 48 bit generato casualmente) viene associato a delle coordinate x, y e z (che rappresentano concettualmente longitudine, latitudine e altitudine). Però queste coordinate non rappresentano realmente la posizione geografica del nodo, pur apparentendo ad un sistema di coordinate sferiche, ma bensì sono il risultato approssimativo della triangolazione tra punti di riferimento selezionato a parità d'altitudine, ovvero altri nodi nella stessa dimensione, dove la triangolazione viene calcolata in base ai tempi di ping tra i nodi, time-based per l'appunto (i tempi di ping potrebbero essere arbitrariamente condizionati da altri fattori, come il route trace o forme di sviamento casuale). Come vagamente anticipato, l'altitudine è un modo per rappresentare la dimensione su cui il nodo opera, in modo che sia matematicamente coerente da gestire per il calcolo delle route migliori, e le dimensioni hanno il fine di isolare su più layer i nodi così di limitare ulteriormente la visione generale dei nodi circostanti. 

### Terminologie
- **Pacchetti dati**: indica una porzione di dati di uno stream, che può corrispondere alla dimensione di un pacchetto TCP/IP o anche di più (come 1024, 2048 etc bytes), lasciando implicita la sua frammentazione.

## Crittografia
Esistono due livelli di crittografia: Point to Point e di routing, che usando entrambe in combinazione sia chiavi asimettriche che simmetriche. La prima serve per permettere al destinatario di verificare l'autenticità del messaggio ed impedire di far vedere il suo contenuto ai nodi di routing, mentre la seconda serve per rendere irriconoscibile la direzione di reindirizzamento di un pacchetto (o un insieme di pacchetti) da un relay. Al contempo il TSNL deve offrire un sistema di aggiornamento costante delle chiavi tra nodi, usando anche nodi intermediari come "testimoni e garanti", per impedire la decrittazione per forza bruta delle connessioni.

Al contempo, per quanto questo sia un sistema di crittografia essenziale, non è sufficiente per consentire un data shuffling efficace. Quindi a questo si aggiungo gli **algoritmi di shuffling**: questi sono degli insiemi di algoritmi che si aggiungono alla crittografia base sia in maniera arbitraria che casuale. Quindi si può trarre vantaggio di linguaggi di scripting standard come l'ECMA script (Javascript) per definire questi algoritmi da applicare dinamicamente per ogni connessione e flusso dati. 

Un esempio di algoritmo di shuffling può riguardare l'accumulare una serie di pacchetti all'interno dello stesso stream dati, dividerli in una serie di blocchi di cui scambiare l'ordine in base a punti di riferimento (seeds) accordati tra i nodi.

Oppure un sistema di "**dummy packets**", così da rendere difficile ricostruire l'effettiva dimensione dei dati in trasmissione sia fuorviare un osservatore da quali dati in trasmissione siano consistenti.

Un altro esempio può riguardato l'applicazione casuale di un cambio di route, che in ogni caso è implementato alla base del protocollo, come spiegato nel paragrafo successivo.

## Routing
Il routing dei pacchetti dati, calcolato dinamicamente in base alle coordinate spaziali dei due indirizzi in connessione, deve offrire anche una particolare diversificazione del percorso per ogni insieme di pacchetti in uscita, così sia da rendere difficile ricostruire sia la serie di connessioni in corso che l'intero stream di dati di una connessione. Il routing deve saper indirizzare con rapidità anche pacchetti inviati una tantum che gestire e approfondire la stabilità e diversificazione di connessioni stabili. In quest'ultimo caso i nodi in comunicazione devono in parallelo studiare e costruire una serie di tracciati stabili da prendere come punti di riferimento affidabili.

La creazione di path stabili avviene su multipli livelli: un nodo può comunicare agevolmente con i suoi nodi di vicinato della stessa dimensione, ma per far cambiare dimensione ad un pacchetto dati può avere solo due nodi di riferimento: un nodo superiore e un nodo inferiore. In casi normali, un pacchetto può essere direzionato unicamente verso la dimensione del destinatario. Però un algoritmo di shuffling può far cambiare liberamente la dimensione di un pacchetto al fine di generare in tempo reale una route imprevedibile. 

### Quantizzazione coordinate
Se un pacchetto dovesse per forza 

**todo: Approfondisci il funzionamento di route tables e alias resolving**

## Checksum
Il TNSL offre anche un sistema di checksum parallelo che permette di inviare pacchetti dati alla destinazione e allo stesso tempo, con un pacchetto separato, il checksum a 64 bit del pacchetto dati già criptato. Dato che il checksum richiede solo 8 bytes, 14 in totale aggiungendo i 6 bytes dell'indirizzo destinatario, è possibile accorpare, quando possibile, più checksum nello stesso pacchetto dati. Quindi si dovrebbe creare un'autostrada parallela con un sistema di routing ad hoc per la distribuzione dei checksum e la consegna per mezzo di tracciati alternativi. 

Per quanto un sistema di checksum possa essere problematico per connessioni di dati in tempo reale che richiedono un'alta ampiezza di banda, al contempo si possono implementare sistemi per la priorità del checksum, come per esempio permettendo la consegna al destinatario dei dati prima ancora di ricevere il checksum, trattanendo unicamente il checksum dei dati ricevuti così da verificarlo appena possibile. In caso di mancata ricezione, il nodo destinatario può richiedere il checksum, in caso di verifica fallita il nodo può interrompere la connessione in corso e propagare un **sub-ICMP** per allarmare i nodi coinvolti di un potenziale *man-in-the-middle*.

# Discussione

## Librerie e protocolli di terze parti

### GUN (JS - NPM)
La libreria GUN ([https://gun.eco/docs/](https://gun.eco/docs/)) per Javascript disponibile su NPM è un punto di riferimento molto valido per la creazione e gestione di database P2P (hash table). Fornisce anche un sistema di autenticazione degli utenti. È una soluzione eccellente per la creazione di punti di riferimento decentralizzati e indipendenti dal progetto Privacy Shield, permettendo una separazione dei compiti che garantisce una migliore resistenza alla censura.

## Riflessioni
- Il protocollo TSNL deve cercare di essere il più portabile e leggero possibile. Per la sua implementazione ho scelto NodeJS, cercando di sfrutta il più possibile codice interpretato e pseudo-interpretato dal runtime di node. Di conseguenza nell'implementazione base del protocollo bisogna evitare l'uso obbligatorio di programmi esterni come IPFS.