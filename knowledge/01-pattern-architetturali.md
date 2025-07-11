# 1. Pattern Architetturali Fondamentali

## 1.1 Model-View-Controller (MVC)

### 1.1.1 Origini e Filosofia del Pattern

Il pattern **Model-View-Controller (MVC)** nasce negli anni '70 presso Xerox PARC come soluzione al problema della crescente complessità delle interfacce utente. La sua filosofia fondamentale si basa sul principio della **separazione delle responsabilità** (Separation of Concerns), che sostiene che ogni componente software dovrebbe avere una singola, ben definita responsabilità.

La genialità di MVC risiede nel riconoscere che in ogni applicazione interattiva esistono tre aspetti fondamentalmente diversi:
- **I dati e le regole che li governano** (Model)
- **La rappresentazione visuale di questi dati** (View) 
- **La logica che coordina le interazioni** (Controller)

Questi tre aspetti hanno cicli di vita, requisiti di cambiamento e responsabilità completamente diverse. Separarli permette di evolverli indipendentemente, riducendo la complessità e aumentando la manutenibilità.

### 1.1.2 Anatomia Concettuale dei Componenti

#### Il Model: Custode della Verità

Il **Model** rappresenta il cuore concettuale dell'applicazione. Non è semplicemente un contenitore di dati, ma l'incarnazione delle regole di business e della logica di dominio. Il Model dovrebbe essere completamente indipendente dalla presentazione e dall'interfaccia utente.

**Responsabilità concettuali del Model:**
- **Integrità dei dati**: Garantisce che i dati rispettino sempre le regole di business
- **Logica di dominio**: Implementa le operazioni specifiche del dominio applicativo
- **Persistenza**: Gestisce il salvataggio e recupero dei dati
- **Notificazione**: Informa i componenti interessati quando lo stato cambia

```java
// Esempio minimale di Model che incapsula logica di business
@Entity
public class Game {
    private String title;
    private LocalDate releaseDate;
    private BigDecimal price;
    
    // Logica di business incapsulata nel Model
    public boolean isNewRelease() {
        return releaseDate.isAfter(LocalDate.now().minusMonths(6));
    }
    
    public void applyDiscount(BigDecimal percentage) {
        if (percentage.compareTo(BigDecimal.ZERO) < 0 || 
            percentage.compareTo(new BigDecimal("100")) > 0) {
            throw new IllegalArgumentException("Sconto non valido");
        }
        this.price = price.multiply(BigDecimal.ONE.subtract(percentage.divide(new BigDecimal("100"))));
    }
}
```

#### La View: Interprete della Realtà

La **View** è l'interfaccia tra il mondo digitale dell'applicazione e la percezione umana. Il suo ruolo va oltre la semplice visualizzazione: è responsabile di tradurre i dati astratti del Model in una rappresentazione comprensibile e utilizzabile dall'utente.

**Principi fondamentali della View:**
- **Passività**: Non dovrebbe mai modificare direttamente i dati
- **Reattività**: Deve riflettere immediatamente i cambiamenti del Model
- **Separazione**: Completamente indipendente dalla logica di business
- **Adattabilità**: Può esistere in multiple forme per lo stesso Model

La View opera secondo il principio della **presentazione dichiarativa**: descrive *cosa* mostrare, non *come* i dati vengono elaborati. Questo permette di cambiare completamente l'aspetto dell'applicazione senza toccare la logica sottostante.

```html
<!-- Esempio di View che si limita alla presentazione -->
<div class="game-card">
    <h3>{{game.title}}</h3>
    <span class="price">€{{game.price}}</span>
    <!-- La logica isNewRelease() è nel Model, la View si limita a mostrarla -->
    <span *ngIf="game.isNewRelease" class="badge">Novità!</span>
</div>
```

#### Il Controller: Direttore d'Orchestra

Il **Controller** è il componente più sottile e spesso frainteso di MVC. Non è un contenitore di logica di business, ma piuttosto un **coordinatore intelligente** che orchestra le interazioni tra Model e View. Il Controller implementa il pattern **Mediator**, fungendo da intermediario che conosce come far comunicare i componenti senza che questi si conoscano direttamente.

**Filosofia del Controller:**
- **Orchestrazione**: Coordina il flusso senza implementare logica di dominio
- **Traduzione**: Converte gli input dell'utente in operazioni sul Model
- **Routing**: Determina quale View utilizzare in base al contesto
- **Gestione dello stato**: Mantiene lo stato della sessione utente

Il Controller opera secondo il principio della **responsabilità limitata**: sa *cosa* fare ma delega *come* farlo. Questo lo rende il punto di ingresso ideale per cross-cutting concerns come autenticazione, logging e validazione.

