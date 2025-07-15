# 4. Query Dinamiche e Paginazione

## 4.1 Query Dinamiche

**Query Dinamiche** sono query costruite a runtime in base a criteri variabili. Risolvono il problema dell'**esplosione combinatoriale** dei metodi di repository per gestire filtri multipli e opzionali.

**Problema delle Query Statiche:**
```java
// Approccio problematico
@Repository
public interface GameRepository extends JpaRepository<Game, Long> {
    Page<Game> findByTitle(String title, Pageable pageable);
    Page<Game> findByTitleAndDeveloper(String title, Developer developer, Pageable pageable);
    Page<Game> findByTitleAndDeveloperAndPriceBetween(...);
    // ... centinaia di combinazioni!
}
```

### Criteria API - Soluzione Dinamica

**Implementazione con Criteria API:**
```java
@Repository
public class GameRepositoryCustomImpl implements GameRepositoryCustom {
    
    @PersistenceContext
    private EntityManager entityManager;
    
    @Override
    public Page<Game> findGamesWithDynamicFilters(
            GameSearchCriteria criteria, Pageable pageable) {
        
        CriteriaBuilder cb = entityManager.getCriteriaBuilder();
        CriteriaQuery<Game> query = cb.createQuery(Game.class);
        Root<Game> root = query.from(Game.class);
        
        // Join condizionali
        Join<Game, Developer> developerJoin = null;
        if (criteria.getDeveloperId() != null) {
            developerJoin = root.join("developer", JoinType.INNER);
        }
        
        // Costruzione predicati dinamici
        List<Predicate> predicates = buildPredicates(cb, root, criteria, developerJoin);
        
        if (!predicates.isEmpty()) {
            query.where(predicates.toArray(new Predicate[0]));
        }
        
        // Ordinamento dinamico
        List<Order> orders = buildOrders(cb, root, pageable.getSort());
        if (!orders.isEmpty()) {
            query.orderBy(orders);
        }
        
        // Esecuzione con paginazione
        TypedQuery<Game> typedQuery = entityManager.createQuery(query);
        typedQuery.setFirstResult((int) pageable.getOffset());
        typedQuery.setMaxResults(pageable.getPageSize());
        
        List<Game> games = typedQuery.getResultList();
        long total = countGamesWithFilters(criteria);
        
        return new PageImpl<>(games, pageable, total);
    }
    
    private List<Predicate> buildPredicates(
            CriteriaBuilder cb, Root<Game> root, 
            GameSearchCriteria criteria, Join<Game, Developer> developerJoin) {
        
        List<Predicate> predicates = new ArrayList<>();
        
        // Filtri dinamici
        if (StringUtils.hasText(criteria.getTitle())) {
            predicates.add(cb.like(
                cb.lower(root.get("title")),
                "%" + criteria.getTitle().toLowerCase() + "%"
            ));
        }
        
        if (criteria.getDeveloperId() != null) {
            predicates.add(cb.equal(
                developerJoin.get("id"), criteria.getDeveloperId()
            ));
        }
        
        if (criteria.getMinPrice() != null) {
            predicates.add(cb.greaterThanOrEqualTo(
                root.get("price"), criteria.getMinPrice()
            ));
        }
        
        if (criteria.getMaxPrice() != null) {
            predicates.add(cb.lessThanOrEqualTo(
                root.get("price"), criteria.getMaxPrice()
            ));
        }
        
        return predicates;
    }
}
```
## Criteri di Ricerca

**DTO per Filtri Dinamici:**
```java
@Data
@Builder
public class GameSearchCriteria {
    // Filtri di testo
    private String title;
    private String developerName;
    
    // Filtri numerici
    private BigDecimal minPrice;
    private BigDecimal maxPrice;
    
    // Filtri per date
    private LocalDate releaseDateFrom;
    private LocalDate releaseDateTo;
    
    // Filtri booleani
    private Boolean available;
    private Boolean onSale;
}
```

## Paginazione

**Implementazione con Spring Data:**
```java
@RestController
public class GameController {
    
    @GetMapping("/games")
    public Page<GameDTO> searchGames(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "20") int size,
            @RequestParam(defaultValue = "releaseDate,desc") String sort,
            GameSearchCriteria criteria) {
        
        Pageable pageable = PageRequest.of(page, size, Sort.by(sort));
        return gameService.findGamesWithFilters(criteria, pageable);
    }
}
```

