# 8. Testing, Configurazione e Deployment

> **Riferimenti teorici**: Questo capitolo implementa le strategie di testing descritte in [Testing Strategies](../knowledge/06-testing-strategies.md) e le configurazioni di deployment

## 8.1 Strategia di Testing

> **Best Practice**: Una strategia di testing completa include test unitari, di integrazione e end-to-end per garantire la qualità del software.

### 8.1.1 Configurazione Test

> **Concetto chiave**: Utilizziamo Testcontainers per creare un ambiente di test isolato con database reale, garantendo test più affidabili.

**TestConfig.java**:
```java
package mc.videogiochi.config;

import org.springframework.boot.test.context.TestConfiguration;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Primary;
import org.springframework.context.annotation.Profile;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.testcontainers.containers.MySQLContainer;

@TestConfiguration
@Profile("test")
public class TestConfig {
    
    @Bean
    @Primary
    public PasswordEncoder testPasswordEncoder() {
        // Usa un encoder più veloce per i test
        return new BCryptPasswordEncoder(4);
    }
    
    @Bean
    public MySQLContainer<?> mysqlContainer() {
        MySQLContainer<?> container = new MySQLContainer<>("mysql:8.0")
                .withDatabaseName("videogiochi_test")
                .withUsername("test")
                .withPassword("test")
                .withReuse(true);
        
        container.start();
        return container;
    }
}
```

**application-test.yml**:
```yaml
spring:
  datasource:
    url: jdbc:tc:mysql:8.0:///videogiochi_test
    driver-class-name: org.testcontainers.jdbc.ContainerDatabaseDriver
    username: test
    password: test
  
  jpa:
    hibernate:
      ddl-auto: create-drop
    show-sql: true
    properties:
      hibernate:
        format_sql: true
        use_sql_comments: true
  
  flyway:
    enabled: false
  
  cache:
    type: simple
  
  mail:
    host: localhost
    port: 3025
    username: test
    password: test
    properties:
      mail:
        smtp:
          auth: false
          starttls:
            enable: false

app:
  security:
    jwt:
      secret: testSecretKeyForJWTTokenGenerationThatIsLongEnoughForHS512Algorithm
      expiration: 3600000 # 1 ora
  
  cache:
    ttl: 300 # 5 minuti
  
  email:
    verification:
      enabled: false

logging:
  level:
    mc.videogiochi: DEBUG
    org.springframework.security: DEBUG
    org.hibernate.SQL: DEBUG
    org.hibernate.type.descriptor.sql.BasicBinder: TRACE
```

### 8.1.2 Test di Repository

