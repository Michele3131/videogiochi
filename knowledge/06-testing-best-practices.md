# 6. Testing e Best Practices

## 6.1 Piramide dei Test

### 6.1.1 Struttura della Piramide

```
        /\     E2E Tests (pochi, lenti, costosi)
       /  \
      /    \   Integration Tests (alcuni, medi)
     /      \
    /        \  Unit Tests (molti, veloci, economici)
   /__________\
```

**Unit Tests (70%)**: Test di singole unità di codice (metodi, classi)
**Integration Tests (20%)**: Test di interazione tra componenti
**End-to-End Tests (10%)**: Test dell'intero flusso applicativo

### 6.1.2 Principi FIRST

- **Fast**: I test devono essere veloci
- **Independent**: I test non devono dipendere l'uno dall'altro
- **Repeatable**: I test devono produrre sempre lo stesso risultato
- **Self-Validating**: I test devono avere un risultato chiaro (pass/fail)
- **Timely**: I test devono essere scritti al momento giusto (TDD/BDD)

## 6.2 Unit Testing

### 6.2.1 Test di Repository Layer

#### Test con @DataJpaTest
```java
@DataJpaTest
@TestPropertySource(properties = {
    "spring.jpa.hibernate.ddl-auto=create-drop",
    "spring.datasource.url=jdbc:h2:mem:testdb",
    "spring.jpa.show-sql=true"
})
class GameRepositoryTest {
    
    @Autowired
    private TestEntityManager entityManager;
    
    @Autowired
    private GameRepository gameRepository;
    
    private Developer testDeveloper;
    private Publisher testPublisher;
    private Genre testGenre;
    private Platform testPlatform;
    
    @BeforeEach
    void setUp() {
        // Setup test data
        testDeveloper = Developer.builder()
            .name("Test Developer")
            .foundedYear(2000)
            .build();
        entityManager.persistAndFlush(testDeveloper);
        
        testPublisher = Publisher.builder()
            .name("Test Publisher")
            .foundedYear(1995)
            .build();
        entityManager.persistAndFlush(testPublisher);
        
        testGenre = Genre.builder()
            .name("Action")
            .description("Action games")
            .build();
        entityManager.persistAndFlush(testGenre);
        
        testPlatform = Platform.builder()
            .name("PC")
            .manufacturer("Various")
            .build();
        entityManager.persistAndFlush(testPlatform);
    }
    
    @Test
    @DisplayName("Should find games by title containing ignore case")
    void shouldFindGamesByTitleContainingIgnoreCase() {
        // Given
        Game game1 = createTestGame("The Legend of Zelda", testDeveloper, testPublisher);
        Game game2 = createTestGame("Super Mario Bros", testDeveloper, testPublisher);
        Game game3 = createTestGame("Zelda: Breath of the Wild", testDeveloper, testPublisher);
        
        entityManager.persistAndFlush(game1);
        entityManager.persistAndFlush(game2);
        entityManager.persistAndFlush(game3);
        
        Pageable pageable = PageRequest.of(0, 10);
        
        // When
        Page<Game> result = gameRepository.findByTitleContainingIgnoreCaseAndActiveTrue(
            "zelda", pageable
        );
        
        // Then
        assertThat(result.getContent()).hasSize(2);
        assertThat(result.getContent())
            .extracting(Game::getTitle)
            .containsExactlyInAnyOrder(
                "The Legend of Zelda",
                "Zelda: Breath of the Wild"
            );
    }
    
    @Test
    @DisplayName("Should find games by developer")
    void shouldFindGamesByDeveloper() {
        // Given
        Developer anotherDeveloper = Developer.builder()
            .name("Another Developer")
            .foundedYear(2010)
            .build();
        entityManager.persistAndFlush(anotherDeveloper);
        
        Game game1 = createTestGame("Game 1", testDeveloper, testPublisher);
        Game game2 = createTestGame("Game 2", anotherDeveloper, testPublisher);
        Game game3 = createTestGame("Game 3", testDeveloper, testPublisher);
        
        entityManager.persistAndFlush(game1);
        entityManager.persistAndFlush(game2);
        entityManager.persistAndFlush(game3);
        
        Pageable pageable = PageRequest.of(0, 10);
        
        // When
        Page<Game> result = gameRepository.findByDeveloperIdAndActiveTrue(
            testDeveloper.getId(), pageable
        );
        
        // Then
        assertThat(result.getContent()).hasSize(2);
        assertThat(result.getContent())
            .extracting(Game::getTitle)
            .containsExactlyInAnyOrder("Game 1", "Game 3");
    }
    
    @Test
    @DisplayName("Should find games by price range")
    void shouldFindGamesByPriceRange() {
        // Given
        Game cheapGame = createTestGame("Cheap Game", testDeveloper, testPublisher);
        cheapGame.setPrice(new BigDecimal("9.99"));
        
        Game midGame = createTestGame("Mid Game", testDeveloper, testPublisher);
        midGame.setPrice(new BigDecimal("29.99"));
        
        Game expensiveGame = createTestGame("Expensive Game", testDeveloper, testPublisher);
        expensiveGame.setPrice(new BigDecimal("59.99"));
        
        entityManager.persistAndFlush(cheapGame);
        entityManager.persistAndFlush(midGame);
        entityManager.persistAndFlush(expensiveGame);
        
        Pageable pageable = PageRequest.of(0, 10);
        
        // When
        Page<Game> result = gameRepository.findByPriceBetweenAndActiveTrue(
            new BigDecimal("20.00"), new BigDecimal("50.00"), pageable
        );
        
        // Then
        assertThat(result.getContent()).hasSize(1);
        assertThat(result.getContent().get(0).getTitle()).isEqualTo("Mid Game");
    }
    
    @Test
    @DisplayName("Should find games with dynamic filters")
    void shouldFindGamesWithDynamicFilters() {
        // Given
        Game actionGame = createTestGame("Action Game", testDeveloper, testPublisher);
        actionGame.setGenres(Set.of(testGenre));
        actionGame.setPlatforms(Set.of(testPlatform));
        actionGame.setPrice(new BigDecimal("39.99"));
        actionGame.setEsrbRating(EsrbRating.TEEN);
        
        entityManager.persistAndFlush(actionGame);
        
        GameSearchCriteria criteria = GameSearchCriteria.builder()
            .title("Action")
            .genreIds(List.of(testGenre.getId()))
            .minPrice(new BigDecimal("30.00"))
            .maxPrice(new BigDecimal("50.00"))
            .esrbRating(EsrbRating.TEEN)
            .build();
        
        Pageable pageable = PageRequest.of(0, 10);
        
        // When
        Page<Game> result = gameRepository.findGamesWithDynamicFilters(criteria, pageable);
        
        // Then
        assertThat(result.getContent()).hasSize(1);
        assertThat(result.getContent().get(0).getTitle()).isEqualTo("Action Game");
    }
    
    @Test
    @DisplayName("Should return empty page when no games match criteria")
    void shouldReturnEmptyPageWhenNoGamesMatchCriteria() {
        // Given
        GameSearchCriteria criteria = GameSearchCriteria.builder()
            .title("Non-existent Game")
            .build();
        
        Pageable pageable = PageRequest.of(0, 10);
        
        // When
        Page<Game> result = gameRepository.findGamesWithDynamicFilters(criteria, pageable);
        
        // Then
        assertThat(result.getContent()).isEmpty();
        assertThat(result.getTotalElements()).isZero();
    }
    
    @Test
    @DisplayName("Should handle pagination correctly")
    void shouldHandlePaginationCorrectly() {
        // Given
        for (int i = 1; i <= 25; i++) {
            Game game = createTestGame("Game " + i, testDeveloper, testPublisher);
            entityManager.persistAndFlush(game);
        }
        
        Pageable firstPage = PageRequest.of(0, 10);
        Pageable secondPage = PageRequest.of(1, 10);
        Pageable thirdPage = PageRequest.of(2, 10);
        
        // When
        Page<Game> page1 = gameRepository.findByActiveTrue(firstPage);
        Page<Game> page2 = gameRepository.findByActiveTrue(secondPage);
        Page<Game> page3 = gameRepository.findByActiveTrue(thirdPage);
        
        // Then
        assertThat(page1.getContent()).hasSize(10);
        assertThat(page1.getTotalElements()).isEqualTo(25);
        assertThat(page1.getTotalPages()).isEqualTo(3);
        assertThat(page1.isFirst()).isTrue();
        assertThat(page1.isLast()).isFalse();
        
        assertThat(page2.getContent()).hasSize(10);
        assertThat(page2.isFirst()).isFalse();
        assertThat(page2.isLast()).isFalse();
        
        assertThat(page3.getContent()).hasSize(5);
        assertThat(page3.isFirst()).isFalse();
        assertThat(page3.isLast()).isTrue();
    }
    
    private Game createTestGame(String title, Developer developer, Publisher publisher) {
        return Game.builder()
            .title(title)
            .description("Test description for " + title)
            .developer(developer)
            .publisher(publisher)
            .releaseDate(LocalDate.now().minusMonths(6))
            .price(new BigDecimal("29.99"))
            .esrbRating(EsrbRating.EVERYONE)
            .active(true)
            .available(true)
            .build();
    }
}
```