## Vantaggi delle Query Dinamiche

- **Flessibilità**: Costruzione runtime delle query
- **Performance**: Solo i join necessari
- **Manutenibilità**: Logica centralizzata
- **Scalabilità**: Gestione efficiente di filtri complessi
- **Type Safety**: Controllo a compile-time con Criteria API

## Best Practices

- Validazione dei parametri di ordinamento
- Limitazione del numero massimo di risultati
- Caching delle query frequenti
- Indicizzazione appropriata del database
- Gestione degli errori e timeout
Le query dinamiche e la paginazione rappresentano strumenti fondamentali per costruire API flessibili e performanti, permettendo agli utenti di filtrare e navigare grandi dataset in modo efficiente attraverso l'uso strategico di Criteria API e Spring Data.
    
    // Costruttore e metodi getter/setter
    
    public Join<Game, Developer> getOrCreateDeveloperJoin() {
        if (developerJoin == null) {
            developerJoin = root.join("developer", JoinType.INNER);
        }
        return developerJoin;
    }
    
    public void addPredicate(Predicate predicate) {
        predicates.add(predicate);
    }
    
    // Altri metodi di utilità...
}
```

## 4.2 Paginazione

### 4.2.1 Concetti Base

**Paginazione** è la tecnica di dividere grandi set di risultati in "pagine" più piccole e gestibili, migliorando le performance e l'esperienza utente.

### 4.2.2 Implementazione con Spring Data

#### Repository con Paginazione
```java
@Repository
public interface GameRepository extends JpaRepository<Game, Long>, GameRepositoryCustom {
    
    // Paginazione semplice
    Page<Game> findByActiveTrue(Pageable pageable);
    
    // Paginazione con filtri
    Page<Game> findByTitleContainingIgnoreCaseAndActiveTrue(
        String title, Pageable pageable);
    
    Page<Game> findByDeveloperIdAndActiveTrue(
        Long developerId, Pageable pageable);
    
    Page<Game> findByPriceBetweenAndActiveTrue(
        BigDecimal minPrice, BigDecimal maxPrice, Pageable pageable);
    
    // Slice per performance migliori (senza count totale)
    Slice<Game> findSliceByActiveTrue(Pageable pageable);
    
    // Query personalizzate con paginazione
    @Query("SELECT g FROM Game g JOIN g.genres gen WHERE gen.id = :genreId AND g.active = true")
    Page<Game> findByGenreId(@Param("genreId") Long genreId, Pageable pageable);
    
    @Query(value = "SELECT g.* FROM games g " +
                   "WHERE g.user_rating >= :minRating " +
                   "AND g.active = true " +
                   "ORDER BY g.user_rating DESC, g.release_date DESC",
           countQuery = "SELECT COUNT(*) FROM games g " +
                       "WHERE g.user_rating >= :minRating " +
                       "AND g.active = true",
           nativeQuery = true)
    Page<Game> findTopRatedGames(@Param("minRating") Double minRating, Pageable pageable);
}
```

#### Service Layer con Paginazione
```java
@Service
@Transactional(readOnly = true)
public class GameService {
    
    private final GameRepository gameRepository;
    private final GameMapper gameMapper;
    
    public PagedResponse<GameDTO> findAllGames(PaginationParams params) {
        Pageable pageable = createPageable(params);
        Page<Game> games = gameRepository.findByActiveTrue(pageable);
        
        List<GameDTO> gameDTOs = games.getContent().stream()
            .map(gameMapper::toDTO)
            .collect(Collectors.toList());
        
        return PagedResponse.of(games, gameDTOs);
    }
    
    public PagedResponse<GameDTO> searchGames(
            GameSearchCriteria criteria, PaginationParams params) {
        
        validateSearchCriteria(criteria);
        Pageable pageable = createPageable(params);
        
        Page<Game> games = gameRepository.findGamesWithDynamicFilters(
            criteria, pageable
        );
        
        List<GameDTO> gameDTOs = games.getContent().stream()
            .map(gameMapper::toDTO)
            .collect(Collectors.toList());
        
        return PagedResponse.of(games, gameDTOs);
    }
    