**GameRepositoryTest.java**:
```java
package mc.videogiochi.repository;

import mc.videogiochi.entity.*;
import mc.videogiochi.enums.EsrbRating;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.boot.test.autoconfigure.orm.jpa.TestEntityManager;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;
import org.springframework.test.context.ActiveProfiles;

import java.time.LocalDate;
import java.util.List;
import java.util.Optional;

import static org.assertj.core.api.Assertions.*;

@DataJpaTest
@ActiveProfiles("test")
class GameRepositoryTest {
    
    @Autowired
    private TestEntityManager entityManager;
    
    @Autowired
    private GameRepository gameRepository;
    
    private Game testGame;
    private Developer testDeveloper;
    private Publisher testPublisher;
    
    @BeforeEach
    void setUp() {
        testDeveloper = Developer.builder()
                .name("Test Developer")
                .foundedYear(2000)
                .country("Italy")
                .build();
        entityManager.persistAndFlush(testDeveloper);
        
        testPublisher = Publisher.builder()
                .name("Test Publisher")
                .foundedYear(1995)
                .country("USA")
                .build();
        entityManager.persistAndFlush(testPublisher);
        
        testGame = Game.builder()
                .title("Test Game")
                .description("A test game for unit testing")
                .releaseDate(LocalDate.of(2023, 1, 15))
                .esrbRating(EsrbRating.TEEN)
                .developer(testDeveloper)
                .publisher(testPublisher)
                .price(29.99)
                .active(true)
                .build();
        entityManager.persistAndFlush(testGame);
    }
    
    @Test
    void findByTitleContainingIgnoreCase_ShouldReturnMatchingGames() {
        // Given
        Pageable pageable = PageRequest.of(0, 10);
        
        // When
        Page<Game> result = gameRepository.findByTitleContainingIgnoreCaseAndActiveTrue("test", pageable);
        
        // Then
        assertThat(result).isNotEmpty();
        assertThat(result.getContent()).hasSize(1);
        assertThat(result.getContent().get(0).getTitle()).isEqualTo("Test Game");
    }
    
    @Test
    void findByDeveloper_ShouldReturnGamesFromDeveloper() {
        // Given
        Pageable pageable = PageRequest.of(0, 10);
        
        // When
        Page<Game> result = gameRepository.findByDeveloperAndActiveTrue(testDeveloper, pageable);
        
        // Then
        assertThat(result).isNotEmpty();
        assertThat(result.getContent()).hasSize(1);
        assertThat(result.getContent().get(0).getDeveloper()).isEqualTo(testDeveloper);
    }
    
    @Test
    void findByReleaseDateBetween_ShouldReturnGamesInDateRange() {
        // Given
        LocalDate startDate = LocalDate.of(2023, 1, 1);
        LocalDate endDate = LocalDate.of(2023, 12, 31);
        Pageable pageable = PageRequest.of(0, 10);
        
        // When
        Page<Game> result = gameRepository.findByReleaseDateBetweenAndActiveTrue(startDate, endDate, pageable);
        
        // Then
        assertThat(result).isNotEmpty();
        assertThat(result.getContent()).hasSize(1);
        assertThat(result.getContent().get(0).getReleaseDate()).isBetween(startDate, endDate);
    }
    
    @Test
    void findByEsrbRating_ShouldReturnGamesWithRating() {
        // Given
        Pageable pageable = PageRequest.of(0, 10);
        
        // When
        Page<Game> result = gameRepository.findByEsrbRatingAndActiveTrue(EsrbRating.TEEN, pageable);
        
        // Then
        assertThat(result).isNotEmpty();
        assertThat(result.getContent()).hasSize(1);
        assertThat(result.getContent().get(0).getEsrbRating()).isEqualTo(EsrbRating.TEEN);
    }
    
    @Test
    void findById_WhenGameExists_ShouldReturnGame() {
        // When
        Optional<Game> result = gameRepository.findById(testGame.getId());
        
        // Then
        assertThat(result).isPresent();
        assertThat(result.get().getTitle()).isEqualTo("Test Game");
    }
    
    @Test
    void findById_WhenGameDoesNotExist_ShouldReturnEmpty() {
        // When
        Optional<Game> result = gameRepository.findById(999L);
        
        // Then
        assertThat(result).isEmpty();
    }
    
    @Test
    void save_ShouldPersistGame() {
        // Given
        Game newGame = Game.builder()
                .title("New Test Game")
                .description("Another test game")
                .releaseDate(LocalDate.of(2024, 6, 1))
                .esrbRating(EsrbRating.EVERYONE)
                .developer(testDeveloper)
                .publisher(testPublisher)
                .price(39.99)
                .active(true)
                .build();
        
        // When
        Game savedGame = gameRepository.save(newGame);
        
        // Then
        assertThat(savedGame.getId()).isNotNull();
        assertThat(savedGame.getTitle()).isEqualTo("New Test Game");
        
        // Verify persistence
        Optional<Game> found = gameRepository.findById(savedGame.getId());
        assertThat(found).isPresent();
        assertThat(found.get().getTitle()).isEqualTo("New Test Game");
    }
    
    @Test
    void delete_ShouldRemoveGame() {
        // Given
        Long gameId = testGame.getId();
        
        // When
        gameRepository.delete(testGame);
        entityManager.flush();
        
        // Then
        Optional<Game> result = gameRepository.findById(gameId);
        assertThat(result).isEmpty();
    }
}
```

### 8.1.3 Test di Service

