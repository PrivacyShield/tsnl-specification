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
Esistono due livelli di crittografia: Point to Point e di routing, che usando in combinazione sia chiavi asimettriche (**RSA**) che simmetriche (**AES**). La prima serve per permettere al destinatario di verificare l'autenticità del messaggio ed impedire di rilevare il suo contenuto ai nodi di routing, mentre la seconda serve per rendere irriconoscibile la direzione di reindirizzamento di un pacchetto (o un insieme di pacchetti) da un relay. Al contempo TSNL deve offrire un sistema di aggiornamento costante delle chiavi tra nodi, usando anche nodi intermediari come "testimoni e garanti", per impedire la decrittazione per forza bruta delle connessioni.

Per quanto questo sia un sistema di crittografia essenziale, non è sufficiente per consentire un data shuffling efficace. Quindi a questo si aggiungo gli **algoritmi di shuffling**: questi sono degli insiemi di algoritmi che si aggiungono alla crittografia base sia in maniera arbitraria che casuale. Quindi si può trarre vantaggio di linguaggi di scripting standard come l'ECMA script (Javascript) per definire questi algoritmi da applicare dinamicamente per ogni connessione e flusso dati. 

Un esempio di algoritmo di shuffling può riguardare l'accumulare una serie di pacchetti all'interno dello stesso stream dati, dividerli in una serie di blocchi di cui scambiare l'ordine in base a punti di riferimento (seeds) accordati tra i nodi.

Oppure un sistema di "**dummy packets**", così da rendere difficile ricostruire l'effettiva dimensione dei dati in trasmissione sia fuorviare un osservatore da quali dati in trasmissione siano consistenti.

Un altro esempio può riguardato l'applicazione casuale di un cambio di route, che in ogni caso è implementato alla base del protocollo, come spiegato nel paragrafo successivo.

## Routing
Il routing dei pacchetti dati, calcolato dinamicamente in base alle coordinate spaziali dei due indirizzi in connessione, deve offrire anche una particolare diversificazione del percorso per ogni insieme di pacchetti in uscita, così sia da rendere difficile ricostruire sia la serie di connessioni in corso che l'intero stream di dati di una connessione. Il routing deve saper indirizzare con rapidità anche pacchetti inviati una tantum che gestire e approfondire la stabilità e diversificazione di connessioni stabili. In quest'ultimo caso i nodi in comunicazione devono in parallelo studiare e costruire una serie di tracciati stabili da prendere come punti di riferimento affidabili.

La creazione di path stabili avviene su multipli livelli: un nodo può comunicare agevolmente con i suoi nodi di vicinato della stessa dimensione, ma per far cambiare dimensione ad un pacchetto dati può avere una serie di nodi di riferimento: una serie di nodi superiori e una inferiore. In casi normali, un pacchetto può essere direzionato unicamente verso la dimensione del destinatario. Però un algoritmo di shuffling può far cambiare liberamente la dimensione di un pacchetto al fine di generare in tempo reale una route imprevedibile. 

### Quantizzazione coordinate
Se un pacchetto dovesse per forza passare per ogni relay frapposto fra due nodi, la latenza nella comunicazione aumenterebbe notevolmente con l'aumentare della distanza. Invece si dividono i nodi in regioni, relativi alla distanza media fra i nodi della rete moltiplicata per 4 (prende il nome di **Reference Region Size**): si prende appunto la latitudine e la longitudine del nodo e la si divide per RRS e si arrotonda il risultato. Questa operazione si esegue radoppiando RRS fino ad ottenere una scala con solo una regione, chiamata **Full Scale**. 

Quindi ogni nodo, oltre ad avere nella propria routes table i suoi nodi vicini, ha anche un insieme di nodi di riferimento per ognuna delle 8 regioni confinanti, eccetto per quelle prive di nodi, per ogni scala fino alla full scale. 

### Coordinate e relativismo
Rappresentare con precisione con coordinate spaziali la mappa dei nodi può apparire confusionario se non in alcuni casi impossibile da definire con certezza. C'è da dire che in realtà non c'è bisogno che questa rappresentazione sia ineccepibile, per quanto l'altitudine funzionando da dimensione richiede maggiore delicatezza nella gestione, rimane strettamente rappresentativa ed arbitraria: è un sistema per creare punti di riferimento stabili su basi però indicative. Si può definire la rappresentazione spaziale della rete TSNL come la media dei punti di vista dei vari nodi che la compongono.