### 6.2.2 Test di Service Layer

#### Test con Mockito
```java
@ExtendWith(MockitoExtension.class)
class GameServiceTest {
    
    @Mock
    private GameRepository gameRepository;
    
    @Mock
    private GameMapper gameMapper;
    
    @Mock
    private UserRepository userRepository;
    
    @Mock
    private UserGameListRepository userGameListRepository;
    
    @InjectMocks
    private GameService gameService;
    
    private Game testGame;
    private GameDTO testGameDTO;
    private User testUser;
    
    @BeforeEach
    void setUp() {
        testGame = Game.builder()
            .id(1L)
            .title("Test Game")
            .description("Test Description")
            .price(new BigDecimal("29.99"))
            .active(true)
            .available(true)
            .build();
        
        testGameDTO = GameDTO.builder()
            .id(1L)
            .title("Test Game")
            .description("Test Description")
            .price(new BigDecimal("29.99"))
            .build();
        
        testUser = User.builder()
            .id(1L)
            .username("testuser")
            .email("test@example.com")
            .active(true)
            .build();
    }
    
    @Test
    @DisplayName("Should find game by ID successfully")
    void shouldFindGameByIdSuccessfully() {
        // Given
        when(gameRepository.findByIdAndActiveTrue(1L))
            .thenReturn(Optional.of(testGame));
        when(gameMapper.toDTO(testGame))
            .thenReturn(testGameDTO);
        
        // When
        GameDTO result = gameService.findGameById(1L);
        
        // Then
        assertThat(result).isNotNull();
        assertThat(result.getId()).isEqualTo(1L);
        assertThat(result.getTitle()).isEqualTo("Test Game");
        
        verify(gameRepository).findByIdAndActiveTrue(1L);
        verify(gameMapper).toDTO(testGame);
    }
    
    @Test
    @DisplayName("Should throw exception when game not found")
    void shouldThrowExceptionWhenGameNotFound() {
        // Given
        when(gameRepository.findByIdAndActiveTrue(999L))
            .thenReturn(Optional.empty());
        
        // When & Then
        assertThatThrownBy(() -> gameService.findGameById(999L))
            .isInstanceOf(ResourceNotFoundException.class)
            .hasMessage("Gioco non trovato con ID: 999");
        
        verify(gameRepository).findByIdAndActiveTrue(999L);
        verifyNoInteractions(gameMapper);
    }
    
    @Test
    @DisplayName("Should create game successfully")
    void shouldCreateGameSuccessfully() {
        // Given
        GameCreateRequest request = GameCreateRequest.builder()
            .title("New Game")
            .description("New Description")
            .price(new BigDecimal("39.99"))
            .developerId(1L)
            .publisherId(1L)
            .build();
        
        Game newGame = Game.builder()
            .title("New Game")
            .description("New Description")
            .price(new BigDecimal("39.99"))
            .active(true)
            .available(true)
            .build();
        
        Game savedGame = Game.builder()
            .id(2L)
            .title("New Game")
            .description("New Description")
            .price(new BigDecimal("39.99"))
            .active(true)
            .available(true)
            .build();
        
        GameDTO expectedDTO = GameDTO.builder()
            .id(2L)
            .title("New Game")
            .description("New Description")
            .price(new BigDecimal("39.99"))
            .build();
        
        when(gameMapper.toEntity(request)).thenReturn(newGame);
        when(gameRepository.save(newGame)).thenReturn(savedGame);
        when(gameMapper.toDTO(savedGame)).thenReturn(expectedDTO);
        
        // When
        GameDTO result = gameService.createGame(request);
        
        // Then
        assertThat(result).isNotNull();
        assertThat(result.getId()).isEqualTo(2L);
        assertThat(result.getTitle()).isEqualTo("New Game");
        
        verify(gameMapper).toEntity(request);
        verify(gameRepository).save(newGame);
        verify(gameMapper).toDTO(savedGame);
    }
    
    @Test
    @DisplayName("Should add game to user list successfully")
    void shouldAddGameToUserListSuccessfully() {
        // Given
        Long userId = 1L;
        Long gameId = 1L;
        ListType listType = ListType.WISHLIST;
        
        when(userRepository.findById(userId))
            .thenReturn(Optional.of(testUser));
        when(gameRepository.findByIdAndActiveTrue(gameId))
            .thenReturn(Optional.of(testGame));
        when(userGameListRepository.existsByUserIdAndGameIdAndListType(userId, gameId, listType))
            .thenReturn(false);
        
        UserGameList savedEntry = UserGameList.builder()
            .user(testUser)
            .game(testGame)
            .listType(listType)
            .build();
        
        when(userGameListRepository.save(any(UserGameList.class)))
            .thenReturn(savedEntry);
        
        // When
        gameService.addGameToUserList(userId, gameId, listType);
        
        // Then
        verify(userRepository).findById(userId);
        verify(gameRepository).findByIdAndActiveTrue(gameId);
        verify(userGameListRepository).existsByUserIdAndGameIdAndListType(userId, gameId, listType);
        verify(userGameListRepository).save(any(UserGameList.class));
    }
    
    @Test
    @DisplayName("Should throw exception when adding duplicate game to list")
    void shouldThrowExceptionWhenAddingDuplicateGameToList() {
        // Given
        Long userId = 1L;
        Long gameId = 1L;
        ListType listType = ListType.WISHLIST;
        
        when(userRepository.findById(userId))
            .thenReturn(Optional.of(testUser));
        when(gameRepository.findByIdAndActiveTrue(gameId))
            .thenReturn(Optional.of(testGame));
        when(userGameListRepository.existsByUserIdAndGameIdAndListType(userId, gameId, listType))
            .thenReturn(true);
        
        // When & Then
        assertThatThrownBy(() -> gameService.addGameToUserList(userId, gameId, listType))
            .isInstanceOf(DuplicateResourceException.class)
            .hasMessage("Il gioco è già presente nella lista WISHLIST");
        
        verify(userGameListRepository, never()).save(any(UserGameList.class));
    }
    
    @Test
    @DisplayName("Should search games with pagination")
    void shouldSearchGamesWithPagination() {
        // Given
        GameSearchCriteria criteria = GameSearchCriteria.builder()
            .title("Test")
            .build();
        
        PaginationParams params = PaginationParams.builder()
            .page(0)
            .size(10)
            .sortBy("title")
            .sortDirection("asc")
            .build();
        
        Pageable pageable = PageRequest.of(0, 10, Sort.by(Sort.Direction.ASC, "title"));
        
        List<Game> games = List.of(testGame);
        Page<Game> gamePage = new PageImpl<>(games, pageable, 1);
        
        List<GameDTO> gameDTOs = List.of(testGameDTO);
        
        when(gameRepository.findGamesWithDynamicFilters(criteria, pageable))
            .thenReturn(gamePage);
        when(gameMapper.toDTO(testGame))
            .thenReturn(testGameDTO);
        
        // When
        PagedResponse<GameDTO> result = gameService.searchGames(criteria, params);
        
        // Then
        assertThat(result).isNotNull();
        assertThat(result.getContent()).hasSize(1);
        assertThat(result.getTotalElements()).isEqualTo(1);
        assertThat(result.getPage()).isEqualTo(0);
        assertThat(result.getSize()).isEqualTo(10);
        
        verify(gameRepository).findGamesWithDynamicFilters(criteria, pageable);
        verify(gameMapper).toDTO(testGame);
    }
    
    @Test
    @DisplayName("Should validate search criteria")
    void shouldValidateSearchCriteria() {
        // Given
        GameSearchCriteria invalidCriteria = GameSearchCriteria.builder()
            .minPrice(new BigDecimal("50.00"))
            .maxPrice(new BigDecimal("30.00")) // Max < Min
            .build();
        
        PaginationParams params = PaginationParams.builder().build();
        
        // When & Then
        assertThatThrownBy(() -> gameService.searchGames(invalidCriteria, params))
            .isInstanceOf(ValidationException.class)
            .hasMessage("Il prezzo minimo non può essere maggiore del prezzo massimo");
        
        verifyNoInteractions(gameRepository);
    }
}
```