**GameServiceTest.java**:
```java
package mc.videogiochi.service;

import mc.videogiochi.dto.request.GameCreateRequest;
import mc.videogiochi.dto.response.GameDTO;
import mc.videogiochi.dto.response.PagedResponse;
import mc.videogiochi.entity.*;
import mc.videogiochi.enums.EsrbRating;
import mc.videogiochi.exception.ResourceNotFoundException;
import mc.videogiochi.mapper.GameMapper;
import mc.videogiochi.repository.GameRepository;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageImpl;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;

import java.time.LocalDate;
import java.util.List;
import java.util.Optional;

import static org.assertj.core.api.Assertions.*;
import static org.mockito.ArgumentMatchers.*;
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class)
class GameServiceTest {
    
    @Mock
    private GameRepository gameRepository;
    
    @Mock
    private GameMapper gameMapper;
    
    @InjectMocks
    private GameService gameService;
    
    private Game testGame;
    private GameDTO testGameDTO;
    private GameCreateRequest testCreateRequest;
    
    @BeforeEach
    void setUp() {
        testGame = Game.builder()
                .id(1L)
                .title("Test Game")
                .description("A test game")
                .releaseDate(LocalDate.of(2023, 1, 15))
                .esrbRating(EsrbRating.TEEN)
                .price(29.99)
                .active(true)
                .build();
        
        testGameDTO = GameDTO.builder()
                .id(1L)
                .title("Test Game")
                .description("A test game")
                .releaseDate(LocalDate.of(2023, 1, 15))
                .esrbRating(EsrbRating.TEEN)
                .price(29.99)
                .build();
        
        testCreateRequest = GameCreateRequest.builder()
                .title("Test Game")
                .description("A test game")
                .releaseDate(LocalDate.of(2023, 1, 15))
                .esrbRating(EsrbRating.TEEN)
                .price(29.99)
                .developerId(1L)
                .publisherId(1L)
                .build();
    }
    
    @Test
    void findGameById_WhenGameExists_ShouldReturnGameDTO() {
        // Given
        when(gameRepository.findByIdAndActiveTrue(1L)).thenReturn(Optional.of(testGame));
        when(gameMapper.toDTO(testGame)).thenReturn(testGameDTO);
        
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
    void findGameById_WhenGameDoesNotExist_ShouldThrowException() {
        // Given
        when(gameRepository.findByIdAndActiveTrue(1L)).thenReturn(Optional.empty());
        
        // When & Then
        assertThatThrownBy(() -> gameService.findGameById(1L))
                .isInstanceOf(ResourceNotFoundException.class)
                .hasMessageContaining("Game non trovato con ID: 1");
        
        verify(gameRepository).findByIdAndActiveTrue(1L);
        verify(gameMapper, never()).toDTO(any());
    }
    
    @Test
    void createGame_WithValidRequest_ShouldReturnCreatedGame() {
        // Given
        when(gameMapper.toEntity(testCreateRequest)).thenReturn(testGame);
        when(gameRepository.save(testGame)).thenReturn(testGame);
        when(gameMapper.toDTO(testGame)).thenReturn(testGameDTO);
        
        // When
        GameDTO result = gameService.createGame(testCreateRequest);
        
        // Then
        assertThat(result).isNotNull();
        assertThat(result.getTitle()).isEqualTo("Test Game");
        
        verify(gameMapper).toEntity(testCreateRequest);
        verify(gameRepository).save(testGame);
        verify(gameMapper).toDTO(testGame);
    }
    
    @Test
    void findAllGames_ShouldReturnPagedResponse() {
        // Given
        Pageable pageable = PageRequest.of(0, 10);
        Page<Game> gamePage = new PageImpl<>(List.of(testGame), pageable, 1);
        
        when(gameRepository.findByActiveTrue(pageable)).thenReturn(gamePage);
        when(gameMapper.toSummaryDTO(testGame)).thenReturn(testGameDTO);
        
        // When
        PagedResponse<GameDTO> result = gameService.findAllGames(pageable);
        
        // Then
        assertThat(result).isNotNull();
        assertThat(result.getContent()).hasSize(1);
        assertThat(result.getTotalElements()).isEqualTo(1);
        assertThat(result.getTotalPages()).isEqualTo(1);
        
        verify(gameRepository).findByActiveTrue(pageable);
        verify(gameMapper).toSummaryDTO(testGame);
    }
    
    @Test
    void deleteGame_WhenGameExists_ShouldMarkAsInactive() {
        // Given
        when(gameRepository.findByIdAndActiveTrue(1L)).thenReturn(Optional.of(testGame));
        when(gameRepository.save(testGame)).thenReturn(testGame);
        
        // When
        gameService.deleteGame(1L);
        
        // Then
        assertThat(testGame.isActive()).isFalse();
        
        verify(gameRepository).findByIdAndActiveTrue(1L);
        verify(gameRepository).save(testGame);
    }
    
    @Test
    void deleteGame_WhenGameDoesNotExist_ShouldThrowException() {
        // Given
        when(gameRepository.findByIdAndActiveTrue(1L)).thenReturn(Optional.empty());
        
        // When & Then
        assertThatThrownBy(() -> gameService.deleteGame(1L))
                .isInstanceOf(ResourceNotFoundException.class)
                .hasMessageContaining("Game non trovato con ID: 1");
        
        verify(gameRepository).findByIdAndActiveTrue(1L);
        verify(gameRepository, never()).save(any());
    }
}
```

