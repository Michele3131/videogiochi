# 4. Implementazione Backend

## 4.1 Setup Iniziale del Progetto

### 4.1.1 Struttura Maven

**pom.xml Completo**:
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
    
    <properties>
        <java.version>21</java.version>
        <mapstruct.version>1.5.5.Final</mapstruct.version>
        <springdoc.version>2.2.0</springdoc.version>
        <testcontainers.version>1.19.1</testcontainers.version>
    </properties>
    
    <dependencies>
        <!-- Spring Boot Starters -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
        
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-cache</artifactId>
        </dependency>
        
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        
        <!-- Database -->
        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
            <scope>runtime</scope>
        </dependency>
        
        <dependency>
            <groupId>org.flywaydb</groupId>
            <artifactId>flyway-core</artifactId>
        </dependency>
        
        <dependency>
            <groupId>org.flywaydb</groupId>
            <artifactId>flyway-mysql</artifactId>
        </dependency>
        
        <!-- Utilities -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        
        <dependency>
            <groupId>org.mapstruct</groupId>
            <artifactId>mapstruct</artifactId>
            <version>${mapstruct.version}</version>
        </dependency>
        
        <!-- Documentation -->
        <dependency>
            <groupId>org.springdoc</groupId>
            <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
            <version>${springdoc.version}</version>
        </dependency>
        
        <!-- Testing -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-test</artifactId>
            <scope>test</scope>
        </dependency>
        
        <dependency>
            <groupId>org.testcontainers</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>${testcontainers.version}</version>
            <scope>test</scope>
        </dependency>
        
        <dependency>
            <groupId>org.testcontainers</groupId>
            <artifactId>mysql</artifactId>
            <version>${testcontainers.version}</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
    
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <annotationProcessorPaths>
                        <path>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                            <version>${lombok.version}</version>
                        </path>
                        <path>
                            <groupId>org.mapstruct</groupId>
                            <artifactId>mapstruct-processor</artifactId>
                            <version>${mapstruct.version}</version>
                        </path>
                    </annotationProcessorPaths>
                </configuration>
            </plugin>
            
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
            
            <plugin>
                <groupId>org.flywaydb</groupId>
                <artifactId>flyway-maven-plugin</artifactId>
                <configuration>
                    <url>jdbc:mysql://localhost:3306/videogiochi</url>
                    <user>videogiochi_app</user>
                    <password>secure_password</password>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

### 4.1.2 Configurazione Application Properties

**application.yml**:
```yaml
spring:
  application:
    name: videogiochi
  
  profiles:
    active: development
  
  datasource:
    url: jdbc:mysql://localhost:3306/videogiochi?useSSL=true&serverTimezone=UTC
    username: ${DB_USERNAME:videogiochi_app}
    password: ${DB_PASSWORD:secure_password}
    driver-class-name: com.mysql.cj.jdbc.Driver
    hikari:
      maximum-pool-size: 20
      minimum-idle: 5
      idle-timeout: 300000
      max-lifetime: 600000
      connection-test-query: SELECT 1
  
  jpa:
    hibernate:
      ddl-auto: validate
    show-sql: false
    properties:
      hibernate:
        dialect: org.hibernate.dialect.MySQL8Dialect
        format_sql: true
        use_sql_comments: true
        jdbc:
          batch_size: 25
        order_inserts: true
        order_updates: true
  
  flyway:
    enabled: true
    locations: classpath:db/migration
    baseline-on-migrate: true
  
  cache:
    type: simple
    cache-names:
      - games
      - users
      - statistics
      - genres
      - platforms
  
  security:
    user:
      name: admin
      password: admin123
      roles: ADMIN

server:
  port: 8080
  servlet:
    context-path: /api
  compression:
    enabled: true
    mime-types: text/html,text/xml,text/plain,text/css,text/javascript,application/javascript,application/json
  
logging:
  level:
    mc.videogiochi: INFO
    org.springframework.security: DEBUG
    org.hibernate.SQL: DEBUG
    org.hibernate.type.descriptor.sql.BasicBinder: TRACE
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} - %msg%n"
    file: "%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n"
  file:
    name: logs/videogiochi.log

app:
  security:
    jwt:
      secret: ${JWT_SECRET:mySecretKey}
      expiration: 86400000 # 24 hours
    session:
      timeout-minutes: 30
      max-login-attempts: 5
  
  upload:
    max-file-size: 10MB
    max-request-size: 10MB
    upload-dir: uploads/
  
  cache:
    enabled: true
    ttl-minutes: 60
  
  pagination:
    default-page-size: 20
    max-page-size: 100

management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
  endpoint:
    health:
      show-details: when-authorized
```