### 6.2.3 Test di Controller Layer

#### Test con @WebMvcTest
```java
@WebMvcTest(GameController.class)
@Import(SecurityConfig.class)
class GameControllerTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @MockBean
    private GameService gameService;
    
    @MockBean
    private JwtTokenProvider jwtTokenProvider;
    
    @MockBean
    private CustomUserDetailsService userDetailsService;
    
    @Autowired
    private ObjectMapper objectMapper;
    
    private GameDTO testGameDTO;
    
    @BeforeEach
    void setUp() {
        testGameDTO = GameDTO.builder()
            .id(1L)
            .title("Test Game")
            .description("Test Description")
            .price(new BigDecimal("29.99"))
            .build();
    }
    
    @Test
    @DisplayName("Should get game by ID successfully")
    void shouldGetGameByIdSuccessfully() throws Exception {
        // Given
        when(gameService.findGameById(1L))
            .thenReturn(testGameDTO);
        
        // When & Then
        mockMvc.perform(get("/api/v1/games/1")
                .contentType(MediaType.APPLICATION_JSON))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.success").value(true))
            .andExpect(jsonPath("$.data.id").value(1))
            .andExpect(jsonPath("$.data.title").value("Test Game"))
            .andExpect(jsonPath("$.data.price").value(29.99));
        
        verify(gameService).findGameById(1L);
    }
    
    @Test
    @DisplayName("Should return 404 when game not found")
    void shouldReturn404WhenGameNotFound() throws Exception {
        // Given
        when(gameService.findGameById(999L))
            .thenThrow(new ResourceNotFoundException("Gioco non trovato con ID: 999"));
        
        // When & Then
        mockMvc.perform(get("/api/v1/games/999")
                .contentType(MediaType.APPLICATION_JSON))
            .andExpect(status().isNotFound())
            .andExpect(jsonPath("$.success").value(false))
            .andExpect(jsonPath("$.message").value("Gioco non trovato con ID: 999"));
    }
    
    @Test
    @DisplayName("Should search games with pagination")
    void shouldSearchGamesWithPagination() throws Exception {
        // Given
        List<GameDTO> games = List.of(testGameDTO);
        PagedResponse<GameDTO> pagedResponse = PagedResponse.<GameDTO>builder()
            .content(games)
            .page(0)
            .size(10)
            .totalElements(1)
            .totalPages(1)
            .first(true)
            .last(true)
            .build();
        
        when(gameService.searchGames(any(GameSearchCriteria.class), any(PaginationParams.class)))
            .thenReturn(pagedResponse);
        
        // When & Then
        mockMvc.perform(get("/api/v1/games/search")
                .param("title", "Test")
                .param("page", "0")
                .param("size", "10")
                .param("sortBy", "title")
                .param("sortDirection", "asc")
                .contentType(MediaType.APPLICATION_JSON))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.content").isArray())
            .andExpect(jsonPath("$.content").value(hasSize(1)))
            .andExpect(jsonPath("$.content[0].title").value("Test Game"))
            .andExpect(jsonPath("$.page").value(0))
            .andExpect(jsonPath("$.size").value(10))
            .andExpect(jsonPath("$.totalElements").value(1));
    }
    
    @Test
    @WithMockUser(roles = "ADMIN")
    @DisplayName("Should create game successfully with admin role")
    void shouldCreateGameSuccessfullyWithAdminRole() throws Exception {
        // Given
        GameCreateRequest request = GameCreateRequest.builder()
            .title("New Game")
            .description("New Description")
            .price(new BigDecimal("39.99"))
            .developerId(1L)
            .publisherId(1L)
            .build();
        
        GameDTO createdGame = GameDTO.builder()
            .id(2L)
            .title("New Game")
            .description("New Description")
            .price(new BigDecimal("39.99"))
            .build();
        
        when(gameService.createGame(any(GameCreateRequest.class)))
            .thenReturn(createdGame);
        
        // When & Then
        mockMvc.perform(post("/api/v1/games")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.success").value(true))
            .andExpect(jsonPath("$.data.id").value(2))
            .andExpect(jsonPath("$.data.title").value("New Game"));
    }
    
    @Test
    @DisplayName("Should return 403 when creating game without admin role")
    void shouldReturn403WhenCreatingGameWithoutAdminRole() throws Exception {
        // Given
        GameCreateRequest request = GameCreateRequest.builder()
            .title("New Game")
            .build();
        
        // When & Then
        mockMvc.perform(post("/api/v1/games")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
            .andExpect(status().isForbidden());
        
        verifyNoInteractions(gameService);
    }
    
    @Test
    @WithMockUser(roles = "ADMIN")
    @DisplayName("Should return 400 for invalid game creation request")
    void shouldReturn400ForInvalidGameCreationRequest() throws Exception {
        // Given
        GameCreateRequest invalidRequest = GameCreateRequest.builder()
            .title("") // Titolo vuoto
            .price(new BigDecimal("-10.00")) // Prezzo negativo
            .build();
        
        // When & Then
        mockMvc.perform(post("/api/v1/games")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(invalidRequest)))
            .andExpect(status().isBadRequest())
            .andExpect(jsonPath("$.success").value(false))
            .andExpect(jsonPath("$.errors").isArray());
        
        verifyNoInteractions(gameService);
    }
}
```