### 8.1.4 Test di Controller

**GameControllerTest.java**:
```java
package mc.videogiochi.controller;

import com.fasterxml.jackson.databind.ObjectMapper;
import mc.videogiochi.dto.request.GameCreateRequest;
import mc.videogiochi.dto.response.GameDTO;
import mc.videogiochi.dto.response.PagedResponse;
import mc.videogiochi.enums.EsrbRating;
import mc.videogiochi.exception.ResourceNotFoundException;
import mc.videogiochi.service.GameService;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;
import org.springframework.http.MediaType;
import org.springframework.security.test.context.support.WithMockUser;
import org.springframework.test.context.ActiveProfiles;
import org.springframework.test.web.servlet.MockMvc;

import java.time.LocalDate;
import java.util.List;

import static org.mockito.ArgumentMatchers.*;
import static org.mockito.Mockito.*;
import static org.springframework.security.test.web.servlet.request.SecurityMockMvcRequestPostProcessors.csrf;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@WebMvcTest(GameController.class)
@ActiveProfiles("test")
class GameControllerTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @MockBean
    private GameService gameService;
    
    @Autowired
    private ObjectMapper objectMapper;
    
    private GameDTO testGameDTO;
    private GameCreateRequest testCreateRequest;
    
    @BeforeEach
    void setUp() {
        testGameDTO = GameDTO.builder()
                .id(1L)
                .title("Test Game")
                .description("A test game")
                .releaseDate(LocalDate.of(2023, 1, 15))
                .esrbRating(EsrbRating.TEEN)
                .price(29.99)
                .build();
        
        testCreateRequest = GameCreateRequest.builder()
                .title("Test Game")
                .description("A test game")
                .releaseDate(LocalDate.of(2023, 1, 15))
                .esrbRating(EsrbRating.TEEN)
                .price(29.99)
                .developerId(1L)
                .publisherId(1L)
                .build();
    }
    
    @Test
    void getAllGames_ShouldReturnPagedResponse() throws Exception {
        // Given
        PagedResponse<GameDTO> pagedResponse = PagedResponse.<GameDTO>builder()
                .content(List.of(testGameDTO))
                .totalElements(1L)
                .totalPages(1)
                .size(10)
                .number(0)
                .build();
        
        when(gameService.findAllGames(any(Pageable.class))).thenReturn(pagedResponse);
        
        // When & Then
        mockMvc.perform(get("/api/v1/games")
                        .param("page", "0")
                        .param("size", "10")
                        .contentType(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.success").value(true))
                .andExpect(jsonPath("$.data.content").isArray())
                .andExpect(jsonPath("$.data.content[0].id").value(1))
                .andExpect(jsonPath("$.data.content[0].title").value("Test Game"))
                .andExpect(jsonPath("$.data.totalElements").value(1));
        
        verify(gameService).findAllGames(any(Pageable.class));
    }
    
    @Test
    void getGameById_WhenGameExists_ShouldReturnGame() throws Exception {
        // Given
        when(gameService.findGameById(1L)).thenReturn(testGameDTO);
        
        // When & Then
        mockMvc.perform(get("/api/v1/games/1")
                        .contentType(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.success").value(true))
                .andExpect(jsonPath("$.data.id").value(1))
                .andExpect(jsonPath("$.data.title").value("Test Game"));
        
        verify(gameService).findGameById(1L);
    }
    
    @Test
    void getGameById_WhenGameDoesNotExist_ShouldReturnNotFound() throws Exception {
        // Given
        when(gameService.findGameById(1L)).thenThrow(new ResourceNotFoundException("Game", 1L));
        
        // When & Then
        mockMvc.perform(get("/api/v1/games/1")
                        .contentType(MediaType.APPLICATION_JSON))
                .andExpect(status().isNotFound())
                .andExpect(jsonPath("$.success").value(false))
                .andExpect(jsonPath("$.message").value("Game non trovato con ID: 1"));
        
        verify(gameService).findGameById(1L);
    }
    
    @Test
    @WithMockUser(roles = "ADMIN")
    void createGame_WithValidRequest_ShouldReturnCreatedGame() throws Exception {
        // Given
        when(gameService.createGame(any(GameCreateRequest.class))).thenReturn(testGameDTO);
        
        // When & Then
        mockMvc.perform(post("/api/v1/games")
                        .with(csrf())
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(objectMapper.writeValueAsString(testCreateRequest)))
                .andExpect(status().isCreated())
                .andExpected(jsonPath("$.success").value(true))
                .andExpect(jsonPath("$.data.id").value(1))
                .andExpect(jsonPath("$.data.title").value("Test Game"));
        
        verify(gameService).createGame(any(GameCreateRequest.class));
    }
    
    @Test
    void createGame_WithoutAuthentication_ShouldReturnUnauthorized() throws Exception {
        // When & Then
        mockMvc.perform(post("/api/v1/games")
                        .with(csrf())
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(objectMapper.writeValueAsString(testCreateRequest)))
                .andExpect(status().isUnauthorized());
        
        verify(gameService, never()).createGame(any());
    }
    
    @Test
    @WithMockUser(roles = "USER")
    void createGame_WithInsufficientRole_ShouldReturnForbidden() throws Exception {
        // When & Then
        mockMvc.perform(post("/api/v1/games")
                        .with(csrf())
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(objectMapper.writeValueAsString(testCreateRequest)))
                .andExpect(status().isForbidden());
        
        verify(gameService, never()).createGame(any());
    }
    
    @Test
    @WithMockUser(roles = "ADMIN")
    void deleteGame_WhenGameExists_ShouldReturnSuccess() throws Exception {
        // Given
        doNothing().when(gameService).deleteGame(1L);
        
        // When & Then
        mockMvc.perform(delete("/api/v1/games/1")
                        .with(csrf())
                        .contentType(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.success").value(true));
        
        verify(gameService).deleteGame(1L);
    }
}
```