    public SliceResponse<GameSummaryDTO> findGameSummaries(
            PaginationParams params) {
        
        Pageable pageable = createPageable(params);
        Slice<Game> games = gameRepository.findSliceByActiveTrue(pageable);
        
        List<GameSummaryDTO> summaries = games.getContent().stream()
            .map(gameMapper::toSummaryDTO)
            .collect(Collectors.toList());
        
        return SliceResponse.of(games, summaries);
    }
    
    private Pageable createPageable(PaginationParams params) {
        // Validazione parametri
        int page = Math.max(0, params.getPage());
        int size = Math.min(Math.max(1, params.getSize()), 100); // Max 100 elementi
        
        // Validazione campo di ordinamento
        String sortBy = validateSortField(params.getSortBy());
        Sort.Direction direction = "desc".equalsIgnoreCase(params.getSortDirection())
            ? Sort.Direction.DESC
            : Sort.Direction.ASC;
        
        Sort sort = Sort.by(direction, sortBy);
        
        return PageRequest.of(page, size, sort);
    }
    
    private String validateSortField(String sortBy) {
        Set<String> allowedFields = Set.of(
            "title", "price", "releaseDate", "userRating", 
            "createdAt", "updatedAt"
        );
        
        return allowedFields.contains(sortBy) ? sortBy : "releaseDate";
    }
    
    private void validateSearchCriteria(GameSearchCriteria criteria) {
        // Validazione range di prezzo
        if (criteria.getMinPrice() != null && criteria.getMaxPrice() != null) {
            if (criteria.getMinPrice().compareTo(criteria.getMaxPrice()) > 0) {
                throw new ValidationException(
                    "Il prezzo minimo non può essere maggiore del prezzo massimo"
                );
            }
        }
        
        // Validazione range di date
        if (criteria.getReleaseDateFrom() != null && criteria.getReleaseDateTo() != null) {
            if (criteria.getReleaseDateFrom().isAfter(criteria.getReleaseDateTo())) {
                throw new ValidationException(
                    "La data di inizio non può essere successiva alla data di fine"
                );
            }
        }
        
        // Validazione lunghezza testo di ricerca
        if (StringUtils.hasText(criteria.getSearchText()) && 
            criteria.getSearchText().length() < 2) {
            throw new ValidationException(
                "Il testo di ricerca deve contenere almeno 2 caratteri"
            );
        }
    }
}
```

### 4.2.3 DTO per Paginazione

#### Response Paginata
```java
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
    private Sort sort;
    
    // Metadati aggiuntivi
    private long numberOfElements;
    private boolean empty;
    
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
            .sort(page.getSort())
            .numberOfElements(page.getNumberOfElements())
            .empty(page.isEmpty())
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
            .sort(page.getSort())
            .numberOfElements(mappedContent.size())
            .empty(mappedContent.isEmpty())
            .build();
    }
}

// Response per Slice (senza count totale)
@Data
@Builder
public class SliceResponse<T> {
    private List<T> content;
    private int page;
    private int size;
    private boolean first;
    private boolean last;
    private boolean hasNext;
    private boolean hasPrevious;
    private Sort sort;
    private long numberOfElements;
    private boolean empty;
    
    public static <T> SliceResponse<T> of(Slice<T> slice) {
        return SliceResponse.<T>builder()
            .content(slice.getContent())
            .page(slice.getNumber())
            .size(slice.getSize())
            .first(slice.isFirst())
            .last(slice.isLast())
            .hasNext(slice.hasNext())
            .hasPrevious(slice.hasPrevious())
            .sort(slice.getSort())
            .numberOfElements(slice.getNumberOfElements())
            .empty(slice.isEmpty())
            .build();
    }
    
    public static <T, U> SliceResponse<U> of(Slice<T> slice, List<U> mappedContent) {
        return SliceResponse.<U>builder()
            .content(mappedContent)
            .page(slice.getNumber())
            .size(slice.getSize())
            .first(slice.isFirst())
            .last(slice.isLast())
            .hasNext(slice.hasNext())
            .hasPrevious(slice.hasPrevious())
            .sort(slice.getSort())
            .numberOfElements(mappedContent.size())
            .empty(mappedContent.isEmpty())
            .build();
    }
}
```

#### Parametri di Paginazione
```java
@Data
@Builder
public class PaginationParams {
    