```java
// Controller che si limita alla coordinazione
@RestController
public class GameController {
    private final GameService gameService;
    
    @GetMapping("/games")
    public ResponseEntity<List<GameDTO>> getGames(@RequestParam String filter) {
        // Traduce la richiesta HTTP in operazione di business
        List<GameDTO> games = gameService.findGamesByFilter(filter);
        // Coordina la risposta senza implementare logica
        return ResponseEntity.ok(games);
    }
}
```

### 1.1.3 I Benefici Strategici di MVC

L'adozione di MVC non è solo una questione di organizzazione del codice, ma una **decisione strategica** che influenza l'intera evoluzione del software.

**Scalabilità Concettuale**: MVC permette di ragionare su problemi complessi dividendoli in domini più gestibili. Quando un'applicazione cresce, la separazione delle responsabilità impedisce che la complessità diventi ingestibile.

**Evoluzione Indipendente**: I tre componenti possono evolvere secondo ritmi diversi. L'interfaccia utente può essere completamente ridisegnata senza toccare la logica di business, o viceversa.

**Testing Strategico**: La separazione permette di testare la logica di business indipendentemente dall'interfaccia, riducendo drasticamente la complessità dei test e aumentandone l'affidabilità.

**Riusabilità Architettonica**: Un Model ben progettato può servire multiple interfacce (web, mobile, API) senza duplicazione di codice.

### 1.1.4 Le Sfide e i Compromessi

Ogni pattern architetturale comporta dei trade-off, e MVC non fa eccezione.

**Complessità Iniziale**: Per applicazioni molto semplici, MVC può sembrare un'over-engineering. Tuttavia, questa complessità iniziale è un investimento che paga dividendi quando l'applicazione cresce.

**Accoppiamento Sottile**: Nonostante la separazione, esiste sempre un accoppiamento concettuale tra i componenti. Il Controller deve conoscere sia il Model che la View, creando una dipendenza a forma di "Y".

**Overhead Cognitivo**: Gli sviluppatori devono costantemente decidere dove posizionare la logica, richiedendo una comprensione profonda dei principi del pattern.

### 1.1.5 MVC nell'Era Moderna: Evoluzione e Adattamenti

L'implementazione moderna di MVC, come quella di Spring Boot, rappresenta un'evoluzione del pattern originale, adattata alle esigenze delle applicazioni enterprise contemporanee.

**Stratificazione del Model**: Il Model moderno non è più un singolo componente, ma una **stratificazione di responsabilità**:
- **Entity**: Rappresentazione dei dati
- **Service**: Logica di business
- **Repository**: Accesso ai dati

Questa stratificazione riflette la crescente complessità delle applicazioni moderne e la necessità di separare ulteriormente le responsabilità.

```java
// Model stratificato in Spring Boot
@Entity
public class Game {
    // Solo dati e validazioni di base
}

@Service
public class GameService {
    // Logica di business complessa
    public GameDTO processGameCreation(GameCreateRequest request) {
        // Orchestrazione di operazioni di business
    }
}
```

**Adattamento alle Architetture Distribuite**: MVC moderno deve confrontarsi con microservizi, API REST e architetture event-driven, mantenendo i suoi principi fondamentali ma adattandoli a contesti distribuiti.

## 1.2 Layered Architecture (Architettura a Livelli)

### 1.2.1 Filosofia della Stratificazione

L'**Architettura a Livelli** rappresenta una delle più antiche e durature metafore organizzative nel software engineering. Ispirata all'architettura di rete OSI e ai principi della geologia, questa architettura riconosce che la complessità software può essere gestita attraverso la **stratificazione delle responsabilità**.

Il principio fondamentale è quello della **dipendenza unidirezionale**: ogni livello può dipendere solo dai livelli sottostanti, mai da quelli superiori. Questo crea una **gerarchia di astrazione** dove ogni livello nasconde la complessità di quelli inferiori, fornendo un'interfaccia semplificata a quelli superiori.

**Vantaggi concettuali della stratificazione:**
- **Isolamento della complessità**: Ogni livello gestisce un tipo specifico di complessità
- **Sostituibilità**: Un livello può essere sostituito senza impattare gli altri
- **Testabilità incrementale**: Ogni livello può essere testato indipendentemente
- **Comprensibilità**: La struttura riflette il modo naturale di pensare ai problemi

### 1.2.2 Anatomia dei Livelli Architetturali

#### Presentation Layer: La Facciata del Sistema

Il **Presentation Layer** è l'ambasciatore del sistema verso il mondo esterno. Non è semplicemente un'interfaccia utente, ma il **traduttore** tra il linguaggio del dominio interno e quello del mondo esterno.