## 6.3 Integration Testing

### 6.3.1 Test di Integrazione Completi

#### Test con @SpringBootTest
```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Testcontainers
@Transactional
class GameIntegrationTest {
    
    @Container
    static MySQLContainer<?> mysql = new MySQLContainer<>("mysql:8.0")
        .withDatabaseName("videogames_test")
        .withUsername("test")
        .withPassword("test");
    
    @Autowired
    private TestRestTemplate restTemplate;
    
    @Autowired
    private GameRepository gameRepository;
    
    @Autowired
    private UserRepository userRepository;
    
    @Autowired
    private DeveloperRepository developerRepository;
    
    @Autowired
    private PublisherRepository publisherRepository;
    
    @Autowired
    private JwtTokenProvider jwtTokenProvider;
    
    @Autowired
    private PasswordEncoder passwordEncoder;
    
    private String adminToken;
    private String userToken;
    private Developer testDeveloper;
    private Publisher testPublisher;
    
    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", mysql::getJdbcUrl);
        registry.add("spring.datasource.username", mysql::getUsername);
        registry.add("spring.datasource.password", mysql::getPassword);
    }
    
    @BeforeEach
    void setUp() {
        // Setup test data
        setupTestUsers();
        setupTestDeveloperAndPublisher();
    }
    
    private void setupTestUsers() {
        // Admin user
        User admin = User.builder()
            .username("admin")
            .email("admin@test.com")
            .password(passwordEncoder.encode("password"))
            .firstName("Admin")
            .lastName("User")
            .roles(Set.of(UserRole.ADMIN))
            .active(true)
            .emailVerified(true)
            .build();
        
        User savedAdmin = userRepository.save(admin);
        UserPrincipal adminPrincipal = UserPrincipal.from(savedAdmin);
        adminToken = jwtTokenProvider.generateAccessToken(adminPrincipal);
        
        // Regular user
        User user = User.builder()
            .username("user")
            .email("user@test.com")
            .password(passwordEncoder.encode("password"))
            .firstName("Regular")
            .lastName("User")
            .roles(Set.of(UserRole.USER))
            .active(true)
            .emailVerified(true)
            .build();
        
        User savedUser = userRepository.save(user);
        UserPrincipal userPrincipal = UserPrincipal.from(savedUser);
        userToken = jwtTokenProvider.generateAccessToken(userPrincipal);
    }
    
    private void setupTestDeveloperAndPublisher() {
        testDeveloper = Developer.builder()
            .name("Test Developer")
            .foundedYear(2000)
            .build();
        testDeveloper = developerRepository.save(testDeveloper);
        
        testPublisher = Publisher.builder()
            .name("Test Publisher")
            .foundedYear(1995)
            .build();
        testPublisher = publisherRepository.save(testPublisher);
    }
    
    @Test
    @DisplayName("Should complete full game lifecycle")
    void shouldCompleteFullGameLifecycle() {
        // 1. Create game as admin
        GameCreateRequest createRequest = GameCreateRequest.builder()
            .title("Integration Test Game")
            .description("A game for integration testing")
            .price(new BigDecimal("49.99"))
            .developerId(testDeveloper.getId())
            .publisherId(testPublisher.getId())
            .releaseDate(LocalDate.now().plusMonths(1))
            .esrbRating(EsrbRating.TEEN)
            .build();
        
        HttpHeaders adminHeaders = new HttpHeaders();
        adminHeaders.setBearerAuth(adminToken);
        HttpEntity<GameCreateRequest> createEntity = new HttpEntity<>(createRequest, adminHeaders);
        
        ResponseEntity<ApiResponse> createResponse = restTemplate.exchange(
            "/api/v1/games",
            HttpMethod.POST,
            createEntity,
            ApiResponse.class
        );
        
        assertThat(createResponse.getStatusCode()).isEqualTo(HttpStatus.CREATED);
        assertThat(createResponse.getBody().isSuccess()).isTrue();
        
        // Extract game ID from response
        @SuppressWarnings("unchecked")
        Map<String, Object> gameData = (Map<String, Object>) createResponse.getBody().getData();
        Long gameId = ((Number) gameData.get("id")).longValue();
        
        // 2. Get game details (public endpoint)
        ResponseEntity<ApiResponse> getResponse = restTemplate.getForEntity(
            "/api/v1/games/" + gameId,
            ApiResponse.class
        );
        
        assertThat(getResponse.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(getResponse.getBody().isSuccess()).isTrue();
        
        // 3. Search for the game
        ResponseEntity<PagedResponse> searchResponse = restTemplate.getForEntity(
            "/api/v1/games/search?title=Integration&page=0&size=10",
            PagedResponse.class
        );
        
        assertThat(searchResponse.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(searchResponse.getBody().getContent()).isNotEmpty();
        
        // 4. Add game to user's wishlist
        HttpHeaders userHeaders = new HttpHeaders();
        userHeaders.setBearerAuth(userToken);
        HttpEntity<Void> userEntity = new HttpEntity<>(userHeaders);
        
        ResponseEntity<ApiResponse> addToListResponse = restTemplate.exchange(
            "/api/v1/users/lists/wishlist/games/" + gameId,
            HttpMethod.POST,
            userEntity,
            ApiResponse.class
        );
        
        assertThat(addToListResponse.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(addToListResponse.getBody().isSuccess()).isTrue();
        
        // 5. Rate the game
        GameRatingRequest ratingRequest = GameRatingRequest.builder()
            .rating(4.5)
            .comment("Great game!")
            .build();
        
        HttpEntity<GameRatingRequest> ratingEntity = new HttpEntity<>(ratingRequest, userHeaders);
        
        ResponseEntity<ApiResponse> ratingResponse = restTemplate.exchange(
            "/api/v1/games/" + gameId + "/rate",
            HttpMethod.POST,
            ratingEntity,
            ApiResponse.class
        );
        
        assertThat(ratingResponse.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(ratingResponse.getBody().isSuccess()).isTrue();
        
        // 6. Update game as admin
        GameUpdateRequest updateRequest = GameUpdateRequest.builder()
            .title("Updated Integration Test Game")
            .description("Updated description")
            .price(new BigDecimal("39.99"))
            .build();
        
        HttpEntity<GameUpdateRequest> updateEntity = new HttpEntity<>(updateRequest, adminHeaders);
        
        ResponseEntity<ApiResponse> updateResponse = restTemplate.exchange(
            "/api/v1/games/" + gameId,
            HttpMethod.PUT,
            updateEntity,
            ApiResponse.class
        );
        
        assertThat(updateResponse.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(updateResponse.getBody().isSuccess()).isTrue();
        
        // 7. Verify changes in database
        Optional<Game> updatedGame = gameRepository.findById(gameId);
        assertThat(updatedGame).isPresent();
        assertThat(updatedGame.get().getTitle()).isEqualTo("Updated Integration Test Game");
        assertThat(updatedGame.get().getPrice()).isEqualTo(new BigDecimal("39.99"));
    }
    
    @Test
    @DisplayName("Should handle authentication and authorization correctly")
    void shouldHandleAuthenticationAndAuthorizationCorrectly() {
        // 1. Try to create game without authentication
        GameCreateRequest createRequest = GameCreateRequest.builder()
            .title("Unauthorized Game")
            .build();
        
        ResponseEntity<ApiResponse> unauthorizedResponse = restTemplate.postForEntity(
            "/api/v1/games",
            createRequest,
            ApiResponse.class
        );
        
        assertThat(unauthorizedResponse.getStatusCode()).isEqualTo(HttpStatus.FORBIDDEN);
        
        // 2. Try to create game with user role (should fail)
        HttpHeaders userHeaders = new HttpHeaders();
        userHeaders.setBearerAuth(userToken);
        HttpEntity<GameCreateRequest> userEntity = new HttpEntity<>(createRequest, userHeaders);
        
        ResponseEntity<ApiResponse> forbiddenResponse = restTemplate.exchange(
            "/api/v1/games",
            HttpMethod.POST,
            userEntity,
            ApiResponse.class
        );
        
        assertThat(forbiddenResponse.getStatusCode()).isEqualTo(HttpStatus.FORBIDDEN);
        
        // 3. Access public endpoints without authentication
        ResponseEntity<PagedResponse> publicResponse = restTemplate.getForEntity(
            "/api/v1/games?page=0&size=10",
            PagedResponse.class
        );
        
        assertThat(publicResponse.getStatusCode()).isEqualTo(HttpStatus.OK);
    }
    
    @Test
    @DisplayName("Should handle validation errors correctly")
    void shouldHandleValidationErrorsCorrectly() {
        // Invalid game creation request
        GameCreateRequest invalidRequest = GameCreateRequest.builder()
            .title("") // Empty title
            .price(new BigDecimal("-10.00")) // Negative price
            .build();
        
        HttpHeaders adminHeaders = new HttpHeaders();
        adminHeaders.setBearerAuth(adminToken);
        HttpEntity<GameCreateRequest> entity = new HttpEntity<>(invalidRequest, adminHeaders);
        
        ResponseEntity<ApiResponse> response = restTemplate.exchange(
            "/api/v1/games",
            HttpMethod.POST,
            entity,
            ApiResponse.class
        );
        
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.BAD_REQUEST);
        assertThat(response.getBody().isSuccess()).isFalse();
        assertThat(response.getBody().getErrors()).isNotEmpty();
    }
    
    @Test
    @DisplayName("Should handle concurrent access correctly")
    void shouldHandleConcurrentAccessCorrectly() throws InterruptedException {
        // Create a game first
        Game game = Game.builder()
            .title("Concurrent Test Game")
            .description("Test description")
            .developer(testDeveloper)
            .publisher(testPublisher)
            .price(new BigDecimal("29.99"))
            .releaseDate(LocalDate.now())
            .esrbRating(EsrbRating.EVERYONE)
            .active(true)
            .available(true)
            .build();
        
        Game savedGame = gameRepository.save(game);
        
        // Simulate concurrent rating submissions
        int numberOfThreads = 10;
        CountDownLatch latch = new CountDownLatch(numberOfThreads);
        AtomicInteger successCount = new AtomicInteger(0);
        
        for (int i = 0; i < numberOfThreads; i++) {
            final int threadId = i;
            new Thread(() -> {
                try {
                    GameRatingRequest ratingRequest = GameRatingRequest.builder()
                        .rating(4.0 + (threadId % 5) * 0.2) // Different ratings
                        .comment("Comment from thread " + threadId)
                        .build();
                    
                    HttpHeaders userHeaders = new HttpHeaders();
                    userHeaders.setBearerAuth(userToken);
                    HttpEntity<GameRatingRequest> entity = new HttpEntity<>(ratingRequest, userHeaders);
                    
                    ResponseEntity<ApiResponse> response = restTemplate.exchange(
                        "/api/v1/games/" + savedGame.getId() + "/rate",
                        HttpMethod.POST,
                        entity,
                        ApiResponse.class
                    );
                    
                    if (response.getStatusCode().is2xxSuccessful()) {
                        successCount.incrementAndGet();
                    }
                } catch (Exception e) {
                    // Expected for concurrent access
                } finally {
                    latch.countDown();
                }
            }).start();
        }
        
        latch.await(10, TimeUnit.SECONDS);
        
        // Only one rating should succeed (user can rate a game only once)
        assertThat(successCount.get()).isEqualTo(1);
    }
}
```

