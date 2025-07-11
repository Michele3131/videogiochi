# 3. Data Transfer Objects (DTO) e Pattern di Mapping

## 3.1 La Filosofia dei Data Transfer Objects

### 3.1.1 Essenza Concettuale del Pattern DTO

Il **Data Transfer Object (DTO)** rappresenta una delle soluzioni più eleganti al problema della **separazione delle responsabilità** nell'architettura software. Non si tratta semplicemente di "oggetti per trasferire dati", ma di una filosofia di design che riconosce la **natura intrinsecamente diversa** tra la rappresentazione interna dei dati (entità di dominio) e la loro rappresentazione esterna (interfacce API).

**Il Principio di Isolamento Semantico**
I DTO incarnano il principio che *ciò che è ottimale per la persistenza non è necessariamente ottimale per la comunicazione*. Mentre le entità di dominio sono progettate per riflettere la logica di business e le relazioni complesse, i DTO sono progettati per l'efficienza della trasmissione e la chiarezza dell'interfaccia.

### 3.1.2 Il Problema dell'Accoppiamento Strutturale

**La Trappola dell'Esposizione Diretta**
Quando esponiamo direttamente le entità di dominio attraverso le API, creiamo un **accoppiamento strutturale** pericoloso. Questo significa che ogni modifica al modello dati interno si riflette automaticamente sull'interfaccia pubblica, violando il principio di **stabilità dell'interfaccia**.

### 3.1.3 Le Dimensioni del Problema: Oltre la Semplice Trasmissione

**1. Il Problema della Contaminazione Semantica**
L'esposizione diretta delle entità contamina il dominio con preoccupazioni di presentazione. Un'entità `Game` progettata per la persistenza potrebbe contenere relazioni lazy, identificatori tecnici, o strutture ottimizzate per le query, che non hanno senso nel contesto di un'API REST.

**2. Il Problema della Rigidità Evolutiva**
Quando l'API è direttamente accoppiata al modello dati, ogni evoluzione del dominio richiede una valutazione dell'impatto sull'interfaccia pubblica. Questo crea una **resistenza al cambiamento** che rallenta l'evoluzione del software.

**3. Il Problema della Performance Semantica**
Le entità sono progettate per la correttezza relazionale, non per l'efficienza della trasmissione. Un DTO può **denormalizzare** i dati, pre-calcolare aggregazioni, e ottimizzare la struttura per il caso d'uso specifico.

**Esempio Concettuale: L'Entità vs il DTO**
```java
// Entità: Ottimizzata per la persistenza e l'integrità
@Entity
public class Game {
    private Long id;  // Chiave tecnica
    private String title;
    
    @ManyToOne(fetch = FetchType.LAZY)
    private Developer developer;  // Relazione lazy per performance
    
    // Logica di dominio
    public boolean isEligibleForDiscount() { /* ... */ }
}

// DTO: Ottimizzato per la comunicazione
public class GameSummaryDTO {
    private String title;
    private String developerName;  // Denormalizzato per efficienza
    private boolean discountEligible;  // Pre-calcolato
}
```
## 3.2 Tassonomia dei DTO: Specializzazione per Contesto

### 3.2.1 La Natura Polimorfica dei DTO

I DTO non sono un concetto monolitico, ma si specializzano in base al **contesto d'uso** e alla **direzione del flusso dati**. Questa specializzazione riflette il principio che *diversi casi d'uso richiedono diverse rappresentazioni degli stessi dati*.

**Response DTO: L'Arte della Sintesi**
I Response DTO incarnano l'arte di **sintetizzare** informazioni complesse in forme consumabili. Non si limitano a "restituire dati", ma curano l'esperienza del consumatore dell'API, pre-elaborando, aggregando e ottimizzando le informazioni.

**Request DTO: La Disciplina della Validazione**
I Request DTO rappresentano la **prima linea di difesa** contro dati inconsistenti. Incorporano non solo la struttura dei dati attesi, ma anche le **regole di validazione** che garantiscono l'integrità semantica dell'input.

