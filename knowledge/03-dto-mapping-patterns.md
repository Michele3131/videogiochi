# 3. Data Transfer Objects (DTO) e Pattern di Mapping

## 3.1 Data Transfer Objects (DTO)

### 3.1.1 Definizione e Scopo

**Data Transfer Object (DTO)** è un pattern di design che definisce oggetti semplici utilizzati per trasferire dati tra diversi livelli di un'applicazione o tra applicazioni diverse. I DTO incapsulano i dati senza contenere logica di business.

### 3.1.2 Problemi Risolti dai DTO

#### Problema: Esposizione Diretta delle Entità
```java
// PROBLEMATICO - Esposizione diretta dell'entità
@RestController
public class GameController {
    
    @GetMapping("/games/{id}")
    public Game getGame(@PathVariable Long id) {
        return gameService.findById(id); // Espone l'entità direttamente!
    }
}

@Entity
public class Game {
    private Long id;
    private String title;
    private String description;
    private BigDecimal price;
    
    // PROBLEMI:
    // 1. Esposizione di campi interni (id tecnico)
    // 2. Possibili lazy loading exceptions
    // 3. Serializzazione di dati sensibili
    // 4. Accoppiamento tra API e modello dati
    // 5. Difficoltà nel versionare l'API
    
    @OneToMany(mappedBy = "game", fetch = FetchType.LAZY)
    private List<Review> reviews; // Potenziale N+1 query!
    
    @ManyToOne(fetch = FetchType.LAZY)
    private Developer developer; // LazyInitializationException!
}
```

#### Soluzione con DTO
```java
// DTO per la risposta
@Data
@Builder
public class GameDTO {
    private Long id;
    private String title;
    private String description;
    private BigDecimal price;
    private LocalDate releaseDate;
    private String esrbRating;
    private Double userRating;
    
    // Dati del developer (denormalizzati)
    private String developerName;
    private String publisherName;
    
    // Statistiche aggregate
    private Integer reviewCount;
    private Double averageRating;
    
    // Metadati
    private LocalDateTime lastUpdated;
}

// Controller con DTO
@RestController
public class GameController {
    
    @GetMapping("/games/{id}")
    public ResponseEntity<GameDTO> getGame(@PathVariable Long id) {
        GameDTO gameDTO = gameService.findGameById(id);
        return ResponseEntity.ok(gameDTO);
    }
}
```

### 3.1.3 Tipi di DTO

#### Response DTO (Output)
```java
// DTO per liste (meno dettagli)
@Data
@Builder
public class GameSummaryDTO {
    private Long id;
    private String title;
    private BigDecimal price;
    private String coverImageUrl;
    private String developerName;
    private Double userRating;
    private LocalDate releaseDate;
}

// DTO per dettagli completi
@Data
@Builder
public class GameDetailDTO {
    private Long id;
    private String title;
    private String description;
    private String longDescription;
    private BigDecimal price;
    private LocalDate releaseDate;
    private String esrbRating;
    private List<String> screenshots;
    private String trailerUrl;
    
    // Informazioni del developer
    private DeveloperDTO developer;
    private PublisherDTO publisher;
    
    // Generi e piattaforme
    private List<GenreDTO> genres;
    private List<PlatformDTO> platforms;
    
    // Statistiche
    private Double averageRating;
    private Integer reviewCount;
    private Integer wishlistCount;
    
    // Metadati
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;
}

// DTO per statistiche
@Data
@Builder
public class GameStatsDTO {
    private Long totalGames;
    private Long activeGames;
    private Double averagePrice;
    private Map<String, Long> gamesByGenre;
    private Map<String, Long> gamesByRating;
    private List<GameSummaryDTO> topRatedGames;
    private List<GameSummaryDTO> newestGames;
}
```

