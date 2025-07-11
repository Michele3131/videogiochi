# 4. Implementazione Backend

> **Riferimenti teorici**: Questo capitolo implementa i pattern descritti in [Pattern Architetturali](../knowledge/01-pattern-architetturali.md) e [DTO Mapping Patterns](../knowledge/03-dto-mapping-patterns.md)

## 4.1 Setup Iniziale del Progetto

### 4.1.1 Configurazione Maven: Fondamenta del Progetto

> **Concetto chiave**: Maven gestisce automaticamente le dipendenze e la compilazione del progetto, semplificando notevolmente lo sviluppo.

**Perché iniziamo con Maven?**
Maven è il nostro sistema di gestione delle dipendenze e build automation. La configurazione corretta del `pom.xml` è cruciale perché definisce:
- Le dipendenze del progetto e le loro versioni
- I plugin necessari per la compilazione e il packaging
- Le proprietà di configurazione globali

**Dove creare il file:** Il file `pom.xml` deve essere posizionato nella **root directory** del progetto (stesso livello della cartella `src/`).

**Struttura del pom.xml spiegata sezione per sezione:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" 
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
        <relativePath/>
    </parent>
    
    <groupId>mc</groupId>
    <artifactId>videogiochi</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>videogiochi</name>
    <description>Libreria Videogiochi - Sistema di gestione collezioni</description>
    
    <!-- SEZIONE 1: Proprietà del Progetto -->
    <!-- Queste proprietà centralizzano le versioni delle dipendenze per evitare duplicazioni -->
    <properties>
        <java.version>21</java.version> <!-- Versione Java: usiamo la 21 per le feature moderne -->
        <mapstruct.version>1.5.5.Final</mapstruct.version> <!-- Per il mapping automatico DTO-Entity -->
        <springdoc.version>2.2.0</springdoc.version> <!-- Per la documentazione API automatica -->
        <testcontainers.version>1.19.1</testcontainers.version> <!-- Per i test di integrazione -->
    </properties>
    
    <dependencies>
        <!-- SEZIONE 2: Spring Boot Starters -->
        <!-- Gli starter sono collezioni pre-configurate di dipendenze per funzionalità specifiche -->
        <!-- Web MVC: Fornisce tutto il necessario per creare API REST -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <!-- Include: Spring MVC, Tomcat embedded, Jackson per JSON, Validation -->
        </dependency>
        
        <!-- JPA/Hibernate: Per l'accesso ai dati e ORM -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
            <!-- Include: Hibernate, Spring Data JPA, Connection pooling -->
        </dependency>
        
        <!-- Security: Autenticazione, autorizzazione, protezione CSRF -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
            <!-- Include: Spring Security Core, Web, Config -->
        </dependency>
        
        <!-- Validation: Validazione automatica dei dati in input -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
            <!-- Include: Hibernate Validator, Bean Validation API -->
        </dependency>
        
        <!-- Cache: Sistema di caching per migliorare le performance -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-cache</artifactId>
            <!-- Include: Spring Cache abstraction, supporto per vari provider -->
        </dependency>
        
        <!-- Thymeleaf: Template engine per le pagine web (opzionale se usi solo API) -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
            <!-- Include: Thymeleaf engine, integrazione Spring -->
        </dependency>
        
        <!-- SEZIONE 3: Database e Persistenza -->
        <!-- Driver MySQL: Connessione al database -->
        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
            <scope>runtime</scope> <!-- Solo a runtime, non serve per la compilazione -->
            <!-- Fornisce il driver JDBC per MySQL 8+ -->
        </dependency>
        
        <!-- Flyway Core: Gestione delle migrazioni del database -->
        <dependency>
            <groupId>org.flywaydb</groupId>
            <artifactId>flyway-core</artifactId>
            <!-- Permette di versionare e applicare modifiche al DB automaticamente -->
        </dependency>
        
        <!-- Flyway MySQL: Supporto specifico per MySQL -->
        <dependency>
            <groupId>org.flywaydb</groupId>
            <artifactId>flyway-mysql</artifactId>
            <!-- Estensioni specifiche per MySQL (sintassi, tipi di dato) -->
        </dependency>
        
        <!-- SEZIONE 4: Librerie di Utilità -->
        <!-- Lombok: Riduce il boilerplate code (getter, setter, costruttori) -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional> <!-- Non transitiva: ogni modulo deve dichiararla -->
            <!-- Genera automaticamente metodi comuni tramite annotazioni -->
        </dependency>
        
        <!-- MapStruct: Mapping automatico tra oggetti (Entity ↔ DTO) -->
        <dependency>
            <groupId>org.mapstruct</groupId>
            <artifactId>mapstruct</artifactId>
            <version>${mapstruct.version}</version>
            <!-- Genera mapper type-safe a compile-time -->
        </dependency>
        
        <!-- SEZIONE 5: Documentazione API -->
        <!-- SpringDoc OpenAPI: Genera automaticamente documentazione Swagger -->
        <dependency>
            <groupId>org.springdoc</groupId>
            <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
            <version>${springdoc.version}</version>
            <!-- Crea interfaccia web per testare le API e documentazione JSON/YAML -->
        </dependency>
        
        <!-- SEZIONE 6: Testing -->
        <!-- Spring Boot Test: Suite completa per test di integrazione -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope> <!-- Solo per i test, non inclusa nel JAR finale -->
            <!-- Include: JUnit 5, Mockito, AssertJ, Hamcrest, Spring Test -->
        </dependency>
        
        <!-- Spring Security Test: Utilità per testare la sicurezza -->
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-test</artifactId>
            <scope>test</scope>
            <!-- Fornisce @WithMockUser, @WithUserDetails per test autenticati -->
        </dependency>
        
        <!-- Testcontainers JUnit: Integrazione con JUnit 5 -->
        <dependency>
            <groupId>org.testcontainers</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>${testcontainers.version}</version>
            <scope>test</scope>
            <!-- Gestisce il ciclo di vita dei container nei test -->
        </dependency>
        
        <!-- Testcontainers MySQL: Container MySQL per test di integrazione -->
        <dependency>
            <groupId>org.testcontainers</groupId>
            <artifactId>mysql</artifactId>
            <version>${testcontainers.version}</version>
            <scope>test</scope>
            <!-- Avvia automaticamente un'istanza MySQL isolata per ogni test -->
        </dependency>
    </dependencies>
    
    <!-- SEZIONE 7: Build Configuration -->
    <!-- I plugin definiscono come compilare, processare e packagere l'applicazione -->
    <build>
        <plugins>
            <!-- Maven Compiler: Configura la compilazione Java -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <!-- Annotation Processors: Elaborano annotazioni a compile-time -->
                    <annotationProcessorPaths>
                        <!-- Lombok: Genera codice da annotazioni (@Getter, @Setter, etc.) -->
                        <path>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                            <version>${lombok.version}</version>
                        </path>
                        <!-- MapStruct: Genera implementazioni dei mapper -->
                        <path>
                            <groupId>org.mapstruct</groupId>
                            <artifactId>mapstruct-processor</artifactId>
                            <version>${mapstruct.version}</version>
                        </path>
                    </annotationProcessorPaths>
                </configuration>
            </plugin>
            
            <!-- Spring Boot Plugin: Crea JAR eseguibile con server embedded -->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <!-- Esclude Lombok dal JAR finale (serve solo a compile-time) -->
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
            
            <!-- Flyway Plugin: Gestisce migrazioni database da Maven -->
            <plugin>
                <groupId>org.flywaydb</groupId>
                <artifactId>flyway-maven-plugin</artifactId>
                <configuration>
                    <!-- Configurazione connessione DB per comandi Maven -->
                    <url>jdbc:mysql://localhost:3306/videogiochi</url>
                    <user>videogiochi_app</user>
                    <password>secure_password</password>
                    <!-- Permette comandi: mvn flyway:migrate, mvn flyway:info, etc. -->
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