## 6.4 Test di Performance

### 6.4.1 Load Testing con JMeter

#### Test Plan Configuration
```xml
<?xml version="1.0" encoding="UTF-8"?>
<jmeterTestPlan version="1.2">
  <hashTree>
    <TestPlan guiclass="TestPlanGui" testclass="TestPlan" testname="Video Games API Load Test">
      <stringProp name="TestPlan.comments">Load test for video games API</stringProp>
      <boolProp name="TestPlan.functional_mode">false</boolProp>
      <boolProp name="TestPlan.serialize_threadgroups">false</boolProp>
      <elementProp name="TestPlan.arguments" elementType="Arguments" guiclass="ArgumentsPanel">
        <collectionProp name="Arguments.arguments"/>
      </elementProp>
      <stringProp name="TestPlan.user_define_classpath"></stringProp>
    </TestPlan>
    <hashTree>
      <!-- Thread Group for Game Search -->
      <ThreadGroup guiclass="ThreadGroupGui" testclass="ThreadGroup" testname="Game Search Load">
        <stringProp name="ThreadGroup.on_sample_error">continue</stringProp>
        <elementProp name="ThreadGroup.main_controller" elementType="LoopController">
          <boolProp name="LoopController.continue_forever">false</boolProp>
          <stringProp name="LoopController.loops">100</stringProp>
        </elementProp>
        <stringProp name="ThreadGroup.num_threads">50</stringProp>
        <stringProp name="ThreadGroup.ramp_time">30</stringProp>
      </ThreadGroup>
      <hashTree>
        <!-- HTTP Request for Game Search -->
        <HTTPSamplerProxy guiclass="HttpTestSampleGui" testclass="HTTPSamplerProxy" testname="Search Games">
          <elementProp name="HTTPsampler.Arguments" elementType="Arguments">
            <collectionProp name="Arguments.arguments">
              <elementProp name="" elementType="HTTPArgument">
                <boolProp name="HTTPArgument.always_encode">false</boolProp>
                <stringProp name="Argument.value">title=action&amp;page=0&amp;size=10</stringProp>
                <stringProp name="Argument.metadata">=</stringProp>
              </elementProp>
            </collectionProp>
          </elementProp>
          <stringProp name="HTTPSampler.domain">localhost</stringProp>
          <stringProp name="HTTPSampler.port">8080</stringProp>
          <stringProp name="HTTPSampler.path">/api/v1/games/search</stringProp>
          <stringProp name="HTTPSampler.method">GET</stringProp>
        </HTTPSamplerProxy>
      </hashTree>
    </hashTree>
  </hashTree>
</jmeterTestPlan>
```