#### Request DTO (Input)
```java
// DTO per creazione
@Data
@Builder
public class GameCreateRequest {
    
    @NotBlank(message = "Il titolo è obbligatorio")
    @Size(min = 1, max = 255, message = "Il titolo deve essere tra 1 e 255 caratteri")
    private String title;
    
    @NotBlank(message = "La descrizione è obbligatoria")
    @Size(min = 10, max = 1000, message = "La descrizione deve essere tra 10 e 1000 caratteri")
    private String description;
    
    @Size(max = 5000, message = "La descrizione lunga non può superare 5000 caratteri")
    private String longDescription;
    
    @NotNull(message = "Il prezzo è obbligatorio")
    @DecimalMin(value = "0.0", inclusive = false, message = "Il prezzo deve essere maggiore di 0")
    @DecimalMax(value = "999.99", message = "Il prezzo non può superare 999.99")
    private BigDecimal price;
    
    @NotNull(message = "La data di rilascio è obbligatoria")
    @PastOrPresent(message = "La data di rilascio non può essere nel futuro")
    private LocalDate releaseDate;
    
    @NotNull(message = "Il rating ESRB è obbligatorio")
    private EsrbRating esrbRating;
    
    @NotNull(message = "Il developer è obbligatorio")
    private Long developerId;
    
    @NotNull(message = "Il publisher è obbligatorio")
    private Long publisherId;
    
    @NotEmpty(message = "Almeno un genere è obbligatorio")
    @Size(max = 5, message = "Massimo 5 generi consentiti")
    private List<Long> genreIds;
    
    @NotEmpty(message = "Almeno una piattaforma è obbligatoria")
    private List<Long> platformIds;
    
    @URL(message = "URL del trailer non valido")
    private String trailerUrl;
    
    private List<@URL(message = "URL screenshot non valido") String> screenshots;
}

// DTO per aggiornamento
@Data
@Builder
public class GameUpdateRequest {
    
    @Size(min = 1, max = 255, message = "Il titolo deve essere tra 1 e 255 caratteri")
    private String title;
    
    @Size(min = 10, max = 1000, message = "La descrizione deve essere tra 10 e 1000 caratteri")
    private String description;
    
    @Size(max = 5000, message = "La descrizione lunga non può superare 5000 caratteri")
    private String longDescription;
    
    @DecimalMin(value = "0.0", inclusive = false, message = "Il prezzo deve essere maggiore di 0")
    @DecimalMax(value = "999.99", message = "Il prezzo non può superare 999.99")
    private BigDecimal price;
    
    @PastOrPresent(message = "La data di rilascio non può essere nel futuro")
    private LocalDate releaseDate;
    
    private EsrbRating esrbRating;
    
    @Size(max = 5, message = "Massimo 5 generi consentiti")
    private List<Long> genreIds;
    
    private List<Long> platformIds;
    
    @URL(message = "URL del trailer non valido")
    private String trailerUrl;
    
    private List<@URL(message = "URL screenshot non valido") String> screenshots;
}

// DTO per ricerca
@Data
@Builder
public class GameSearchCriteria {
    private String title;
    private Long developerId;
    private Long publisherId;
    private List<Long> genreIds;
    private List<Long> platformIds;
    private BigDecimal minPrice;
    private BigDecimal maxPrice;
    private LocalDate releaseDateFrom;
    private LocalDate releaseDateTo;
    private EsrbRating esrbRating;
    private Double minRating;
    private String sortBy; // title, price, releaseDate, rating
    private String sortDirection; // asc, desc
}
```