**Responsabilità concettuali:**
- **Traduzione dei protocolli**: Converte richieste HTTP in chiamate di dominio
- **Gestione del contesto**: Mantiene informazioni sulla sessione e l'utente
- **Validazione sintattica**: Verifica la correttezza formale degli input
- **Serializzazione**: Converte oggetti di dominio in formati di trasporto

```java
@RestController
public class GameController {
    private final GameService gameService;
    
    @GetMapping("/games")
    public ResponseEntity<List<GameDTO>> getGames(
            @Valid @ModelAttribute GameSearchCriteria criteria) {
        // Il controller traduce la richiesta HTTP in operazione di dominio
        List<GameDTO> games = gameService.searchGames(criteria);
        return ResponseEntity.ok(games);
    }
}
```

#### Business Layer: Il Cuore Pulsante del Dominio

Il **Business Layer** è dove risiede l'essenza dell'applicazione. Qui vengono implementate le regole che definiscono *cosa* significa essere un videogioco, *come* si comporta un utente, *quando* un'operazione è valida.

**Responsabilità concettuali del Business Layer:**
- **Orchestrazione del Dominio**: Coordina operazioni complesse che coinvolgono multiple entità
- **Applicazione delle Regole**: Implementa le invarianti di business che definiscono la validità delle operazioni
- **Gestione Transazionale**: Garantisce la consistenza dei dati attraverso operazioni atomiche
- **Integrazione di Servizi**: Coordina l'interazione con servizi esterni e altri bounded context

**Principi Architetturali:**
- **Statelessness**: I servizi non mantengono stato tra le chiamate
- **Idempotenza**: Le operazioni producono lo stesso risultato se ripetute
- **Fail-Fast**: Validazione immediata per prevenire stati inconsistenti
- **Event-Driven**: Pubblicazione di eventi per comunicazione asincrona

Il Business Layer opera come **orchestratore intelligente**, combinando dati da multiple sorgenti, applicando regole complesse e coordinando side-effects come notifiche e audit logging.
```

#### Data Access Layer: Il Custode della Persistenza

Il **Data Access Layer** rappresenta il confine tra il mondo degli oggetti e quello relazionale. È responsabile di **tradurre** le operazioni di dominio in operazioni di persistenza, nascondendo la complessità del database sottostante.

**Responsabilità Architetturali:**
- **Astrazione della Persistenza**: Nasconde i dettagli implementativi del database
- **Ottimizzazione delle Query**: Gestisce performance e strategie di caricamento
- **Gestione delle Transazioni**: Coordina operazioni atomiche sui dati
- **Mapping Oggetto-Relazionale**: Traduce tra paradigmi diversi

**Strategie di Implementazione:**

**Query Derivate**: Utilizzano convenzioni di naming per generare automaticamente query SQL, riducendo il boilerplate e aumentando la leggibilità.

**Query Personalizzate**: Per logiche complesse che non possono essere espresse attraverso convenzioni, permettendo controllo fine sulla generazione SQL.

**Criteria API**: Per query dinamiche costruite a runtime, offrendo type-safety e flessibilità per filtri configurabili.

**Query Native**: Per ottimizzazioni specifiche del database o operazioni che richiedono funzionalità SQL avanzate.

**Principi di Design:**
- **Single Responsibility**: Ogni repository gestisce una singola entità aggregata
- **Interface Segregation**: Interfacce specifiche per diversi tipi di operazioni
- **Dependency Inversion**: Dipendenza da astrazioni, non da implementazioni concrete
```

### 1.2.3 Vantaggi dell'Architettura a Livelli

1. **Separazione delle Responsabilità**: Ogni livello ha un compito specifico
2. **Riusabilità**: I livelli inferiori possono essere riutilizzati
3. **Testabilità**: Ogni livello può essere testato indipendentemente
4. **Manutenibilità**: Modifiche isolate a un livello
5. **Scalabilità**: I livelli possono essere scalati indipendentemente

### 1.2.4 Svantaggi e Considerazioni

1. **Performance**: Overhead dovuto ai livelli multipli
2. **Complessità**: Può essere eccessivo per applicazioni semplici
3. **Rigidità**: Difficile modificare la struttura una volta stabilita
4. **Accoppiamento**: Dipendenze tra livelli adiacenti

## 1.3 Repository Pattern

### 1.3.1 Definizione e Scopo

Il **Repository Pattern** incapsula la logica necessaria per accedere alle fonti di dati. Centralizza le funzionalità di accesso ai dati comuni, fornendo una migliore manutenibilità e disaccoppiando l'infrastruttura o la tecnologia utilizzata per accedere ai database dal livello del modello di dominio.