### Altitudini e dimensioni
Le altitudini rappresentano le dimensioni. Ma cosa porta un dispositivo ad avere determinate altitudini? Questo succede quando la triangolazione lo colloca in una determinata longitudine e altitudine ma, relativamente agli altri nodi vicini, ha una latenza maggiore. Altri nodi apparterranno alla stessa latitudine se hanno lo stesso gap di latenza senza averlo con i nodi della stessa altitudine.

Però, per quanto la dimensione sia direttamente attribuibile all'altitudine, la sensibilità a questa e la sua precisione è scelto in rapporto agli altri nodi così da ottenere il rapporto migliore tra velocità e ridondanza. 

Un esempio che può rendere evidente il concetto di altitudine e dimensione può essere un dispositivo connesso ad internet usando la connessione mobile rispetto ad uno connesso tramite WAN Starlink dove, grazie alla comunicazione diretta fra i vari satelliti della rete, la comunicazione con altri dispositivi connessi via satellite anche se dall'altro capo del mondo è decisamente più rapida.

**todo: Approfondisci il funzionamento di route tables e alias resolving**

## Checksum
Il TNSL offre anche un sistema di checksum parallelo che permette di inviare pacchetti dati alla destinazione e allo stesso tempo, con un pacchetto separato, il checksum a 64 bit del pacchetto dati già criptato. Dato che il checksum richiede solo 8 bytes, 14 in totale aggiungendo i 6 bytes dell'indirizzo destinatario, è possibile accorpare, quando possibile, più checksum nello stesso pacchetto dati. Quindi si dovrebbe creare un'autostrada parallela con un sistema di routing ad hoc per la distribuzione dei checksum e la consegna per mezzo di tracciati alternativi. 

Per quanto un sistema di checksum possa essere problematico per connessioni di dati in tempo reale che richiedono un'alta ampiezza di banda, al contempo si possono implementare sistemi per la priorità del checksum, come per esempio permettendo la consegna al destinatario dei dati prima ancora di ricevere il checksum, trattanendo unicamente il checksum dei dati ricevuti così da verificarlo appena possibile. In caso di mancata ricezione, il nodo destinatario può richiedere il checksum, in caso di verifica fallita il nodo può interrompere la connessione in corso e propagare un **sub-ICMP** per allarmare i nodi coinvolti di un potenziale *man-in-the-middle*.

## Ruoli e attributi
Ogni nodo può avere particolari ruoli e attributi all'interno della rete in base alla loro velocità, latenza, stabilità, tempo online e reputazione. Questi attributi danno una particolare priorità e responsabilità/potere all'interno di una regione se non dell'intera rete. Per esempio sono essenziali una serie di nodi in grado di permettere l'inizializzazione di un nodo alla rete sia agendo come punto di riferimento con il suo indirizzo IP, sia gestendo e fornendo informazioni essenziali al nodo per localizzarsi nella giusta regione, ottenendo le migliori coordinate e di conseguenza vicini.

Via andando, i ruoli possono portare un nodo a specializzarsi in funzioni come la gestione del traffico del checksum, che richiede bassa latenza e buona capacità di calcolo del processore, come altri ruoli possono trarre vantaggio da una buona capacità di archiviazione a propria disposizione.

## Dimensioni
La definzione delle dimensioni per mezzo dell'altitudine di un nodo ha il fine di localizzare in maniera coerente un connessione ad internet che pare più o meno veloce rispetto alla media senza però "rompere" la rappresentazione geografica fatti dei vari nodi. Ponendo quindi un nodo più in alto o più in basso, si può permettere di mantenere la stessa longitudine e latitudine. 

La definizione a "dimensione" cerca di trarre vantaggio dalla possibilità che i nodi della stessa dimensione possano comunicare fra di loro in modo più rapido, rendendo il loro utilizzo vantaggioso per le comunicazioni sulla lunga distanza. Al contempo, però, etichettare a tutti i costi ogni connessione e irrigidire il cambio di dimensioni da parte di una path può creare un ambiente favorevole all'abbuso di determinate path e dimensioni, rendendo più facile intercettare e controllare le connessioni in corso da parte di un malintenzionato.