## 4.2 Layer Domain - Entità JPA

### 4.2.1 Entità Base

**BaseEntity.java**:
```java
package mc.videogiochi.domain.entity;

import jakarta.persistence.*;
import lombok.Getter;
import lombok.Setter;
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.annotation.LastModifiedDate;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;

import java.time.LocalDateTime;

@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
@Getter
@Setter
public abstract class BaseEntity {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @CreatedDate
    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt;
    
    @LastModifiedDate
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;
    
    @Version
    private Long version;
}
```

### 4.2.2 Entità Principali

**User.java**:
```java
package mc.videogiochi.domain.entity;

import jakarta.persistence.*;
import jakarta.validation.constraints.*;
import lombok.*;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;

import java.time.LocalDate;
import java.time.LocalDateTime;
import java.util.*;

@Entity
@Table(name = "users")
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class User extends BaseEntity implements UserDetails {
    
    @Column(name = "username", unique = true, nullable = false, length = 50)
    @NotBlank(message = "Username è obbligatorio")
    @Size(min = 3, max = 50, message = "Username deve essere tra 3 e 50 caratteri")
    private String username;
    
    @Column(name = "email", unique = true, nullable = false, length = 100)
    @NotBlank(message = "Email è obbligatoria")
    @Email(message = "Email non valida")
    private String email;
    
    @Column(name = "password_hash", nullable = false)
    @NotBlank(message = "Password è obbligatoria")
    private String password;
    
    @Column(name = "first_name", length = 50)
    private String firstName;
    
    @Column(name = "last_name", length = 50)
    private String lastName;
    
    @Column(name = "birth_date")
    private LocalDate birthDate;
    
    @Column(name = "avatar_url", length = 500)
    private String avatarUrl;
    
    @Column(name = "bio", columnDefinition = "TEXT")
    private String bio;
    
    @Column(name = "is_active")
    @Builder.Default
    private Boolean isActive = true;
    
    @Column(name = "email_verified")
    @Builder.Default
    private Boolean emailVerified = false;
    
    @Column(name = "last_login")
    private LocalDateTime lastLogin;
    
    @ManyToMany(fetch = FetchType.EAGER)
    @JoinTable(
        name = "user_roles",
        joinColumns = @JoinColumn(name = "user_id"),
        inverseJoinColumns = @JoinColumn(name = "role_id")
    )
    @Builder.Default
    private Set<Role> roles = new HashSet<>();
    
    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL, orphanRemoval = true)
    @Builder.Default
    private List<UserGameList> gameLists = new ArrayList<>();
    
    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL, orphanRemoval = true)
    @Builder.Default
    private List<CustomList> customLists = new ArrayList<>();
    
    // UserDetails implementation
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return roles.stream()
                .map(role -> new SimpleGrantedAuthority("ROLE_" + role.getName()))
                .toList();
    }
    
    @Override
    public boolean isAccountNonExpired() {
        return true;
    }
    
    @Override
    public boolean isAccountNonLocked() {
        return isActive;
    }
    
    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }
    
    @Override
    public boolean isEnabled() {
        return isActive && emailVerified;
    }
    
    // Helper methods
    public String getFullName() {
        if (firstName != null && lastName != null) {
            return firstName + " " + lastName;
        }
        return username;
    }
    
    public void addRole(Role role) {
        roles.add(role);
        role.getUsers().add(this);
    }
    
    public void removeRole(Role role) {
        roles.remove(role);
        role.getUsers().remove(this);
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