### 8.1.5 Test di Integrazione

**GameIntegrationTest.java**:
```java
package mc.videogiochi.integration;

import com.fasterxml.jackson.databind.ObjectMapper;
import mc.videogiochi.dto.request.GameCreateRequest;
import mc.videogiochi.dto.request.LoginRequest;
import mc.videogiochi.dto.response.JwtResponse;
import mc.videogiochi.enums.EsrbRating;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureWebMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.test.context.ActiveProfiles;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.MvcResult;
import org.springframework.transaction.annotation.Transactional;

import java.time.LocalDate;

import static org.springframework.security.test.web.servlet.request.SecurityMockMvcRequestPostProcessors.csrf;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureWebMvc
@ActiveProfiles("test")
@Transactional
class GameIntegrationTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @Autowired
    private ObjectMapper objectMapper;
    
    private String adminToken;
    private String userToken;
    
    @BeforeEach
    void setUp() throws Exception {
        // Ottieni token admin
        LoginRequest adminLogin = new LoginRequest("admin", "admin123");
        MvcResult adminResult = mockMvc.perform(post("/api/v1/auth/login")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(objectMapper.writeValueAsString(adminLogin)))
                .andExpect(status().isOk())
                .andReturn();
        
        String adminResponse = adminResult.getResponse().getContentAsString();
        JwtResponse adminJwtResponse = objectMapper.readValue(adminResponse, JwtResponse.class);
        adminToken = "Bearer " + adminJwtResponse.getToken();
        
        // Ottieni token user
        LoginRequest userLogin = new LoginRequest("user", "user123");
        MvcResult userResult = mockMvc.perform(post("/api/v1/auth/login")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(objectMapper.writeValueAsString(userLogin)))
                .andExpect(status().isOk())
                .andReturn();
        
        String userResponse = userResult.getResponse().getContentAsString();
        JwtResponse userJwtResponse = objectMapper.readValue(userResponse, JwtResponse.class);
        userToken = "Bearer " + userJwtResponse.getToken();
    }
    
    @Test
    void completeGameWorkflow_ShouldWork() throws Exception {
        // 1. Crea un nuovo gioco (come admin)
        GameCreateRequest createRequest = GameCreateRequest.builder()
                .title("Integration Test Game")
                .description("A game created during integration testing")
                .releaseDate(LocalDate.of(2024, 6, 1))
                .esrbRating(EsrbRating.EVERYONE)
                .price(39.99)
                .developerId(1L)
                .publisherId(1L)
                .build();
        
        MvcResult createResult = mockMvc.perform(post("/api/v1/games")
                        .with(csrf())
                        .header("Authorization", adminToken)
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(objectMapper.writeValueAsString(createRequest)))
                .andExpect(status().isCreated())
                .andExpect(jsonPath("$.success").value(true))
                .andExpect(jsonPath("$.data.title").value("Integration Test Game"))
                .andReturn();
        
        // Estrai l'ID del gioco creato
        String createResponse = createResult.getResponse().getContentAsString();
        Long gameId = objectMapper.readTree(createResponse).get("data").get("id").asLong();
        
        // 2. Recupera il gioco (pubblico)
        mockMvc.perform(get("/api/v1/games/" + gameId)
                        .contentType(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.success").value(true))
                .andExpect(jsonPath("$.data.id").value(gameId))
                .andExpect(jsonPath("$.data.title").value("Integration Test Game"));
        
        // 3. Cerca giochi per titolo (pubblico)
        mockMvc.perform(get("/api/v1/games/search/title")
                        .param("title", "Integration")
                        .contentType(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.success").value(true))
                .andExpect(jsonPath("$.data.content").isArray())
                .andExpect(jsonPath("$.data.content[0].title").value("Integration Test Game"));
        
        // 4. Tenta di eliminare come user (dovrebbe fallire)
        mockMvc.perform(delete("/api/v1/games/" + gameId)
                        .with(csrf())
                        .header("Authorization", userToken)
                        .contentType(MediaType.APPLICATION_JSON))
                .andExpect(status().isForbidden());
        
        // 5. Elimina il gioco (come admin)
        mockMvc.perform(delete("/api/v1/games/" + gameId)
                        .with(csrf())
                        .header("Authorization", adminToken)
                        .contentType(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.success").value(true));
        
        // 6. Verifica che il gioco non sia più accessibile
        mockMvc.perform(get("/api/v1/games/" + gameId)
                        .contentType(MediaType.APPLICATION_JSON))
                .andExpect(status().isNotFound());
    }
}
```