### 3.1.4 DTO per Paginazione
```java
// Parametri di paginazione
@Data
@Builder
public class PaginationParams {
    
    @Min(value = 0, message = "Il numero di pagina deve essere >= 0")
    private int page = 0;
    
    @Min(value = 1, message = "La dimensione della pagina deve essere >= 1")
    @Max(value = 100, message = "La dimensione della pagina deve essere <= 100")
    private int size = 10;
    
    private String sortBy = "id";
    private String sortDirection = "asc";
    
    public Pageable toPageable() {
        Sort.Direction direction = "desc".equalsIgnoreCase(sortDirection) 
            ? Sort.Direction.DESC 
            : Sort.Direction.ASC;
        
        return PageRequest.of(page, size, Sort.by(direction, sortBy));
    }
}

// Risposta paginata
@Data
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
    
    public static <T> PagedResponse<T> of(Page<T> page) {
        return PagedResponse.<T>builder()
            .content(page.getContent())
            .page(page.getNumber())
            .size(page.getSize())
            .totalElements(page.getTotalElements())
            .totalPages(page.getTotalPages())
            .first(page.isFirst())
            .last(page.isLast())
            .hasNext(page.hasNext())
            .hasPrevious(page.hasPrevious())
            .build();
    }
    
    public static <T, U> PagedResponse<U> of(Page<T> page, List<U> mappedContent) {
        return PagedResponse.<U>builder()
            .content(mappedContent)
            .page(page.getNumber())
            .size(page.getSize())
            .totalElements(page.getTotalElements())
            .totalPages(page.getTotalPages())
            .first(page.isFirst())
            .last(page.isLast())
            .hasNext(page.hasNext())
            .hasPrevious(page.hasPrevious())
            .build();
    }
}
```

## 3.2 Pattern di Mapping

### 3.2.1 Mapping Manuale

#### Vantaggi e Svantaggi
```java
// Mapping manuale - Controllo completo ma verboso
@Component
public class GameMapper {
    
    public GameDTO toDTO(Game game) {
        if (game == null) {
            return null;
        }
        
        return GameDTO.builder()
            .id(game.getId())
            .title(game.getTitle())
            .description(game.getDescription())
            .price(game.getPrice())
            .releaseDate(game.getReleaseDate())
            .esrbRating(game.getEsrbRating().name())
            .userRating(game.getUserRating())
            .developerName(game.getDeveloper() != null ? game.getDeveloper().getName() : null)
            .publisherName(game.getPublisher() != null ? game.getPublisher().getName() : null)
            .reviewCount(game.getReviews() != null ? game.getReviews().size() : 0)
            .averageRating(calculateAverageRating(game.getReviews()))
            .lastUpdated(game.getUpdatedAt())
            .build();
    }
    
    public Game toEntity(GameCreateRequest request) {
        if (request == null) {
            return null;
        }
        
        Game game = new Game();
        game.setTitle(request.getTitle());
        game.setDescription(request.getDescription());
        game.setLongDescription(request.getLongDescription());
        game.setPrice(request.getPrice());
        game.setReleaseDate(request.getReleaseDate());
        game.setEsrbRating(request.getEsrbRating());
        game.setTrailerUrl(request.getTrailerUrl());
        game.setScreenshots(request.getScreenshots());
        game.setActive(true);
        game.setCreatedAt(LocalDateTime.now());
        game.setUpdatedAt(LocalDateTime.now());
        
        return game;
    }
    
    public void updateEntity(Game game, GameUpdateRequest request) {
        if (request == null || game == null) {
            return;
        }
        
        if (request.getTitle() != null) {
            game.setTitle(request.getTitle());
        }
        if (request.getDescription() != null) {
            game.setDescription(request.getDescription());
        }
        if (request.getLongDescription() != null) {
            game.setLongDescription(request.getLongDescription());
        }
        if (request.getPrice() != null) {
            game.setPrice(request.getPrice());
        }
        if (request.getReleaseDate() != null) {
            game.setReleaseDate(request.getReleaseDate());
        }
        if (request.getEsrbRating() != null) {
            game.setEsrbRating(request.getEsrbRating());
        }
        if (request.getTrailerUrl() != null) {
            game.setTrailerUrl(request.getTrailerUrl());
        }
        if (request.getScreenshots() != null) {
            game.setScreenshots(request.getScreenshots());
        }
        
        game.setUpdatedAt(LocalDateTime.now());
    }
    
    private Double calculateAverageRating(List<Review> reviews) {
        if (reviews == null || reviews.isEmpty()) {
            return null;
        }
        
        return reviews.stream()
            .mapToDouble(Review::getRating)
            .average()
            .orElse(0.0);
    }
}
```