### 6.4.2 Performance Testing con Spring Boot

#### Microbenchmark con JMH
```java
@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.MILLISECONDS)
@State(Scope.Benchmark)
@Fork(value = 2, jvmArgs = {"-Xms2G", "-Xmx2G"})
@Warmup(iterations = 3)
@Measurement(iterations = 5)
public class GameServiceBenchmark {
    
    private GameService gameService;
    private GameRepository gameRepository;
    private GameMapper gameMapper;
    
    @Setup
    public void setup() {
        // Setup mock dependencies
        gameRepository = Mockito.mock(GameRepository.class);
        gameMapper = Mockito.mock(GameMapper.class);
        gameService = new GameService(gameRepository, gameMapper, null, null);
        
        // Setup test data
        setupTestData();
    }
    
    @Benchmark
    public PagedResponse<GameDTO> benchmarkGameSearch() {
        GameSearchCriteria criteria = GameSearchCriteria.builder()
            .title("action")
            .build();
        
        PaginationParams params = PaginationParams.builder()
            .page(0)
            .size(10)
            .build();
        
        return gameService.searchGames(criteria, params);
    }
    
    @Benchmark
    public GameDTO benchmarkGameRetrieval() {
        return gameService.findGameById(1L);
    }
    
    private void setupTestData() {
        // Setup mock responses
        List<Game> games = createTestGames(100);
        Page<Game> gamePage = new PageImpl<>(games.subList(0, 10));
        
        when(gameRepository.findGamesWithDynamicFilters(any(), any()))
            .thenReturn(gamePage);
        
        when(gameRepository.findByIdAndActiveTrue(1L))
            .thenReturn(Optional.of(games.get(0)));
        
        when(gameMapper.toDTO(any(Game.class)))
            .thenReturn(createTestGameDTO());
    }
    
    private List<Game> createTestGames(int count) {
        List<Game> games = new ArrayList<>();
        for (int i = 0; i < count; i++) {
            games.add(Game.builder()
                .id((long) i)
                .title("Game " + i)
                .description("Description " + i)
                .price(new BigDecimal("29.99"))
                .build());
        }
        return games;
    }
    
    private GameDTO createTestGameDTO() {
        return GameDTO.builder()
            .id(1L)
            .title("Test Game")
            .description("Test Description")
            .price(new BigDecimal("29.99"))
            .build();
    }
}
```

