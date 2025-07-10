# 5. Service Layer e Mappers

## 5.1 Mappers con MapStruct

### 5.1.1 Configurazione Base MapStruct

**MapperConfig.java**:
```java
package mc.videogiochi.mapper.config;

import org.mapstruct.MapperConfig;
import org.mapstruct.ReportingPolicy;
import org.mapstruct.NullValuePropertyMappingStrategy;
import org.mapstruct.NullValueCheckStrategy;

@MapperConfig(
    componentModel = "spring",
    unmappedTargetPolicy = ReportingPolicy.WARN,
    nullValuePropertyMappingStrategy = NullValuePropertyMappingStrategy.IGNORE,
    nullValueCheckStrategy = NullValueCheckStrategy.ALWAYS
)
public interface MapperConfig {
}
```

### 5.1.2 Mapper Principali

**GameMapper.java**:
```java
package mc.videogiochi.mapper;

import mc.videogiochi.domain.entity.*;
import mc.videogiochi.dto.request.GameCreateRequest;
import mc.videogiochi.dto.request.GameUpdateRequest;
import mc.videogiochi.dto.response.*;
import mc.videogiochi.mapper.config.MapperConfig;
import org.mapstruct.*;
import org.springframework.beans.factory.annotation.Autowired;

import java.util.List;
import java.util.Set;

@Mapper(config = MapperConfig.class, uses = {DeveloperMapper.class, PublisherMapper.class, GenreMapper.class, PlatformMapper.class})
public abstract class GameMapper {
    
    @Autowired
    protected UserGameListRepository userGameListRepository;
    
    // Entity to DTO
    @Mapping(target = "userStatus", source = ".", qualifiedByName = "mapUserStatus")
    public abstract GameDTO toDTO(Game game);
    
    @Mapping(target = "developerName", source = "developer.name")
    @Mapping(target = "publisherName", source = "publisher.name")
    @Mapping(target = "inWishlist", ignore = true)
    @Mapping(target = "isPlayed", ignore = true)
    @Mapping(target = "userRating", ignore = true)
    public abstract GameSummaryDTO toSummaryDTO(Game game);
    
    public abstract List<GameDTO> toDTOList(List<Game> games);
    
    public abstract List<GameSummaryDTO> toSummaryDTOList(List<Game> games);
    
    // Request to Entity
    @Mapping(target = "id", ignore = true)
    @Mapping(target = "createdAt", ignore = true)
    @Mapping(target = "updatedAt", ignore = true)
    @Mapping(target = "version", ignore = true)
    @Mapping(target = "averageRating", constant = "0")
    @Mapping(target = "totalRatings", constant = "0")
    @Mapping(target = "isActive", constant = "true")
    @Mapping(target = "totalInWishlists", constant = "0")
    @Mapping(target = "totalCompleted", constant = "0")
    @Mapping(target = "popularityScore", constant = "0")
    @Mapping(target = "developer", source = "developerId", qualifiedByName = "mapDeveloper")
    @Mapping(target = "publisher", source = "publisherId", qualifiedByName = "mapPublisher")
    @Mapping(target = "genres", source = "genreIds", qualifiedByName = "mapGenres")
    @Mapping(target = "platforms", source = "platformIds", qualifiedByName = "mapPlatforms")
    @Mapping(target = "userGameLists", ignore = true)
    public abstract Game toEntity(GameCreateRequest request);
    
    // Update mapping
    @Mapping(target = "id", ignore = true)
    @Mapping(target = "createdAt", ignore = true)
    @Mapping(target = "updatedAt", ignore = true)
    @Mapping(target = "version", ignore = true)
    @Mapping(target = "averageRating", ignore = true)
    @Mapping(target = "totalRatings", ignore = true)
    @Mapping(target = "totalInWishlists", ignore = true)
    @Mapping(target = "totalCompleted", ignore = true)
    @Mapping(target = "popularityScore", ignore = true)
    @Mapping(target = "developer", source = "developerId", qualifiedByName = "mapDeveloper")
    @Mapping(target = "publisher", source = "publisherId", qualifiedByName = "mapPublisher")
    @Mapping(target = "genres", source = "genreIds", qualifiedByName = "mapGenres")
    @Mapping(target = "platforms", source = "platformIds", qualifiedByName = "mapPlatforms")
    @Mapping(target = "userGameLists", ignore = true)
    public abstract void updateEntity(@MappingTarget Game game, GameUpdateRequest request);
    
    // Custom mapping methods
    @Named("mapUserStatus")
    protected UserGameStatusDTO mapUserStatus(Game game) {
        // Questo metodo sarà implementato nel service per includere il contesto utente
        return null;
    }
    
    @Named("mapDeveloper")
    protected Developer mapDeveloper(Long developerId) {
        if (developerId == null) return null;
        Developer developer = new Developer();
        developer.setId(developerId);
        return developer;
    }
    
    @Named("mapPublisher")
    protected Publisher mapPublisher(Long publisherId) {
        if (publisherId == null) return null;
        Publisher publisher = new Publisher();
        publisher.setId(publisherId);
        return publisher;
    }
    
    @Named("mapGenres")
    protected Set<Genre> mapGenres(List<Long> genreIds) {
        if (genreIds == null || genreIds.isEmpty()) return Set.of();
        return genreIds.stream()
                .map(id -> {
                    Genre genre = new Genre();
                    genre.setId(id);
                    return genre;
                })
                .collect(java.util.stream.Collectors.toSet());
    }
    
    @Named("mapPlatforms")
    protected Set<Platform> mapPlatforms(List<Long> platformIds) {
        if (platformIds == null || platformIds.isEmpty()) return Set.of();
        return platformIds.stream()
                .map(id -> {
                    Platform platform = new Platform();
                    platform.setId(id);
                    return platform;
                })
                .collect(java.util.stream.Collectors.toSet());
    }
}
```