### 3.2.2 MapStruct - Mapping Automatico

#### Configurazione Base
```java
// Configurazione MapStruct
@Mapper(
    componentModel = "spring",
    unmappedTargetPolicy = ReportingPolicy.WARN,
    nullValuePropertyMappingStrategy = NullValuePropertyMappingStrategy.IGNORE
)
public interface GameMapper {
    
    // Mapping semplice
    @Mapping(target = "developerName", source = "developer.name")
    @Mapping(target = "publisherName", source = "publisher.name")
    @Mapping(target = "esrbRating", source = "esrbRating", qualifiedByName = "esrbToString")
    @Mapping(target = "reviewCount", source = "reviews", qualifiedByName = "reviewsToCount")
    @Mapping(target = "averageRating", source = "reviews", qualifiedByName = "reviewsToAverage")
    GameDTO toDTO(Game game);
    
    // Mapping per lista
    List<GameDTO> toDTOList(List<Game> games);
    
    // Mapping da request a entity
    @Mapping(target = "id", ignore = true)
    @Mapping(target = "developer", ignore = true)
    @Mapping(target = "publisher", ignore = true)
    @Mapping(target = "genres", ignore = true)
    @Mapping(target = "platforms", ignore = true)
    @Mapping(target = "reviews", ignore = true)
    @Mapping(target = "userGameLists", ignore = true)
    @Mapping(target = "active", constant = "true")
    @Mapping(target = "createdAt", expression = "java(java.time.LocalDateTime.now())")
    @Mapping(target = "updatedAt", expression = "java(java.time.LocalDateTime.now())")
    Game toEntity(GameCreateRequest request);
    
    // Update mapping
    @Mapping(target = "id", ignore = true)
    @Mapping(target = "developer", ignore = true)
    @Mapping(target = "publisher", ignore = true)
    @Mapping(target = "genres", ignore = true)
    @Mapping(target = "platforms", ignore = true)
    @Mapping(target = "reviews", ignore = true)
    @Mapping(target = "userGameLists", ignore = true)
    @Mapping(target = "active", ignore = true)
    @Mapping(target = "createdAt", ignore = true)
    @Mapping(target = "updatedAt", expression = "java(java.time.LocalDateTime.now())")
    void updateEntity(@MappingTarget Game game, GameUpdateRequest request);
    
    // Metodi di supporto
    @Named("esrbToString")
    default String esrbToString(EsrbRating esrbRating) {
        return esrbRating != null ? esrbRating.name() : null;
    }
    
    @Named("reviewsToCount")
    default Integer reviewsToCount(List<Review> reviews) {
        return reviews != null ? reviews.size() : 0;
    }
    
    @Named("reviewsToAverage")
    default Double reviewsToAverage(List<Review> reviews) {
        if (reviews == null || reviews.isEmpty()) {
            return null;
        }
        
        return reviews.stream()
            .mapToDouble(Review::getRating)
            .average()
            .orElse(0.0);
    }
}
```