    @Min(value = 0, message = "Il numero di pagina deve essere >= 0")
    private int page = 0;
    
    @Min(value = 1, message = "La dimensione della pagina deve essere >= 1")
    @Max(value = 100, message = "La dimensione della pagina deve essere <= 100")
    private int size = 10;
    
    private String sortBy = "releaseDate";
    
    @Pattern(regexp = "^(asc|desc)$", message = "La direzione deve essere 'asc' o 'desc'")
    private String sortDirection = "desc";
    
    // Metodi di utilità
    public Pageable toPageable() {
        Sort.Direction direction = "desc".equalsIgnoreCase(sortDirection)
            ? Sort.Direction.DESC
            : Sort.Direction.ASC;
        
        return PageRequest.of(page, size, Sort.by(direction, sortBy));
    }
    
    public Pageable toPageable(String defaultSortBy) {
        String actualSortBy = StringUtils.hasText(sortBy) ? sortBy : defaultSortBy;
        
        Sort.Direction direction = "desc".equalsIgnoreCase(sortDirection)
            ? Sort.Direction.DESC
            : Sort.Direction.ASC;
        
        return PageRequest.of(page, size, Sort.by(direction, actualSortBy));
    }
    
    // Validazione
    public void validate() {
        if (page < 0) {
            throw new ValidationException("Il numero di pagina deve essere >= 0");
        }
        
        if (size < 1 || size > 100) {
            throw new ValidationException(
                "La dimensione della pagina deve essere tra 1 e 100"
            );
        }
        
        if (StringUtils.hasText(sortDirection) && 
            !"asc".equalsIgnoreCase(sortDirection) && 
            !"desc".equalsIgnoreCase(sortDirection)) {
            throw new ValidationException(
                "La direzione di ordinamento deve essere 'asc' o 'desc'"
            );
        }
    }
}
```

### 4.2.4 Controller con Paginazione

#### REST Controller
```java
@RestController
@RequestMapping("/api/v1/games")
@Validated
public class GameController {
    
    private final GameService gameService;
    
    @GetMapping
    public ResponseEntity<PagedResponse<GameDTO>> getGames(
            @Valid @ModelAttribute PaginationParams paginationParams) {
        
        PagedResponse<GameDTO> response = gameService.findAllGames(paginationParams);
        return ResponseEntity.ok(response);
    }
    
    @GetMapping("/search")
    public ResponseEntity<PagedResponse<GameDTO>> searchGames(
            @Valid @ModelAttribute GameSearchCriteria criteria,
            @Valid @ModelAttribute PaginationParams paginationParams) {
        
        PagedResponse<GameDTO> response = gameService.searchGames(
            criteria, paginationParams
        );
        return ResponseEntity.ok(response);
    }
    
    @GetMapping("/summaries")
    public ResponseEntity<SliceResponse<GameSummaryDTO>> getGameSummaries(
            @Valid @ModelAttribute PaginationParams paginationParams) {
        
        SliceResponse<GameSummaryDTO> response = gameService.findGameSummaries(
            paginationParams
        );
        return ResponseEntity.ok(response);
    }
    
    @GetMapping("/by-genre/{genreId}")
    public ResponseEntity<PagedResponse<GameDTO>> getGamesByGenre(
            @PathVariable Long genreId,
            @Valid @ModelAttribute PaginationParams paginationParams) {
        
        PagedResponse<GameDTO> response = gameService.findGamesByGenre(
            genreId, paginationParams
        );
        return ResponseEntity.ok(response);
    }
    
    @GetMapping("/recommendations")
    @PreAuthorize("hasRole('USER')")
    public ResponseEntity<PagedResponse<GameDTO>> getRecommendations(
            @Valid @ModelAttribute PaginationParams paginationParams,
            Authentication authentication) {
        
        UserPrincipal user = (UserPrincipal) authentication.getPrincipal();
        PagedResponse<GameDTO> response = gameService.findRecommendedGames(
            user.getId(), paginationParams
        );
        return ResponseEntity.ok(response);
    }
}
```

### 4.2.5 Ottimizzazioni per Performance

#### Cursor-based Pagination
```java
// Per dataset molto grandi, la paginazione offset-based può essere lenta
// Cursor-based pagination è più efficiente