Questo mi porta ad approfondire la questione per trarne dei princìpi fondamentali sulla definizione ad uso delle dimensioni:

- L'altitudine di un nodo va ad incidere sulla sua conseguenza dimensione <ins>in questo caso</ins>, perchè è utile al fine per cui il protocollo vuole essere utilizzato.
- All'interno della stessa rete, quindi, possono essere definiti più tipi di dimensioni a cui un nodo può appartenere (che si lega anche strettamente agli attributi di un nodo definiti), come per esempio la capienza della memoria a sua disposizione, i tempi di elaboramento di un'operazione, l'uptime etc.
- In precedenza è stata data una regola stringente sul come un pacchetto dati possa cambiare dimensione durante la sua trasmissione. Però permettere in maniera così stringente la trasmissione su una dimensione differente può creare delle lacune nel sistema di data shuffling. Quindi queste regole stringenti andrebbero tenute solo per le situazioni di effettivo vantaggio con una maggiore priorità rispetto ad altri scambi di dati, che però devono avvenire ugualmente come su un qualsiasi nodo della rete, in base all'effettiva capacità di trasmissione della connessione del nodo. Ma un abuso di questa strutturazione, anche se in normali casi aumenta l'efficienza nell'uso dei nodi, crea una rete dal comportamento facilmente prevedibile. 

### Come definire le dimensioni "mean"
Le dimensione "mean" sta per mezzo, ovvero mezzo di comunicazione. Anche se questo non va confuso con l'ISP ma più genericamente con la tecnlogica usata per la trasmissione e il ruoting dei dati, le due cose possono comunque coincidere. 
Quindi l'altitudine verrà importata al fine di trovare l'altezza tale che al contempo giustifichi al triangolazione longitudinale e latitudinale con gli altri nodi, mentre le creazione di una dimensione *de-facto* avviene quando un nodo con latenze maggiori con i suoi nodi vicini, non risulta avere lo stesso rapporto di latenza con altri nodi della stessa altitudine, formando di fatto una dimensione fra loro.
Le dimensioni mean non vanno definite in base all'ISP usato o ai ruoter in comune, unicamente in base ai valori delle performance ottenute in fase di inizializzazione e aggiornamento.

## Distributed Hash Table
Ogni nodo della rete, seppur senza richiedere particolari spazio di archiviazione, avrà la responsabilità di gestire in maniera casuale e ridondante una porzione dati delle tabelle necessarie per l'accesso rapido ad informazioni essenziali, come la risoluzione degli indirizzi alias.

## Esempi e flussi
Esempi pratici e rappresentazione del funzionamento delle operazioni nella rete può semplificare notevolmente sia la spiegazione che la compresione del funzionamento del TSNL.

### A. Connessione di un nodo alla rete
I primi 2 punti possono essere bypassati nel caso non fosse la prima connessione del nodo alla rete con lo stesso ISP nella stessa regione.

#### **1. Recupero degli indirizzi di riferimento**
Con l'ausilio di un database P2P, prendendo il protocollo GUN come riferimento, si legge la tabella contente gli indirizzi IP dei server master, ovvero al livello superiore.

#### **2. Connessione al master node e ricerca regione**
Il master node fornisce almeno un paio di indirizzi IP di nodi per ogni regione a livello superiore, il client calcola la regione con minore latenza e richiede altri nodi di riferimento per ognuna delle 9 regioni della corrente regione, ripetendo l'operazione finchè non raggiunge la regione di scala 1:1. 

#### **3. Calcolo nodi vicini**
Su selezione casuale per ogni radiante, con l'aiuto dei nodi della stessa regione, ottiene dei nodi di referenza per ognuna delle 4 direzioni possibili. Una maggiore spartizione in radianti delle direzioni di riferimento potrebbero paradossalmente creare delle route riconoscibili in base all'indirizzo destinatario. La stessa operazione va fatta per ogni regione superiore e per la dimensione superiore ed inferiore.

I ping dei nodi vicini va eseguito contempoeanemente, così da autoamticamente escludere le correnti fluttuazioni della rete. Ad questo, può essere aggiunto un superficiale traceroute/tracert per definire con maggiore profondità il tipo di connessione. 