#### Mapping Complesso con Dipendenze
```java
@Mapper(
    componentModel = "spring",
    uses = {DeveloperMapper.class, PublisherMapper.class, GenreMapper.class},
    unmappedTargetPolicy = ReportingPolicy.WARN
)
public interface GameDetailMapper {
    
    // Mapping completo con oggetti nested
    @Mapping(target = "developer", source = "developer")
    @Mapping(target = "publisher", source = "publisher")
    @Mapping(target = "genres", source = "genres")
    @Mapping(target = "platforms", source = "platforms")
    @Mapping(target = "averageRating", source = "reviews", qualifiedByName = "calculateAverage")
    @Mapping(target = "reviewCount", source = "reviews", qualifiedByName = "countReviews")
    @Mapping(target = "wishlistCount", source = "userGameLists", qualifiedByName = "countWishlists")
    GameDetailDTO toDetailDTO(Game game);
    
    @Named("calculateAverage")
    default Double calculateAverage(List<Review> reviews) {
        if (reviews == null || reviews.isEmpty()) {
            return null;
        }
        
        return reviews.stream()
            .filter(review -> review.getRating() != null)
            .mapToDouble(Review::getRating)
            .average()
            .orElse(0.0);
    }
    
    @Named("countReviews")
    default Integer countReviews(List<Review> reviews) {
        return reviews != null ? reviews.size() : 0;
    }
    
    @Named("countWishlists")
    default Integer countWishlists(List<UserGameList> userGameLists) {
        if (userGameLists == null) {
            return 0;
        }
        
        return (int) userGameLists.stream()
            .filter(ugl -> ugl.getListType() == ListType.WISHLIST)
            .count();
    }
}

// Mapper per Developer
@Mapper(componentModel = "spring")
public interface DeveloperMapper {
    
    @Mapping(target = "gameCount", source = "games", qualifiedByName = "countActiveGames")
    DeveloperDTO toDTO(Developer developer);
    
    @Named("countActiveGames")
    default Integer countActiveGames(List<Game> games) {
        if (games == null) {
            return 0;
        }
        
        return (int) games.stream()
            .filter(Game::isActive)
            .count();
    }
}
```

### 3.2.3 Mapping Condizionale e Personalizzato

#### Mapping Basato su Contesto
```java
@Mapper(componentModel = "spring")
public interface GameMapper {
    
    // Mapping diverso per utenti autenticati
    @Mapping(target = "userRating", source = "game", qualifiedByName = "getUserRating")
    @Mapping(target = "inWishlist", source = "game", qualifiedByName = "isInWishlist")
    @Mapping(target = "owned", source = "game", qualifiedByName = "isOwned")
    GameDTO toDTOForUser(Game game, @Context User currentUser);
    
    // Mapping per amministratori (più dettagli)
    @Mapping(target = "createdBy", source = "createdBy.username")
    @Mapping(target = "lastModifiedBy", source = "lastModifiedBy.username")
    @Mapping(target = "totalRevenue", source = ".", qualifiedByName = "calculateRevenue")
    GameAdminDTO toAdminDTO(Game game);
    
    @Named("getUserRating")
    default Double getUserRating(Game game, @Context User currentUser) {
        if (currentUser == null || game.getReviews() == null) {
            return null;
        }
        
        return game.getReviews().stream()
            .filter(review -> review.getUser().getId().equals(currentUser.getId()))
            .findFirst()
            .map(Review::getRating)
            .orElse(null);
    }
    
    @Named("isInWishlist")
    default Boolean isInWishlist(Game game, @Context User currentUser) {
        if (currentUser == null || game.getUserGameLists() == null) {
            return false;
        }
        
        return game.getUserGameLists().stream()
            .anyMatch(ugl -> ugl.getUser().getId().equals(currentUser.getId()) 
                          && ugl.getListType() == ListType.WISHLIST);
    }
    
    @Named("isOwned")
    default Boolean isOwned(Game game, @Context User currentUser) {
        if (currentUser == null || game.getUserGameLists() == null) {
            return false;
        }
        
        return game.getUserGameLists().stream()
            .anyMatch(ugl -> ugl.getUser().getId().equals(currentUser.getId()) 
                          && ugl.getListType() == ListType.OWNED);
    }
    
    @Named("calculateRevenue")
    default BigDecimal calculateRevenue(Game game) {
        if (game.getUserGameLists() == null) {
            return BigDecimal.ZERO;
        }
        
        long ownedCount = game.getUserGameLists().stream()
            .filter(ugl -> ugl.getListType() == ListType.OWNED)
            .count();
        
        return game.getPrice().multiply(BigDecimal.valueOf(ownedCount));
    }
}
```