## 6.5 Best Practices

### 6.5.1 Organizzazione dei Test

#### Struttura delle Directory
```
src/test/java/
├── unit/
│   ├── repository/
│   ├── service/
│   └── controller/
├── integration/
│   ├── api/
│   └── database/
├── performance/
└── e2e/
```

#### Naming Conventions
```java
// Pattern: should[ExpectedBehavior]When[StateUnderTest]
@Test
void shouldReturnGameWhenValidIdProvided() { }

@Test
void shouldThrowExceptionWhenGameNotFound() { }

@Test
void shouldCreateGameWhenValidDataProvided() { }
```

### 6.5.2 Test Data Management

#### Test Data Builders
```java
public class GameTestDataBuilder {
    
    private Long id = 1L;
    private String title = "Default Game";
    private String description = "Default Description";
    private BigDecimal price = new BigDecimal("29.99");
    private Developer developer;
    private Publisher publisher;
    private LocalDate releaseDate = LocalDate.now();
    private EsrbRating esrbRating = EsrbRating.EVERYONE;
    private boolean active = true;
    private boolean available = true;
    
    public static GameTestDataBuilder aGame() {
        return new GameTestDataBuilder();
    }
    
    public GameTestDataBuilder withId(Long id) {
        this.id = id;
        return this;
    }
    
    public GameTestDataBuilder withTitle(String title) {
        this.title = title;
        return this;
    }
    
    public GameTestDataBuilder withPrice(BigDecimal price) {
        this.price = price;
        return this;
    }
    
    public GameTestDataBuilder withDeveloper(Developer developer) {
        this.developer = developer;
        return this;
    }
    
    public GameTestDataBuilder withPublisher(Publisher publisher) {
        this.publisher = publisher;
        return this;
    }
    
    public GameTestDataBuilder withReleaseDate(LocalDate releaseDate) {
        this.releaseDate = releaseDate;
        return this;
    }
    
    public GameTestDataBuilder withEsrbRating(EsrbRating esrbRating) {
        this.esrbRating = esrbRating;
        return this;
    }
    
    public GameTestDataBuilder inactive() {
        this.active = false;
        return this;
    }
    
    public GameTestDataBuilder unavailable() {
        this.available = false;
        return this;
    }
    
    public Game build() {
        return Game.builder()
            .id(id)
            .title(title)
            .description(description)
            .price(price)
            .developer(developer)
            .publisher(publisher)
            .releaseDate(releaseDate)
            .esrbRating(esrbRating)
            .active(active)
            .available(available)
            .build();
    }
}

// Usage
@Test
void shouldFindExpensiveGames() {
    // Given
    Game expensiveGame = aGame()
        .withTitle("Expensive Game")
        .withPrice(new BigDecimal("79.99"))
        .build();
    
    Game cheapGame = aGame()
        .withTitle("Cheap Game")
        .withPrice(new BigDecimal("9.99"))
        .build();
    
    // Test implementation...
}
```

