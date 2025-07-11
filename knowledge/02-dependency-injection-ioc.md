# 2. Dependency Injection e Inversion of Control

## 2.1 La Rivoluzione Concettuale dell'Inversion of Control

### 2.1.1 Genesi di un Paradigma

**Inversion of Control (IoC)** rappresenta una delle più profonde rivoluzioni concettuali nella progettazione software. Non è semplicemente una tecnica, ma un **cambio di paradigma** che ridefinisce il rapporto tra oggetti e le loro dipendenze.

Nella programmazione tradizionale, ogni oggetto è un "dittatore" che controlla completamente la creazione e gestione delle sue dipendenze. IoC inverte questa dinamica: gli oggetti diventano "cittadini" di un ecosistema più ampio, dove un'autorità esterna (il container IoC) gestisce le relazioni e le dipendenze.

Questa inversione non è solo tecnica, ma **filosofica**: sposta il focus dal "come creare" al "cosa fare", permettendo agli oggetti di concentrarsi sulla loro responsabilità principale senza preoccuparsi della gestione delle dipendenze.

### 2.1.2 L'Anatomia del Problema: Dipendenze Hard-Coded

Per comprendere il valore di IoC, dobbiamo prima analizzare i problemi intrinseci della gestione tradizionale delle dipendenze.

```java
// Esempio di accoppiamento forte - anti-pattern
public class GameService {
    private final GameRepository repository;
    private final NotificationService notifier;
    
    public GameService() {
        // Ogni dipendenza è hard-coded - problematico!
        this.repository = new DatabaseGameRepository();
        this.notifier = new EmailNotificationService();
    }
    
    public void publishGame(Game game) {
        repository.save(game);
        notifier.notify("Game published: " + game.getTitle());
    }
}
```

**Analisi dei problemi fondamentali:**

**Accoppiamento Ontologico**: La classe non dipende da *cosa* fa un repository, ma da *come* è implementato. Questo viola il principio di astrazione e rende il codice fragile ai cambiamenti.

**Rigidità Configurativa**: Ogni modifica alle dipendenze richiede modifiche al codice sorgente, rendendo impossibile la configurazione runtime o l'adattamento a contesti diversi.

**Testabilità Compromessa**: L'impossibilità di sostituire le dipendenze rende i test unitari estremamente difficili, forzando l'uso di test di integrazione più lenti e complessi.

**Violazione della Responsabilità Singola**: La classe gestisce sia la logica di business che la costruzione delle dipendenze, mescolando responsabilità diverse.

### 2.1.3 La Trasformazione attraverso IoC

L'applicazione di IoC trasforma radicalmente l'architettura del codice, creando un sistema più flessibile e manutenibile.

```java
// Trasformazione con IoC - la classe diventa "pura"
public class GameService {
    private final GameRepository repository;
    private final NotificationService notifier;
    private final AuditLogger auditor;
    
    // Le dipendenze vengono fornite dall'esterno
    public GameService(GameRepository repository, 
                      NotificationService notifier,
                      AuditLogger auditor) {
        this.repository = repository;
        this.notifier = notifier;
        this.auditor = auditor;
    }
    
    public void publishGame(Game game) {
        // Focus esclusivo sulla logica di business
        repository.save(game);
        notifier.notify("Game published: " + game.getTitle());
        auditor.log("GAME_PUBLISHED", game.getId());
    }
}
```

**Benefici della trasformazione:**

**Purezza Funzionale**: La classe diventa "pura" nel senso che la sua logica dipende solo dagli input ricevuti, non da dettagli implementativi nascosti.

**Composabilità**: Diverse implementazioni possono essere combinate per creare comportamenti diversi senza modificare il codice della classe.

**Testabilità Intrinseca**: La possibilità di iniettare mock rende i test naturali e veloci, permettendo di testare la logica in isolamento.

**Configurabilità Dinamica**: Il comportamento può essere modificato a runtime semplicemente cambiando le dipendenze iniettate.

## 2.2 Dependency Injection: L'Arte della Composizione

### 2.2.1 Oltre la Definizione: Una Filosofia di Composizione

