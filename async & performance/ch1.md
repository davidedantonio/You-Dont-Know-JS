# Tu non conosci JS: Async e Performance
# Capitolo 1: Asincronia: Ora e Dopo

Una delle parti più importanti e spesso incomprese della programmazione in un linguaggio come JavaScript è come esprimere e manipolare il comportamento del programma distribuito su un periodo di tempo.

Questo non riguarda solo ciò che accade dall'inizio di un ciclo `for` alla fine di un ciclo` for`, che ovviamente richiede *un certo tempo* (microsecondi in millisecondi) per essere completato. Riguarda cosa succede quando parte del tuo programma è in esecuzione *ora*, e un'altra parte del tuo programma viene eseguita *dopo* -- c'è uno spazio tra *ora* e *dopo* dove il tuo programma non è in esecuzione.

Praticamente tutti i programmi non banali mai scritti (specialmente in JS) hanno in qualche modo dovuto gestire questo gap, sia che si tratti di aspettare l'input dell'utente, richiedere dati da un database o da un file system, inviare dati attraverso la rete e attendere un risposta o esecuzione di un'attività ripetuta a intervalli di tempo prestabiliti (come un'animazione). In tutti questi scenari, il tuo programma deve gestire lo stato attraverso il divario nel tempo. Come dicono notoriamente a Londra (al baratro tra la porta della metropolitana e il binario): "pensa al divario".

In effetti, la relazione tra l'*ora* e il *dopo* nel tuo programma è al centro della programmazione asincrona.

Sicuramente la programmazione asincrona è esistita sin dall'inizio di JS. Ma la maggior parte degli sviluppatori di JS non ha mai considerato con attenzione esattamente come e perché è cresciuta nei loro programmi, o esplorato *altri* modi per gestirla. L'approccio *abbastanza buono* è sempre stato l'umile funzione di callback. Molti oggi insistono sul fatto che le callbacks sono più che sufficienti.

Tuttavia, mentre JS continua a crescere sia in termini di portata che di complessità, per soddisfare le sempre maggiori esigenze di un linguaggio di programmazione di prima classe che gira nei browser e sui server e ogni dispositivo immaginabile nel mezzo, i dolori con cui gestiamo l'asincronia stanno diventando sempre più paralizzanti e invocano approcci che sono entrambi più capaci e più ragionevoli.

Mentre tutto questo può sembrare alquanto astratto in questo momento, ti assicuro che lo affronteremo in modo più completo e concreto mentre andando avanti in questo libro. Esploreremo una serie di nuove tecniche per la programmazione asincrona di JavaScript nei prossimi capitoli.

Ma prima che possiamo arrivarci, dovremo capire molto più profondamente che cos'è l'asincronia e come funziona in JS.

## Un Programma in Blocchi

Puoi scrivere il tuo programma JS in un file *.js*, ma il tuo programma è quasi certamente composto da diversi blocchi, uno solo dei quali sta per essere eseguito *ora*, mentre il resto verrà eseguito *successivamente*. L'unità più comune di *blocco* è la `funzione`.

Il problema che la maggior parte degli sviluppatori nuovi di JS sembrano avere è che *successivamente* non avviene in modo rigoroso e immediatamente dopo *ora*. In altre parole, le attività che non sono completate *ora*, per definizione, sono completate in modo asincrono, e quindi non avremo il comportamento di bloccante come si era previsto o voluto.

Consideriamo:

```js
// ajax(..) è una funzione Ajax fornita da una libreria
var data = ajax( "http://some.url.1" );

console.log( data );
// Oops! `data` generalmente non avranno i risultati Ajax
```

Probabilmente siete a conoscenza del fatto che le richieste Ajax standard non vengono completate in modo sincrono, il che significa che la funzione `ajax (..)` non ha ancora alcun valore da ritornare per essere assegnato alla variabile `data`. Se `ajax (..)` *potrebbe* bloccare fino a quando la risposta è arrivata, allora l'assegnazione `data = ..` funzionerebbe correttamente.

Ma non è così che utilizziamo Ajax. Facciamo una richiesta Ajax asincrona *ora*, e non otterremo i risultati prima di *dopo*.

Il modo più semplice (ma sicuramente non il solo, o necessariamente anche il migliore!) di "aspettare" da *ora* fino a *dopo* è di usare una funzione, comunemente chiamata funzione di callback:

```js
// ajax(..) è una funzione Ajax fornita da una libreria
ajax( "http://some.url.1", function laMiaFunzioneDiCallback(data){

  console.log( data ); // Yay, Ho dei dati in `data`!

} );
```

**Attenzione:** Potresti aver sentito che è possibile effettuare richieste Ajax sincrone. Anche se questo è tecnicamente vero, non dovresti mai, mai farlo, in nessuna circostanza, perché blocca l'interfaccia utente del browser (pulsanti, menu, barre di scorrimento, ecc.) E impedisce qualsiasi interazione dell'utente. Questa è un'idea terribile e dovrebbe sempre essere evitata.

Prima di protestare per il disaccordo, no, il tuo desiderio è quello di evitare il caos delle callback e *non* giustificare il blocco, di una chiamata Ajax sincrona.

Ad esempio, considera questo codice:

```js
function ora() {
  return 21;
}

function dopo() {
  risposta = risposta * 2;
  console.log( "Significato della vita:", risposta );
}

var risposta = ora();

setTimeout( dopo, 1000 ); // Significato della vita: 42
```

Ci sono due blocchi in questo programma: quello che verrà eseguito *ora* e quello che verrà eseguito *dopo*. Dovrebbe essere abbastanza ovvio quali siano questi due blocchi, ma siamo super espliciti:

Ora:
```js
function ora() {
  return 21;
}

function dopo() { .. }

var risposta = ora();

setTimeout( dopo, 1000 );
```

Later:
```js
risposta = risposta * 2;
console.log( "Significato della vita:", risposta );
```
Il blocco *ora* viene eseguito immediatamente non appena si esegue il programma. Ma `setTimeout (..)` imposta anche un evento (un timeout) che si verifica *dopo*, quindi il contenuto della funzione `dopo()` verrà eseguito in un secondo momento (1.000 millisecondi da ora).