**UserMapper.java**:
```java
package mc.videogiochi.mapper;

import mc.videogiochi.domain.entity.User;
import mc.videogiochi.dto.request.UserRegistrationRequest;
import mc.videogiochi.dto.request.UserUpdateRequest;
import mc.videogiochi.dto.response.UserDTO;
import mc.videogiochi.dto.response.UserProfileDTO;
import mc.videogiochi.mapper.config.MapperConfig;
import org.mapstruct.*;

import java.util.List;

@Mapper(config = MapperConfig.class)
public interface UserMapper {
    
    // Entity to DTO
    @Mapping(target = "fullName", source = ".", qualifiedByName = "getFullName")
    @Mapping(target = "roleNames", source = "roles", qualifiedByName = "mapRoleNames")
    UserDTO toDTO(User user);
    
    @Mapping(target = "fullName", source = ".", qualifiedByName = "getFullName")
    @Mapping(target = "totalGames", ignore = true)
    @Mapping(target = "totalWishlist", ignore = true)
    @Mapping(target = "averageRating", ignore = true)
    @Mapping(target = "favoriteGenres", ignore = true)
    UserProfileDTO toProfileDTO(User user);
    
    List<UserDTO> toDTOList(List<User> users);
    
    // Request to Entity
    @Mapping(target = "id", ignore = true)
    @Mapping(target = "createdAt", ignore = true)
    @Mapping(target = "updatedAt", ignore = true)
    @Mapping(target = "version", ignore = true)
    @Mapping(target = "password", ignore = true) // Sarà gestito dal service
    @Mapping(target = "isActive", constant = "true")
    @Mapping(target = "emailVerified", constant = "false")
    @Mapping(target = "lastLogin", ignore = true)
    @Mapping(target = "roles", ignore = true)
    @Mapping(target = "gameLists", ignore = true)
    @Mapping(target = "customLists", ignore = true)
    User toEntity(UserRegistrationRequest request);
    
    // Update mapping
    @Mapping(target = "id", ignore = true)
    @Mapping(target = "createdAt", ignore = true)
    @Mapping(target = "updatedAt", ignore = true)
    @Mapping(target = "version", ignore = true)
    @Mapping(target = "username", ignore = true)
    @Mapping(target = "email", ignore = true)
    @Mapping(target = "password", ignore = true)
    @Mapping(target = "isActive", ignore = true)
    @Mapping(target = "emailVerified", ignore = true)
    @Mapping(target = "lastLogin", ignore = true)
    @Mapping(target = "roles", ignore = true)
    @Mapping(target = "gameLists", ignore = true)
    @Mapping(target = "customLists", ignore = true)
    void updateEntity(@MappingTarget User user, UserUpdateRequest request);
    
    // Custom mapping methods
    @Named("getFullName")
    default String getFullName(User user) {
        return user.getFullName();
    }
    
    @Named("mapRoleNames")
    default List<String> mapRoleNames(Set<Role> roles) {
        return roles.stream()
                .map(Role::getName)
                .toList();
    }
}
```