**Dependency Injection** non è semplicemente una tecnica di implementazione di IoC, ma una **filosofia di composizione** che trasforma il modo in cui pensiamo alla costruzione di software. Rappresenta il passaggio da un modello "costruttivo" (dove ogni oggetto costruisce le sue dipendenze) a un modello "compositivo" (dove le dipendenze vengono assemblate dall'esterno).

Questa filosofia riflette un principio più profondo: **la separazione tra configurazione e utilizzo**. Gli oggetti si concentrano su *cosa* fare (utilizzo), mentre un meccanismo esterno si occupa di *come* assemblarli (configurazione).

### 2.2.2 Le Modalità di Iniezione: Strategie di Composizione

Esistono diverse strategie per iniettare dipendenze, ognuna con implicazioni filosofiche e pratiche diverse.

#### Constructor Injection: L'Immutabilità come Virtù

Il **Constructor Injection** rappresenta la forma più pura di dependency injection. Riflette il principio che un oggetto dovrebbe essere **completo e immutabile** dal momento della sua creazione.

```java
@Service
public class GameService {
    private final GameRepository repository;
    private final GameValidator validator;
    private final EventPublisher eventPublisher;
    
    // Tutte le dipendenze fornite alla costruzione
    public GameService(GameRepository repository,
                      GameValidator validator, 
                      EventPublisher eventPublisher) {
        this.repository = repository;
        this.validator = validator;
        this.eventPublisher = eventPublisher;
    }
    
    // L'oggetto è completo e pronto all'uso
}
```

**Filosofia del Constructor Injection:**

**Completezza Ontologica**: Un oggetto esiste completamente solo quando tutte le sue dipendenze essenziali sono presenti. Non può esistere in uno stato "parziale".

**Immutabilità Strutturale**: Una volta creato, l'oggetto non cambia la sua struttura interna, garantendo prevedibilità e thread-safety.

**Fail-Fast Principle**: Gli errori di configurazione vengono rilevati immediatamente alla creazione, non durante l'esecuzione.

#### Setter Injection: La Flessibilità dell'Opzionalità

Il **Setter Injection** rappresenta un approccio più flessibile, adatto quando alcune dipendenze sono opzionali o quando l'oggetto può essere riconfigurato dopo la creazione.

```java
@Service
public class GameService {
    private GameRepository repository;
    private NotificationService notificationService; // Opzionale
    
    @Autowired
    public void setRepository(GameRepository repository) {
        this.repository = repository;
    }
    
    @Autowired(required = false)
    public void setNotificationService(NotificationService service) {
        this.notificationService = service;
    }
    
    public void publishGame(Game game) {
        repository.save(game);
        
        // Comportamento adattivo basato sulla presenza della dipendenza
        Optional.ofNullable(notificationService)
                .ifPresent(service -> service.notify("Game published: " + game.getTitle()));
    }
}
```

**Quando utilizzare Setter Injection:**
- **Dipendenze opzionali**: Quando il servizio può funzionare anche senza certe dipendenze
- **Riconfigurazione runtime**: Quando le dipendenze possono cambiare durante il ciclo di vita
- **Compatibilità legacy**: Quando si deve integrare con codice esistente

#### Field Injection (Sconsigliato)
```java
@Service
public class GameService {
    
    @Autowired // SCONSIGLIATO!
    private GameRepository gameRepository;
    
    @Autowired
    private EmailService emailService;
    
    // Problemi:
    // - Difficile testare
    // - Dipendenze nascoste
    // - Possibili NullPointerException
    // - Accoppiamento con il framework
}
```

### 2.2.3 Spring Framework e Dependency Injection

#### Configurazione con Annotazioni
```java
// Configurazione dei Bean
@Configuration
@EnableJpaRepositories(basePackages = "com.videogames.repository")
@ComponentScan(basePackages = "com.videogames")
public class AppConfig {
    
    @Bean
    @Primary
    public GameRepository gameRepository(EntityManager entityManager) {
        return new GameRepositoryImpl(entityManager);
    }
    
    @Bean
    @ConditionalOnProperty(name = "email.enabled", havingValue = "true")
    public EmailService emailService() {
        return new SmtpEmailService();
    }
    
    @Bean
    @ConditionalOnProperty(name = "email.enabled", havingValue = "false", matchIfMissing = true)
    public EmailService mockEmailService() {
        return new MockEmailService();
    }
    
    @Bean
    @Scope("prototype") // Nuovo bean per ogni richiesta
    public GameProcessor gameProcessor() {
        return new GameProcessor();
    }
    
    @Bean
    @Lazy // Inizializzazione lazy
    public ExpensiveService expensiveService() {
        return new ExpensiveService();
    }
}
```

#### Annotazioni di Stereotipo
```java
// Service Layer
@Service
@Transactional
public class GameService {
    // Logica di business
}

// Repository Layer
@Repository
public class GameRepositoryImpl implements GameRepository {
    // Accesso ai dati
}

// Controller Layer
@RestController
@RequestMapping("/api/v1/games")
public class GameController {
    // Gestione HTTP
}

// Configuration
@Configuration
public class DatabaseConfig {
    // Configurazione
}

// Component generico
@Component
public class GameValidator {
    // Validazione
}
```

#### Qualificatori per Disambiguare
```java
// Interfaccia comune
public interface NotificationService {
    void sendNotification(String message);
}

// Implementazioni multiple
@Service
@Qualifier("email")
public class EmailNotificationService implements NotificationService {
    @Override
    public void sendNotification(String message) {
        // Invio email
    }
}

@Service
@Qualifier("sms")
public class SmsNotificationService implements NotificationService {
    @Override
    public void sendNotification(String message) {
        // Invio SMS
    }
}

// Utilizzo con qualificatori
@Service
public class GameService {
    private final NotificationService emailService;
    private final NotificationService smsService;
    
    public GameService(@Qualifier("email") NotificationService emailService,
                      @Qualifier("sms") NotificationService smsService) {
        this.emailService = emailService;
        this.smsService = smsService;
    }
    
    public void notifyGameRelease(Game game) {
        String message = "Nuovo gioco disponibile: " + game.getTitle();
        emailService.sendNotification(message);
        smsService.sendNotification(message);
    }
}
```

### 2.2.4 Scopi dei Bean (Bean Scopes)

#### Singleton (Default)
```java
@Service
// @Scope("singleton") - default
public class GameService {
    // Una sola istanza per l'intero container
    // Condivisa tra tutte le richieste
}
```

#### Prototype
```java
@Service
@Scope("prototype")
public class GameProcessor {
    // Nuova istanza ogni volta che viene richiesta
    // Utile per oggetti stateful
}
```

#### Request (Web Applications)
```java
@Component
@Scope("request")
public class RequestContext {
    // Una istanza per ogni richiesta HTTP
    // Automaticamente distrutta alla fine della richiesta
}
```

#### Session (Web Applications)
```java
@Component
@Scope("session")
public class UserSession {
    // Una istanza per ogni sessione HTTP
    // Mantiene stato durante la sessione
}
```

### 2.2.5 Lifecycle dei Bean

#### Inizializzazione e Distruzione
```java
@Service
public class GameCacheService {
    private Cache<String, Game> gameCache;
    
    @PostConstruct
    public void initialize() {
        // Inizializzazione dopo l'iniezione delle dipendenze
        gameCache = CacheBuilder.newBuilder()
            .maximumSize(1000)
            .expireAfterWrite(30, TimeUnit.MINUTES)
            .build();
        
        log.info("GameCacheService inizializzato");
    }
    
    @PreDestroy
    public void cleanup() {
        // Pulizia prima della distruzione del bean
        if (gameCache != null) {
            gameCache.invalidateAll();
        }
        log.info("GameCacheService distrutto");
    }
    
    public void cacheGame(String key, Game game) {
        gameCache.put(key, game);
    }
    
    public Optional<Game> getCachedGame(String key) {
        return Optional.ofNullable(gameCache.getIfPresent(key));
    }
}
```

#### Bean Factory Post Processors
```java
@Component
public class CustomBeanPostProcessor implements BeanPostProcessor {
    
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) 
            throws BeansException {
        
        if (bean instanceof GameService) {
            log.info("Inizializzazione GameService: {}", beanName);
        }
        
        return bean;
    }
    
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) 
            throws BeansException {
        
        if (bean instanceof GameService) {
            log.info("GameService inizializzato: {}", beanName);
        }
        
        return bean;
    }
}
```

## 2.3 Configurazione Avanzata

### 2.3.1 Profili Spring
```java
// Configurazione per ambiente di sviluppo
@Configuration
@Profile("dev")
public class DevConfig {
    
    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.H2)
            .build();
    }
    
    @Bean
    public EmailService emailService() {
        return new MockEmailService(); // Mock per sviluppo
    }
}

// Configurazione per produzione
@Configuration
@Profile("prod")
public class ProdConfig {
    
    @Bean
    public DataSource dataSource() {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl("jdbc:mysql://prod-db:3306/videogames");
        config.setUsername("${db.username}");
        config.setPassword("${db.password}");
        config.setMaximumPoolSize(20);
        return new HikariDataSource(config);
    }
    
    @Bean
    public EmailService emailService() {
        return new SmtpEmailService(); // Servizio reale
    }
}

// Attivazione profili
// application.yml
# spring:
#   profiles:
#     active: dev

// Oppure via JVM
// -Dspring.profiles.active=prod
```

### 2.3.2 Configurazione Condizionale
```java
@Configuration
public class ConditionalConfig {
    
    @Bean
    @ConditionalOnProperty(
        name = "cache.enabled", 
        havingValue = "true", 
        matchIfMissing = false
    )
    public CacheManager cacheManager() {
        return new CaffeineCacheManager();
    }
    
    @Bean
    @ConditionalOnMissingBean(CacheManager.class)
    public CacheManager noCacheManager() {
        return new NoOpCacheManager();
    }
    
    @Bean
    @ConditionalOnClass(RedisTemplate.class)
    public RedisTemplate<String, Object> redisTemplate() {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        // Configurazione Redis
        return template;
    }
    
    @Bean
    @ConditionalOnWebApplication
    public WebMvcConfigurer webConfig() {
        return new WebMvcConfigurer() {
            // Configurazione web
        };
    }
}
```

### 2.3.3 Configurazione Esterna
```java
// Proprietà di configurazione
@ConfigurationProperties(prefix = "app.game")
@Data
@Component
public class GameProperties {
    private int maxGamesPerUser = 100;
    private boolean enableRecommendations = true;
    private Duration cacheExpiration = Duration.ofMinutes(30);
    private List<String> allowedGenres = new ArrayList<>();
    private Map<String, String> defaultSettings = new HashMap<>();
    
    @NestedConfigurationProperty
    private EmailSettings email = new EmailSettings();
    
    @Data
    public static class EmailSettings {
        private boolean enabled = true;
        private String from = "noreply@videogames.com";
        private String subject = "Notifica Videogiochi";
    }
}

// Utilizzo delle proprietà
@Service
public class GameService {
    private final GameProperties gameProperties;
    
    public GameService(GameProperties gameProperties) {
        this.gameProperties = gameProperties;
    }
    
    public void validateUserGameLimit(User user) {
        int userGameCount = getUserGameCount(user);
        if (userGameCount >= gameProperties.getMaxGamesPerUser()) {
            throw new BusinessException(
                "Limite massimo di giochi raggiunto: " + 
                gameProperties.getMaxGamesPerUser()
            );
        }
    }
    
    public List<Game> getRecommendations(User user) {
        if (!gameProperties.isEnableRecommendations()) {
            return Collections.emptyList();
        }
        
        // Logica di raccomandazione
        return findRecommendedGames(user);
    }
}
```

```yaml
# application.yml
app:
  game:
    max-games-per-user: 150
    enable-recommendations: true
    cache-expiration: PT45M
    allowed-genres:
      - ACTION
      - ADVENTURE
      - RPG
      - STRATEGY
    default-settings:
      theme: dark
      language: it
    email:
      enabled: true
      from: "games@mycompany.com"
      subject: "Aggiornamenti Videogiochi"
```

## 2.4 Testing con Dependency Injection

### 2.4.1 Unit Testing
```java
@ExtendWith(MockitoExtension.class)
class GameServiceTest {
    
    @Mock
    private GameRepository gameRepository;
    
    @Mock
    private EmailService emailService;
    
    @Mock
    private AuditService auditService;
    
    @InjectMocks
    private GameService gameService;
    
    @Test
    void shouldCreateGameSuccessfully() {
        // Given
        Game game = Game.builder()
            .title("Test Game")
            .price(BigDecimal.valueOf(59.99))
            .build();
        
        when(gameRepository.save(any(Game.class)))
            .thenReturn(game);
        
        // When
        gameService.createGame(game);
        
        // Then
        verify(gameRepository).save(game);
        verify(emailService).sendNotification(contains("Test Game"));
        verify(auditService).logAction(eq("GAME_CREATED"), any());
    }
    
    @Test
    void shouldHandleEmailServiceFailure() {
        // Given
        Game game = Game.builder().title("Test Game").build();
        
        when(gameRepository.save(any(Game.class))).thenReturn(game);
        doThrow(new RuntimeException("Email service down"))
            .when(emailService).sendNotification(any());
        
        // When & Then
        assertThatThrownBy(() -> gameService.createGame(game))
            .isInstanceOf(RuntimeException.class)
            .hasMessage("Email service down");
        
        verify(gameRepository).save(game);
        verify(auditService, never()).logAction(any(), any());
    }
}
```

### 2.4.2 Integration Testing
```java
@SpringBootTest
@TestPropertySource(properties = {
    "spring.datasource.url=jdbc:h2:mem:testdb",
    "spring.jpa.hibernate.ddl-auto=create-drop"
})
class GameServiceIntegrationTest {
    
    @Autowired
    private GameService gameService;
    
    @Autowired
    private GameRepository gameRepository;
    
    @MockBean // Mock solo il servizio email
    private EmailService emailService;
    
    @Test
    @Transactional
    void shouldCreateAndRetrieveGame() {
        // Given
        Game game = Game.builder()
            .title("Integration Test Game")
            .price(BigDecimal.valueOf(49.99))
            .releaseDate(LocalDate.now())
            .build();
        
        // When
        Game savedGame = gameService.createGame(game);
        Optional<Game> retrievedGame = gameService.findById(savedGame.getId());
        
        // Then
        assertThat(retrievedGame).isPresent();
        assertThat(retrievedGame.get().getTitle()).isEqualTo("Integration Test Game");
        
        verify(emailService).sendNotification(contains("Integration Test Game"));
    }
}
```

### 2.4.3 Test Configuration
```java
@TestConfiguration
public class TestConfig {
    
    @Bean
    @Primary
    public EmailService mockEmailService() {
        return Mockito.mock(EmailService.class);
    }
    
    @Bean
    @Primary
    public PasswordEncoder testPasswordEncoder() {
        // Encoder veloce per i test
        return new BCryptPasswordEncoder(4);
    }
    
    @Bean
    public Clock testClock() {
        // Clock fisso per test deterministici
        return Clock.fixed(
            Instant.parse("2024-01-01T00:00:00Z"), 
            ZoneOffset.UTC
        );
    }
}
```

## 2.5 Best Practices

### 2.5.1 Principi Generali

1. **Preferire Constructor Injection**
   - Garantisce immutabilità
   - Fail-fast behavior
   - Migliore testabilità

2. **Evitare Dipendenze Circolari**
```java
// PROBLEMATICO - Dipendenza circolare
@Service
public class GameService {
    private final UserService userService;
    
    public GameService(UserService userService) {
        this.userService = userService;
    }
}

@Service
public class UserService {
    private final GameService gameService; // CIRCOLARE!
    
    public UserService(GameService gameService) {
        this.gameService = gameService;
    }
}

// SOLUZIONE - Introdurre un servizio intermedio
@Service
public class UserGameService {
    private final GameRepository gameRepository;
    private final UserRepository userRepository;
    
    // Logica che coinvolge entrambi
}
```

3. **Utilizzare Interfacce per le Dipendenze**
```java
// BUONA PRATICA
@Service
public class GameService {
    private final GameRepository gameRepository; // Interfaccia
    private final NotificationService notificationService; // Interfaccia
    
    public GameService(GameRepository gameRepository,
                      NotificationService notificationService) {
        this.gameRepository = gameRepository;
        this.notificationService = notificationService;
    }
}
```

4. **Gestire le Dipendenze Opzionali**
```java
@Service
public class GameService {
    private final GameRepository gameRepository;
    private final Optional<EmailService> emailService;
    
    public GameService(GameRepository gameRepository,
                      @Autowired(required = false) EmailService emailService) {
        this.gameRepository = gameRepository;
        this.emailService = Optional.ofNullable(emailService);
    }
    
    public void createGame(Game game) {
        gameRepository.save(game);
        
        emailService.ifPresent(service -> 
            service.sendNotification("Nuovo gioco: " + game.getTitle())
        );
    }
}
```

5. **Configurazione Esplicita per Logica Complessa**
```java
@Configuration
public class GameConfig {
    
    @Bean
    public GameService gameService(
            GameRepository gameRepository,
            @Qualifier("async") NotificationService notificationService,
            GameProperties properties) {
        
        GameService service = new GameService(
            gameRepository, 
            notificationService
        );
        
        // Configurazione complessa
        service.setMaxGamesPerUser(properties.getMaxGamesPerUser());
        service.setValidationRules(createValidationRules(properties));
        
        return service;
    }
    
    private List<ValidationRule> createValidationRules(GameProperties properties) {
        // Logica di creazione delle regole
        return Arrays.asList(
            new PriceValidationRule(properties.getMinPrice(), properties.getMaxPrice()),
            new TitleValidationRule(properties.getMinTitleLength()),
            new ReleaseDateValidationRule()
        );
    }
}
```

L'Inversion of Control e la Dependency Injection sono principi fondamentali per creare applicazioni modulari, testabili e manutenibili. Spring Framework fornisce un potente container IoC che semplifica notevolmente l'implementazione di questi pattern, permettendo agli sviluppatori di concentrarsi sulla logica di business piuttosto che sulla gestione delle dipendenze.