### 3.1.3 Tipi di DTO

#### Response DTO (Output)

I Response DTO seguono il **principio della granularità semantica**: ogni DTO è progettato per un contesto d'uso specifico, ottimizzando sia le performance che l'esperienza utente.

**DTO per Liste (Summary)**: Contengono solo le informazioni essenziali per la visualizzazione in elenchi. Seguono il principio del **minimo carico cognitivo**, presentando solo i dati necessari per permettere all'utente di identificare e confrontare rapidamente gli elementi.

**DTO per Dettagli (Detail)**: Rappresentano la **vista completa** di un'entità, includendo tutte le informazioni necessarie per una visualizzazione dettagliata. Incorporano anche dati derivati e aggregati che sarebbero costosi da calcolare in tempo reale.

**DTO per Statistiche (Stats)**: Specializzati nella presentazione di **dati aggregati** e **metriche derivate**. Questi DTO incapsulano logiche di business complesse, trasformando dati grezzi in informazioni significative per l'analisi e il reporting.

#### Request DTO (Input)

I Request DTO implementano il **principio della validazione anticipata** e della **sicurezza per progettazione**. Ogni DTO di input rappresenta un **contratto semantico** che definisce non solo la struttura dei dati, ma anche le regole di business che li governano.

**DTO per Creazione (Create)**: Definiscono tutti i campi obbligatori per la creazione di una nuova entità. Implementano **validazioni stringenti** che riflettono le regole di business del dominio. Ogni annotazione di validazione non è solo un controllo tecnico, ma l'espressione di un **invariante di dominio**.

**DTO per Aggiornamento (Update)**: Seguono il **principio della modifica parziale**, dove tutti i campi sono opzionali ma quelli presenti devono rispettare le stesse regole di validazione della creazione. Questo approccio permette aggiornamenti granulari senza compromettere l'integrità dei dati.

**DTO per Ricerca (Search/Filter)**: Incapsulano la **logica di query** in oggetti tipizzati, trasformando parametri di ricerca potenzialmente complessi in strutture dati comprensibili e validabili. Permettono di costruire query dinamiche mantenendo la type-safety.

### 3.1.4 DTO per Paginazione

La paginazione rappresenta un **pattern architetturale fondamentale** per la gestione di grandi dataset. I DTO di paginazione incapsulano non solo i parametri tecnici (page, size), ma anche la **semantica della navigazione** e dell'ordinamento.

**Principi di Progettazione**:
- **Limitazione delle Risorse**: Prevenzione di query che potrebbero sovraccaricare il sistema
- **Esperienza Utente Consistente**: Standardizzazione dei parametri di navigazione
- **Sicurezza**: Validazione dei parametri per prevenire attacchi di tipo DoS
- **Performance**: Ottimizzazione delle query attraverso parametri predefiniti

I DTO di paginazione trasformano parametri HTTP grezzi in oggetti di dominio tipizzati, permettendo una gestione uniforme della paginazione in tutto il sistema.
La **risposta paginata** incapsula non solo i dati richiesti, ma anche i **metadati di navigazione** necessari per costruire interfacce utente intuitive. Questo approccio elimina la necessità di query aggiuntive per ottenere informazioni sulla struttura del dataset.
```

## 3.2 Pattern di Mapping

### 3.2.1 Mapping Manuale

Il mapping manuale rappresenta l'approccio più **esplicito e controllabile** per la trasformazione tra entità e DTO. Questo pattern offre il massimo controllo sulla logica di trasformazione, permettendo di implementare regole di business complesse direttamente nel processo di mapping.

**Vantaggi Filosofici**:
- **Trasparenza Semantica**: Ogni trasformazione è esplicita e comprensibile
- **Controllo Granulare**: Possibilità di implementare logiche di business specifiche
- **Debugging Facilitato**: Ogni passaggio è tracciabile e modificabile
- **Flessibilità Massima**: Nessuna limitazione imposta da framework esterni

**Svantaggi Architetturali**:
- **Verbosità**: Richiede codice boilerplate significativo
- **Manutenzione**: Ogni modifica alle entità richiede aggiornamenti manuali
- **Rischio di Inconsistenza**: Possibili errori umani nella sincronizzazione

Il mapping manuale è ideale quando la **logica di trasformazione** è complessa o quando si necessita di **performance ottimizzate** attraverso controllo diretto del processo.
```