## 5.2 Service Layer

### 5.2.1 Service Base

**BaseService.java**:
```java
package mc.videogiochi.service;

import mc.videogiochi.exception.ResourceNotFoundException;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.jpa.repository.JpaRepository;

import java.util.List;
import java.util.Optional;

public abstract class BaseService<T, ID, R extends JpaRepository<T, ID>> {
    
    protected final R repository;
    
    protected BaseService(R repository) {
        this.repository = repository;
    }
    
    public T findById(ID id) {
        return repository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException(
                    getEntityName() + " non trovato con ID: " + id));
    }
    
    public Optional<T> findByIdOptional(ID id) {
        return repository.findById(id);
    }
    
    public List<T> findAll() {
        return repository.findAll();
    }
    
    public Page<T> findAll(Pageable pageable) {
        return repository.findAll(pageable);
    }
    
    public T save(T entity) {
        return repository.save(entity);
    }
    
    public List<T> saveAll(List<T> entities) {
        return repository.saveAll(entities);
    }
    
    public void deleteById(ID id) {
        if (!repository.existsById(id)) {
            throw new ResourceNotFoundException(
                getEntityName() + " non trovato con ID: " + id);
        }
        repository.deleteById(id);
    }
    
    public void delete(T entity) {
        repository.delete(entity);
    }
    
    public boolean existsById(ID id) {
        return repository.existsById(id);
    }
    
    public long count() {
        return repository.count();
    }
    
    protected abstract String getEntityName();
}
```

### 5.2.2 Game Service