## 8.2 Configurazioni Avanzate

### 8.2.1 Configurazione Cache

**CacheConfig.java**:
```java
package mc.videogiochi.config;

import com.github.benmanes.caffeine.cache.Caffeine;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.cache.CacheManager;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.cache.caffeine.CaffeineCacheManager;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.concurrent.TimeUnit;

@Configuration
@EnableCaching
public class CacheConfig {
    
    @Value("${app.cache.ttl:3600}")
    private long cacheTtl;
    
    @Value("${app.cache.max-size:1000}")
    private long cacheMaxSize;
    
    @Bean
    public CacheManager cacheManager() {
        CaffeineCacheManager cacheManager = new CaffeineCacheManager();
        cacheManager.setCaffeine(caffeineCacheBuilder());
        cacheManager.setCacheNames("games", "users", "genres", "platforms", "developers", "publishers");
        return cacheManager;
    }
    
    private Caffeine<Object, Object> caffeineCacheBuilder() {
        return Caffeine.newBuilder()
                .maximumSize(cacheMaxSize)
                .expireAfterWrite(cacheTtl, TimeUnit.SECONDS)
                .recordStats();
    }
}
```

### 8.2.2 Configurazione Async

**AsyncConfig.java**:
```java
package mc.videogiochi.config;

import lombok.extern.slf4j.Slf4j;
import org.springframework.aop.interceptor.AsyncUncaughtExceptionHandler;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.annotation.AsyncConfigurer;
import org.springframework.scheduling.annotation.EnableAsync;
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;

import java.util.concurrent.Executor;

@Configuration
@EnableAsync
@Slf4j
public class AsyncConfig implements AsyncConfigurer {
    
    @Bean(name = "taskExecutor")
    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("VideoGiochi-Async-");
        executor.setWaitForTasksToCompleteOnShutdown(true);
        executor.setAwaitTerminationSeconds(60);
        executor.initialize();
        return executor;
    }
    
    @Override
    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
        return (throwable, method, objects) -> {
            log.error("Errore in metodo asincrono: {} - {}", method.getName(), throwable.getMessage(), throwable);
        };
    }
}
```

### 8.2.3 Configurazione Monitoring

**MonitoringConfig.java**:
```java
package mc.videogiochi.config;

import io.micrometer.core.aop.TimedAspect;
import io.micrometer.core.instrument.MeterRegistry;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.EnableAspectJAutoProxy;

@Configuration
@EnableAspectJAutoProxy
public class MonitoringConfig {
    
    @Bean
    public TimedAspect timedAspect(MeterRegistry registry) {
        return new TimedAspect(registry);
    }
}
```

## 8.3 Deployment

### 8.3.1 Dockerfile