#### **4. Consolidazione peers**
Una volta effettuato l'handshake con i vicini si può consolidare la propria posizione come nodo nella rete così da poter usato per il routing dei pacchetti. Nel caso il nodo si frapponesse fra due nodi nella stessa regione, questi sostituirebbero il vicino con il nuovo nodo.

#### E se non ci fosse alcun master node?
Come si comporta il primo nodo in assoluto a connettersi alla rete? In questo caso si da' per assunto che le sue coordinate corrispondano al punto zero. Il nodo ricoprerebbe in automatico il ruolo di master node aggiungendo il suo indirizzo IP alla tabella GUN. 

Va considerato il caso particolare in cui, per qualsiasi ragione, il nodo si convincesse erroneamente di essere l'unico nodo disponibile. In quel caso è necessario implementare algoritmi di controllo, affinchè il nodo "si disconnetta" e riconnetta alla rete reale. In alcuni casi è persino plausibile la possibilità che si formino due reti separate composte da indirizzi IP che non possono comunicare fra loro, per qualsiasi ragione come a causa di un blocco regionale degli IP. In tal caso bisogna accettare e gestire la co-esistenza delle due reti e preferibilmente implementare l'uso di tunnel. Va valutato anche il caso in cui per qualsiasi ragione il blocco si risolvesse, se riconnettere tutti i nodi della rete più piccola alla rete maggiore oppure implementare un sistema di propagazione dell'offset (che al contempo potrebbe esporre una rete ad un attacco auto distruttivo), così da collegare in maniera trasparente le due reti. L'effettiva complessità nella gestione di questa condizione dipenderà anche dall'implementazione finale del protocollo.

### B. Invito di un pacchetto dati ad un altro nodo
Sono stanco, come sempre

# Discussione

## Precisazioni
- Il protocollo, specie nelle fasi iniziali, può apparire confusionario ed incerto. C'è da precisare che molte definizioni verrano chiarite durante l'implementazione pratica del protocollo.
- È importante integrare un sistema per la verifica dell'integrità sia degli algoritmi di shuffling che per il programma stesso, che sia impacchettato o come codice sorgente Javascript.

## Tecnologie esterne

### GUN (JS - NPM)
La libreria GUN ([https://gun.eco/docs/](https://gun.eco/docs/)) per Javascript disponibile su NPM è un punto di riferimento molto valido per la creazione e gestione di database P2P (hash table). Fornisce anche un sistema di autenticazione degli utenti. È una soluzione eccellente per la creazione di punti di riferimento decentralizzati e indipendenti dal progetto Privacy Shield, permettendo una separazione dei compiti che garantisce una migliore resistenza alla censura.

### QUIC
QUIC è un transportation layer alternativo a TCP, volto a renderlo obsoleto. [https://en.wikipedia.org/wiki/QUIC](https://en.wikipedia.org/wiki/QUIC)

### ICE (NAT management)
[https://en.wikipedia.org/wiki/Interactive_Connectivity_Establishment](https://en.wikipedia.org/wiki/Interactive_Connectivity_Establishment)

Interactive Connectivity Establishment (ICE) is a technique used in computer networking to find ways for two computers to talk to each other as directly as possible in peer-to-peer networking.

Usato per ottenere la via di contatto più diretta possibile fra due indirizzi IP. 

## Hardware
Data la sua portabilità, potrebbe essere un notevole punto forza fornire una distribuzione Linux per **Raspberry PI** con i protocolli Privacy Shield, in versione server con interfaccia web UI e in versione **media center** collegandolo ad un televisore. Il costo dei Raspberry PI 4 (ormai obsoleti) partono da 30 euro, rendendolo un dispositivo competitivo, da porre come dispositivo DMZ in una connessione ad internet.

## Riflessioni
- Il protocollo TSNL deve cercare di essere il più portabile e leggero possibile. Per la sua implementazione ho scelto NodeJS, cercando di sfrutta il più possibile codice interpretato e pseudo-interpretato dal runtime di node. Di conseguenza nell'implementazione base del protocollo bisogna evitare l'uso obbligatorio di programmi esterni come IPFS.
- Si potrebbe ridurre anche la dimensione del checksum a 48 bit per consentire un buon rapporto definizione/peso.
- Per come si sta strutturando il protocollo, questo può essere riutilizzato per la creazione di reti di computazione distribuita.