# 2. Architettura del Sistema

## 2.1 Panoramica Architetturale

### 2.1.1 Principi Architetturali

L'architettura del sistema segue i seguenti principi fondamentali:

- **Separation of Concerns**: Ogni componente ha una responsabilità specifica
- **Dependency Inversion**: Le dipendenze puntano verso astrazioni
- **Single Responsibility**: Ogni classe ha un unico motivo per cambiare
- **Open/Closed Principle**: Aperto per estensioni, chiuso per modifiche
- **DRY (Don't Repeat Yourself)**: Evitare duplicazione di codice
- **KISS (Keep It Simple, Stupid)**: Preferire soluzioni semplici

### 2.1.2 Pattern Architetturali Utilizzati

#### Model-View-Controller (MVC)
- **Model**: Entità JPA, DTO, Business Logic
- **View**: Template Thymeleaf, JSON Response
- **Controller**: REST Controller, Web Controller

#### Layered Architecture
```
┌─────────────────────────────────────┐
│           Presentation Layer        │  ← Controllers, Views
├─────────────────────────────────────┤
│            Service Layer            │  ← Business Logic
├─────────────────────────────────────┤
│          Repository Layer           │  ← Data Access
├─────────────────────────────────────┤
│            Domain Layer             │  ← Entities, DTOs
└─────────────────────────────────────┘
```

#### Dependency Injection (IoC)
- Gestione automatica delle dipendenze tramite Spring Container
- Configurazione tramite annotazioni e Java Config
- Lifecycle management dei bean

## 2.2 Architettura Fisica

### 2.2.1 Deployment Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Load Balancer │    │   Web Server    │    │    Database     │
│    (Nginx)      │◄──►│  (Spring Boot)  │◄──►│    (MySQL)      │
│                 │    │                 │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Static Files  │    │   Application   │    │   Data Storage  │
│   (CSS, JS,     │    │   Server        │    │   (Persistent)  │
│    Images)      │    │   (Embedded)    │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### 2.2.2 Componenti Infrastrutturali

#### Web Server
- **Tomcat Embedded**: Server applicativo integrato in Spring Boot
- **Configurazione**: application.properties per tuning performance
- **SSL/TLS**: Terminazione SSL a livello di load balancer

#### Database
- **MySQL 8.x**: Database relazionale principale
- **Connection Pooling**: HikariCP per gestione connessioni
- **Backup Strategy**: Backup incrementali giornalieri

#### Caching
- **Spring Cache**: Caching a livello applicativo
- **Redis** (opzionale): Cache distribuita per scalabilità

## 2.3 Architettura Logica

### 2.3.1 Package Structure

```
mc.videogiochi/
├── config/                 # Configurazioni Spring
│   ├── SecurityConfig.java
│   ├── DatabaseConfig.java
│   └── CacheConfig.java
├── controller/             # Presentation Layer
│   ├── web/               # Web Controllers (Thymeleaf)
│   └── api/               # REST Controllers
├── service/               # Business Logic Layer
│   ├── impl/              # Implementazioni servizi
│   └── interfaces/        # Interfacce servizi
├── repository/            # Data Access Layer
│   ├── jpa/              # Repository JPA
│   └── custom/           # Query personalizzate
├── domain/               # Domain Layer
│   ├── entity/           # Entità JPA
│   ├── dto/              # Data Transfer Objects
│   └── enums/            # Enumerazioni
├── security/             # Sicurezza
│   ├── authentication/   # Gestione autenticazione
│   └── authorization/    # Gestione autorizzazioni
├── util/                 # Utilities
│   ├── mapper/           # Mapping Entity ↔ DTO
│   ├── validator/        # Validatori custom
│   └── helper/           # Classi di supporto
└── exception/            # Gestione eccezioni
    ├── custom/           # Eccezioni personalizzate
    └── handler/          # Exception handlers
```

### 2.3.2 Layer Responsibilities

#### Presentation Layer (Controllers)
**Responsabilità**:
- Gestione richieste HTTP
- Validazione input
- Serializzazione/Deserializzazione
- Gestione sessioni e sicurezza

**Componenti**:
```java
@RestController
@RequestMapping("/api/games")
public class GameApiController {
    
    @Autowired
    private GameService gameService;
    
    @GetMapping
    public ResponseEntity<PagedResponse<GameDTO>> getGames(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "20") int size,
            @RequestParam(required = false) String search) {
        // Delegazione al service layer
    }
}
```

#### Service Layer (Business Logic)
**Responsabilità**:
- Implementazione logica di business
- Orchestrazione operazioni complesse
- Gestione transazioni
- Validazione business rules

**Componenti**:
```java
@Service
@Transactional
public class GameServiceImpl implements GameService {
    
    @Autowired
    private GameRepository gameRepository;
    
    @Autowired
    private GameMapper gameMapper;
    
    @Override
    public PagedResponse<GameDTO> searchGames(GameSearchCriteria criteria) {
        // Implementazione logica di ricerca
    }
}
```

#### Repository Layer (Data Access)
**Responsabilità**:
- Accesso ai dati
- Query personalizzate
- Gestione persistenza
- Ottimizzazioni database

**Componenti**:
```java
@Repository
public interface GameRepository extends JpaRepository<Game, Long>, 
                                       GameRepositoryCustom {
    
    @Query("SELECT g FROM Game g WHERE g.title LIKE %:title%")
    Page<Game> findByTitleContaining(@Param("title") String title, 
                                     Pageable pageable);
}
```

#### Domain Layer (Entities & DTOs)
**Responsabilità**:
- Definizione modello dati
- Regole di validazione
- Mapping relazioni
- Incapsulamento stato

## 2.4 Pattern di Design Implementati

### 2.4.1 Repository Pattern

**Scopo**: Incapsulare la logica di accesso ai dati

**Implementazione**:
```java
public interface GameRepository extends JpaRepository<Game, Long> {
    // Query methods derivate
    List<Game> findByGenreAndReleaseDateBetween(
        Genre genre, LocalDate start, LocalDate end);
    
    // Query personalizzate
    @Query("SELECT g FROM Game g JOIN g.userGameLists ugl " +
           "WHERE ugl.user.id = :userId AND ugl.listType = :listType")
    List<Game> findUserGamesByListType(
        @Param("userId") Long userId, 
        @Param("listType") ListType listType);
}
```

**Vantaggi**:
- Separazione logica business da accesso dati
- Testabilità migliorata
- Riusabilità query
- Astrazione dal tipo di persistenza

### 2.4.2 Data Transfer Object (DTO) Pattern

**Scopo**: Trasferimento dati tra layer senza esporre entità interne

**Implementazione**:
```java
public class GameDTO {
    private Long id;
    private String title;
    private String description;
    private LocalDate releaseDate;
    private List<String> genres;
    private DeveloperDTO developer;
    private Double averageRating;
    
    // Costruttori, getter, setter
}

public class GameSummaryDTO {
    private Long id;
    private String title;
    private String coverImageUrl;
    private Double averageRating;
    
    // Versione ridotta per liste
}
```

**Vantaggi**:
- Controllo su dati esposti
- Versioning API facilitato
- Performance ottimizzate
- Sicurezza migliorata

### 2.4.3 Service Layer Pattern

**Scopo**: Incapsulare logica di business e orchestrare operazioni

**Implementazione**:
```java
@Service
@Transactional
public class UserGameListService {
    
    @Autowired
    private UserGameListRepository userGameListRepository;
    
    @Autowired
    private GameRepository gameRepository;
    
    @Autowired
    private NotificationService notificationService;
    
    public void addGameToUserList(Long userId, Long gameId, 
                                 ListType listType, Integer rating) {
        // 1. Validazione business rules
        validateGameNotAlreadyInList(userId, gameId, listType);
        
        // 2. Creazione entità
        UserGameList userGameList = new UserGameList();
        userGameList.setUserId(userId);
        userGameList.setGameId(gameId);
        userGameList.setListType(listType);
        userGameList.setRating(rating);
        
        // 3. Persistenza
        userGameListRepository.save(userGameList);
        
        // 4. Operazioni collaterali
        if (listType == ListType.COMPLETED) {
            updateUserStatistics(userId);
            notificationService.sendCompletionNotification(userId, gameId);
        }
    }
}
```

### 2.4.4 Factory Pattern

**Scopo**: Creazione oggetti complessi con logica centralizzata

**Implementazione**:
```java
@Component
public class SearchCriteriaFactory {
    
    public GameSearchCriteria createFromRequest(GameSearchRequest request) {
        GameSearchCriteria criteria = new GameSearchCriteria();
        
        if (StringUtils.hasText(request.getTitle())) {
            criteria.setTitleFilter(request.getTitle().trim().toLowerCase());
        }
        
        if (request.getGenres() != null && !request.getGenres().isEmpty()) {
            criteria.setGenreFilter(request.getGenres());
        }
        
        if (request.getYearFrom() != null || request.getYearTo() != null) {
            criteria.setYearRangeFilter(
                request.getYearFrom(), request.getYearTo());
        }
        
        return criteria;
    }
}
```

### 2.4.5 Strategy Pattern

**Scopo**: Algoritmi intercambiabili per diverse strategie di business

**Implementazione**:
```java
public interface RecommendationStrategy {
    List<GameDTO> recommend(Long userId, int limit);
}

@Component
public class SimilarGamesStrategy implements RecommendationStrategy {
    @Override
    public List<GameDTO> recommend(Long userId, int limit) {
        // Logica basata su giochi simili
    }
}

@Component
public class CollaborativeFilteringStrategy implements RecommendationStrategy {
    @Override
    public List<GameDTO> recommend(Long userId, int limit) {
        // Logica collaborative filtering
    }
}

@Service
public class RecommendationService {
    
    @Autowired
    private Map<String, RecommendationStrategy> strategies;
    
    public List<GameDTO> getRecommendations(Long userId, String strategyName) {
        RecommendationStrategy strategy = strategies.get(strategyName);
        return strategy.recommend(userId, 10);
    }
}
```

## 2.5 Gestione delle Dipendenze

### 2.5.1 Dependency Injection con Spring

**Configurazione tramite Annotazioni**:
```java
@Service
public class GameService {
    
    private final GameRepository gameRepository;
    private final GameMapper gameMapper;
    private final CacheManager cacheManager;
    
    // Constructor injection (raccomandato)
    public GameService(GameRepository gameRepository,
                      GameMapper gameMapper,
                      CacheManager cacheManager) {
        this.gameRepository = gameRepository;
        this.gameMapper = gameMapper;
        this.cacheManager = cacheManager;
    }
}
```

**Configurazione Java Config**:
```java
@Configuration
public class ServiceConfig {
    
    @Bean
    @ConditionalOnProperty(name = "app.caching.enabled", havingValue = "true")
    public CacheManager cacheManager() {
        return new ConcurrentMapCacheManager("games", "users");
    }
    
    @Bean
    @Profile("development")
    public DataSource developmentDataSource() {
        // Configurazione datasource per sviluppo
    }
}
```

### 2.5.2 Gestione Scope e Lifecycle

**Bean Scopes**:
- **Singleton**: Default, una istanza per container
- **Prototype**: Nuova istanza ad ogni richiesta
- **Request**: Una istanza per richiesta HTTP
- **Session**: Una istanza per sessione HTTP

**Lifecycle Callbacks**:
```java
@Component
public class DatabaseInitializer {
    
    @PostConstruct
    public void initialize() {
        // Inizializzazione dopo creazione bean
    }
    
    @PreDestroy
    public void cleanup() {
        // Pulizia prima distruzione bean
    }
}
```

## 2.6 Gestione Configurazioni

### 2.6.1 Profili di Ambiente

**application.properties** (comune):
```properties
# Configurazioni comuni
spring.application.name=videogiochi
spring.jpa.hibernate.ddl-auto=validate
spring.jpa.show-sql=false
```

**application-development.properties**:
```properties
# Configurazioni sviluppo
spring.datasource.url=jdbc:mysql://localhost:3306/videogiochi_dev
spring.jpa.show-sql=true
logging.level.mc.videogiochi=DEBUG
```

**application-production.properties**:
```properties
# Configurazioni produzione
spring.datasource.url=${DATABASE_URL}
spring.jpa.show-sql=false
logging.level.root=WARN
```

### 2.6.2 Configuration Properties

```java
@ConfigurationProperties(prefix = "app")
@Component
public class ApplicationProperties {
    
    private Security security = new Security();
    private Cache cache = new Cache();
    private Upload upload = new Upload();
    
    public static class Security {
        private int sessionTimeoutMinutes = 30;
        private int maxLoginAttempts = 5;
        // getter, setter
    }
    
    public static class Cache {
        private boolean enabled = true;
        private int ttlMinutes = 60;
        // getter, setter
    }
}
```

## 2.7 Gestione Errori e Logging

### 2.7.1 Exception Handling

**Global Exception Handler**:
```java
@ControllerAdvice
public class GlobalExceptionHandler {
    
    private static final Logger logger = LoggerFactory.getLogger(GlobalExceptionHandler.class);
    
    @ExceptionHandler(EntityNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleEntityNotFound(EntityNotFoundException ex) {
        logger.warn("Entity not found: {}", ex.getMessage());
        
        ErrorResponse error = new ErrorResponse(
            "ENTITY_NOT_FOUND", 
            ex.getMessage(),
            System.currentTimeMillis()
        );
        
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }
    
    @ExceptionHandler(ValidationException.class)
    public ResponseEntity<ErrorResponse> handleValidation(ValidationException ex) {
        logger.warn("Validation error: {}", ex.getMessage());
        
        ErrorResponse error = new ErrorResponse(
            "VALIDATION_ERROR", 
            ex.getMessage(),
            System.currentTimeMillis()
        );
        
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(error);
    }
}
```

### 2.7.2 Logging Strategy

**Configurazione Logback**:
```xml
<configuration>
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/videogiochi.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>logs/videogiochi.%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    
    <logger name="mc.videogiochi" level="DEBUG"/>
    <logger name="org.springframework.security" level="DEBUG"/>
    
    <root level="INFO">
        <appender-ref ref="STDOUT"/>
        <appender-ref ref="FILE"/>
    </root>
</configuration>
```

## 2.8 Performance e Scalabilità

### 2.8.1 Caching Strategy

**Configurazione Cache**:
```java
@Configuration
@EnableCaching
public class CacheConfig {
    
    @Bean
    public CacheManager cacheManager() {
        ConcurrentMapCacheManager cacheManager = new ConcurrentMapCacheManager();
        cacheManager.setCacheNames(Arrays.asList("games", "users", "statistics"));
        return cacheManager;
    }
}

@Service
public class GameService {
    
    @Cacheable(value = "games", key = "#id")
    public GameDTO getGameById(Long id) {
        // Metodo cachato
    }
    
    @CacheEvict(value = "games", key = "#gameDTO.id")
    public GameDTO updateGame(GameDTO gameDTO) {
        // Invalidazione cache
    }
}
```

### 2.8.2 Database Optimization

**Connection Pooling**:
```properties
# HikariCP configuration
spring.datasource.hikari.maximum-pool-size=20
spring.datasource.hikari.minimum-idle=5
spring.datasource.hikari.idle-timeout=300000
spring.datasource.hikari.max-lifetime=600000
```

**Query Optimization**:
```java
@Entity
@Table(name = "games")
@NamedEntityGraph(
    name = "Game.withDeveloper",
    attributeNodes = @NamedAttributeNode("developer")
)
public class Game {
    // Definizione entità con entity graph per evitare N+1
}

@Repository
public interface GameRepository extends JpaRepository<Game, Long> {
    
    @EntityGraph("Game.withDeveloper")
    @Query("SELECT g FROM Game g WHERE g.genre = :genre")
    List<Game> findByGenreWithDeveloper(@Param("genre") Genre genre);
}
```

## 2.9 Sicurezza Architetturale

### 2.9.1 Security Layers

```
┌─────────────────────────────────────┐
│        Network Security             │  ← HTTPS, Firewall
├─────────────────────────────────────┤
│      Application Security           │  ← Authentication, Authorization
├─────────────────────────────────────┤
│        Data Security                │  ← Encryption, Validation
└─────────────────────────────────────┘
```

### 2.9.2 Security Configuration

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(authz -> authz
                .requestMatchers("/api/public/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            )
            .csrf(csrf -> csrf.disable())
            .headers(headers -> headers
                .frameOptions().deny()
                .contentTypeOptions().and()
                .httpStrictTransportSecurity(hstsConfig -> hstsConfig
                    .maxAgeInSeconds(31536000)
                    .includeSubdomains(true)
                )
            );
        
        return http.build();
    }
}
```

## 2.10 Testing Architecture

### 2.10.1 Test Pyramid

```
        ┌─────────────┐
        │     E2E     │  ← Integration Tests
        │    Tests    │
        └─────────────┘
      ┌─────────────────┐
      │  Integration    │  ← Service Layer Tests
      │     Tests       │
      └─────────────────┘
    ┌───────────────────────┐
    │     Unit Tests        │  ← Repository, Utility Tests
    └───────────────────────┘
```

### 2.10.2 Test Configuration

```java
@SpringBootTest
@TestPropertySource(locations = "classpath:application-test.properties")
@Transactional
public class GameServiceIntegrationTest {
    
    @Autowired
    private GameService gameService;
    
    @MockBean
    private NotificationService notificationService;
    
    @Test
    public void shouldCreateGameSuccessfully() {
        // Test di integrazione
    }
}
```

Questa architettura fornisce una base solida per lo sviluppo di un'applicazione scalabile, manutenibile e sicura, implementando le best practices e i pattern più consolidati nel mondo Spring Boot.