#### Mapping con Validazione
```java
@Mapper(componentModel = "spring")
public interface GameMapper {
    
    @Mapping(target = "id", ignore = true)
    @Mapping(target = "active", constant = "true")
    @Mapping(target = "createdAt", expression = "java(java.time.LocalDateTime.now())")
    @Mapping(target = "updatedAt", expression = "java(java.time.LocalDateTime.now())")
    @Mapping(target = "price", source = "price", qualifiedByName = "validatePrice")
    @Mapping(target = "title", source = "title", qualifiedByName = "sanitizeTitle")
    Game toEntity(GameCreateRequest request);
    
    @Named("validatePrice")
    default BigDecimal validatePrice(BigDecimal price) {
        if (price == null) {
            throw new ValidationException("Il prezzo è obbligatorio");
        }
        
        if (price.compareTo(BigDecimal.ZERO) <= 0) {
            throw new ValidationException("Il prezzo deve essere maggiore di zero");
        }
        
        if (price.compareTo(BigDecimal.valueOf(999.99)) > 0) {
            throw new ValidationException("Il prezzo non può superare 999.99");
        }
        
        return price.setScale(2, RoundingMode.HALF_UP);
    }
    
    @Named("sanitizeTitle")
    default String sanitizeTitle(String title) {
        if (title == null) {
            return null;
        }
        
        // Rimuove caratteri speciali e normalizza
        String sanitized = title.trim()
            .replaceAll("[^a-zA-Z0-9\\s\\-\\:]", "")
            .replaceAll("\\s+", " ");
        
        if (sanitized.length() > 255) {
            sanitized = sanitized.substring(0, 255);
        }
        
        return sanitized;
    }
}
```

## 3.3 Strategie di Mapping Avanzate

### 3.3.1 Mapping Lazy e Performance

#### Projection per Performance
```java
// Interface-based projection
public interface GameSummaryProjection {
    Long getId();
    String getTitle();
    BigDecimal getPrice();
    String getDeveloperName();
    Double getUserRating();
    LocalDate getReleaseDate();
}

// Class-based projection
@Data
@AllArgsConstructor
public class GameSummaryProjection {
    private Long id;
    private String title;
    private BigDecimal price;
    private String developerName;
    private Double userRating;
    private LocalDate releaseDate;
}

// Repository con projection
@Repository
public interface GameRepository extends JpaRepository<Game, Long> {
    
    @Query("SELECT g.id as id, g.title as title, g.price as price, " +
           "d.name as developerName, g.userRating as userRating, " +
           "g.releaseDate as releaseDate " +
           "FROM Game g JOIN g.developer d WHERE g.active = true")
    Page<GameSummaryProjection> findGameSummaries(Pageable pageable);
    
    @Query("SELECT new com.videogames.dto.GameSummaryProjection(" +
           "g.id, g.title, g.price, d.name, g.userRating, g.releaseDate) " +
           "FROM Game g JOIN g.developer d WHERE g.active = true")
    Page<GameSummaryProjection> findGameSummariesConstructor(Pageable pageable);
}
```

#### Mapping con EntityGraph
```java
@Repository
public interface GameRepository extends JpaRepository<Game, Long> {
    
    @EntityGraph(attributePaths = {"developer", "publisher", "genres", "platforms"})
    @Query("SELECT g FROM Game g WHERE g.id = :id AND g.active = true")
    Optional<Game> findByIdWithDetails(@Param("id") Long id);
    
    @EntityGraph(attributePaths = {"developer", "publisher"})
    Page<Game> findByActiveTrue(Pageable pageable);
}

// Service con mapping ottimizzato
@Service
public class GameService {
    
    public GameDetailDTO findGameDetails(Long id) {
        Game game = gameRepository.findByIdWithDetails(id)
            .orElseThrow(() -> new ResourceNotFoundException("Gioco non trovato"));
        
        return gameDetailMapper.toDetailDTO(game);
    }
    
    public PagedResponse<GameDTO> findAllGames(Pageable pageable) {
        Page<Game> games = gameRepository.findByActiveTrue(pageable);
        
        List<GameDTO> gameDTOs = games.getContent().stream()
            .map(gameMapper::toDTO)
            .collect(Collectors.toList());
        
        return PagedResponse.of(games, gameDTOs);
    }
}
```