### 4.1.2 Configurazione Application Properties: Il Cuore dell'Applicazione

> **Suggerimento per principianti**: Il file `application.yml` è come il "pannello di controllo" della tua applicazione - qui configuri database, sicurezza, logging e molto altro.

**Perché usiamo application.yml invece di application.properties?**
YAML offre una sintassi più leggibile e strutturata, ideale per configurazioni complesse con gerarchie annidate.

**Dove posizionare il file:** Il file `application.yml` deve essere inserito in `src/main/resources/`.

**Struttura delle configurazioni spiegata sezione per sezione:**
```yaml
# SEZIONE 1: Configurazione Base dell'Applicazione
spring:
  application:
    name: videogiochi  # Nome dell'applicazione per logging e monitoring
  
  profiles:
    active: development  # Profilo attivo (development, test, production)
    # I profili permettono configurazioni diverse per ambienti diversi
  
  # SEZIONE 2: Configurazione Database
  datasource:
    # URL con parametri di sicurezza e timezone
    url: jdbc:mysql://localhost:3306/videogiochi?useSSL=true&serverTimezone=UTC
    # Variabili d'ambiente con fallback per sicurezza
    username: ${DB_USERNAME:videogiochi_app}  # Usa env var o default
    password: ${DB_PASSWORD:secure_password}   # Usa env var o default
    driver-class-name: com.mysql.cj.jdbc.Driver  # Driver MySQL 8+
    
    # HikariCP: Pool di connessioni ad alte performance
    hikari:
      maximum-pool-size: 20      # Max connessioni simultanee
      minimum-idle: 5            # Connessioni sempre aperte
      idle-timeout: 300000       # Timeout connessioni inattive (5 min)
      max-lifetime: 600000       # Vita massima connessione (10 min)
      connection-test-query: SELECT 1  # Query per testare connessioni
  
  # SEZIONE 3: Configurazione JPA/Hibernate
  jpa:
    hibernate:
      ddl-auto: validate  # Non modifica schema, solo validazione (sicuro per prod)
    show-sql: false       # Non mostra SQL in console (attivare solo per debug)
    properties:
      hibernate:
        dialect: org.hibernate.dialect.MySQL8Dialect  # Ottimizzazioni MySQL 8
        format_sql: true          # Formatta SQL nei log per leggibilità
        use_sql_comments: true    # Aggiunge commenti SQL per debug
        jdbc:
          batch_size: 25          # Raggruppa operazioni per performance
        order_inserts: true       # Ottimizza ordine INSERT
        order_updates: true       # Ottimizza ordine UPDATE
  
  # SEZIONE 4: Configurazione Flyway (Migrazioni Database)
  flyway:
    enabled: true                        # Abilita migrazioni automatiche
    locations: classpath:db/migration    # Cartella script SQL
    baseline-on-migrate: true            # Crea baseline se DB non vuoto
  
  # SEZIONE 5: Configurazione Cache
  cache:
    type: simple              # Cache in memoria (per sviluppo)
    cache-names:              # Nomi cache predefinite
      - games                 # Cache per giochi
      - users                 # Cache per utenti
      - statistics            # Cache per statistiche
      - genres                # Cache per generi
      - platforms             # Cache per piattaforme
  
  # SEZIONE 6: Configurazione Security (Base)
  security:
    user:                     # Utente di default per sviluppo
      name: admin
      password: admin123      # CAMBIARE in produzione!
      roles: ADMIN            # Ruolo amministratore

# SEZIONE 7: Configurazione Server
server:
  port: 8080                    # Porta dell'applicazione
  servlet:
    context-path: /api          # Prefisso per tutte le API (/api/users, /api/games, etc.)
  compression:
    enabled: true               # Compressione HTTP per ridurre traffico
    # Tipi MIME da comprimere per ottimizzare le performance
    mime-types: text/html,text/xml,text/plain,text/css,text/javascript,application/javascript,application/json
  
# SEZIONE 8: Configurazione Logging
logging:
  level:                        # Livelli di log per package specifici
    mc.videogiochi: INFO        # Log applicazione: solo INFO e superiori
    org.springframework.security: DEBUG  # Debug sicurezza per sviluppo
    org.hibernate.SQL: DEBUG    # Mostra query SQL eseguite
    org.hibernate.type.descriptor.sql.BasicBinder: TRACE  # Mostra parametri query
  pattern:
    # Formato log console: timestamp - messaggio
    console: "%d{yyyy-MM-dd HH:mm:ss} - %msg%n"
    # Formato log file: timestamp [thread] livello logger - messaggio
    file: "%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n"
  file:
    name: logs/videogiochi.log  # File di log (cartella logs/ creata automaticamente)

# SEZIONE 9: Configurazioni Custom dell'Applicazione
app:
  security:
    jwt:                        # Configurazione JSON Web Token
      secret: ${JWT_SECRET:mySecretKey}  # Chiave segreta (usa variabile ambiente!)
      expiration: 86400000      # Scadenza token: 24 ore in millisecondi
    session:
      timeout-minutes: 30       # Timeout sessione utente
      max-login-attempts: 5     # Tentativi login prima del blocco
  
  upload:                       # Configurazione upload file
    max-file-size: 10MB         # Dimensione massima singolo file
    max-request-size: 10MB      # Dimensione massima richiesta HTTP
    upload-dir: uploads/        # Cartella di destinazione upload
  
  cache:                        # Configurazioni cache custom
    enabled: true               # Abilita caching applicazione
    ttl-minutes: 60             # Time-to-live cache: 1 ora
  
  pagination:                   # Configurazioni paginazione
    default-page-size: 20       # Elementi per pagina di default
    max-page-size: 100          # Massimo elementi per pagina

# SEZIONE 10: Configurazione Actuator (Monitoring)
management:
  endpoints:
    web:
      exposure:
        # Endpoint esposti per monitoring (health check, metriche, etc.)
        include: health,info,metrics,prometheus
  endpoint:
    health:
      show-details: when-authorized  # Dettagli health solo se autorizzato
```