**Dockerfile**:
```dockerfile
# Build stage
FROM maven:3.9.4-eclipse-temurin-17 AS build

WORKDIR /app

# Copy pom.xml and download dependencies
COPY pom.xml .
RUN mvn dependency:go-offline -B

# Copy source code and build
COPY src ./src
RUN mvn clean package -DskipTests

# Runtime stage
FROM eclipse-temurin:17-jre-alpine

# Create app user
RUN addgroup -g 1001 -S appgroup && \
    adduser -u 1001 -S appuser -G appgroup

WORKDIR /app

# Copy jar from build stage
COPY --from=build /app/target/*.jar app.jar

# Change ownership
RUN chown -R appuser:appgroup /app

# Switch to non-root user
USER appuser

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:8080/actuator/health || exit 1

# Expose port
EXPOSE 8080

# Run application
ENTRYPOINT ["java", "-jar", "/app/app.jar"]
```

### 8.3.2 Docker Compose

**docker-compose.yml**:
```yaml
version: '3.8'

services:
  mysql:
    image: mysql:8.0
    container_name: videogiochi-mysql
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: videogiochi
      MYSQL_USER: videogiochi_user
      MYSQL_PASSWORD: videogiochi_pass
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql
      - ./docker/mysql/init:/docker-entrypoint-initdb.d
    networks:
      - videogiochi-network
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      timeout: 20s
      retries: 10

  redis:
    image: redis:7-alpine
    container_name: videogiochi-redis
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    networks:
      - videogiochi-network
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 30s
      timeout: 3s
      retries: 5

  app:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: videogiochi-app
    environment:
      SPRING_PROFILES_ACTIVE: docker
      SPRING_DATASOURCE_URL: jdbc:mysql://mysql:3306/videogiochi
      SPRING_DATASOURCE_USERNAME: videogiochi_user
      SPRING_DATASOURCE_PASSWORD: videogiochi_pass
      SPRING_REDIS_HOST: redis
      SPRING_REDIS_PORT: 6379
    ports:
      - "8080:8080"
    depends_on:
      mysql:
        condition: service_healthy
      redis:
        condition: service_healthy
    networks:
      - videogiochi-network
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:8080/actuator/health"]
      interval: 30s
      timeout: 10s
      retries: 5
      start_period: 40s

volumes:
  mysql_data:
  redis_data:

networks:
  videogiochi-network:
    driver: bridge
```

**application-docker.yml**:
```yaml
spring:
  datasource:
    url: ${SPRING_DATASOURCE_URL}
    username: ${SPRING_DATASOURCE_USERNAME}
    password: ${SPRING_DATASOURCE_PASSWORD}
    hikari:
      maximum-pool-size: 20
      minimum-idle: 5
      connection-timeout: 20000
      idle-timeout: 300000
      max-lifetime: 1200000
  
  jpa:
    hibernate:
      ddl-auto: validate
    show-sql: false
    properties:
      hibernate:
        dialect: org.hibernate.dialect.MySQL8Dialect
        format_sql: false
        use_sql_comments: false
        jdbc:
          batch_size: 20
        order_inserts: true
        order_updates: true
  
  flyway:
    enabled: true
    baseline-on-migrate: true
    validate-on-migrate: true
  
  cache:
    type: redis
  
  data:
    redis:
      host: ${SPRING_REDIS_HOST:localhost}
      port: ${SPRING_REDIS_PORT:6379}
      timeout: 2000ms
      lettuce:
        pool:
          max-active: 8
          max-idle: 8
          min-idle: 0

management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
  endpoint:
    health:
      show-details: when-authorized
  metrics:
    export:
      prometheus:
        enabled: true

logging:
  level:
    mc.videogiochi: INFO
    org.springframework.security: WARN
    org.hibernate.SQL: WARN
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} - %msg%n"
    file: "%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n"
  file:
    name: /app/logs/videogiochi.log
    max-size: 100MB
    max-history: 30
```

### 8.3.3 Script di Deployment