### 3.2.2 MapStruct - Mapping Automatico

MapStruct rappresenta l'evoluzione del mapping attraverso **generazione di codice compile-time**. Questo approccio combina la semplicità dichiarativa con le performance del codice generato, eliminando il runtime overhead tipico della reflection.

**Filosofia di MapStruct**:
- **Convenzione su Configurazione**: Mapping automatico per campi con nomi corrispondenti
- **Sicurezza Compile-Time**: Errori di mapping rilevati durante la compilazione
- **Performance Ottimizzate**: Codice generato equivalente al mapping manuale
- **Estensibilità**: Possibilità di personalizzare la logica attraverso metodi custom

**Principi Architetturali**:
- **Separazione delle Responsabilità**: Il mapper si occupa solo della trasformazione
- **Immutabilità**: Supporto nativo per oggetti immutabili
- **Composizione**: Riutilizzo di mapper attraverso dependency injection
- **Null Safety**: Gestione automatica dei valori null

MapStruct è ideale per applicazioni dove la **produttività** e la **manutenibilità** sono prioritarie, specialmente in presenza di modelli di dati complessi con molte relazioni.

#### Mapping Complesso con Dipendenze

Il **mapping compositivo** rappresenta uno dei pattern più potenti di MapStruct. Attraverso il meccanismo `uses`, è possibile creare una **rete di mapper interdipendenti** che gestiscono automaticamente la trasformazione di oggetti complessi e delle loro relazioni.

**Principi del Mapping Compositivo**:
- **Separazione delle Responsabilità**: Ogni mapper gestisce un singolo tipo di entità
- **Riutilizzo**: I mapper possono essere composti per creare trasformazioni complesse
- **Dependency Injection**: Integrazione automatica con il container Spring
- **Lazy Loading**: Gestione intelligente delle relazioni per evitare N+1 queries

Questo approccio permette di costruire **architetture di mapping scalabili** dove l'aggiunta di nuove entità non richiede modifiche ai mapper esistenti.
```

### 3.2.3 Mapping Condizionale e Personalizzato

#### Mapping Basato su Contesto

Il **mapping contestuale** rappresenta l'evoluzione del pattern DTO verso la **personalizzazione semantica**. Questo approccio riconosce che la stessa entità può avere rappresentazioni diverse a seconda del **contesto d'uso** e dell'**identità dell'osservatore**.

**Principi del Mapping Contestuale**:
- **Personalizzazione**: Ogni utente riceve una vista personalizzata dei dati
- **Sicurezza**: Informazioni sensibili vengono filtrate in base ai permessi
- **Performance**: Solo i dati necessari vengono calcolati e trasferiti
- **Esperienza Utente**: L'interfaccia si adatta al ruolo e alle preferenze dell'utente

**Architettura Multi-Vista**:
- **Vista Pubblica**: Informazioni base accessibili a tutti
- **Vista Utente**: Dati personalizzati per utenti autenticati
- **Vista Amministratore**: Informazioni complete per la gestione

Questo pattern elimina la necessità di query multiple per ottenere informazioni contestuali, incorporando la logica di personalizzazione direttamente nel processo di mapping.
```

#### Mapping con Validazione

Il **mapping validativo** integra la logica di trasformazione con la **validazione semantica** e la **sanitizzazione dei dati**. Questo approccio implementa il principio della **sicurezza per progettazione**, garantendo che i dati siano sempre in uno stato consistente e sicuro.