**GameService.java**:
```java
package mc.videogiochi.service;

import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import mc.videogiochi.domain.entity.*;
import mc.videogiochi.domain.enums.ListType;
import mc.videogiochi.dto.request.*;
import mc.videogiochi.dto.response.*;
import mc.videogiochi.exception.*;
import mc.videogiochi.mapper.GameMapper;
import mc.videogiochi.repository.jpa.*;
import mc.videogiochi.service.cache.CacheService;
import org.springframework.cache.annotation.*;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.math.BigDecimal;
import java.util.List;
import java.util.Set;

@Service
@RequiredArgsConstructor
@Slf4j
@Transactional(readOnly = true)
public class GameService extends BaseService<Game, Long, GameRepository> {
    
    private final GameMapper gameMapper;
    private final DeveloperRepository developerRepository;
    private final PublisherRepository publisherRepository;
    private final GenreRepository genreRepository;
    private final PlatformRepository platformRepository;
    private final UserGameListRepository userGameListRepository;
    private final CacheService cacheService;
    private final SecurityService securityService;
    
    @Override
    protected String getEntityName() {
        return "Game";
    }
    
    // ==================== CRUD Operations ====================
    
    @Cacheable(value = "games", key = "#id")
    public GameDTO findGameById(Long id) {
        Game game = findById(id);
        GameDTO gameDTO = gameMapper.toDTO(game);
        
        // Aggiungi informazioni utente se autenticato
        securityService.getCurrentUser().ifPresent(user -> {
            UserGameStatusDTO userStatus = getUserGameStatus(game, user);
            gameDTO.setUserStatus(userStatus);
        });
        
        return gameDTO;
    }
    
    @Cacheable(value = "games", key = "'all-' + #pageable.pageNumber + '-' + #pageable.pageSize")
    public PagedResponse<GameSummaryDTO> findAllGames(Pageable pageable) {
        Page<Game> gamesPage = repository.findByIsActiveTrue(pageable);
        List<GameSummaryDTO> gameDTOs = gameMapper.toSummaryDTOList(gamesPage.getContent());
        
        // Aggiungi informazioni utente per ogni gioco
        securityService.getCurrentUser().ifPresent(user -> 
            enrichWithUserInfo(gameDTOs, user));
        
        return PagedResponse.of(
            gameDTOs,
            gamesPage.getNumber(),
            gamesPage.getSize(),
            gamesPage.getTotalElements()
        );
    }
    
    public PagedResponse<GameSummaryDTO> searchGames(GameSearchCriteria criteria, Pageable pageable) {
        Page<Game> gamesPage = repository.findByCriteria(criteria, pageable);
        List<GameSummaryDTO> gameDTOs = gameMapper.toSummaryDTOList(gamesPage.getContent());
        
        securityService.getCurrentUser().ifPresent(user -> 
            enrichWithUserInfo(gameDTOs, user));
        
        return PagedResponse.of(
            gameDTOs,
            gamesPage.getNumber(),
            gamesPage.getSize(),
            gamesPage.getTotalElements()
        );
    }
    
    @Transactional
    @CacheEvict(value = "games", allEntries = true)
    public GameDTO createGame(GameCreateRequest request) {
        log.info("Creazione nuovo gioco: {}", request.getTitle());
        
        // Validazioni
        validateGameCreation(request);
        
        // Mapping e salvataggio
        Game game = gameMapper.toEntity(request);
        
        // Carica entità correlate
        loadRelatedEntities(game, request);
        
        Game savedGame = repository.save(game);
        log.info("Gioco creato con ID: {}", savedGame.getId());
        
        return gameMapper.toDTO(savedGame);
    }
    
    @Transactional
    @CacheEvict(value = "games", key = "#id")
    public GameDTO updateGame(Long id, GameUpdateRequest request) {
        log.info("Aggiornamento gioco ID: {}", id);
        
        Game existingGame = findById(id);
        
        // Validazioni
        validateGameUpdate(existingGame, request);
        
        // Update
        gameMapper.updateEntity(existingGame, request);
        
        // Aggiorna entità correlate se necessario
        if (request.getGenreIds() != null) {
            updateGameGenres(existingGame, request.getGenreIds());
        }
        
        if (request.getPlatformIds() != null) {
            updateGamePlatforms(existingGame, request.getPlatformIds());
        }
        
        Game updatedGame = repository.save(existingGame);
        log.info("Gioco aggiornato: {}", updatedGame.getTitle());
        
        return gameMapper.toDTO(updatedGame);
    }
    
    @Transactional
    @CacheEvict(value = "games", key = "#id")
    public void deleteGame(Long id) {
        log.info("Eliminazione gioco ID: {}", id);
        
        Game game = findById(id);
        
        // Soft delete
        game.setIsActive(false);
        repository.save(game);
        
        log.info("Gioco eliminato (soft delete): {}", game.getTitle());
    }
    
    // ==================== Search & Filter Operations ====================
    
    public PagedResponse<GameSummaryDTO> findGamesByTitle(String title, Pageable pageable) {
        Page<Game> gamesPage = repository.findByTitleContainingIgnoreCaseAndIsActiveTrue(title, pageable);
        List<GameSummaryDTO> gameDTOs = gameMapper.toSummaryDTOList(gamesPage.getContent());
        
        securityService.getCurrentUser().ifPresent(user -> 
            enrichWithUserInfo(gameDTOs, user));
        
        return PagedResponse.of(
            gameDTOs,
            gamesPage.getNumber(),
            gamesPage.getSize(),
            gamesPage.getTotalElements()
        );
    }
    
    public PagedResponse<GameSummaryDTO> findGamesByGenre(Long genreId, Pageable pageable) {
        Page<Game> gamesPage = repository.findByGenreId(genreId, pageable);
        List<GameSummaryDTO> gameDTOs = gameMapper.toSummaryDTOList(gamesPage.getContent());
        
        securityService.getCurrentUser().ifPresent(user -> 
            enrichWithUserInfo(gameDTOs, user));
        
        return PagedResponse.of(
            gameDTOs,
            gamesPage.getNumber(),
            gamesPage.getSize(),
            gamesPage.getTotalElements()
        );
    }
    
    public PagedResponse<GameSummaryDTO> findGamesByPlatform(Long platformId, Pageable pageable) {
        Page<Game> gamesPage = repository.findByPlatformId(platformId, pageable);
        List<GameSummaryDTO> gameDTOs = gameMapper.toSummaryDTOList(gamesPage.getContent());
        
        securityService.getCurrentUser().ifPresent(user -> 
            enrichWithUserInfo(gameDTOs, user));
        
        return PagedResponse.of(
            gameDTOs,
            gamesPage.getNumber(),
            gamesPage.getSize(),
            gamesPage.getTotalElements()
        );
    }
    
    public PagedResponse<GameSummaryDTO> findGamesByDeveloper(Long developerId, Pageable pageable) {
        Page<Game> gamesPage = repository.findByDeveloperId(developerId, pageable);
        List<GameSummaryDTO> gameDTOs = gameMapper.toSummaryDTOList(gamesPage.getContent());
        
        securityService.getCurrentUser().ifPresent(user -> 
            enrichWithUserInfo(gameDTOs, user));
        
        return PagedResponse.of(
            gameDTOs,
            gamesPage.getNumber(),
            gamesPage.getSize(),
            gamesPage.getTotalElements()
        );
    }
    
    // ==================== Recommendation System ====================
    
    public PagedResponse<GameSummaryDTO> getRecommendationsForUser(Long userId, Pageable pageable) {
        Page<Game> gamesPage = repository.findRecommendationsForUser(userId, pageable);
        List<GameSummaryDTO> gameDTOs = gameMapper.toSummaryDTOList(gamesPage.getContent());
        
        return PagedResponse.of(
            gameDTOs,
            gamesPage.getNumber(),
            gamesPage.getSize(),
            gamesPage.getTotalElements()
        );
    }
    
    public List<GameSummaryDTO> getSimilarGames(Long gameId, int limit) {
        Pageable pageable = Pageable.ofSize(limit);
        List<Game> similarGames = repository.findSimilarGames(gameId, pageable);
        return gameMapper.toSummaryDTOList(similarGames);
    }
    
    // ==================== Statistics ====================
    
    @Cacheable(value = "statistics", key = "'game-stats'")
    public GameStatisticsDTO getGameStatistics() {
        long totalGames = repository.countActiveGames();
        BigDecimal averageRating = repository.getOverallAverageRating();
        
        return GameStatisticsDTO.builder()
                .totalGames(totalGames)
                .averageRating(averageRating)
                .build();
    }
    
    // ==================== Rating System ====================
    
    @Transactional
    @CacheEvict(value = "games", key = "#gameId")
    public void updateGameRating(Long gameId, BigDecimal newRating) {
        Game game = findById(gameId);
        game.updateRating(newRating);
        repository.save(game);
        
        // Aggiorna popularity score
        updatePopularityScore(game);
    }
    
    // ==================== Helper Methods ====================
    
    private void validateGameCreation(GameCreateRequest request) {
        // Verifica che developer esista
        if (!developerRepository.existsById(request.getDeveloperId())) {
            throw new ResourceNotFoundException("Developer non trovato con ID: " + request.getDeveloperId());
        }
        
        // Verifica che publisher esista (se specificato)
        if (request.getPublisherId() != null && !publisherRepository.existsById(request.getPublisherId())) {
            throw new ResourceNotFoundException("Publisher non trovato con ID: " + request.getPublisherId());
        }
        
        // Verifica che tutti i generi esistano
        List<Long> existingGenreIds = genreRepository.findAllById(request.getGenreIds())
                .stream().map(Genre::getId).toList();
        if (existingGenreIds.size() != request.getGenreIds().size()) {
            throw new ValidationException("Uno o più generi specificati non esistono");
        }
        
        // Verifica che tutte le piattaforme esistano
        List<Long> existingPlatformIds = platformRepository.findAllById(request.getPlatformIds())
                .stream().map(Platform::getId).toList();
        if (existingPlatformIds.size() != request.getPlatformIds().size()) {
            throw new ValidationException("Una o più piattaforme specificate non esistono");
        }
    }
    
    private void validateGameUpdate(Game existingGame, GameUpdateRequest request) {
        // Validazioni simili a quelle di creazione
        // ma solo per i campi che vengono aggiornati
    }
    
    private void loadRelatedEntities(Game game, GameCreateRequest request) {
        // Carica developer
        Developer developer = developerRepository.findById(request.getDeveloperId())
                .orElseThrow(() -> new ResourceNotFoundException("Developer non trovato"));
        game.setDeveloper(developer);
        
        // Carica publisher se specificato
        if (request.getPublisherId() != null) {
            Publisher publisher = publisherRepository.findById(request.getPublisherId())
                    .orElseThrow(() -> new ResourceNotFoundException("Publisher non trovato"));
            game.setPublisher(publisher);
        }
        
        // Carica generi
        Set<Genre> genres = Set.copyOf(genreRepository.findAllById(request.getGenreIds()));
        game.setGenres(genres);
        
        // Carica piattaforme
        Set<Platform> platforms = Set.copyOf(platformRepository.findAllById(request.getPlatformIds()));
        game.setPlatforms(platforms);
    }
    
    private void updateGameGenres(Game game, List<Long> genreIds) {
        game.getGenres().clear();
        Set<Genre> newGenres = Set.copyOf(genreRepository.findAllById(genreIds));
        game.setGenres(newGenres);
    }
    
    private void updateGamePlatforms(Game game, List<Long> platformIds) {
        game.getPlatforms().clear();
        Set<Platform> newPlatforms = Set.copyOf(platformRepository.findAllById(platformIds));
        game.setPlatforms(newPlatforms);
    }
    
    private UserGameStatusDTO getUserGameStatus(Game game, User user) {
        return userGameListRepository.findByUserAndGame(user, game)
                .map(userGameList -> UserGameStatusDTO.builder()
                        .listType(userGameList.getListType())
                        .rating(userGameList.getRating())
                        .playedHours(userGameList.getPlayedHours())
                        .completionPercentage(userGameList.getCompletionPercentage())
                        .notes(userGameList.getNotes())
                        .addedAt(userGameList.getCreatedAt())
                        .build())
                .orElse(null);
    }
    
    private void enrichWithUserInfo(List<GameSummaryDTO> gameDTOs, User user) {
        List<Long> gameIds = gameDTOs.stream().map(GameSummaryDTO::getId).toList();
        
        List<UserGameList> userGameLists = userGameListRepository.findByUserAndGameIdIn(user, gameIds);
        
        Map<Long, UserGameList> gameListMap = userGameLists.stream()
                .collect(Collectors.toMap(
                    ugl -> ugl.getGame().getId(),
                    Function.identity()
                ));
        
        gameDTOs.forEach(gameDTO -> {
            UserGameList userGameList = gameListMap.get(gameDTO.getId());
            if (userGameList != null) {
                gameDTO.setInWishlist(userGameList.getListType() == ListType.WISHLIST);
                gameDTO.setIsPlayed(userGameList.getListType() == ListType.PLAYED);
                gameDTO.setUserRating(userGameList.getRating());
            } else {
                gameDTO.setInWishlist(false);
                gameDTO.setIsPlayed(false);
                gameDTO.setUserRating(null);
            }
        });
    }
    
    private void updatePopularityScore(Game game) {
        // Algoritmo semplificato per calcolare popularity score
        // Basato su: rating medio, numero di rating, numero in wishlist, ecc.
        BigDecimal popularityScore = game.getAverageRating()
                .multiply(BigDecimal.valueOf(Math.log(game.getTotalRatings() + 1)))
                .multiply(BigDecimal.valueOf(Math.log(game.getTotalInWishlists() + 1)));
        
        game.setPopularityScore(popularityScore);
    }
}
```

