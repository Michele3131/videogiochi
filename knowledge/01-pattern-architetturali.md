# 1. Pattern Architetturali Fondamentali

## 1.1 Model-View-Controller (MVC)

### 1.1.1 La Filosofia della Separazione

Il pattern **Model-View-Controller (MVC)** rappresenta una delle più importanti rivoluzioni concettuali nella progettazione software. Nato negli anni '70, MVC non è semplicemente un modo di organizzare il codice, ma una **filosofia di pensiero** che riconosce la natura intrinsecamente diversa di tre aspetti fondamentali di ogni applicazione interattiva.

**Il Principio Fondamentale**: La separazione delle responsabilità (Separation of Concerns) sostiene che problemi diversi dovrebbero essere risolti da componenti diversi. In un'applicazione, distinguiamo:

- **I dati e le regole che li governano** (Model)
- **Come questi dati vengono presentati** (View)
- **Come l'utente interagisce con il sistema** (Controller)

Questa separazione non è arbitraria: riflette il fatto che questi tre aspetti evolvono secondo ritmi e logiche completamente diverse. L'interfaccia utente può cambiare per seguire nuove tendenze di design, la logica di business può evolversi per nuovi requisiti, e i dati possono essere ristrutturati per performance, ma ciascuno di questi cambiamenti dovrebbe essere indipendente dagli altri.

### 1.1.2 Il Model: Il Cuore Logico

Il **Model** non è semplicemente un contenitore di dati, ma l'**incarnazione delle regole di business**. È il componente che "sa" cosa significa essere un videogioco, quali operazioni sono valide, e come i dati si relazionano tra loro.

**Caratteristiche concettuali del Model:**
- **Autonomia**: Funziona indipendentemente da come viene presentato
- **Integrità**: Garantisce che i dati rispettino sempre le regole di business
- **Responsabilità**: Contiene tutta la logica specifica del dominio

```java
// Esempio minimale: il Model incapsula la logica di business
public class Game {
    private String title;
    private LocalDate releaseDate;
    private BigDecimal price;
    
    // Il Model "sa" cosa significa essere una novità
    public boolean isNewRelease() {
        return releaseDate.isAfter(LocalDate.now().minusMonths(6));
    }
    
    // Il Model gestisce le proprie regole di business
    public void applyDiscount(BigDecimal percentage) {
        if (percentage.compareTo(BigDecimal.ZERO) < 0) {
            throw new IllegalArgumentException("Sconto non valido");
        }
        this.price = price.multiply(BigDecimal.ONE.subtract(percentage.divide(new BigDecimal("100"))));
    }
}
```

### 1.1.3 La View: L'Interprete

La **View** è il traduttore tra il mondo astratto dei dati e la percezione umana. Il suo compito è rendere comprensibili e utilizzabili le informazioni contenute nel Model, senza mai modificarle direttamente.

**Principi della View:**
- **Passività**: Non modifica mai i dati, li presenta soltanto
- **Reattività**: Riflette immediatamente i cambiamenti del Model
- **Adattabilità**: Lo stesso Model può avere multiple rappresentazioni

```html
<!-- La View si limita a presentare, non a elaborare -->
<div class="game-card">
    <h3>{{game.title}}</h3>
    <span class="price">€{{game.price}}</span>
    <span *ngIf="game.isNewRelease" class="badge">Novità!</span>
</div>
```

### 1.1.4 Il Controller: Il Coordinatore

Il **Controller** è il componente più sottile di MVC. Non contiene logica di business, ma **coordina** le interazioni tra Model e View. È il punto di ingresso che traduce le azioni dell'utente in operazioni sul Model.

**Responsabilità del Controller:**
- **Traduzione**: Converte input utente in operazioni di business
- **Coordinazione**: Orchestra l'interazione tra componenti
- **Routing**: Determina quale View utilizzare

```java
// Il Controller coordina senza implementare logica
@RestController
public class GameController {
    private final GameService gameService;
    
    @GetMapping("/games")
    public ResponseEntity<List<GameDTO>> getGames(@RequestParam String filter) {
        // Traduce HTTP in operazione di business
        List<GameDTO> games = gameService.findGamesByFilter(filter);
        return ResponseEntity.ok(games);
    }
}
```

### 1.1.5 I Benefici della Separazione

**Manutenibilità**: Ogni componente può evolversi indipendentemente. Cambiare l'interfaccia utente non richiede modifiche alla logica di business.

**Testabilità**: Ogni componente può essere testato in isolamento, semplificando la verifica della correttezza.

**Riusabilità**: Lo stesso Model può supportare multiple View (web, mobile, API).

**Scalabilità del Team**: Sviluppatori diversi possono lavorare su componenti diversi senza interferenze.

### 1.1.6 MVC nel Contesto Moderno

MVC rimane fondamentale nelle architetture moderne, evolvendosi in varianti come MVVM per framework reattivi. Nel nostro progetto:
- **Model**: Entità JPA con logica di business
- **View**: Template Thymeleaf e risposte JSON
- **Controller**: Spring MVC Controllers per coordinazione HTTP

La stratificazione moderna del Model (Entity, Service, Repository) riflette la crescente complessità delle applicazioni enterprise.

## 1.2 Layered Architecture (Architettura a Livelli)

### 1.2.1 La Filosofia della Stratificazione

L'**Architettura a Livelli** applica il principio della **stratificazione delle responsabilità** per gestire la complessità software. Ogni livello ha una responsabilità specifica e può dipendere solo dai livelli sottostanti, creando una gerarchia di astrazione.

**Principio Fondamentale**: La dipendenza unidirezionale garantisce che ogni livello nasconda la complessità di quelli inferiori, fornendo un'interfaccia semplificata a quelli superiori.

