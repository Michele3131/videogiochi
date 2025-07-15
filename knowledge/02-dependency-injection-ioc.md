# 2. Dependency Injection e Inversion of Control

## 2.1 Inversion of Control (IoC): La Rivoluzione Concettuale

### 2.1.1 Il Problema delle Dipendenze Hard-Coded

Nel paradigma tradizionale, gli oggetti creano e gestiscono le proprie dipendenze, causando **accoppiamento forte** e difficoltà di testing.

```java
public class GameService {
    public GameService() {
        // Dipendenze hard-coded - accoppiamento forte
        this.gameRepository = new MySQLGameRepository();
        this.emailService = new SMTPEmailService();
    }
}
```

**Problemi:**
- **Accoppiamento Forte**: Legato a implementazioni specifiche
- **Difficoltà di Testing**: Impossibile testare in isolamento
- **Rigidità**: Difficile cambiare implementazioni
- **Responsabilità Multiple**: L'oggetto gestisce sia logica che creazione dipendenze

### 2.1.2 La Trasformazione attraverso IoC

L'**Inversion of Control** inverte la responsabilità: un **container esterno** crea e inietta le dipendenze.

**Principi Fondamentali:**
- **Inversione della Responsabilità**: Gli oggetti ricevono dipendenze dall'esterno
- **Controllo Centralizzato**: Container IoC gestisce creazione e ciclo di vita
- **Configurazione Esterna**: Dipendenze configurate fuori dal codice business

```java
@Service
public class GameService {
    private final GameRepository gameRepository;
    private final EmailService emailService;
    
    // Constructor Injection - dipendenze iniettate
    public GameService(GameRepository gameRepository, EmailService emailService) {
        this.gameRepository = gameRepository;
        this.emailService = emailService;
    }
}
```

**Vantaggi:**
- **Disaccoppiamento**: Dipendenza da astrazioni, non implementazioni
- **Testabilità**: Possibilità di iniettare mock per test isolati
- **Flessibilità**: Diverse implementazioni senza modifiche al codice
- **Configurabilità**: Setup diversi per ambienti diversi

## 2.2 Dependency Injection: L'Arte della Composizione

### 2.2.1 La Filosofia della Dependency Injection

La **Dependency Injection (DI)** è l'implementazione pratica dei principi IoC. È una **filosofia di composizione** che riconosce che la forza di un sistema risiede nelle **relazioni** tra componenti, non nei singoli componenti.

**Principi Fondamentali:**
- **Composizione over Inheritance**: Comporre comportamenti invece di ereditarli
- **Dependency Inversion**: Dipendere da astrazioni, non da implementazioni
- **Single Responsibility**: Una sola ragione per cambiare
- **Open/Closed**: Aperto all'estensione, chiuso alla modifica

### 2.2.2 Constructor Injection: L'Eleganza della Semplicità

Il **Constructor Injection** è la forma più pura di DI. Garantisce che un oggetto sia sempre in uno stato valido e completo.

```java
@Service
public class GameRecommendationService {
    private final GameRepository gameRepository;
    private final UserPreferenceService userPreferenceService;
    private final RecommendationAlgorithm algorithm;
    
    public GameRecommendationService(
            GameRepository gameRepository,
            UserPreferenceService userPreferenceService,
            RecommendationAlgorithm algorithm) {
        this.gameRepository = gameRepository;
        this.userPreferenceService = userPreferenceService;
        this.algorithm = algorithm;
    }
}
```

**Vantaggi:**
- **Immutabilità**: Dipendenze final, thread-safe
- **Fail-Fast**: Errori immediati se mancano dipendenze
- **Trasparenza**: Dipendenze esplicite nella signature
- **Testabilità**: Facile creazione di istanze per test

### 2.2.3 Setter Injection: Flessibilità Controllata

Il **Setter Injection** offre flessibilità per dipendenze opzionali o configurazione post-creazione.

```java
@Service
public class GameAnalyticsService {
    private final GameRepository gameRepository;
    private CacheManager cacheManager; // Opzionale
    
    public GameAnalyticsService(GameRepository gameRepository) {
        this.gameRepository = gameRepository;
    }
    
    @Autowired(required = false)
    public void setCacheManager(CacheManager cacheManager) {
        this.cacheManager = cacheManager;
    }
}
```

**Casi d'Uso:**
- Dipendenze opzionali
- Configurazione circolare (rara)
- Riconfigurabilità runtime
- Integrazione con sistemi legacy

### 2.2.4 Field Injection: Convenienza vs Buone Pratiche

Il **Field Injection** è conveniente ma nasconde dipendenze e complica il testing.

```java
@Service
public class GameNotificationService {
    @Autowired
    private GameRepository gameRepository; // Sconsigliato
    
    @Autowired
    private EmailService emailService;
}
```