**Principi della Validazione Integrata**:
- **Fail-Fast**: Errori rilevati immediatamente durante la trasformazione
- **Sanitizzazione**: Normalizzazione automatica dei dati in input
- **Invarianti di Dominio**: Applicazione delle regole di business durante il mapping
- **Tracciabilità**: Ogni errore è associato al campo specifico che lo ha generato

**Vantaggi Architetturali**:
- **Centralizzazione**: Logica di validazione concentrata nei mapper
- **Riutilizzo**: Regole di validazione condivise tra diversi contesti
- **Performance**: Validazione e trasformazione in un singolo passaggio
- **Manutenibilità**: Modifiche alle regole di business localizzate

Questo pattern è particolarmente efficace per **API pubbliche** dove la qualità e la sicurezza dei dati sono critiche.
```

## 3.3 Strategie di Mapping Avanzate

### 3.3.1 Mapping Lazy e Performance

#### Projection per Performance

Le **projection** rappresentano una strategia fondamentale per ottimizzare le performance quando si lavora con DTO. Questo pattern permette di **selezionare solo i campi necessari** direttamente a livello di database, eliminando il trasferimento di dati non utilizzati.

**Tipi di Projection**:
- **Interface-based**: Definiscono un contratto per i dati richiesti
- **Class-based**: Utilizzano costruttori per la materializzazione diretta
- **Dynamic**: Permettono selezioni di campi configurabili a runtime

**Vantaggi delle Projection**:
- **Riduzione del Traffico di Rete**: Solo i dati necessari vengono trasferiti
- **Ottimizzazione della Memoria**: Minore utilizzo di heap per oggetti più piccoli
- **Performance del Database**: Query più efficienti con SELECT specifiche
- **Scalabilità**: Migliore gestione di grandi volumi di dati
```

#### Mapping con EntityGraph

L'**EntityGraph** rappresenta una strategia avanzata per il controllo del **lazy loading** in JPA. Questo pattern permette di definire esattamente quali relazioni devono essere caricate eagerly, ottimizzando le performance e prevenendo il problema N+1.

**Principi dell'EntityGraph**:
- **Controllo Granulare**: Definizione precisa delle relazioni da caricare
- **Prevenzione N+1**: Eliminazione di query multiple attraverso JOIN
- **Flessibilità**: Diversi grafi per diversi casi d'uso
- **Performance Predictable**: Comportamento di caricamento deterministico

**Strategie di Applicazione**:
- **Fetch Graphs**: Caricano solo le relazioni specificate
- **Load Graphs**: Caricano le relazioni specificate più quelle eager di default
- **Dynamic Graphs**: Costruzione programmatica per casi complessi
```

### 3.3.2 Mapping Asincrono

#### Mapping con CompletableFuture

Il **mapping asincrono** diventa essenziale quando la trasformazione DTO richiede operazioni costose come chiamate a servizi esterni, calcoli complessi o arricchimenti di dati. Questo pattern permette di **parallelizzare** le operazioni di mapping, migliorando significativamente le performance.

**Scenari di Applicazione**:
- **Arricchimento Dati**: Integrazione con servizi esterni
- **Calcoli Complessi**: Elaborazioni che richiedono tempo significativo
- **Batch Processing**: Trasformazione di grandi volumi di dati
- **Aggregazioni**: Combinazione di dati da multiple sorgenti

**Vantaggi del Mapping Asincrono**:
- **Throughput Migliorato**: Parallelizzazione delle operazioni
- **Responsività**: Non blocco del thread principale
- **Scalabilità**: Migliore utilizzo delle risorse di sistema
- **Resilienza**: Gestione degli errori per singole operazioni
```

### 3.3.3 Caching dei Mapping

#### Cache per DTO Costosi