**deploy.sh**:
```bash
#!/bin/bash

set -e

echo "🚀 Avvio deployment applicazione VideoGiochi..."

# Variabili
APP_NAME="videogiochi"
DOCKER_COMPOSE_FILE="docker-compose.yml"
BACKUP_DIR="./backups"
DATE=$(date +"%Y%m%d_%H%M%S")

# Funzioni
log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $1"
}

error_exit() {
    log "❌ ERRORE: $1"
    exit 1
}

backup_database() {
    log "📦 Creazione backup database..."
    mkdir -p $BACKUP_DIR
    
    docker exec videogiochi-mysql mysqldump \
        -u videogiochi_user \
        -pvideogiochi_pass \
        videogiochi > "$BACKUP_DIR/backup_$DATE.sql" || error_exit "Backup database fallito"
    
    log "✅ Backup creato: $BACKUP_DIR/backup_$DATE.sql"
}

stop_services() {
    log "🛑 Arresto servizi esistenti..."
    docker-compose -f $DOCKER_COMPOSE_FILE down || true
}

build_application() {
    log "🔨 Build applicazione..."
    mvn clean package -DskipTests || error_exit "Build fallito"
}

start_services() {
    log "🚀 Avvio servizi..."
    docker-compose -f $DOCKER_COMPOSE_FILE up -d || error_exit "Avvio servizi fallito"
}

wait_for_health() {
    log "⏳ Attesa che i servizi siano pronti..."
    
    # Attendi MySQL
    for i in {1..30}; do
        if docker exec videogiochi-mysql mysqladmin ping -h localhost --silent; then
            log "✅ MySQL è pronto"
            break
        fi
        if [ $i -eq 30 ]; then
            error_exit "MySQL non è diventato disponibile in tempo"
        fi
        sleep 2
    done
    
    # Attendi applicazione
    for i in {1..60}; do
        if curl -f http://localhost:8080/actuator/health > /dev/null 2>&1; then
            log "✅ Applicazione è pronta"
            break
        fi
        if [ $i -eq 60 ]; then
            error_exit "Applicazione non è diventata disponibile in tempo"
        fi
        sleep 5
    done
}

run_tests() {
    log "🧪 Esecuzione test di smoke..."
    
    # Test endpoint di health
    curl -f http://localhost:8080/actuator/health || error_exit "Health check fallito"
    
    # Test endpoint pubblico
    curl -f http://localhost:8080/api/v1/games?size=1 || error_exit "Test API fallito"
    
    log "✅ Test completati con successo"
}

cleanup_old_images() {
    log "🧹 Pulizia immagini Docker obsolete..."
    docker image prune -f || true
}

# Esecuzione deployment
log "🎯 Inizio deployment di $APP_NAME"

# Verifica prerequisiti
command -v docker >/dev/null 2>&1 || error_exit "Docker non installato"
command -v docker-compose >/dev/null 2>&1 || error_exit "Docker Compose non installato"
command -v mvn >/dev/null 2>&1 || error_exit "Maven non installato"

# Backup se i servizi sono già in esecuzione
if docker ps | grep -q videogiochi-mysql; then
    backup_database
fi

# Deployment
stop_services
build_application
start_services
wait_for_health
run_tests
cleanup_old_images

log "🎉 Deployment completato con successo!"
log "📊 Applicazione disponibile su: http://localhost:8080"
log "📈 Metriche disponibili su: http://localhost:8080/actuator/prometheus"
log "🏥 Health check: http://localhost:8080/actuator/health"
```

### 8.3.4 Configurazione CI/CD

**.github/workflows/ci-cd.yml**:
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
          MYSQL_DATABASE: videogiochi_test
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
    
    - name: Run tests
      run: mvn clean test
      env:
        SPRING_PROFILES_ACTIVE: test
    
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

  build-and-deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up JDK 17
      uses: actions/setup-java@v3
      with:
        java-version: '17'
        distribution: 'temurin'
    
    - name: Build application
      run: mvn clean package -DskipTests
    
    - name: Build Docker image
      run: docker build -t videogiochi:${{ github.sha }} .
    
    - name: Run security scan
      uses: aquasecurity/trivy-action@master
      with:
        image-ref: 'videogiochi:${{ github.sha }}'
        format: 'sarif'
        output: 'trivy-results.sarif'
    
    - name: Upload Trivy scan results
      uses: github/codeql-action/upload-sarif@v2
      with:
        sarif_file: 'trivy-results.sarif'
    
    # Deploy to staging/production based on branch
    - name: Deploy to staging
      if: github.ref == 'refs/heads/develop'
      run: |
        echo "Deploy to staging environment"
        # Add staging deployment commands here
    
    - name: Deploy to production
      if: github.ref == 'refs/heads/main'
      run: |
        echo "Deploy to production environment"
        # Add production deployment commands here
```

Questo documento completa la documentazione del backend, coprendo testing, configurazioni avanzate e deployment, fornendo una base solida per lo sviluppo, il testing e la messa in produzione dell'applicazione.