### 1.3.2 Implementazione in Spring Data JPA

#### Architettura Stratificata dei Repository

L'implementazione moderna del Repository Pattern in Spring Data JPA segue una **architettura stratificata** che combina convenzioni, personalizzazioni e ottimizzazioni.

**Livelli di Astrazione:**

**1. Repository Base Generico**
Fornisce operazioni comuni a tutte le entità, implementando pattern trasversali come soft delete, auditing e operazioni batch. Questo livello elimina la duplicazione di codice e garantisce consistenza comportamentale.

**2. Repository Specifico di Dominio**
Estende il repository base con operazioni specifiche dell'entità, utilizzando query derivate per operazioni semplici e query personalizzate per logiche complesse.
**3. Repository Personalizzato**
Per query complesse che richiedono logica procedurale, costruzione dinamica di criteri o ottimizzazioni specifiche del database.

**Strategie di Query:**

**Query Derivate**: Utilizzano convenzioni di naming per generare automaticamente implementazioni. Ideali per operazioni semplici e leggibili.

**Query Dichiarative**: Definite tramite annotazioni JPQL o SQL nativo. Offrono controllo preciso sulla generazione delle query.

**Query Statistiche**: Specializzate per aggregazioni e calcoli, spesso utilizzate per dashboard e reporting.

**Query Native**: Per sfruttare funzionalità specifiche del database o ottimizzazioni di performance critiche.
```

#### Repository Personalizzato: Gestione della Complessità

I **Repository Personalizzati** rappresentano il livello più sofisticato dell'architettura di accesso ai dati, dove vengono implementate logiche che non possono essere espresse attraverso convenzioni o query dichiarative.

**Scenari di Utilizzo:**

**Query Dinamiche**: Costruzione runtime di criteri di ricerca basati su input utente variabili. Utilizzano Criteria API per garantire type-safety e performance ottimali.

**Algoritmi di Raccomandazione**: Implementano logiche complesse che combinano preferenze utente, analisi comportamentali e algoritmi di machine learning per suggerire contenuti pertinenti.

**Aggregazioni Statistiche**: Calcolano metriche di business attraverso query ottimizzate che spesso richiedono JOIN complessi e funzioni di aggregazione avanzate.

**Ottimizzazioni Specifiche**: Implementano strategie di performance per scenari critici, utilizzando hint del database, query native o tecniche di caching avanzate.

**Principi di Implementazione:**
- **Separation of Concerns**: Interfaccia separata dall'implementazione
- **Composability**: Metodi riutilizzabili per costruire query complesse
- **Performance-First**: Ottimizzazione delle query per scenari ad alto carico
- **Maintainability**: Codice leggibile e ben documentato per logiche complesse
```

### 1.3.3 Vantaggi Strategici del Repository Pattern

**Disaccoppiamento Architetturale**: Il Repository Pattern crea una **barriera di astrazione** tra la logica di business e i dettagli di persistenza, permettendo l'evoluzione indipendente di entrambi i livelli.

**Testabilità Incrementale**: Facilita la creazione di test unitari attraverso mock repositories, permettendo di testare la logica di business senza dipendenze dal database.

**Centralizzazione della Conoscenza**: Concentra tutta la logica di accesso ai dati in un singolo punto, eliminando la duplicazione e garantendo consistenza nelle operazioni.

**Evoluzione Controllata**: Permette modifiche alle strategie di persistenza senza impattare il codice client, supportando refactoring e ottimizzazioni incrementali.

### 1.3.4 Principi di Design Avanzati

**Single Responsibility Principle**: Ogni repository gestisce esclusivamente una singola entità aggregata, mantenendo coesione e riducendo l'accoppiamento.

**Interface Segregation**: Utilizzo di interfacce specifiche per diversi tipi di operazioni (lettura, scrittura, query complesse), evitando dipendenze non necessarie.

**Dependency Inversion**: Dipendenza da astrazioni piuttosto che da implementazioni concrete, facilitando l'inversione di controllo e la testabilità.

**Performance by Design**: Implementazione di strategie di ottimizzazione come EntityGraph, query batch e caching a livello di repository per garantire performance scalabili.

**Naming Semantico**: Convenzioni di naming che riflettono l'intento business piuttosto che i dettagli implementativi, migliorando la leggibilità e la manutenibilità.

Questi pattern architetturali rappresentano i **pilastri fondamentali** per costruire applicazioni enterprise robuste, fornendo una base solida per la separazione delle responsabilità, la scalabilità e l'evoluzione controllata del software.