Ogni volta che si avvolge una porzione di codice in una `funzione` e si specifica che deve essere eseguita in risposta a qualche evento (timer, clic del mouse, risposta Ajax, ecc.), Si sta creando un blocco *dopo* del codice, e quindi stai introducendo l'asincronia al tuo programma.

### Console Asincrona

Non ci sono specifiche o set di requisiti su come funzionano i metodi `console.*` - non sono ufficialmente parte di JavaScript, ma sono invece aggiunti a JS dall'*ambiente ospitante* (vedi *Tipi e grammatica* di questa serie di libri).

Quindi, diversi browser e ambienti JS fanno ciò che vogliono, il che a volte può portare a comportamenti che possono confondere.

In particolare, ci sono alcuni browser e alcune circostanze dove `console.log(..)` in realtà non emette immediatamente ciò che gli viene dato. Il motivo principale per cui ciò potrebbe accadere è che l'I/O è solitamente la parte lenta e bloccante di molti programmi (non solo in JS). Quindi, potrebbe funzionare meglio (dal punto di vista della pagina/interfaccia utente) per un browser che gestisce l'I/O della console in modo asincrono in background, senza che tu nemmeno sappia che ciò è accaduto.

Uno scenario non molto comune, ma possibile, in cui ciò potrebbe essere *osservabile* (non dal codice stesso ma dall'esterno):

```js
var a = {
  indice: 1
};

// dopo
console.log( a ); // ??

// anche dopo
a.indice++;
```
Normalmente ci aspettiamo di vedere l'oggetto `a` essere istantaneamente catturato nel momento esatto dell'istruzione `console.log(..)`, stampando qualcosa come `{indice: 1}`, tale che nella prossima istruzione quando `a.indice++` viene eseguita, sta modificando qualcosa di diverso, o solo immediatamente dopo, l'output di `a`.

Nella maggior parte dei casi, il codice precedente produrrà probabilmente una rappresentazione di oggetti nella console degli strumenti per sviluppatori che è ciò che ti aspetteresti. Ma è possibile che questo stesso codice possa essere eseguito in una situazione in cui il browser sentiva la necessità di posticipare l'I/O della console in background, nel qual caso è *possibile* che al momento della rappresentazione dell'oggetto nella console del browser, l'istruzione `a.index++` sia già stata eseguita e quindi mostrerà `{index: 2}`.

È un bersaglio mobile in quali condizioni esattamente l'I/O della `console` verrà differito, o anche se sarà osservabile. Basta essere consapevoli di questa possibile asincronicità in I/O nel caso in cui si verifichino problemi nel debug in cui gli oggetti sono stati modificati *dopo* un'istruzione `console.log(..)` e tuttavia si vedranno le modifiche impreviste.

**Nota:** Se ti imbatti in questo raro scenario, l'opzione migliore è usare i breakpoint nel tuo debugger JS invece di affidarti all'output di `console`. L'opzione migliore sarebbe quella di forzare una "istantanea" dell'oggetto in questione serializzandolo su una `stringa`, come con `JSON.stringify(..)`.

## Ciclo degli eventi

Facciamo una affermazione (forse scioccante): nonostante abbia chiaramente permesso il codice asincrono JS (come il timeout che abbiamo appena visto), fino a poco tempo fa (ES6), lo stesso JavaScript non ha mai avuto alcuna nozione diretta di asincronia incorporata.

** Cosa!? ** Sembra un'affermazione pazzesca, giusto? In effetti, è abbastanza vero. Il motore JS stesso non ha mai fatto niente di più che eseguire un singolo blocco del programma in un dato momento, quando richiesto.

"Chiesto a." Da chi? Questa è la parte importante!

Il motore JS non funziona separatamente. Funziona all'interno di un *ambiente ospitante*, che è per la maggior parte degli sviluppatori il tipico browser web. Negli ultimi anni (ma non esclusivamente), JS si è espanso oltre il browser in altri ambienti, come i server, tramite cose come Node.js. Infatti, JavaScript viene incorporato in tutti i tipi di dispositivi in questi giorni, dai robot alle lampadine.

Ma l'unico "thread" comune (che è uno scherzo asincrono non così sottile, per quello che vale) di tutti questi ambienti è che hanno un meccanismo in loro che gestisce l'esecuzione di più blocchi del programma *nel tempo*, nei momenti in cui viene invocato il motore JS, chiamato "ciclo degli eventi".

In altre parole, il motore JS non ha avuto alcun senso innato di *tempo*, ma è stato invece un ambiente di esecuzione su richiesta per qualsiasi snippet arbitrario di JS. È l'ambiente circostante che ha sempre *schedulato* "eventi" (esecuzioni di codice JS).

Così, per esempio, quando il tuo programma JS fa una richiesta Ajax per recuperare alcuni dati da un server, tu immetti il codice della "risposta" in una funzione (comunemente chiamata "callback"), e il motore JS dice all'ambiente ospitante, "Ehi, ho intenzione di sospendere l'esecuzione per ora, ma ogni volta che finisci con quella richiesta, ed hai alcuni dati, ti preghiamo di *chiamare* questa funzione *di nuovo*."

Il browser viene quindi configurato per ascoltare la risposta dalla rete e, quando ha qualcosa da darti, pianifica la funzione di callback da eseguire inserendola nel *ciclo degli eventi*.

Allora, cos'è il *ciclo di eventi*?

Concettualizziamolo prima attraverso un codice finto:

```js
// `cicloEventi` è un array che funge da coda (first-in, first-out)
var cicloEventi = [ ];
var evento;

// continua a ciclare "per sempre"
while (true) {
  // esegui un "tick"
  if (cicloEventi.length > 0) {
    // prendi il prossimo evento dalla coda
    evento = cicloEventi.shift();

    // ora, esegui il prossimo evento
    try {
      evento();
    }
    catch (err) {
      comunicaErrore(err);
    }
  }
}
```
Questo è, naturalmente, uno pseudocodice notevolmente semplificato per illustrare i concetti. Ma dovrebbe essere sufficiente per aiutare a capire meglio.

Come puoi vedere, c'è un ciclo infinito in esecuzione rappresentato dal ciclo `while`, e ogni iterazione di questo ciclo è chiamato "tick". Per ogni tick, se un evento è in attesa sulla coda, viene rimosso ed eseguito. Questi eventi sono le tue funzioni di callback.

È importante notare che `setTimeout (..)` non mette la tua callback nella coda del ciclo degli eventi. Quello che fa è impostare un timer; quando il timer scade, l'ambiente colloca la callback nel ciclo degli eventi, in modo tale che alcuni tick futuri la raccolgano e la eseguano.

Cosa succede se ci sono già 20 elementi nel ciclo degli eventi in quel momento? La tua callback attende. Si allinea dietro agli altri - normalmente non c'è un percorso per anticipare la coda e saltare in fila. Questo spiega perché i timer `setTimeout (..)` potrebbero non funzionare con una precisione temporale perfetta. L'unica garanzia (in parole povere) è che la tua callback viene eseguita *prima* dell'intervallo di tempo specificato, ma può essere eseguita a partire da quel momento, a seconda dello stato della coda degli eventi.

In altre parole, il tuo programma è generalmente suddiviso in molti piccoli blocchi, che si succedono uno dopo l'altro nella coda del ciclo degli eventi. E tecnicamente, altri eventi non correlati direttamente al tuo programma possono essere intercalati all'interno della coda.

**Nota:** Abbiamo menzionato "fino a poco tempo fa" in relazione a ES6 cambiando la natura di dove viene gestita la coda del ciclo di eventi. Per lo più è un tecnicismo formale, ma ES6 specifica come funziona il ciclo degli eventi, il che significa che tecnicamente è nell'ambito di competenza del motore JS, piuttosto che solo dell'*ambiente di ospitante*. Una delle ragioni principali di questo cambiamento è l'introduzione di ES6 delle Promises, di cui parleremo nel Capitolo 3, poiché richiedono la possibilità di avere un controllo diretto e preciso sulle operazioni pianificate sulla coda del ciclo di eventi (vedi la discussione di `setTimeout(..0)` nella sezione "Cooperazione").

## Threads Paralleli

È molto comune confondere i termini "asincrono" e "parallelo", ma in realtà sono abbastanza diversi. Ricorda, asincrono riguarda il divario tra *ora* e *più tardi*. Mentre il parallelo riguarda le cose che possono accadere simultaneamente.

Gli strumenti più comuni per l'esecuzione parallela sono processi e thread. Processi e thread vengono eseguiti in modo indipendente e possono essere eseguiti simultaneamente: su processori separati o persino su computer separati, ma più thread possono condividere la memoria di un singolo processo.

Un ciclo di eventi, al contrario, suddivide il lavoro in attività e le esegue in serie, impedendo l'accesso parallelo e le modifiche alla memoria condivisa. Il parallelismo e la "serializzazione" possono coesistere sotto forma di ciclo di eventi cooperanti in threads separati.

L'interlacciamento di thread paralleli di esecuzione e l'interlacciamento di eventi asincroni si verificano a livelli molto diversi di granularità.

Per esempio:

```js
function dopo() {
  risposta = risposta * 2;
  console.log( "Significato della vita:", risposta );
}
```
Mentre l'intero contenuto di `dopo()` sarebbe considerato come un singolo elemento nella coda del ciclo degli eventi, e si pensa ad un thread su cui questo codice dev'essere eseguito, in realtà ci sono una dozzina di operazioni di basso livello diverse. Ad esempio, `risposta = risposta * 2` richiede prima il caricamento del valore corrente di `risposta`, quindi mette `2` da qualche parte, poi esegue la moltiplicazione, poi prende il risultato e lo memorizza in `risposta`.

In un ambiente a thread singolo, non importa se gli elementi nella coda del thread sono operazioni di basso livello, perché nulla può interrompere il thread. Ma se si dispone di un sistema parallelo, in cui due thread diversi operano nello stesso programma, è molto probabile che si abbia un comportamento imprevedibile.

Consideriamo:

```js
var a = 20;

function foo() {
  a = a + 1;
}

function bar() {
  a = a * 2;
}

// ajax(..) è una funzione Ajax fornita da una libreria
ajax( "http://some.url.1", foo );
ajax( "http://some.url.2", bar );
```
Nel comportamento a thread singolo di JavaScript, se `foo()` viene eseguito prima di `bar()`, il risultato è che `a` è `42`, ma se `bar()` viene eseguito prima di `foo()` il risultato in `a` sarà `41`.

Se gli eventi JS che condividono gli stessi dati eseguiti in parallelo, tuttavia, i problemi sarebbero molto più sottili. Considera questi due blocchi di pseudocodice come i thread che possono eseguire il codice rispettivamente in `foo()` e `bar()`, e considera cosa succede se vengono eseguiti esattamente nello stesso momento:

Thread 1 (`X` e` Y` sono locazioni di memoria temporanee):
```
foo():
  a. memorizza il valore di `a` in `X`
  b. memorizza `1` in `Y`
  c. somma `X` e `Y`, memorizza il risultato in `X`
  d. memorizza il valore di `X` in `a`
```

Thread 2 (`X` and `Y` sono locazioni di memoria temporanee):
```
bar():
  a. memorizza il valore di `a` in `X`
  b. memorizza `2` in `Y`
  c. moltiplica `X` e `Y`, memorizza il valore in `X`
  d. memorizza il valore di `X` in `a`
```

Ora, diciamo che i due thread vengono eseguiti veramente in parallelo. Probabilmente puoi individuare il problema, giusto? Usano le posizioni di memoria condivisa `X` e `Y` per i loro passaggi temporanei.

Qual è il risultato finale in `a` se i passaggi si verificano in questo modo?

```
1a  (memorizza il valore di `a` in `X`   ==> `20`)
2a  (memorizza il valore di `a` in `X`   ==> `20`)
1b  (memorizza `1` in `Y`   ==> `1`)
2b  (memorizza `2` in `Y`   ==> `2`)
1c  (somma `X` e `Y`, memorizza il risultato in `X`   ==> `22`)
1d  (memorizza il valore di `X` in `a`   ==> `22`)
2c  (moltiplica `X` e `Y`, memorizza il risultato in `X`   ==> `44`)
2d  (memorizza il valore di `X` in `a`   ==> `44`)
```

Il risultato in `a` sarà `44`. Ma per quanto riguarda questo ordinamento?

```
1a  (memorizza il valore di `a` in `X`   ==> `20`)
2a  (memorizza il valore di `a` in `X`   ==> `20`)
2b  (memorizza `2` in `Y`   ==> `2`)
1b  (memorizza `1` in `Y`   ==> `1`)
2c  (moltiplica `X` e `Y`, memorizza il risultato in `X`   ==> `20`)
1c  (somma `X` e `Y`, memorizza il risultato in `X`   ==> `21`)
1d  (memorizza il valore di `X` in `a`   ==> `21`)
2d  (memorizza il valore di `X` in `a`   ==> `21`)
```

Il risultato in `a` sarà `21`.

Quindi, la programmazione a thread è molto complicata, perché se non si adottano misure particolari per evitare che questo tipo di interruzione/interlacciamento accada, è possibile ottenere un comportamento molto sorprendente e non deterministico che spesso porta a mal di testa.

JavaScript non condivide mai i dati tra thread, il che significa che *quel* livello di non determinismo non è un problema. Ma questo non significa che JS sia sempre deterministico. Ricorda prima, dove l'ordinamento relativo di `foo()` e `bar()` produce due risultati diversi (`41` o `42`)?

**Nota:** Potrebbe non essere ancora evidente, ma non tutto il nondeterminismo è cattivo. A volte è irrilevante e talvolta è intenzionale. Ne vedremo altri esempi in tutto questo e nei prossimi capitoli.

### Esecuzione-per-Completare

A causa del single-threading di JavaScript, il codice all'interno di `foo()` (e `bar()`) è atomico, il che significa che una volta che `foo()` inizia a funzionare, la totalità del suo codice finirà prima di qualsiasi il codice in `bar()` possa essere eseguito, o viceversa. Questo è chiamato comportamento "Esecuzione-per-Completare".

In effetti, la semantica Esecuzione-per-Completare è più ovvia quando `foo()` e `bar()` contengono più codice, come:

```js
var a = 1;
var b = 2;

function foo() {
  a++;
  b = b * a;
  a = b + 3;
}

function bar() {
  b--;
  a = 8 + b;
  b = a * 2;
}

// ajax(..) è una funzione Ajax fornita da una libreria
ajax( "http://some.url.1", foo );
ajax( "http://some.url.2", bar );
```

Poiché `foo()` non può essere interrotto da `bar()`, e `bar()` non può essere interrotto da `foo()`, questo programma ha solo due possibili risultati a seconda di quale inizia per primo - se il threading era presente e le singole istruzioni in `foo()` e `bar()` potevano essere intercalate, il numero di possibili risultati sarebbe notevolmente aumentato!

Il Blocco 1 è sincrono (accade *ora*), ma i blocchi 2 e 3 sono asincroni (accadono *dopo*), il che significa che la loro esecuzione sarà separata da un intervallo di tempo.

Blocco 1:
```js
var a = 1;
var b = 2;
```

Blocco 2 (`foo()`):
```js
a++;
b = b * a;
a = b + 3;
```

Blocco 3 (`bar()`):
```js
b--;
a = 8 + b;
b = a * 2;
```

I blocchi 2 and 3 possono essere eseguiti entrambi per primi, quindi ci sono due possibili risultati per questo programma, come illustrato qui:

Risultato 1:
```js
var a = 1;
var b = 2;

// foo()
a++;
b = b * a;
a = b + 3;

// bar()
b--;
a = 8 + b;
b = a * 2;

a; // 11
b; // 22
```

Risultato 2:
```js
var a = 1;
var b = 2;

// bar()
b--;
a = 8 + b;
b = a * 2;

// foo()
a++;
b = b * a;
a = b + 3;

a; // 183
b; // 180
```

Due risultati dello stesso codice significa che abbiamo ancora il nondeterminismo! Ma è a livello di ordinamento della funzione (evento), piuttosto che al livello di ordinamento delle istruzioni (o, di fatto, il livello di ordinamento delle operazioni) come avviene con i thread. In altre parole, è *più deterministico* di quanto sarebbero stati i thread.

Come si applica al comportamento di JavaScript, questo nondeterminismo di ordinamento delle funzioni è il termine comune "condizione di razza", in quanto `foo()` e `bar()` si sfidano l'uno contro l'altro per vedere quali sono le prime. In particolare, si tratta di una "condizione di competizione" perché non è possibile prevedere in modo affidabile come si verificheranno `a` e `b`.

**Nota:** Se in JS c'era una funzione che in qualche modo non aveva un comportamento da Esecuzione-per-Completare, potremmo avere molti più risultati possibili, giusto? Si scopre che ES6 introduce proprio una cosa del genere (vedi Capitolo 4 "Generatori"), ma non preoccuparti adesso, ci torneremo in seguito!

## Concorrenza

Immaginiamo un sito che visualizza un elenco di aggiornamenti di stato (ad esempio un feed di notizie sui social network) che viene caricato progressivamente mentre l'utente scorre lungo l'elenco. Per fare in modo che tale caratteristica funzioni correttamente, (almeno) due "processi" separati dovranno essere eseguiti *simultaneamente* (cioè, durante la stessa finestra di tempo, ma non necessariamente nello stesso istante).

**Nota:** Stiamo usando un "processo" tra virgolette qui perché non sono veri processi a livello di sistema operativo nel senso scientifico del computer. Sono processi virtuali, o attività, che rappresentano una serie sequenziale di operazioni logicamente connesse. Preferiremo semplicemente "processare" su "task" perché, in termini, corrisponderà alle definizioni dei concetti che stiamo esplorando.

Il primo "processo" risponderà agli eventi `onscroll` (facendo richieste Ajax per nuovi contenuti) questi si attivano quando l'utente fa scrollare la pagina più in basso. Il secondo "processo" riceverà le risposte Ajax (per visualizzare il contenuto sulla pagina).

Ovviamente, se un utente scorre abbastanza veloce, è possibile che vengano attivati due o più eventi `onscroll` durante il tempo necessario per ottenere la prima risposta ed il processo, e quindi si avranno tutti gli eventi `onscroll` ed eventi di risposta Ajax eseguiti rapidamente, intercalati l'uno con l'altro.

La concorrenza è quando due o più "processi" sono eseguiti simultaneamente nello stesso periodo, indipendentemente dal fatto che le loro singole operazioni costituenti avvengano *in parallelo* (nello stesso istante su processori o core separati) o meno. Si può pensare alla concorrenza come a un parallelismo dei "processi" (o livelli di attività), in opposizione al parallelismo a livello di operazione (thread del processore separato).

**Nota:** La concorrenza introduce anche una nozione facoltativa su questi "processi" che interagiscono tra loro. Ci torneremo su dopo.

Per una data finestra temporale (i pochi secondi di scrolling dell'utente), avremo un "processo" indipendente raffigurato da una serie di eventi/operazioni:

"Processo" 1 (evento `onscroll`):
```
onscroll, richiesta 1
onscroll, richiesta 2
onscroll, richiesta 3
onscroll, richiesta 4
onscroll, richiesta 5
onscroll, richiesta 6
onscroll, richiesta 7
```

"Processo" 2 (eventi risposta Ajax):
```
risposta 1
risposta 2
risposta 3
risposta 4
risposta 5
risposta 6
risposta 7
```

È abbastanza probabile che un evento `onscroll` e un evento di risposta Ajax possano essere pronti per essere elaborati esattamente nello stesso *momento*. Ad esempio, visualizziamo questi eventi in una timeline:

```
onscroll, richiesta 1
onscroll, richiesta 2          risposta 1
onscroll, richiesta 3          risposta 2
risposta 3
onscroll, richiesta 4
onscroll, richiesta 5
onscroll, richiesta 6          risposta 4
onscroll, richiesta 7
risposta 6
risposta 5
risposta 7
```

Ma, tornando alla nostra nozione del ciclo degli eventi nella prima sezione del capitolo, JS sarà in grado di gestire un evento alla volta, quindi o `onscroll, richiesta 2` sta per accadere prima che `risposta 1` sta per essere eseguita, non possono accadere letteralmente nello stesso momento. Proprio come i bambini in una mensa scolastica, a prescindere dalla folla che formano fuori dalle porte, dovranno unirsi in un'unica fila per pranzare!

Visualizziamo l'interlacciamento di tutti questi eventi sulla coda del ciclo degli eventi.

Event Loop Queue:
```
onscroll, richiesta 1   <--- Processo 1 inizia
onscroll, richiesta 2
risposta 1            <--- Processo 2 inizia
onscroll, richiesta 3
risposta 2
risposta 3
onscroll, richiesta 4
onscroll, richiesta 5
onscroll, richiesta 6
risposta 4
onscroll, richiesta 7   <--- Processo 1 finito
risposta 6
risposta 5
risposta 7            <--- Processo 2 finito
```

"Processo 1" e "Processo 2" vengono eseguiti simultaneamente (a livello di attività in parallelo), ma i loro singoli eventi vengono eseguiti in sequenza nella coda del ciclo di eventi.

A proposito, nota come `response 6` e` response 5` sono tornati fuori dall'ordine previsto?

Il ciclo di eventi a thread singolo è un'espressione di concorrenza (ce ne sono sicuramente altri, qui ci torneremo più avanti).

### Noninteracting

Poiché due o più "processi" intercalano i loro passaggi/eventi contemporaneamente all'interno dello stesso programma, non necessariamente devono interagire tra loro se le attività non sono correlate. **Se non interagiscono, il non determinismo è perfettamente accettabile.**

Per esempio:

```js
var res = {};

function foo(results) {
  res.foo = results;
}

function bar(results) {
  res.bar = results;
}

// ajax(..) è una funzione Ajax fornita da una libreria
ajax( "http://some.url.1", foo );
ajax( "http://some.url.2", bar );
```

`foo ()` e `bar ()` sono due "processi" concomitanti, ed è indeterminato in quale ordine saranno eseguiti. Ma abbiamo costruito il programma in modo che non abbia importanza quale ordine vengono eseguiti, perché agiscono in modo indipendente e come tali non hanno bisogno di interagire.

Questo non è un bug "race condition", poiché il codice funzionerà sempre correttamente, indipendentemente dall'ordinamento.

### Interazione

Più comunemente, i "processi" concorrenti dovranno necessariamente interagire, indirettamente attraverso l'ambito e/o il DOM. Quando si verifica tale interazione, è necessario coordinare queste interazioni per prevenire "race conditions", come descritto in precedenza.

Ecco un semplice esempio di due "processi" concorrenti che interagiscono a causa di un ordinamento implicito, che è solo *a volte non rispettato*:

```js
var res = [];

function risposta(data) {
  res.push( data );
}

// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", risposta );
ajax( "http://some.url.2", risposta );
```

I "processi" concorrenti sono le due invocazioni a `risposta()` che saranno fatte per gestire le risposte Ajax. Possono essere eseguite entrambe per prime.

Supponiamo che il comportamento previsto sia che `res[0]` ha il risultato della chiamata `"http: //some.url.1"` e `res[1]` ha il risultato della chiamata a `"http://some.url.2"`. A volte sarà così, ma a volte saranno invertite, dipende da quale chiamata finisce prima. C'è una buona probabilità che questo nondeterminismo sia un bug "race condition".

**Nota:** Sii estremamente diffidente nei confronti delle ipotesi che potresti tendere a trovarti in queste situazioni. Ad esempio, non è raro che uno sviluppatore osservi che `"http://some.url.2"` è "sempre" molto più lento a rispondere di `"http://some.url.1"`, forse in virtù di quali attività compiono (ad esempio, uno esegue un'attività di database e l'altro recupera un file statico), quindi l'ordine sembra essere sempre quello previsto. Anche se entrambe le richieste vengono inviate allo stesso server e *questo* rispetta intenzionalmente in un determinato ordine, non esiste alcuna garanzia *reale* dell'ordine in cui le risposte arriveranno nel browser.

Quindi, per affrontare una tale condizione, è possibile gestire l'ordine dell'interazione:

```js
var res = [];

function risposta(data) {
  if (data.url == "http://some.url.1") {
    res[0] = data;
  }
  else if (data.url == "http://some.url.2") {
    res[1] = data;
  }
}

// ajax(..) è una funzione Ajax fornita da una libreria
ajax( "http://some.url.1", risposta );
ajax( "http://some.url.2", risposta );
```
Indipendentemente dalla prima risposta Ajax, esaminiamo `data.url` (supponendo che uno sia restituito dal server, ovviamente!) per capire quale posizione occuperanno i dati di risposta nell'array `res`. `res[0]` manterrà sempre il risultato `"http://some.url.1"` e `res[1]` manterrà sempre il risultato di `"http://some.url.2"`. Attraverso una semplice coordinazione, abbiamo eliminato il non determinismo della "race condition".

Lo stesso ragionamento di questo scenario si applicherebbe se più chiamate simultanee di funzioni interagissero l'una con l'altra attraverso il DOM condiviso, come uno che aggiorna il contenuto di un `<div>` e l'altro che aggiorna lo stile o gli attributi di `<div>` (ad esempio, per rendere visibile l'elemento DOM una volta che possiede un contenuto). Probabilmente non vorresti mostrare l'elemento DOM prima che abbia un contenuto, quindi il coordinamento deve garantire un corretto ordine.

Alcuni scenari di concorrenza sono *sempre interrotti* (non solo *a volte*) senza interazione coordinata. Consideriamo:

```js
var a, b;

function foo(x) {
  a = x * 2;
  baz();
}

function bar(y) {
  b = y * 2;
  baz();
}

function baz() {
  console.log(a + b);
}

// ajax(..) è una funzione Ajax fornita da una libreria
ajax( "http://some.url.1", foo );
ajax( "http://some.url.2", bar );
```
In questo esempio, se `foo()` o `bar()` viene invocata per prima, invocherà sempre `baz()` troppo presto (ed `a` o `b` sarà ancora `undefined`), ma la seconda invocazione di `baz()` funzionerà, dato che sia `a` che `b` saranno disponibili.

Ci sono diversi modi per affrontare una tale condizione. Ecco un modo semplice:

```js
var a, b;

function foo(x) {
  a = x * 2;
  if (a && b) {
    baz();
  }
}

function bar(y) {
  b = y * 2;
  if (a && b) {
    baz();
  }
}

function baz() {
  console.log( a + b );
}

// ajax(..) è una funzione Ajax fornita da una libreria
ajax( "http://some.url.1", foo );
ajax( "http://some.url.2", bar );
```


La condizione `if (a && b)` che racchiude l'invocazione di `baz()` è tradizionalmente chiamato "gate", perché non siamo sicuri in quale ordine `a` e `b` arriveranno, ma aspettiamo entrambi per arrivarci prima di procedere ad aprire il gate (invocazione di `baz()`).

Un'altra condizione di interazione concorrente che potresti incontrare è a volte chiamata "gara", ma più correttamente chiamata "latch". La cartteristica principale è un comportamento dove "solo il primo vince". Qui, il nondeterminismo è accettabile, in quanto stai dicendo esplicitamente che al traguardo della "gara" ci sarà un solo vincitore.

Consideriamo il seguente codice:

```js
var a;

function foo(x) {
  a = x * 2;
  baz();
}

function bar(x) {
  a = x / 2;
  baz();
}

function baz() {
  console.log( a );
}

// ajax(..) è una funzione Ajax fornita da una libreria
ajax( "http://some.url.1", foo );
ajax( "http://some.url.2", bar );
```

Qualunque evento (`foo()` o `bar()`) invocato per ultimo non solo sovrascriverà il valore assegnato ad `a` dall'altro, ma duplicherà anche la chiamata a `baz()` (probabilmente un comportamento indesiderato).

Quindi, possiamo coordinare l'interazione con un semplice latch, e lasciar passare solo il primo:

```js
var a;

function foo(x) {
  if (a == undefined) {
    a = x * 2;
    baz();
  }
}

function bar(x) {
  if (a == undefined) {
    a = x / 2;
    baz();
  }
}

function baz() {
  console.log( a );
}

// ajax(..) è una funzione Ajax fornita da una libreria
ajax( "http://some.url.1", foo );
ajax( "http://some.url.2", bar );
```
La condizione `if (a == undefined)` lascia passare solo primo tra `foo()` o `bar()`, mentre la seconda (in effetti tutte le chiamate successive alla prima) verranno semplicemente ignorate. Non c'è solo virtù nell'essere al secondo posto!

**Nota:** In tutti questi scenari, abbiamo utilizzato variabili globali per scopi illustrativi semplicistici, ma non c'è nulla nel nostro ragionamento qui che lo richiede. Finché le funzioni in questione possono accedere alle variabili (tramite scope), funzioneranno come previsto. Affidarsi a variabili lessicali (date uno sguardo al titolo *Scope & Closures* in questa serie di libri), ed in effetti le variabili globali utilizzate in questi esempi, sono uno svantaggio evidente in questo tipo di gestione della concorrenza. Mentre analizzeremo i prossimi capitoli, vedremo altri modi di gestione della concorrenza che sono molto più puliti.

### Cooperazione

Un'altra tipologia di gestione della concorrenza è chiamata "concorrenza cooperativa". Qui, l'attenzione non cala sull'interazione tramite la condivisione del valore negli ambiti (anche se questo è ovviamente ancora permesso!). L'obiettivo è di eseguire un "processo" di lunga durata e suddividerlo in passaggi o gruppi in modo che altri "processi" concorrenti abbiano la possibilità di agganciare le loro operazioni nella coda del ciclo di eventi.

Ad esempio, si consideri un gestore di risposta Ajax che deve eseguire un lungo elenco di risultati per trasformare i valori. Useremo `Array#map(..)` per mantenere il codice breve:

```js
var res = [];

// `risposta(..)` riceve una serie di risultati dalla chiamata Ajax
function risposta(data) {
  // aggiungere all'array `res` esistente
  res = res.concat(
    // crea una nuovo array trasformato con tutti i valori di `data` raddoppiati
    data.map( function(val){
      return val * 2;
    } )
  );
}

// ajax(..) è una funzione Ajax fornita da una libreria
ajax( "http://some.url.1", risposta );
ajax( "http://some.url.2", risposta );
```

Se `"http://some.url.1"` recupera prima i risultati, l'intero elenco verrà mappato in `res` tutto in una volta. Se si tratta di poche migliaia o poco meno di registrazioni, questo non è generalmente un grosso problema. Ma se si tratta di 10 milioni di record, ci vorrà un po 'di tempo per eseguire (alcuni secondi su un laptop potente, molto più a lungo su un dispositivo mobile, ecc.).

Mentre un tale "processo" è in esecuzione, non può accadere nient'altro nella pagina, comprese altre chiamate `risposta(..)`, nessun aggiornamento dell'interfaccia utente, nemmeno gli eventi dell'utente come lo scorrimento, la digitazione, il click del pulsante e cose simili. È piuttosto doloroso.

Quindi, per creare un sistema concorrente più cooperativo, uno più amichevole e che non impalla la coda del ciclo di eventi, è possibile elaborare questi risultati in batch in modo asincrono, dopo che ognuno "ritorna" al ciclo degli eventi per eseguire altri eventi in attesa.

Ecco un approccio molto semplice:

```js
var res = [];

// `risposta(..)` riceve una serie di risultati dalla chiamata Ajax
function risposta(data) {
  // let's just do 1000 at a time
  var chunk = data.splice( 0, 1000 );

  // aggiungere all'array `res` esistente
  res = res.concat(
    // crea una nuovo array trasformato con tutti i valori di `data` raddoppiati
    chunk.map( function(val){
      return val * 2;
    } )
  );

  // anything left to process?
  if (data.length > 0) {
    // async schedule next batch
    setTimeout( function(){
      response( data );
    }, 0 );
  }
}

// ajax(..) è una funzione Ajax fornita da una libreria
ajax( "http://some.url.1", response );
ajax( "http://some.url.2", response );
```

Elaboriamo il set di dati in blocchi di dimensioni massime di 1.000 articoli. In tal modo, garantiamo un "processo" di breve periodo, anche se ciò significa molti più "processi" in successione, poiché l'interleaving sulla coda del ciclo di eventi ci darà un sito/app molto più reattivo (performante).

Naturalmente, non stiamo coordinando le interazioni con l'ordinamento di nessuno di questi "processi", quindi l'ordine dei risultati in `res` non sarà prevedibile. Se fosse richiesto l'ordine, avresti bisogno di usare tecniche di interazione come quelle che abbiamo discusso in precedenza, o di quelle di cui parleremo nei prossimi capitoli di questo libro.

Usiamo un `setTimeout(.. 0)` (hack) per la schedulazione asincrona, che in pratica significa semplicemente "agganciare questa funzione alla fine della coda del ciclo di eventi corrente."

**Nota:** `setTimeout (.. 0)` non sta tecnicamente inserendo un elemento direttamente nella coda del ciclo di eventi. Il timer inserirà l'evento alla sua prossima opportunità. Ad esempio, due chiamate `setTimeout(.. 0)` successive non sarebbero strettamente garantite per essere elaborate in ordine di chiamata, quindi *è* possibile che si abbiano varie condizioni come un cumulo di timer il cui ordinamento non è prevedibile. In Node.js, un approccio simile è `process.nextTick(..)`. Nonostante quanto sia conveniente (e di solito più performante), non esiste un unico modo diretto (almeno finora) in tutti gli ambienti per garantire un ordinamento asincrono degli eventi. Tratteremo questo argomento in modo più dettagliato nella prossima sezione.

## Jobs

A partire da ES6, c'è un nuovo concetto sovrapposto alla coda del ciclo di eventi, chiamato "Jobs queue". L'esperienza più probabile che avrai sarà con il comportamento asincrono delle Promises (vedi Capitolo 3).

Sfortunatamente, al momento è un meccanismo senza un'API esposta, e quindi dimostrarlo è un po' più complicato. Quindi dovremo semplicemente descriverlo concettualmente, in modo tale che quando discuteremo del comportamento asincrono con Promises nel Capitolo 3, capirai in che modo tali azioni vengono pianificate ed elaborate.

Quindi, il modo migliore per pensare a ciò che ho trovato è che la "Jobs queue" è una coda che pende dalla fine di ogni tick nella coda del ciclo di eventi. Alcune azioni implicite da asinc che possono verificarsi durante un segno di spunta del ciclo di eventi non causeranno l'aggiunta di un nuovo evento alla coda del ciclo di eventi, ma aggiungeranno un elemento (noto anche come Job) alla fine della coda di lavoro del tick corrente. .

È come dire "oh, ecco quest'altra cosa che devo fare *dopo*, ma assicurati che accada subito prima che possa accadere qualsiasi altra cosa".

Oppure, per usare una metafora: la coda del ciclo degli eventi è come un giro in un parco dei divertimenti, dove una volta finito il giro, devi andare dietro linea per rifarlo di nuovo. Ma la Jobs queue è come finire il viaggio, ma poi tagliare in linea e riprendere.

Un Job può anche creare più Jobs da aggiungere alla fine della stessa coda. Quindi, è teoricamente possibile che un "ciclo" di Job (un Job che continua ad aggiungere un altro Job, ecc.) possa girare indefinitamente, quindi affamando il programma della possibilità di passare al prossimo tick del ciclo di eventi. Questo sarebbe concettualmente quasi lo stesso di un semplice ciclo infinito o in esecuzione (come `while (true)..`) nel codice.

I Jobs sono un po' come lo spirito dell'hack `setTimeout(.. 0)`, ma implementato in modo tale da avere un ordinamento molto più definito e garantito: **più tardi, ma il prima possibile**.

Immaginiamo un'API per schedulare i Job (direttamente, senza nessun hack), e chiamiamola `schedula(..)`. Consideriamo:

```js
console.log( "A" );

setTimeout( function(){
  console.log( "B" );
}, 0 );

// theoretical "Job API"
schedula( function(){
  console.log( "C" );

  schedule( function(){
    console.log( "D" );
  } );
} );
```

Ci si potrebbe aspettare che stampi `A B C D`, ma invece stampa `A C D B`, perché i Job vengono eseguiti alla fine del tick corrente del ciclo di eventi, e il timer si attiva per pianificarlo al *prossimo* tick del ciclo degli eventi (se disponibile!).

Nel Capitolo 3 vedremo che il comportamento asincrono di Promises si basa su Jobs, quindi è importante aver ben chiaro come ciò è collegato al comportamento del ciclo di eventi.

## Ordinamento delle dichiarazioni

L'ordine in cui scriviamo le righe di codice del nostro programma non è necessariamente lo stesso ordine in cui il motore JS le eseguirà. Potrebbe sembrare una strana dichiarazione da fare, quindi lo esploreremo solo brevemente.

Ma prima di farlo, dovremmo essere chiari su qualcosa: le regole/grammatica del linguaggio (vedi il *Tipi e grammatica* titolo di questa serie di libri) dettano un comportamento molto prevedibile e affidabile per l'ordinamento delle dichiarazioni dal punto di vista del programma. Quindi ciò di cui stiamo discutendo sono **cose che non dovresti mai essere in grado di osservare** nel tuo programma JS.

**Attenzione:** Se sei mai in grado di *osservare* la compilazione di istruzioni del compilatore come stiamo per illustrare, sarebbe una chiara violazione delle specifiche, e sarebbe indubbiamente dovuta a un bug nel motore JS in questione -- un bug che dovrebbe essere prontamente segnalato e risolto! Ma è molto più comune che tu *sospetti* che qualcosa di pazzesco stia accadendo nel motore JS, quando in realtà è solo un bug (probabilmente una "race condition"!) Nel tuo codice - quindi guarda prima lì, e ancora e ancora . Il debugger JS, usando i breakpoint e scorrendo il codice riga per riga, sarà il tuo strumento più potente per individuare questi bug *nel tuo codice*.

Consideriamo:

```js
var a, b;

a = 10;
b = 30;

a = a + 1;
b = b + 1;

console.log( a + b ); // 42
```

Questo codice non ha alcuna asincronia espressa (tranne il raro I/O asincrono della `console` discusso in precedenza!), Quindi l'ipotesi più probabile è che viene elaborato riga per riga dall'alto verso il basso.

Ma è *possibile* che il motore JS, dopo aver compilato questo codice (sì, JS si compila - vedi il titolo *Scope & Closures* di questa serie di libri!) potrebbe avere l'opportunità di eseguire il tuo codice più velocemente riorganizzando (in modo sicuro) l'ordine delle righe di codice. In sostanza, finché non è possibile osservare il riordinamento, è tutto leale.

Ad esempio, il motore potrebbe scoprire che è più veloce eseguire il codice in questo modo:

```js
var a, b;

a = 10;
a++;

b = 30;
b++;

console.log( a + b ); // 42
```

O in quest'altro modo:

```js
var a, b;

a = 11;
b = 31;

console.log( a + b ); // 42
```

o anche:

```js
// siccome `a` e `b` non vengono più usati, possiamo
// possiamo loggare inline e non ne abbiamo più bisogno!
console.log( 42 ); // 42
```

In tutti questi casi, il motore JS sta eseguendo ottimizzazioni sicure durante la sua compilazione, poiché il risultato *osservabile* alla fine sarà lo stesso.

Ma ecco uno scenario in cui queste ottimizzazioni specifiche non sarebbero sicure e quindi non dovrebbero essere consentite (ovviamente, per non dire che non è affatto ottimizzato):

```js
var a, b;

a = 10;
b = 30;

// necessitiamo di `a` e `b` nel loro stato pre-incrementato!
console.log( a * b ); // 300

a = a + 1;
b = b + 1;

console.log( a + b ); // 42
```
Altri esempi in cui il riordino del compilatore potrebbe creare effetti collaterali osservabili (e quindi deve essere vietato) include cose come qualsiasi chiamata a funzione con effetti collaterali (anche e soprattutto funzioni getter), o oggetti Proxy ES6 (vedere il *ES6 e oltre* di questa serie di libri).

Consideriamo:

```js
function foo() {
  console.log( b );
  return 1;
}

var a, b, c;

// ES5.1 sintassi getter
c = {
  get bar() {
    console.log( a );
    return 1;
  }
};

a = 10;
b = 30;

a += foo(); // 30
b += c.bar; // 11

console.log( a + b );	// 42
```

Se non fosse per le istruzioni `console.log(..)` in questo frammento (usato solo come uno strumento comodo per illustrare un effetto collaterale osservabile), il motore JS probabilmente sarebbe stato libero, e se lo avesse voluto (chissà se lo sarebbe!?), per riordinare il codice in:

```js
// ...

a = 10 + foo();
b = 30 + c.bar;

// ...
```

Mentre la semantica JS per fortuna ci protegge dagli incubi *osservabili* che il riordino delle istruzioni del compilatore sembra essere pericoloso, è comunque importante capire quanto sia sottile il collegamento tra il modo in cui il codice sorgente è stato creato (in modo top-down) e il modo in cui viene eseguito dopo la compilazione.

Il riordino delle istruzioni del compilatore è quasi una micro-metafora della concorrenza e dell'interazione. Come concetto generale, tale consapevolezza può aiutarti a capire meglio i problemi relativi al flusso del codice JS asincrono.

## Revisioni

Un programma JavaScript è (praticamente) sempre suddiviso in due o più blocchi, dove il primo blocco viene eseguito *ora* ed il blocco successivo viene eseguito *più tardi*, come risposta a un evento. Anche se il programma viene eseguito blocco-per-blocco, tutti condividono lo stesso accesso all'ambito e allo stato del programma, quindi ogni modifica allo stato viene eseguita in base allo stato precedente.

Ogni volta che ci sono eventi da eseguire, il *ciclo degli eventi* viene eseguito finché la coda non è vuota. Ogni iterazione del ciclo degli eventi è una "spunta". L'interazione dell'utente, l'I/O ed i timer accodano gli eventi nella coda degli eventi.

In qualsiasi momento, solo un evento alla volta può essere elaborato dalla coda. Mentre un evento è in esecuzione, può generare direttamente o indirettamente uno o più eventi successivi.

La concorrenza è quando due o più catene di eventi si intrecciano nel tempo, in modo tale che da una prospettiva di alto livello, sembrano essere in esecuzione *simultaneamente* (anche se in un dato momento viene elaborato solo un evento).

Spesso è necessario eseguire una qualche forma di coordinamento dell'interazione tra questi "processi" concomitanti (distinti dai processi del sistema operativo), ad esempio per garantire l'ordine o per prevenire "condizioni di competizione". Questi "processi" possono anche *cooperare* dividendosi in blocchi più piccoli e consentendo altri interleaving "di processo".