## 4.2 Layer Domain - Entità JPA: La Struttura dei Dati

> **Pattern implementato**: Questo layer implementa il **Domain Model** del pattern MVC. Vedi [Pattern Architetturali](../knowledge/01-pattern-architetturali.md) per i dettagli teorici.

**Perché iniziamo con le entità?**
Le entità JPA rappresentano la struttura dei nostri dati e definiscono come l'applicazione interagisce con il database. Sono il cuore del Domain Layer.

**Dove posizionare le entità:** Tutte le entità vanno nel package `src/main/java/mc/videogiochi/domain/entity/`.

### 4.2.1 Entità Base: Fondamenta Comuni

> **Best Practice**: La BaseEntity implementa il principio DRY (Don't Repeat Yourself) - tutti i campi comuni sono definiti una sola volta.

**Perché creare una BaseEntity?**
Evita duplicazione di codice per campi comuni (ID, timestamp, versioning) e garantisce consistenza tra tutte le entità.

**BaseEntity.java** - Posizionare in `src/main/java/mc/videogiochi/domain/entity/BaseEntity.java`:
```java
package mc.videogiochi.domain.entity;

// Import JPA per persistenza
import jakarta.persistence.*;
// Import Lombok per ridurre boilerplate
import lombok.Getter;
import lombok.Setter;
// Import Spring Data per auditing automatico
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.annotation.LastModifiedDate;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;

import java.time.LocalDateTime;

// @MappedSuperclass: Non crea tabella, ma eredita campi alle entità figlie
@MappedSuperclass
// @EntityListeners: Abilita auditing automatico (createdAt, updatedAt)
@EntityListeners(AuditingEntityListener.class)
// Lombok: genera automaticamente getter e setter
@Getter
@Setter
public abstract class BaseEntity {
    
    // Chiave primaria auto-incrementale
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY) // MySQL AUTO_INCREMENT
    private Long id;
    
    // Timestamp creazione: impostato automaticamente alla creazione
    @CreatedDate
    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt;
    
    // Timestamp ultima modifica: aggiornato automaticamente
    @LastModifiedDate
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;
    
    // Versioning per Optimistic Locking (prevenzione conflitti concorrenti)
    @Version
    private Long version;
}
```

### 4.2.2 Entità Principali: Modellazione del Dominio

**User.java** - Posizionare in `src/main/java/mc/videogiochi/domain/entity/User.java`:

> **Integrazione con Spring Security**: User implementa UserDetails per integrarsi seamlessly con il sistema di autenticazione di Spring.

**Perché User implementa UserDetails?**
Spring Security richiede questa interfaccia per l'autenticazione. Integriamo direttamente la logica di sicurezza nell'entità per semplicità.

**Spiegazione delle annotazioni e campi:**
```java
package mc.videogiochi.domain.entity;

// Import per JPA e validazione
import jakarta.persistence.*;
import jakarta.validation.constraints.*;
// Import Lombok per ridurre boilerplate code
import lombok.*;
// Import Spring Security per autenticazione
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;

import java.time.LocalDate;
import java.time.LocalDateTime;
import java.util.*;

/**
 * Entità User: rappresenta un utente del sistema
 * - Estende BaseEntity per ereditare campi comuni (id, timestamps, version)
 * - Implementa UserDetails per integrazione con Spring Security
 * - Utilizza Lombok per ridurre il codice boilerplate
 */
@Entity // Marca la classe come entità JPA (mappa su tabella database)
@Table(name = "users") // Specifica il nome della tabella nel database
@Getter // Lombok: genera automaticamente tutti i metodi getter
@Setter // Lombok: genera automaticamente tutti i metodi setter
@NoArgsConstructor // Lombok: genera costruttore senza parametri
@AllArgsConstructor // Lombok: genera costruttore con tutti i parametri
@Builder // Lombok: genera pattern Builder per creazione oggetti
public class User extends BaseEntity implements UserDetails {
    
    // Campo username: identificativo univoco dell'utente
    @Column(name = "username", unique = true, nullable = false, length = 50)
    @NotBlank(message = "Username è obbligatorio")  // Validazione: campo non può essere vuoto
    @Size(min = 3, max = 50, message = "Username deve essere tra 3 e 50 caratteri") // Validazione lunghezza
    private String username;
    
    // Campo email: per comunicazioni e login alternativo
    @Column(name = "email", unique = true, nullable = false, length = 100)
    @NotBlank(message = "Email è obbligatoria") // Validazione: campo obbligatorio
    @Email(message = "Email non valida")  // Validazione: formato email corretto
    private String email;
    
    // Password hashata (IMPORTANTE: mai salvare password in chiaro per sicurezza!)
    @Column(name = "password_hash", nullable = false)
    @NotBlank(message = "Password è obbligatoria")
    private String password;  // Sarà hashata dal service prima del salvataggio nel database
    
    // Dati personali opzionali dell'utente
    @Column(name = "first_name", length = 50)
    private String firstName; // Nome dell'utente
    
    @Column(name = "last_name", length = 50)
    private String lastName; // Cognome dell'utente
    
    @Column(name = "birth_date")
    private LocalDate birthDate; // Data di nascita per calcolare età
    
    @Column(name = "avatar_url", length = 500)
    private String avatarUrl; // URL dell'immagine profilo
    
    @Column(name = "bio", columnDefinition = "TEXT")
    private String bio; // Biografia/descrizione utente (testo lungo)
    
    // Campi di stato dell'account
    @Column(name = "is_active")
    @Builder.Default // Lombok: valore di default quando si usa il Builder
    private Boolean isActive = true; // Account attivo/disattivato
    
    @Column(name = "email_verified")
    @Builder.Default
    private Boolean emailVerified = false; // Email verificata tramite link
    
    @Column(name = "last_login")
    private LocalDateTime lastLogin; // Timestamp ultimo accesso
    
    // Relazione Many-to-Many con Role: un utente può avere più ruoli
    @ManyToMany(fetch = FetchType.EAGER) // EAGER: carica subito i ruoli (necessario per Spring Security)
    @JoinTable( // Tabella di join per la relazione many-to-many
        name = "user_roles", // Nome della tabella intermedia
        joinColumns = @JoinColumn(name = "user_id"), // Colonna che referenzia User
        inverseJoinColumns = @JoinColumn(name = "role_id") // Colonna che referenzia Role
    )
    @Builder.Default
    private Set<Role> roles = new HashSet<>(); // Set evita duplicati
    
    // Relazione One-to-Many con UserGameList: un utente ha molte liste di giochi
    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL, orphanRemoval = true)
    @Builder.Default
    private List<UserGameList> gameLists = new ArrayList<>();
    
    // Relazione One-to-Many con CustomList: un utente può creare liste personalizzate
    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL, orphanRemoval = true)
    @Builder.Default
    private List<CustomList> customLists = new ArrayList<>();
    
    // ========== IMPLEMENTAZIONE USERDETAILS (richiesta da Spring Security) ==========
    
    /**
     * Restituisce i ruoli dell'utente come GrantedAuthority
     * Spring Security usa questo metodo per l'autorizzazione
     */
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return roles.stream()
                .map(role -> new SimpleGrantedAuthority("ROLE_" + role.getName()))
                .toList();
    }
    
    @Override
    public boolean isAccountNonExpired() {
        return true; // Gli account non scadono mai in questo sistema
    }
    
    @Override
    public boolean isAccountNonLocked() {
        return isActive; // Account bloccato se non attivo
    }
    
    @Override
    public boolean isCredentialsNonExpired() {
        return true; // Le password non scadono mai in questo sistema
    }
    
    @Override
    public boolean isEnabled() {
        return isActive && emailVerified; // Account abilitato solo se attivo E email verificata
    }
    
    // ========== METODI HELPER (utility per semplificare operazioni comuni) ==========
    
    /**
     * Restituisce il nome completo dell'utente
     * Se nome e cognome non sono disponibili, usa lo username
     */
    public String getFullName() {
        if (firstName != null && lastName != null) {
            return firstName + " " + lastName;
        }
        return username; // Fallback su username se nome/cognome mancanti
    }
    
    /**
     * Aggiunge un ruolo all'utente
     * Gestisce automaticamente la relazione bidirezionale
     */
    public void addRole(Role role) {
        roles.add(role); // Aggiunge il ruolo a questo utente
        role.getUsers().add(this); // Aggiunge questo utente al ruolo (relazione bidirezionale)
    }
    
    /**
     * Rimuove un ruolo dall'utente
     * Gestisce automaticamente la relazione bidirezionale
     */
    public void removeRole(Role role) {
        roles.remove(role); // Rimuove il ruolo da questo utente
        role.getUsers().remove(this); // Rimuove questo utente dal ruolo (relazione bidirezionale)
    }
}
```

**Game.java**:
```java
package mc.videogiochi.domain.entity;

import jakarta.persistence.*;
import jakarta.validation.constraints.*;
import lombok.*;
import mc.videogiochi.domain.enums.EsrbRating;

import java.math.BigDecimal;
import java.time.LocalDate;
import java.util.*;

@Entity
@Table(name = "games")
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
@NamedEntityGraph(
    name = "Game.withDeveloperAndPublisher",
    attributeNodes = {
        @NamedAttributeNode("developer"),
        @NamedAttributeNode("publisher")
    }
)
public class Game extends BaseEntity {
    
    @Column(name = "title", nullable = false, length = 200)
    @NotBlank(message = "Titolo è obbligatorio")
    @Size(max = 200, message = "Titolo non può superare 200 caratteri")
    private String title;
    
    @Column(name = "description", columnDefinition = "TEXT")
    private String description;
    
    @Column(name = "release_date")
    private LocalDate releaseDate;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "developer_id")
    private Developer developer;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "publisher_id")
    private Publisher publisher;
    
    @Column(name = "cover_image_url", length = 500)
    private String coverImageUrl;
    
    @Column(name = "trailer_url", length = 500)
    private String trailerUrl;
    
    @Column(name = "average_rating", precision = 3, scale = 2)
    @DecimalMin(value = "0.00", message = "Rating deve essere >= 0")
    @DecimalMax(value = "10.00", message = "Rating deve essere <= 10")
    @Builder.Default
    private BigDecimal averageRating = BigDecimal.ZERO;
    
    @Column(name = "total_ratings")
    @Min(value = 0, message = "Numero rating deve essere >= 0")
    @Builder.Default
    private Integer totalRatings = 0;
    
    @Column(name = "metacritic_score")
    @Min(value = 0, message = "Metacritic score deve essere >= 0")
    @Max(value = 100, message = "Metacritic score deve essere <= 100")
    private Integer metacriticScore;
    
    @Enumerated(EnumType.STRING)
    @Column(name = "esrb_rating")
    private EsrbRating esrbRating;
    
    @Column(name = "price", precision = 8, scale = 2)
    @DecimalMin(value = "0.00", message = "Prezzo deve essere >= 0")
    private BigDecimal price;
    
    @Column(name = "is_active")
    @Builder.Default
    private Boolean isActive = true;
    
    // Relazioni Many-to-Many
    @ManyToMany
    @JoinTable(
        name = "game_genres",
        joinColumns = @JoinColumn(name = "game_id"),
        inverseJoinColumns = @JoinColumn(name = "genre_id")
    )
    @Builder.Default
    private Set<Genre> genres = new HashSet<>();
    
    @ManyToMany
    @JoinTable(
        name = "game_platforms",
        joinColumns = @JoinColumn(name = "game_id"),
        inverseJoinColumns = @JoinColumn(name = "platform_id")
    )
    @Builder.Default
    private Set<Platform> platforms = new HashSet<>();
    
    @OneToMany(mappedBy = "game", cascade = CascadeType.ALL, orphanRemoval = true)
    @Builder.Default
    private List<UserGameList> userGameLists = new ArrayList<>();
    
    // Campi denormalizzati per performance
    @Column(name = "total_in_wishlists")
    @Builder.Default
    private Integer totalInWishlists = 0;
    
    @Column(name = "total_completed")
    @Builder.Default
    private Integer totalCompleted = 0;
    
    @Column(name = "popularity_score", precision = 10, scale = 4)
    @Builder.Default
    private BigDecimal popularityScore = BigDecimal.ZERO;
    
    // Helper methods
    public void addGenre(Genre genre) {
        genres.add(genre);
        genre.getGames().add(this);
    }
    
    public void removeGenre(Genre genre) {
        genres.remove(genre);
        genre.getGames().remove(this);
    }
    
    public void addPlatform(Platform platform) {
        platforms.add(platform);
        platform.getGames().add(this);
    }
    
    public void removePlatform(Platform platform) {
        platforms.remove(platform);
        platform.getGames().remove(this);
    }
    
    public void updateRating(BigDecimal newRating) {
        if (totalRatings == 0) {
            averageRating = newRating;
            totalRatings = 1;
        } else {
            BigDecimal totalScore = averageRating.multiply(BigDecimal.valueOf(totalRatings));
            totalScore = totalScore.add(newRating);
            totalRatings++;
            averageRating = totalScore.divide(BigDecimal.valueOf(totalRatings), 2, BigDecimal.ROUND_HALF_UP);
        }
    }
}
```

### 4.2.3 Enumerazioni

**EsrbRating.java**:
```java
package mc.videogiochi.domain.enums;

public enum EsrbRating {
    E("Everyone"),
    E10_PLUS("Everyone 10+"),
    T("Teen"),
    M("Mature 17+"),
    AO("Adults Only 18+"),
    RP("Rating Pending");
    
    private final String description;
    
    EsrbRating(String description) {
        this.description = description;
    }
    
    public String getDescription() {
        return description;
    }
}
```

**ListType.java**:
```java
package mc.videogiochi.domain.enums;

public enum ListType {
    PLAYED("Giocati"),
    WISHLIST("Lista Desideri"),
    PLAYING("Sto Giocando"),
    DROPPED("Abbandonati"),
    CUSTOM("Lista Personalizzata");
    
    private final String displayName;
    
    ListType(String displayName) {
        this.displayName = displayName;
    }
    
    public String getDisplayName() {
        return displayName;
    }
}
```

## 4.3 Layer Repository - Accesso ai Dati

### 4.3.1 Repository Base

**BaseRepository.java**:
```java
package mc.videogiochi.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.JpaSpecificationExecutor;
import org.springframework.data.repository.NoRepositoryBean;

@NoRepositoryBean
public interface BaseRepository<T, ID> extends JpaRepository<T, ID>, JpaSpecificationExecutor<T> {
}
```

### 4.3.2 Repository Specifici

**GameRepository.java**:
```java
package mc.videogiochi.repository.jpa;

import mc.videogiochi.domain.entity.Game;
import mc.videogiochi.domain.enums.EsrbRating;
import mc.videogiochi.repository.BaseRepository;
import mc.videogiochi.repository.custom.GameRepositoryCustom;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.jpa.repository.EntityGraph;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;

import java.math.BigDecimal;
import java.time.LocalDate;
import java.util.List;
import java.util.Optional;

@Repository
public interface GameRepository extends BaseRepository<Game, Long>, GameRepositoryCustom {
    
    // Query methods derivate
    Page<Game> findByIsActiveTrue(Pageable pageable);
    
    Page<Game> findByTitleContainingIgnoreCaseAndIsActiveTrue(String title, Pageable pageable);
    
    List<Game> findByReleaseDateBetweenAndIsActiveTrue(LocalDate startDate, LocalDate endDate);
    
    List<Game> findByAverageRatingGreaterThanEqualAndIsActiveTrue(BigDecimal minRating);
    
    List<Game> findByEsrbRatingAndIsActiveTrue(EsrbRating esrbRating);
    
    // Query con EntityGraph per evitare N+1
    @EntityGraph("Game.withDeveloperAndPublisher")
    @Query("SELECT g FROM Game g WHERE g.isActive = true")
    Page<Game> findAllActiveWithDeveloperAndPublisher(Pageable pageable);
    
    @EntityGraph("Game.withDeveloperAndPublisher")
    Optional<Game> findByIdAndIsActiveTrue(Long id);
    
    // Query personalizzate
    @Query("SELECT g FROM Game g JOIN g.genres gen WHERE gen.id = :genreId AND g.isActive = true")
    Page<Game> findByGenreId(@Param("genreId") Long genreId, Pageable pageable);
    
    @Query("SELECT g FROM Game g JOIN g.platforms p WHERE p.id = :platformId AND g.isActive = true")
    Page<Game> findByPlatformId(@Param("platformId") Long platformId, Pageable pageable);
    
    @Query("SELECT g FROM Game g WHERE g.developer.id = :developerId AND g.isActive = true")
    Page<Game> findByDeveloperId(@Param("developerId") Long developerId, Pageable pageable);
    
    @Query("SELECT g FROM Game g WHERE g.publisher.id = :publisherId AND g.isActive = true")
    Page<Game> findByPublisherId(@Param("publisherId") Long publisherId, Pageable pageable);
    
    // Query per statistiche
    @Query("SELECT COUNT(g) FROM Game g WHERE g.isActive = true")
    long countActiveGames();
    
    @Query("SELECT AVG(g.averageRating) FROM Game g WHERE g.isActive = true AND g.totalRatings > 0")
    BigDecimal getOverallAverageRating();
    
    // Query per raccomandazioni
    @Query("SELECT g FROM Game g JOIN g.genres gen WHERE gen IN " +
           "(SELECT gen2 FROM Game g2 JOIN g2.genres gen2 WHERE g2.id = :gameId) " +
           "AND g.id != :gameId AND g.isActive = true " +
           "ORDER BY g.averageRating DESC, g.totalRatings DESC")
    List<Game> findSimilarGames(@Param("gameId") Long gameId, Pageable pageable);
    
    // Full-text search
    @Query(value = "SELECT * FROM games g WHERE MATCH(g.title, g.description) AGAINST (:searchTerm IN NATURAL LANGUAGE MODE) AND g.is_active = true", 
           nativeQuery = true)
    List<Game> fullTextSearch(@Param("searchTerm") String searchTerm);
}
```

**GameRepositoryCustom.java**:
```java
package mc.videogiochi.repository.custom;

import mc.videogiochi.domain.entity.Game;
import mc.videogiochi.dto.request.GameSearchCriteria;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;

public interface GameRepositoryCustom {
    
    Page<Game> findByCriteria(GameSearchCriteria criteria, Pageable pageable);
    
    Page<Game> findRecommendationsForUser(Long userId, Pageable pageable);
}
```

**GameRepositoryCustomImpl.java**:
```java
package mc.videogiochi.repository.custom;

import jakarta.persistence.EntityManager;
import jakarta.persistence.PersistenceContext;
import jakarta.persistence.TypedQuery;
import jakarta.persistence.criteria.*;
import mc.videogiochi.domain.entity.*;
import mc.videogiochi.domain.enums.ListType;
import mc.videogiochi.dto.request.GameSearchCriteria;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageImpl;
import org.springframework.data.domain.Pageable;
import org.springframework.stereotype.Repository;
import org.springframework.util.StringUtils;

import java.util.ArrayList;
import java.util.List;

@Repository
public class GameRepositoryCustomImpl implements GameRepositoryCustom {
    
    @PersistenceContext
    private EntityManager entityManager;
    
    @Override
    public Page<Game> findByCriteria(GameSearchCriteria criteria, Pageable pageable) {
        CriteriaBuilder cb = entityManager.getCriteriaBuilder();
        CriteriaQuery<Game> query = cb.createQuery(Game.class);
        Root<Game> root = query.from(Game.class);
        
        List<Predicate> predicates = new ArrayList<>();
        
        // Filtro base: solo giochi attivi
        predicates.add(cb.isTrue(root.get("isActive")));
        
        // Filtro per titolo
        if (StringUtils.hasText(criteria.getTitle())) {
            predicates.add(cb.like(
                cb.lower(root.get("title")), 
                "%" + criteria.getTitle().toLowerCase() + "%"
            ));
        }
        
        // Filtro per generi
        if (criteria.getGenreIds() != null && !criteria.getGenreIds().isEmpty()) {
            Join<Game, Genre> genreJoin = root.join("genres");
            predicates.add(genreJoin.get("id").in(criteria.getGenreIds()));
        }
        
        // Filtro per piattaforme
        if (criteria.getPlatformIds() != null && !criteria.getPlatformIds().isEmpty()) {
            Join<Game, Platform> platformJoin = root.join("platforms");
            predicates.add(platformJoin.get("id").in(criteria.getPlatformIds()));
        }
        
        // Filtro per sviluppatore
        if (criteria.getDeveloperId() != null) {
            predicates.add(cb.equal(root.get("developer").get("id"), criteria.getDeveloperId()));
        }
        
        // Filtro per publisher
        if (criteria.getPublisherId() != null) {
            predicates.add(cb.equal(root.get("publisher").get("id"), criteria.getPublisherId()));
        }
        
        // Filtro per anno di rilascio
        if (criteria.getReleaseYearFrom() != null) {
            predicates.add(cb.greaterThanOrEqualTo(
                cb.function("YEAR", Integer.class, root.get("releaseDate")),
                criteria.getReleaseYearFrom()
            ));
        }
        
        if (criteria.getReleaseYearTo() != null) {
            predicates.add(cb.lessThanOrEqualTo(
                cb.function("YEAR", Integer.class, root.get("releaseDate")),
                criteria.getReleaseYearTo()
            ));
        }
        
        // Filtro per rating
        if (criteria.getMinRating() != null) {
            predicates.add(cb.greaterThanOrEqualTo(root.get("averageRating"), criteria.getMinRating()));
        }
        
        if (criteria.getMaxRating() != null) {
            predicates.add(cb.lessThanOrEqualTo(root.get("averageRating"), criteria.getMaxRating()));
        }
        
        // Filtro per ESRB rating
        if (criteria.getEsrbRating() != null) {
            predicates.add(cb.equal(root.get("esrbRating"), criteria.getEsrbRating()));
        }
        
        query.where(predicates.toArray(new Predicate[0]));
        
        // Ordinamento
        if (pageable.getSort().isSorted()) {
            List<Order> orders = new ArrayList<>();
            pageable.getSort().forEach(sortOrder -> {
                if (sortOrder.isAscending()) {
                    orders.add(cb.asc(root.get(sortOrder.getProperty())));
                } else {
                    orders.add(cb.desc(root.get(sortOrder.getProperty())));
                }
            });
            query.orderBy(orders);
        } else {
            // Ordinamento di default per popolarità
            query.orderBy(
                cb.desc(root.get("popularityScore")),
                cb.desc(root.get("averageRating"))
            );
        }
        
        // Esecuzione query paginata
        TypedQuery<Game> typedQuery = entityManager.createQuery(query);
        typedQuery.setFirstResult((int) pageable.getOffset());
        typedQuery.setMaxResults(pageable.getPageSize());
        
        List<Game> games = typedQuery.getResultList();
        
        // Count query per paginazione
        CriteriaQuery<Long> countQuery = cb.createQuery(Long.class);
        Root<Game> countRoot = countQuery.from(Game.class);
        countQuery.select(cb.count(countRoot));
        countQuery.where(predicates.toArray(new Predicate[0]));
        
        Long total = entityManager.createQuery(countQuery).getSingleResult();
        
        return new PageImpl<>(games, pageable, total);
    }
    
    @Override
    public Page<Game> findRecommendationsForUser(Long userId, Pageable pageable) {
        // Query complessa per raccomandazioni basate su:
        // 1. Generi dei giochi che l'utente ha valutato positivamente
        // 2. Giochi simili a quelli nella wishlist
        // 3. Collaborative filtering semplificato
        
        String jpql = """
            SELECT DISTINCT g FROM Game g
            JOIN g.genres gen
            WHERE g.isActive = true
            AND g.id NOT IN (
                SELECT ugl.game.id FROM UserGameList ugl 
                WHERE ugl.user.id = :userId
            )
            AND gen.id IN (
                SELECT gen2.id FROM UserGameList ugl2 
                JOIN ugl2.game.genres gen2
                WHERE ugl2.user.id = :userId 
                AND ugl2.rating >= 7
            )
            ORDER BY g.averageRating DESC, g.totalRatings DESC
            """;
        
        TypedQuery<Game> query = entityManager.createQuery(jpql, Game.class);
        query.setParameter("userId", userId);
        query.setFirstResult((int) pageable.getOffset());
        query.setMaxResults(pageable.getPageSize());
        
        List<Game> games = query.getResultList();
        
        // Count query semplificata
        String countJpql = """
            SELECT COUNT(DISTINCT g) FROM Game g
            JOIN g.genres gen
            WHERE g.isActive = true
            AND g.id NOT IN (
                SELECT ugl.game.id FROM UserGameList ugl 
                WHERE ugl.user.id = :userId
            )
            AND gen.id IN (
                SELECT gen2.id FROM UserGameList ugl2 
                JOIN ugl2.game.genres gen2
                WHERE ugl2.user.id = :userId 
                AND ugl2.rating >= 7
            )
            """;
        
        TypedQuery<Long> countQuery = entityManager.createQuery(countJpql, Long.class);
        countQuery.setParameter("userId", userId);
        Long total = countQuery.getSingleResult();
        
        return new PageImpl<>(games, pageable, total);
    }
}
```

## 4.4 Layer DTO - Data Transfer Objects

### 4.4.1 DTO Base

**BaseDTO.java**:
```java
package mc.videogiochi.dto;

import lombok.Getter;
import lombok.Setter;

import java.time.LocalDateTime;

@Getter
@Setter
public abstract class BaseDTO {
    private Long id;
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;
    private Long version;
}
```

### 4.4.2 DTO Response

**GameDTO.java**:
```java
package mc.videogiochi.dto.response;

import lombok.*;
import mc.videogiochi.domain.enums.EsrbRating;
import mc.videogiochi.dto.BaseDTO;

import java.math.BigDecimal;
import java.time.LocalDate;
import java.util.List;

@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class GameDTO extends BaseDTO {
    
    private String title;
    private String description;
    private LocalDate releaseDate;
    private DeveloperDTO developer;
    private PublisherDTO publisher;
    private String coverImageUrl;
    private String trailerUrl;
    private BigDecimal averageRating;
    private Integer totalRatings;
    private Integer metacriticScore;
    private EsrbRating esrbRating;
    private BigDecimal price;
    private Boolean isActive;
    
    private List<GenreDTO> genres;
    private List<PlatformDTO> platforms;
    
    // Campi statistici
    private Integer totalInWishlists;
    private Integer totalCompleted;
    private BigDecimal popularityScore;
    
    // Campi per utente corrente (se autenticato)
    private UserGameStatusDTO userStatus;
}
```

**GameSummaryDTO.java**:
```java
package mc.videogiochi.dto.response;

import lombok.*;

import java.math.BigDecimal;
import java.time.LocalDate;

@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class GameSummaryDTO {
    
    private Long id;
    private String title;
    private LocalDate releaseDate;
    private String coverImageUrl;
    private BigDecimal averageRating;
    private Integer totalRatings;
    private String developerName;
    private String publisherName;
    
    // Campi per liste
    private Boolean inWishlist;
    private Boolean isPlayed;
    private Integer userRating;
}
```

### 4.4.3 DTO Request

**GameCreateRequest.java**:
```java
package mc.videogiochi.dto.request;

import jakarta.validation.constraints.*;
import lombok.*;
import mc.videogiochi.domain.enums.EsrbRating;

import java.math.BigDecimal;
import java.time.LocalDate;
import java.util.List;

@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class GameCreateRequest {
    
    @NotBlank(message = "Titolo è obbligatorio")
    @Size(max = 200, message = "Titolo non può superare 200 caratteri")
    private String title;
    
    @Size(max = 5000, message = "Descrizione non può superare 5000 caratteri")
    private String description;
    
    private LocalDate releaseDate;
    
    @NotNull(message = "Sviluppatore è obbligatorio")
    private Long developerId;
    
    private Long publisherId;
    
    @Pattern(regexp = "^https?://.*\\.(jpg|jpeg|png|gif)$", 
             message = "URL immagine non valido")
    private String coverImageUrl;
    
    @Pattern(regexp = "^https?://.*", 
             message = "URL trailer non valido")
    private String trailerUrl;
    
    @Min(value = 0, message = "Metacritic score deve essere >= 0")
    @Max(value = 100, message = "Metacritic score deve essere <= 100")
    private Integer metacriticScore;
    
    private EsrbRating esrbRating;
    
    @DecimalMin(value = "0.00", message = "Prezzo deve essere >= 0")
    private BigDecimal price;
    
    @NotEmpty(message = "Almeno un genere è obbligatorio")
    private List<Long> genreIds;
    
    @NotEmpty(message = "Almeno una piattaforma è obbligatoria")
    private List<Long> platformIds;
}
```

**GameSearchCriteria.java**:
```java
package mc.videogiochi.dto.request;

import lombok.*;
import mc.videogiochi.domain.enums.EsrbRating;

import java.math.BigDecimal;
import java.util.List;

@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class GameSearchCriteria {
    
    private String title;
    private List<Long> genreIds;
    private List<Long> platformIds;
    private Long developerId;
    private Long publisherId;
    private Integer releaseYearFrom;
    private Integer releaseYearTo;
    private BigDecimal minRating;
    private BigDecimal maxRating;
    private EsrbRating esrbRating;
    private String sortBy;
    private String sortDirection;
    
    // Filtri avanzati
    private Boolean hasTrailer;
    private Boolean hasMetacriticScore;
    private BigDecimal minPrice;
    private BigDecimal maxPrice;
}
```

### 4.4.4 DTO Paginazione

**PagedResponse.java**:
```java
package mc.videogiochi.dto.response;

import lombok.*;

import java.util.List;

@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class PagedResponse<T> {
    
    private List<T> content;
    private int page;
    private int size;
    private long totalElements;
    private int totalPages;
    private boolean first;
    private boolean last;
    private boolean hasNext;
    private boolean hasPrevious;
    
    public static <T> PagedResponse<T> of(List<T> content, int page, int size, long totalElements) {
        int totalPages = (int) Math.ceil((double) totalElements / size);
        
        return PagedResponse.<T>builder()
                .content(content)
                .page(page)
                .size(size)
                .totalElements(totalElements)
                .totalPages(totalPages)
                .first(page == 0)
                .last(page >= totalPages - 1)
                .hasNext(page < totalPages - 1)
                .hasPrevious(page > 0)
                .build();
    }
}
```

Questo documento continua con l'implementazione completa del backend, includendo i service layer, i controller, la sicurezza e tutti gli altri componenti necessari per un'applicazione Spring Boot robusta e scalabile.