Il **caching dei DTO** rappresenta una strategia cruciale per ottimizzare le performance quando la trasformazione o il calcolo dei dati è costoso. Questo pattern è particolarmente efficace per **dati aggregati**, **statistiche** e **viste complesse** che cambiano raramente.

**Strategie di Caching**:
- **Entity-Level**: Cache delle entità prima del mapping
- **DTO-Level**: Cache del risultato finale della trasformazione
- **Partial**: Cache di componenti specifici del DTO
- **Computed**: Cache di valori calcolati e aggregazioni

**Considerazioni Architetturali**:
- **Invalidazione**: Strategie per mantenere la coerenza dei dati
- **TTL (Time To Live)**: Gestione dell'obsolescenza automatica
- **Warming**: Pre-caricamento di cache per dati critici
- **Eviction**: Politiche di rimozione per gestire la memoria
```

## 3.4 Best Practices

### 3.4.1 Principi Generali

**1. Separazione delle Responsabilità**
- **DTO**: Esclusivamente per il trasferimento e la rappresentazione dei dati
- **Entity**: Gestione della persistenza e delle relazioni di dominio
- **Mapper**: Logica di trasformazione isolata e testabile

**2. Immutabilità dei DTO**
I DTO dovrebbero essere **immutabili per progettazione**, utilizzando il pattern Builder e campi final. Questo garantisce thread-safety e previene modifiche accidentali durante il trasferimento.

**3. Validazione Stratificata**
- **Sintassi**: Validazione della struttura e dei tipi
- **Semantica**: Validazione delle regole di business
- **Contesto**: Validazione basata sullo stato dell'applicazione

**4. Naming Convention Semantico**
I nomi dei DTO devono riflettere il loro **scopo semantico** (es. `GameSummaryDTO`, `GameDetailDTO`) piuttosto che la loro struttura tecnica.

**5. Versionamento dei DTO**
Implementare strategie di versionamento per gestire l'evoluzione delle API senza rompere la compatibilità con i client esistenti.
// DTO immutabile
@Value
@Builder
public class GameDTO {
    Long id;
    String title;
    String description;
    BigDecimal price;
    LocalDate releaseDate;
    
    // Tutti i campi sono final e immutabili
}
```

3. **Validazione nei Request DTO**
```java
@Data
@Builder
public class GameCreateRequest {
    
    @NotBlank
    @Size(min = 1, max = 255)
    private String title;
    
    @NotNull
    @DecimalMin("0.01")
    @DecimalMax("999.99")
    private BigDecimal price;
    
    @NotNull
    @PastOrPresent
    private LocalDate releaseDate;
}
```

4. **Naming Convention Consistente**
   - `*DTO` per response
   - `*Request` per input
   - `*Response` per wrapper di risposta
   - `*Criteria` per filtri di ricerca

5. **Gestione dei Null Values**
```java
@Mapper(
    componentModel = "spring",
    nullValuePropertyMappingStrategy = NullValuePropertyMappingStrategy.IGNORE,
    nullValueCheckStrategy = NullValueCheckStrategy.ALWAYS
)
public interface GameMapper {
    
    @Mapping(target = "developerName", 
             source = "developer.name", 
             defaultValue = "Sconosciuto")
    GameDTO toDTO(Game game);
}
```

6. **Versionamento dei DTO**
```java
// Versione 1
package com.videogames.dto.v1;

@Data
public class GameDTO {
    private Long id;
    private String title;
    private BigDecimal price;
}

// Versione 2 - Aggiunta di nuovi campi
package com.videogames.dto.v2;

@Data
public class GameDTO {
    private Long id;
    private String title;
    private BigDecimal price;
    private String description; // Nuovo campo
    private LocalDate releaseDate; // Nuovo campo
}
```

I DTO e i pattern di mapping sono essenziali per creare API robuste e manutenibili, fornendo un livello di astrazione tra il modello di dominio interno e l'interfaccia esterna dell'applicazione.