**Problemi:**
- Dipendenze nascoste
- Difficoltà di testing senza container
- Impossibilità di usare final
- Accoppiamento al framework

### 2.2.5 Spring Framework: Configurazione Dichiarativa

Spring trasforma la dependency injection in un **linguaggio dichiarativo**. Le annotazioni rendono il codice una **dichiarazione di intenti** piuttosto che istruzioni imperative.

```java
@Configuration
public class GameConfiguration {
    
    @Bean
    @Primary
    public RecommendationAlgorithm collaborativeFiltering() {
        return new CollaborativeFilteringAlgorithm();
    }
    
    @Bean
    @ConditionalOnProperty(name = "recommendation.algorithm", havingValue = "content")
    public RecommendationAlgorithm contentBased() {
        return new ContentBasedAlgorithm();
    }
    
    @Bean
    @Scope("prototype")
    public GameSession gameSession() {
        return new GameSession();
    }
    
    @Bean
    @Lazy
    public ExpensiveAnalyticsService analyticsService() {
        return new ExpensiveAnalyticsService();
    }
}
```

**Annotazioni Chiave:**
- **@Primary**: Implementazione predefinita
- **@ConditionalOnProperty**: Creazione condizionale
- **@Scope**: Controllo del ciclo di vita
- **@Lazy**: Creazione ritardata

#### Stereotipi Architetturali

Gli **stereotipi** sono dichiarazioni architetturali che comunicano il ruolo di ogni componente:

```java
@Service // Logica di business
public class GameRecommendationService { }

@Repository // Accesso ai dati
public class GameRepositoryImpl { }

@RestController // Interfaccia REST
public class GameController { }

@Component // Componente generico
public class GameEventListener { }
```

**Semantica:**
- **@Service**: Logica di business e regole di dominio
- **@Repository**: Accesso ai dati con traduzione automatica delle eccezioni
- **@RestController**: Gestione richieste HTTP con serializzazione automatica
- **@Component**: Componente generico per casi non specifici

#### Qualificatori per Disambiguare

Quando esistono multiple implementazioni della stessa interfaccia, i **qualificatori** permettono di specificare quale implementazione utilizzare:

```java
@Service
@Qualifier("email")
public class EmailNotificationService implements NotificationService { }

@Service
@Qualifier("sms")
public class SmsNotificationService implements NotificationService { }

@Service
public class GameService {
    public GameService(@Qualifier("email") NotificationService emailService,
                      @Qualifier("sms") NotificationService smsService) {
        // Iniezione specifica per qualificatore
    }
}
```

## 2.3 Vantaggi Architetturali

### 2.3.1 Testabilità Rivoluzionaria

IoC e DI trasformano il testing permettendo l'isolamento completo delle unità:

```java
@ExtendWith(MockitoExtension.class)
class GameServiceTest {
    @Mock private GameRepository gameRepository;
    @Mock private EmailService emailService;
    @InjectMocks private GameService gameService;
    
    @Test
    void shouldCreateGameSuccessfully() {
        // Test con dipendenze controllate
    }
}
```

### 2.3.2 Flessibilità Configurabile

Cambiare implementazioni senza modificare il codice client:

```java
@Profile("dev")
@Configuration
public class DevConfig {
    @Bean
    public EmailService emailService() {
        return new MockEmailService(); // Mock per sviluppo
    }
}

@Profile("prod")
@Configuration
public class ProdConfig {
    @Bean
    public EmailService emailService() {
        return new SmtpEmailService(); // SMTP per produzione
    }
}
```

### 2.3.3 Manutenibilità e Modularità

La separazione delle responsabilità porta a codice più pulito e modulare:

**Prima (Accoppiato):**
```java
public class GameService {
    private GameRepository repository = new GameRepositoryImpl(); // Rigido
}
```

**Dopo (Disaccoppiato):**
```java
@Service
public class GameService {
    private final GameRepository repository;
    
    public GameService(GameRepository repository) {
        this.repository = repository; // Flessibile
    }
}
```

## 2.4 Conclusioni

IoC e DI rappresentano una **trasformazione paradigmatica** da un modello "costruttivo" a uno "compositivo":

**Trasformazioni Chiave:**
- **Controllo**: Da imperativo a dichiarativo
- **Struttura**: Da accoppiamento a composizione
- **Adattabilità**: Da rigidità a flessibilità

**Benefici Sistemici:**
- **Testabilità**: Testing isolato e controllato
- **Manutenibilità**: Codice modulare e pulito
- **Flessibilità**: Adattamento rapido ai requisiti
- **Riusabilità**: Componenti indipendenti

Questi principi costituiscono la base per architetture moderne, scalabili e mantenibili, e sono fondamentali per pattern avanzati come microservizi e domain-driven design.