@Data
@Builder
public class CursorPaginationParams {
    private String cursor; // ID dell'ultimo elemento della pagina precedente
    private int size = 10;
    private String sortBy = "id";
    private String sortDirection = "asc";
}

@Repository
public interface GameRepository extends JpaRepository<Game, Long> {
    
    // Cursor-based pagination
    @Query("SELECT g FROM Game g WHERE g.id > :cursor AND g.active = true ORDER BY g.id ASC")
    List<Game> findGamesAfterCursor(@Param("cursor") Long cursor, Pageable pageable);
    
    @Query("SELECT g FROM Game g WHERE g.releaseDate < :cursor AND g.active = true " +
           "ORDER BY g.releaseDate DESC, g.id DESC")
    List<Game> findGamesBeforeDateCursor(
        @Param("cursor") LocalDate cursor, Pageable pageable);
}

@Data
@Builder
public class CursorResponse<T> {
    private List<T> content;
    private String nextCursor;
    private String previousCursor;
    private boolean hasNext;
    private boolean hasPrevious;
    private int size;
}
```

#### Caching per Query Frequenti
```java
@Service
public class GameService {
    
    @Cacheable(value = "popularGames", key = "#pageable.pageNumber + '_' + #pageable.pageSize")
    public PagedResponse<GameDTO> findPopularGames(Pageable pageable) {
        Page<Game> games = gameRepository.findPopularGames(pageable);
        
        List<GameDTO> gameDTOs = games.getContent().stream()
            .map(gameMapper::toDTO)
            .collect(Collectors.toList());
        
        return PagedResponse.of(games, gameDTOs);
    }
    
    @Cacheable(value = "gameSearch", 
               key = "#criteria.hashCode() + '_' + #pageable.pageNumber + '_' + #pageable.pageSize")
    public PagedResponse<GameDTO> searchGamesWithCache(
            GameSearchCriteria criteria, Pageable pageable) {
        
        // Implementazione con cache
        return searchGames(criteria, PaginationParams.builder()
            .page(pageable.getPageNumber())
            .size(pageable.getPageSize())
            .build());
    }
}
```

## 4.3 Best Practices

### 4.3.1 Performance

1. **Utilizzare Projection per Liste**
```java
// Invece di caricare entità complete per le liste
@Query("SELECT new com.videogames.dto.GameSummaryDTO(" +
       "g.id, g.title, g.price, d.name, g.userRating) " +
       "FROM Game g JOIN g.developer d WHERE g.active = true")
Page<GameSummaryDTO> findGameSummariesOptimized(Pageable pageable);
```

2. **Limitare la Dimensione delle Pagine**
```java
public static final int MAX_PAGE_SIZE = 100;
public static final int DEFAULT_PAGE_SIZE = 10;

private int validatePageSize(int size) {
    return Math.min(Math.max(1, size), MAX_PAGE_SIZE);
}
```

3. **Utilizzare Slice per Performance**
```java
// Quando non serve il count totale, usa Slice
Slice<Game> games = gameRepository.findSliceByActiveTrue(pageable);
```

4. **Indicizzazione Database**
```sql
-- Indici per query di ricerca frequenti
CREATE INDEX idx_games_title ON games(title);
CREATE INDEX idx_games_release_date ON games(release_date);
CREATE INDEX idx_games_price ON games(price);
CREATE INDEX idx_games_user_rating ON games(user_rating);
CREATE INDEX idx_games_active_release_date ON games(active, release_date);

-- Indice composto per filtri comuni
CREATE INDEX idx_games_search ON games(active, developer_id, price, release_date);
```

### 4.3.2 Sicurezza

1. **Validazione Input**
2. **Limitazione Risorse**
3. **Sanitizzazione Query**

Le query dinamiche e la paginazione sono tecniche fondamentali per creare applicazioni scalabili e user-friendly, permettendo agli utenti di navigare e filtrare grandi quantità di dati in modo efficiente.