### 3.3.2 Mapping Asincrono

#### Mapping con CompletableFuture
```java
@Service
public class GameService {
    
    @Async
    public CompletableFuture<List<GameDTO>> findGamesAsync(List<Long> gameIds) {
        List<Game> games = gameRepository.findAllById(gameIds);
        
        List<GameDTO> gameDTOs = games.parallelStream()
            .map(this::mapToDTO)
            .collect(Collectors.toList());
        
        return CompletableFuture.completedFuture(gameDTOs);
    }
    
    private GameDTO mapToDTO(Game game) {
        // Mapping complesso che può richiedere tempo
        GameDTO dto = gameMapper.toDTO(game);
        
        // Arricchimento con dati esterni
        enrichWithExternalData(dto);
        
        return dto;
    }
    
    private void enrichWithExternalData(GameDTO dto) {
        // Chiamate a servizi esterni, calcoli complessi, etc.
        // Simulazione di operazione costosa
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}
```

### 3.3.3 Caching dei Mapping

#### Cache per DTO Costosi
```java
@Service
public class GameService {
    
    @Cacheable(value = "gameDetails", key = "#id")
    public GameDetailDTO findGameDetails(Long id) {
        Game game = gameRepository.findByIdWithDetails(id)
            .orElseThrow(() -> new ResourceNotFoundException("Gioco non trovato"));
        
        return gameDetailMapper.toDetailDTO(game);
    }
    
    @Cacheable(value = "gameStats", key = "'all'")
    public GameStatsDTO getGameStatistics() {
        // Calcoli costosi per statistiche
        long totalGames = gameRepository.count();
        long activeGames = gameRepository.countByActiveTrue();
        Double averagePrice = gameRepository.getAveragePrice();
        
        Map<String, Long> gamesByGenre = gameRepository.getGameCountByGenre();
        Map<String, Long> gamesByRating = gameRepository.getGameCountByRating();
        
        List<Game> topRated = gameRepository.findTop10ByOrderByUserRatingDesc();
        List<GameSummaryDTO> topRatedDTOs = topRated.stream()
            .map(gameMapper::toSummaryDTO)
            .collect(Collectors.toList());
        
        List<Game> newest = gameRepository.findTop10ByOrderByReleaseDateDesc();
        List<GameSummaryDTO> newestDTOs = newest.stream()
            .map(gameMapper::toSummaryDTO)
            .collect(Collectors.toList());
        
        return GameStatsDTO.builder()
            .totalGames(totalGames)
            .activeGames(activeGames)
            .averagePrice(averagePrice)
            .gamesByGenre(gamesByGenre)
            .gamesByRating(gamesByRating)
            .topRatedGames(topRatedDTOs)
            .newestGames(newestDTOs)
            .build();
    }
    
    @CacheEvict(value = {"gameDetails", "gameStats"}, allEntries = true)
    public GameDTO updateGame(Long id, GameUpdateRequest request) {
        // Aggiornamento che invalida la cache
        Game game = gameRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("Gioco non trovato"));
        
        gameMapper.updateEntity(game, request);
        Game savedGame = gameRepository.save(game);
        
        return gameMapper.toDTO(savedGame);
    }
}
```

## 3.4 Best Practices

### 3.4.1 Principi Generali

1. **Separazione delle Responsabilità**
   - DTO per trasferimento dati
   - Entity per persistenza
   - Mapper per conversione

2. **Immutabilità dei DTO**
```java
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