### 5.2.3 User Service

**UserService.java**:
```java
package mc.videogiochi.service;

import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import mc.videogiochi.domain.entity.*;
import mc.videogiochi.dto.request.*;
import mc.videogiochi.dto.response.*;
import mc.videogiochi.exception.*;
import mc.videogiochi.mapper.UserMapper;
import mc.videogiochi.repository.jpa.*;
import org.springframework.cache.annotation.*;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.time.LocalDateTime;
import java.util.List;
import java.util.Optional;

@Service
@RequiredArgsConstructor
@Slf4j
@Transactional(readOnly = true)
public class UserService extends BaseService<User, Long, UserRepository> {
    
    private final UserMapper userMapper;
    private final PasswordEncoder passwordEncoder;
    private final RoleRepository roleRepository;
    private final EmailService emailService;
    
    @Override
    protected String getEntityName() {
        return "User";
    }
    
    // ==================== CRUD Operations ====================
    
    @Cacheable(value = "users", key = "#id")
    public UserDTO findUserById(Long id) {
        User user = findById(id);
        return userMapper.toDTO(user);
    }
    
    public UserProfileDTO findUserProfile(Long id) {
        User user = findById(id);
        UserProfileDTO profile = userMapper.toProfileDTO(user);
        
        // Aggiungi statistiche
        enrichUserProfile(profile, user);
        
        return profile;
    }
    
    public Optional<UserDTO> findByUsername(String username) {
        return repository.findByUsername(username)
                .map(userMapper::toDTO);
    }
    
    public Optional<UserDTO> findByEmail(String email) {
        return repository.findByEmail(email)
                .map(userMapper::toDTO);
    }
    
    public PagedResponse<UserDTO> findAllUsers(Pageable pageable) {
        Page<User> usersPage = repository.findByIsActiveTrue(pageable);
        List<UserDTO> userDTOs = userMapper.toDTOList(usersPage.getContent());
        
        return PagedResponse.of(
            userDTOs,
            usersPage.getNumber(),
            usersPage.getSize(),
            usersPage.getTotalElements()
        );
    }
    
    @Transactional
    @CacheEvict(value = "users", allEntries = true)
    public UserDTO createUser(UserRegistrationRequest request) {
        log.info("Registrazione nuovo utente: {}", request.getUsername());
        
        // Validazioni
        validateUserRegistration(request);
        
        // Mapping e preparazione
        User user = userMapper.toEntity(request);
        user.setPassword(passwordEncoder.encode(request.getPassword()));
        
        // Assegna ruolo di default
        Role userRole = roleRepository.findByName("USER")
                .orElseThrow(() -> new SystemException("Ruolo USER non trovato"));
        user.addRole(userRole);
        
        User savedUser = repository.save(user);
        log.info("Utente registrato con ID: {}", savedUser.getId());
        
        // Invia email di verifica
        emailService.sendVerificationEmail(savedUser);
        
        return userMapper.toDTO(savedUser);
    }
    
    @Transactional
    @CacheEvict(value = "users", key = "#id")
    public UserDTO updateUser(Long id, UserUpdateRequest request) {
        log.info("Aggiornamento utente ID: {}", id);
        
        User existingUser = findById(id);
        
        // Validazioni
        validateUserUpdate(existingUser, request);
        
        // Update
        userMapper.updateEntity(existingUser, request);
        
        User updatedUser = repository.save(existingUser);
        log.info("Utente aggiornato: {}", updatedUser.getUsername());
        
        return userMapper.toDTO(updatedUser);
    }
    
    @Transactional
    @CacheEvict(value = "users", key = "#id")
    public void deactivateUser(Long id) {
        log.info("Disattivazione utente ID: {}", id);
        
        User user = findById(id);
        user.setIsActive(false);
        repository.save(user);
        
        log.info("Utente disattivato: {}", user.getUsername());
    }
    
    // ==================== Authentication Operations ====================
    
    @Transactional
    public void updateLastLogin(String username) {
        repository.findByUsername(username)
                .ifPresent(user -> {
                    user.setLastLogin(LocalDateTime.now());
                    repository.save(user);
                });
    }
    
    @Transactional
    public void verifyEmail(String username) {
        User user = repository.findByUsername(username)
                .orElseThrow(() -> new ResourceNotFoundException("Utente non trovato: " + username));
        
        user.setEmailVerified(true);
        repository.save(user);
        
        log.info("Email verificata per utente: {}", username);
    }
    
    @Transactional
    public void changePassword(Long userId, ChangePasswordRequest request) {
        User user = findById(userId);
        
        // Verifica password corrente
        if (!passwordEncoder.matches(request.getCurrentPassword(), user.getPassword())) {
            throw new ValidationException("Password corrente non corretta");
        }
        
        // Aggiorna password
        user.setPassword(passwordEncoder.encode(request.getNewPassword()));
        repository.save(user);
        
        log.info("Password cambiata per utente: {}", user.getUsername());
    }
    
    // ==================== Role Management ====================
    
    @Transactional
    @CacheEvict(value = "users", key = "#userId")
    public void addRoleToUser(Long userId, String roleName) {
        User user = findById(userId);
        Role role = roleRepository.findByName(roleName)
                .orElseThrow(() -> new ResourceNotFoundException("Ruolo non trovato: " + roleName));
        
        user.addRole(role);
        repository.save(user);
        
        log.info("Ruolo {} aggiunto all'utente {}", roleName, user.getUsername());
    }
    
    @Transactional
    @CacheEvict(value = "users", key = "#userId")
    public void removeRoleFromUser(Long userId, String roleName) {
        User user = findById(userId);
        Role role = roleRepository.findByName(roleName)
                .orElseThrow(() -> new ResourceNotFoundException("Ruolo non trovato: " + roleName));
        
        user.removeRole(role);
        repository.save(user);
        
        log.info("Ruolo {} rimosso dall'utente {}", roleName, user.getUsername());
    }
    
    // ==================== Helper Methods ====================
    
    private void validateUserRegistration(UserRegistrationRequest request) {
        // Verifica username univoco
        if (repository.existsByUsername(request.getUsername())) {
            throw new ValidationException("Username già in uso: " + request.getUsername());
        }
        
        // Verifica email univoca
        if (repository.existsByEmail(request.getEmail())) {
            throw new ValidationException("Email già in uso: " + request.getEmail());
        }
        
        // Validazione password
        validatePassword(request.getPassword());
    }
    
    private void validateUserUpdate(User existingUser, UserUpdateRequest request) {
        // Validazioni per aggiornamento
        // (meno restrittive della registrazione)
    }
    
    private void validatePassword(String password) {
        if (password.length() < 8) {
            throw new ValidationException("Password deve essere di almeno 8 caratteri");
        }
        
        if (!password.matches(".*[A-Z].*")) {
            throw new ValidationException("Password deve contenere almeno una lettera maiuscola");
        }
        
        if (!password.matches(".*[a-z].*")) {
            throw new ValidationException("Password deve contenere almeno una lettera minuscola");
        }
        
        if (!password.matches(".*[0-9].*")) {
            throw new ValidationException("Password deve contenere almeno un numero");
        }
    }
    
    private void enrichUserProfile(UserProfileDTO profile, User user) {
        // Calcola statistiche utente
        long totalGames = user.getGameLists().size();
        long totalWishlist = user.getGameLists().stream()
                .mapToLong(ugl -> ugl.getListType() == ListType.WISHLIST ? 1 : 0)
                .sum();
        
        Double averageRating = user.getGameLists().stream()
                .filter(ugl -> ugl.getRating() != null)
                .mapToInt(UserGameList::getRating)
                .average()
                .orElse(0.0);
        
        profile.setTotalGames(totalGames);
        profile.setTotalWishlist(totalWishlist);
        profile.setAverageRating(BigDecimal.valueOf(averageRating));
        
        // Calcola generi preferiti
        Map<String, Long> genreCount = user.getGameLists().stream()
                .filter(ugl -> ugl.getRating() != null && ugl.getRating() >= 7)
                .flatMap(ugl -> ugl.getGame().getGenres().stream())
                .collect(Collectors.groupingBy(Genre::getName, Collectors.counting()));
        
        List<String> favoriteGenres = genreCount.entrySet().stream()
                .sorted(Map.Entry.<String, Long>comparingByValue().reversed())
                .limit(5)
                .map(Map.Entry::getKey)
                .toList();
        
        profile.setFavoriteGenres(favoriteGenres);
    }
}
```

Questo documento continua con l'implementazione completa dei service layer, includendo la gestione delle liste utente, il sistema di cache, e tutti gli altri servizi necessari per il funzionamento dell'applicazione.