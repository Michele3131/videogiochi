# 6. Testing e Best Practices

## Piramide dei Test

```
        /\     E2E Tests (10%)
       /  \
      /    \   Integration Tests (20%)
     /      \
    /        \  Unit Tests (70%)
   /__________\
```

**Principi FIRST:**
- **Fast**: Test veloci
- **Independent**: Test indipendenti
- **Repeatable**: Risultati consistenti
- **Self-Validating**: Risultato chiaro (pass/fail)
- **Timely**: Scritti al momento giusto

## Unit Testing

**Repository Test:**
```java
@DataJpaTest
class GameRepositoryTest {
    
    @Autowired
    private TestEntityManager entityManager;
    
    @Autowired
    private GameRepository gameRepository;
    
    @Test
    void shouldFindGamesByTitle() {
        // Given
        Game game = Game.builder()
            .title("Test Game")
            .active(true)
            .build();
        entityManager.persistAndFlush(game);
        
        // When
        Page<Game> result = gameRepository.findByTitleContainingIgnoreCaseAndActiveTrue(
            "test", PageRequest.of(0, 10)
        );
        
        // Then
        assertThat(result.getContent()).hasSize(1);
        assertThat(result.getContent().get(0).getTitle()).isEqualTo("Test Game");
    }
}
```

**Service Test:**
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
    void shouldFindGameById() {
        // Given
        Game game = Game.builder().id(1L).title("Test").build();
        GameDTO dto = GameDTO.builder().id(1L).title("Test").build();
        
        when(gameRepository.findByIdAndActiveTrue(1L)).thenReturn(Optional.of(game));
        when(gameMapper.toDTO(game)).thenReturn(dto);
        
        // When
        GameDTO result = gameService.findGameById(1L);
        
        // Then
        assertThat(result.getTitle()).isEqualTo("Test");
        verify(gameRepository).findByIdAndActiveTrue(1L);
    }
    
    @Test
    void shouldThrowExceptionWhenGameNotFound() {
        // Given
        when(gameRepository.findByIdAndActiveTrue(999L)).thenReturn(Optional.empty());
        
        // When & Then
        assertThatThrownBy(() -> gameService.findGameById(999L))
            .isInstanceOf(ResourceNotFoundException.class);
    }
}
```

## Integration Testing

**Test con @SpringBootTest:**
```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Transactional
class GameIntegrationTest {
    
    @Autowired
    private TestRestTemplate restTemplate;
    
    @Test
    void shouldCreateAndRetrieveGame() {
        // Given
        GameCreateRequest request = new GameCreateRequest("New Game", "Description");
        
        // When
        ResponseEntity<GameDTO> createResponse = restTemplate.postForEntity(
            "/api/v1/games", request, GameDTO.class
        );
        
        // Then
        assertThat(createResponse.getStatusCode()).isEqualTo(HttpStatus.CREATED);
        assertThat(createResponse.getBody().getTitle()).isEqualTo("New Game");
    }
}
```

## End-to-End Testing

**Test con Selenium/TestContainers:**
```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Testcontainers
class GameE2ETest {
    
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:13")
        .withDatabaseName("testdb")
        .withUsername("test")
        .withPassword("test");
    
    @Test
    void shouldCompleteGamePurchaseFlow() {
        // Test del flusso completo: ricerca -> dettagli -> acquisto
        // Utilizzando WebDriver per simulare interazioni utente
    }
}
```

## Best Practices

- **Test Isolation**: Ogni test deve essere indipendente
- **Test Data**: Utilizzare builder pattern per creare dati di test
- **Mocking**: Mock solo le dipendenze esterne, non la logica di business
- **Assertions**: Utilizzare AssertJ per assertions più leggibili
- **Coverage**: Puntare al 80% di code coverage per la logica di business
- **Performance**: I test unit devono essere veloci (<100ms)
- **Naming**: Nomi descrittivi che spiegano cosa viene testato
- **AAA Pattern**: Arrange, Act, Assert per strutturare i test

**Configurazione Test:**
```java
@TestConfiguration
public class TestConfig {
    
    @Bean
    @Primary
    public Clock testClock() {
        return Clock.fixed(Instant.parse("2023-01-01T00:00:00Z"), ZoneOffset.UTC);
    }
    
    @Bean
    @Primary
    public EmailService mockEmailService() {
        return Mockito.mock(EmailService.class);
    }
}
```

Il testing rappresenta un pilastro fondamentale per garantire la qualità del software, permettendo di identificare bug precocemente, facilitare il refactoring e documentare il comportamento atteso del sistema.