**Benefici della Stratificazione:**
- **Isolamento**: Ogni livello gestisce un tipo specifico di complessità
- **Sostituibilità**: Un livello può essere sostituito senza impattare gli altri
- **Testabilità**: Ogni livello può essere testato indipendentemente
- **Comprensibilità**: Struttura che riflette il pensiero naturale sui problemi

### 1.2.2 I Tre Livelli Fondamentali

#### Presentation Layer: L'Interfaccia

Il **Presentation Layer** è il traduttore tra il mondo esterno e il sistema interno. Gestisce protocolli di comunicazione, validazione degli input e serializzazione delle risposte.

**Responsabilità:**
- Traduzione delle richieste in operazioni di dominio
- Gestione del contesto utente e sessione
- Validazione sintattica degli input
- Serializzazione delle risposte

```java
@RestController
public class GameController {
    @GetMapping("/games")
    public ResponseEntity<List<GameDTO>> getGames(@Valid GameSearchCriteria criteria) {
        List<GameDTO> games = gameService.searchGames(criteria);
        return ResponseEntity.ok(games);
    }
}
```

#### Business Layer: Il Cuore Logico

Il **Business Layer** contiene l'essenza dell'applicazione: le regole che definiscono cosa significa essere un videogioco, come si comporta un utente, quando un'operazione è valida.

**Responsabilità:**
- Orchestrazione di operazioni complesse tra multiple entità
- Applicazione delle regole di business e invarianti
- Gestione transazionale per garantire consistenza
- Coordinazione con servizi esterni

**Principi:**
- **Statelessness**: Nessuno stato mantenuto tra chiamate
- **Idempotenza**: Stesso risultato per operazioni ripetute
- **Fail-Fast**: Validazione immediata degli input

```java
@Service
public class GameService {
    public GameDTO createGame(GameCreateRequest request) {
        // Validazione regole di business
        validateGameRules(request);
        // Orchestrazione operazioni
        Game game = gameRepository.save(new Game(request));
        return gameMapper.toDTO(game);
    }
}
```
```

#### Data Access Layer: Il Custode dei Dati

Il **Data Access Layer** traduce le operazioni di dominio in operazioni di persistenza, nascondendo la complessità del database sottostante.

**Responsabilità:**
- Astrazione della persistenza dal database specifico
- Ottimizzazione delle query e strategie di caricamento
- Mapping tra oggetti e strutture relazionali
- Gestione delle transazioni sui dati

**Strategie:**
- **Query Derivate**: Convenzioni di naming per generazione automatica
- **Query Personalizzate**: Controllo fine per logiche complesse
- **Criteria API**: Query dinamiche type-safe
- **Query Native**: Ottimizzazioni specifiche del database

```java
public interface GameRepository extends JpaRepository<Game, Long> {
    // Query derivata
    List<Game> findByTitleContaining(String title);
    
    // Query personalizzata
    @Query("SELECT g FROM Game g WHERE g.price < :maxPrice")
    List<Game> findAffordableGames(@Param("maxPrice") BigDecimal maxPrice);
}
```

### 1.2.3 Vantaggi e Svantaggi

**Vantaggi:**
- Separazione chiara delle responsabilità
- Riusabilità dei livelli inferiori
- Testabilità indipendente di ogni livello
- Manutenibilità attraverso modifiche isolate

**Svantaggi:**
- Overhead di performance per livelli multipli
- Complessità eccessiva per applicazioni semplici
- Rigidità nella modifica della struttura
```

## 1.3 Repository Pattern

### 1.3.1 La Filosofia del Repository

Il **Repository Pattern** incapsula la logica di accesso ai dati, fornendo un'interfaccia uniforme per le operazioni di persistenza. Disaccoppia il dominio dall'infrastruttura di persistenza, permettendo di cambiare database senza impattare la logica di business.

**Principi Fondamentali:**
- **Astrazione**: Nasconde i dettagli di persistenza al dominio
- **Centralizzazione**: Concentra la logica di accesso ai dati
- **Testabilità**: Permette mock e stub per i test
- **Consistenza**: Fornisce un'interfaccia uniforme

### 1.3.2 Implementazione in Spring Data JPA

Spring Data JPA implementa il Repository Pattern attraverso diverse strategie:

**Query Derivate**: Convenzioni di naming per generazione automatica
```java
List<Game> findByTitleContaining(String title);
```

**Query Dichiarative**: Controllo preciso con JPQL
```java
@Query("SELECT g FROM Game g WHERE g.price < :maxPrice")
List<Game> findAffordableGames(@Param("maxPrice") BigDecimal maxPrice);
```

**Repository Personalizzati**: Per logiche complesse
```java
public interface GameRepositoryCustom {
    List<Game> findWithComplexCriteria(GameSearchCriteria criteria);
}
```

### 1.3.3 Vantaggi del Pattern

- **Disaccoppiamento**: Separazione tra dominio e persistenza
- **Testabilità**: Facilita unit testing con mock
- **Manutenibilità**: Centralizza la logica di accesso ai dati
- **Flessibilità**: Permette cambi di tecnologia di persistenza
## Conclusione

Questi pattern architetturali rappresentano i **pilastri fondamentali** per costruire applicazioni enterprise robuste:

- **MVC** fornisce separazione delle responsabilità tra presentazione, logica e dati
- **Layered Architecture** organizza la complessità attraverso livelli con dipendenze unidirezionali
- **Repository Pattern** disaccoppia il dominio dalla persistenza

Ogni pattern affronta specifici problemi architetturali, e la loro combinazione crea una base solida per applicazioni scalabili e manutenibili.