### 6.5.3 Mocking Best Practices

#### Effective Mocking
```java
@ExtendWith(MockitoExtension.class)
class GameServiceTest {
    
    @Mock
    private GameRepository gameRepository;
    
    @Mock
    private GameMapper gameMapper;
    
    @InjectMocks
    private GameService gameService;
    
    @Test
    void shouldHandleRepositoryException() {
        // Given
        when(gameRepository.findByIdAndActiveTrue(1L))
            .thenThrow(new DataAccessException("Database error") {});
        
        // When & Then
        assertThatThrownBy(() -> gameService.findGameById(1L))
            .isInstanceOf(DataAccessException.class)
            .hasMessage("Database error");
    }
    
    @Test
    void shouldVerifyInteractions() {
        // Given
        Game game = aGame().build();
        GameDTO gameDTO = GameDTO.builder().build();
        
        when(gameRepository.findByIdAndActiveTrue(1L))
            .thenReturn(Optional.of(game));
        when(gameMapper.toDTO(game))
            .thenReturn(gameDTO);
        
        // When
        gameService.findGameById(1L);
        
        // Then
        verify(gameRepository).findByIdAndActiveTrue(1L);
        verify(gameMapper).toDTO(game);
        verifyNoMoreInteractions(gameRepository, gameMapper);
    }
    
    @Test
    void shouldUseArgumentMatchers() {
        // Given
        when(gameRepository.save(any(Game.class)))
            .thenAnswer(invocation -> {
                Game game = invocation.getArgument(0);
                game.setId(1L);
                return game;
            });
        
        // When
        GameCreateRequest request = GameCreateRequest.builder()
            .title("New Game")
            .build();
        
        gameService.createGame(request);
        
        // Then
        verify(gameRepository).save(argThat(game -> 
            "New Game".equals(game.getTitle())
        ));
    }
}
```

### 6.5.4 Test Configuration

#### Test Profiles
```yaml
# application-test.yml
spring:
  datasource:
    url: jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=FALSE
    driver-class-name: org.h2.Driver
    username: sa
    password: 
  
  jpa:
    hibernate:
      ddl-auto: create-drop
    show-sql: true
    properties:
      hibernate:
        format_sql: true
        dialect: org.hibernate.dialect.H2Dialect
  
  cache:
    type: none
  
  mail:
    host: localhost
    port: 2525
    username: test
    password: test

app:
  jwt:
    secret: dGVzdC1zZWNyZXQtZm9yLWp3dC10b2tlbi1zaWduaW5nLXRoYXQtaXMtbG9uZy1lbm91Z2g=
    access-token-expiration: 900000
    refresh-token-expiration: 604800000

logging:
  level:
    com.videogames: DEBUG
    org.springframework.security: DEBUG
    org.hibernate.SQL: DEBUG
    org.hibernate.type.descriptor.sql.BasicBinder: TRACE
```

#### Test Configuration Class
```java
@TestConfiguration
public class TestConfig {
    
    @Bean
    @Primary
    public PasswordEncoder testPasswordEncoder() {
        // Faster password encoder for tests
        return new BCryptPasswordEncoder(4);
    }
    
    @Bean
    @Primary
    public Clock testClock() {
        // Fixed clock for predictable tests
        return Clock.fixed(
            Instant.parse("2023-01-01T00:00:00Z"),
            ZoneOffset.UTC
        );
    }
    
    @Bean
    @Primary
    public EmailService mockEmailService() {
        return Mockito.mock(EmailService.class);
    }
}
```

### 6.5.5 Continuous Integration

#### GitHub Actions Workflow
```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    
    services:
      mysql:
        image: mysql:8.0
        env:
          MYSQL_ROOT_PASSWORD: root
          MYSQL_DATABASE: videogames_test
        ports:
          - 3306:3306
        options: --health-cmd="mysqladmin ping" --health-interval=10s --health-timeout=5s --health-retries=3
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up JDK 17
      uses: actions/setup-java@v3
      with:
        java-version: '17'
        distribution: 'temurin'
    
    - name: Cache Maven dependencies
      uses: actions/cache@v3
      with:
        path: ~/.m2
        key: ${{ runner.os }}-m2-${{ hashFiles('**/pom.xml') }}
        restore-keys: ${{ runner.os }}-m2
    
    - name: Run unit tests
      run: mvn test -Dspring.profiles.active=test
    
    - name: Run integration tests
      run: mvn verify -Dspring.profiles.active=test
    
    - name: Generate test report
      uses: dorny/test-reporter@v1
      if: success() || failure()
      with:
        name: Maven Tests
        path: target/surefire-reports/*.xml
        reporter: java-junit
    
    - name: Code coverage
      run: mvn jacoco:report
    
    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v3
      with:
        file: target/site/jacoco/jacoco.xml
        flags: unittests
        name: codecov-umbrella
```

I test sono fondamentali per garantire la qualità del software e la fiducia nel codice. Una strategia di testing ben strutturata, che include unit test, integration test e test end-to-end, insieme a best practices come il TDD e l'uso di test data builders, contribuisce significativamente al successo del progetto.