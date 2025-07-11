# 6. Controller REST e Sicurezza

> **Riferimenti teorici**: Questo capitolo implementa il **Controller Layer** del pattern MVC e le strategie di sicurezza descritte in [Pattern Architetturali](../knowledge/01-pattern-architetturali.md) e [Sicurezza Web](../knowledge/04-sicurezza-web.md)

## 6.1 Controller Layer

> **Pattern implementato**: I Controller gestiscono le richieste HTTP e coordinano tra Presentation Layer e Service Layer.

### 6.1.1 Controller Base

> **Best Practice**: Il BaseController centralizza la gestione delle risposte API, garantendo consistenza in tutta l'applicazione.

**BaseController.java**:
```java
package mc.videogiochi.controller;

import mc.videogiochi.dto.response.ApiResponse;
import mc.videogiochi.exception.ValidationException;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.validation.BindingResult;
import org.springframework.validation.FieldError;

import java.util.HashMap;
import java.util.Map;

public abstract class BaseController {
    
    protected <T> ResponseEntity<ApiResponse<T>> success(T data) {
        return ResponseEntity.ok(ApiResponse.success(data));
    }
    
    protected <T> ResponseEntity<ApiResponse<T>> success(T data, String message) {
        return ResponseEntity.ok(ApiResponse.success(data, message));
    }
    
    protected <T> ResponseEntity<ApiResponse<T>> created(T data) {
        return ResponseEntity.status(HttpStatus.CREATED)
                .body(ApiResponse.success(data, "Risorsa creata con successo"));
    }
    
    protected ResponseEntity<ApiResponse<Void>> noContent() {
        return ResponseEntity.ok(ApiResponse.success(null, "Operazione completata con successo"));
    }
    
    protected void validateRequest(BindingResult bindingResult) {
        if (bindingResult.hasErrors()) {
            Map<String, String> errors = new HashMap<>();
            for (FieldError error : bindingResult.getFieldErrors()) {
                errors.put(error.getField(), error.getDefaultMessage());
            }
            throw new ValidationException("Errori di validazione", errors);
        }
    }
}
```

**ApiResponse.java**:
```java
package mc.videogiochi.dto.response;

import lombok.*;
import com.fasterxml.jackson.annotation.JsonInclude;

import java.time.LocalDateTime;
import java.util.Map;

@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
@JsonInclude(JsonInclude.Include.NON_NULL)
public class ApiResponse<T> {
    
    private boolean success;
    private String message;
    private T data;
    private Map<String, String> errors;
    private LocalDateTime timestamp;
    private String path;
    
    public static <T> ApiResponse<T> success(T data) {
        return ApiResponse.<T>builder()
                .success(true)
                .data(data)
                .timestamp(LocalDateTime.now())
                .build();
    }
    
    public static <T> ApiResponse<T> success(T data, String message) {
        return ApiResponse.<T>builder()
                .success(true)
                .message(message)
                .data(data)
                .timestamp(LocalDateTime.now())
                .build();
    }
    
    public static <T> ApiResponse<T> error(String message) {
        return ApiResponse.<T>builder()
                .success(false)
                .message(message)
                .timestamp(LocalDateTime.now())
                .build();
    }
    
    public static <T> ApiResponse<T> error(String message, Map<String, String> errors) {
        return ApiResponse.<T>builder()
                .success(false)
                .message(message)
                .errors(errors)
                .timestamp(LocalDateTime.now())
                .build();
    }
}
```

### 6.1.2 Game Controller

> **Responsabilità**: Il GameController espone le API REST per la gestione dei videogiochi, implementando autorizzazione basata sui ruoli.

**GameController.java**:
```java
package mc.videogiochi.controller;

import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.Parameter;
import io.swagger.v3.oas.annotations.responses.ApiResponses;
import io.swagger.v3.oas.annotations.responses.ApiResponse as SwaggerApiResponse;
import io.swagger.v3.oas.annotations.security.SecurityRequirement;
import io.swagger.v3.oas.annotations.tags.Tag;
import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import mc.videogiochi.dto.request.*;
import mc.videogiochi.dto.response.*;
import mc.videogiochi.service.GameService;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.Sort;
import org.springframework.data.web.PageableDefault;
import org.springframework.http.ResponseEntity;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.validation.BindingResult;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/v1/games")
@RequiredArgsConstructor
@Slf4j
@Tag(name = "Games", description = "API per la gestione dei videogiochi")
public class GameController extends BaseController {
    
    private final GameService gameService;
    
    @GetMapping
    @Operation(
        summary = "Ottieni lista paginata di giochi",
        description = "Restituisce una lista paginata di tutti i giochi attivi"
    )
    @ApiResponses({
        @SwaggerApiResponse(responseCode = "200", description = "Lista giochi recuperata con successo"),
        @SwaggerApiResponse(responseCode = "400", description = "Parametri di paginazione non validi")
    })
    public ResponseEntity<ApiResponse<PagedResponse<GameSummaryDTO>>> getAllGames(
            @PageableDefault(size = 20, sort = "title", direction = Sort.Direction.ASC) 
            @Parameter(description = "Parametri di paginazione") Pageable pageable) {
        
        log.debug("Richiesta lista giochi - Pagina: {}, Dimensione: {}", 
                 pageable.getPageNumber(), pageable.getPageSize());
        
        PagedResponse<GameSummaryDTO> games = gameService.findAllGames(pageable);
        return success(games);
    }
    
    @GetMapping("/{id}")
    @Operation(
        summary = "Ottieni dettagli gioco",
        description = "Restituisce i dettagli completi di un gioco specifico"
    )
    @ApiResponses({
        @SwaggerApiResponse(responseCode = "200", description = "Gioco trovato"),
        @SwaggerApiResponse(responseCode = "404", description = "Gioco non trovato")
    })
    public ResponseEntity<ApiResponse<GameDTO>> getGameById(
            @PathVariable @Parameter(description = "ID del gioco") Long id) {
        
        log.debug("Richiesta dettagli gioco ID: {}", id);
        
        GameDTO game = gameService.findGameById(id);
        return success(game);
    }
    
    @PostMapping
    @PreAuthorize("hasRole('ADMIN') or hasRole('MODERATOR')")
    @SecurityRequirement(name = "bearerAuth")
    @Operation(
        summary = "Crea nuovo gioco",
        description = "Crea un nuovo gioco nel sistema. Richiede ruolo ADMIN o MODERATOR"
    )
    @ApiResponses({
        @SwaggerApiResponse(responseCode = "201", description = "Gioco creato con successo"),
        @SwaggerApiResponse(responseCode = "400", description = "Dati di input non validi"),
        @SwaggerApiResponse(responseCode = "403", description = "Accesso negato")
    })
    public ResponseEntity<ApiResponse<GameDTO>> createGame(
            @Valid @RequestBody GameCreateRequest request,
            BindingResult bindingResult) {
        
        validateRequest(bindingResult);
        
        log.info("Creazione nuovo gioco: {}", request.getTitle());
        
        GameDTO createdGame = gameService.createGame(request);
        return created(createdGame);
    }
    
    @PutMapping("/{id}")
    @PreAuthorize("hasRole('ADMIN') or hasRole('MODERATOR')")
    @SecurityRequirement(name = "bearerAuth")
    @Operation(
        summary = "Aggiorna gioco",
        description = "Aggiorna i dati di un gioco esistente. Richiede ruolo ADMIN o MODERATOR"
    )
    @ApiResponses({
        @SwaggerApiResponse(responseCode = "200", description = "Gioco aggiornato con successo"),
        @SwaggerApiResponse(responseCode = "400", description = "Dati di input non validi"),
        @SwaggerApiResponse(responseCode = "404", description = "Gioco non trovato"),
        @SwaggerApiResponse(responseCode = "403", description = "Accesso negato")
    })
    public ResponseEntity<ApiResponse<GameDTO>> updateGame(
            @PathVariable Long id,
            @Valid @RequestBody GameUpdateRequest request,
            BindingResult bindingResult) {
        
        validateRequest(bindingResult);
        
        log.info("Aggiornamento gioco ID: {}", id);
        
        GameDTO updatedGame = gameService.updateGame(id, request);
        return success(updatedGame, "Gioco aggiornato con successo");
    }
    
    @DeleteMapping("/{id}")
    @PreAuthorize("hasRole('ADMIN')")
    @SecurityRequirement(name = "bearerAuth")
    @Operation(
        summary = "Elimina gioco",
        description = "Elimina un gioco dal sistema (soft delete). Richiede ruolo ADMIN"
    )
    @ApiResponses({
        @SwaggerApiResponse(responseCode = "200", description = "Gioco eliminato con successo"),
        @SwaggerApiResponse(responseCode = "404", description = "Gioco non trovato"),
        @SwaggerApiResponse(responseCode = "403", description = "Accesso negato")
    })
    public ResponseEntity<ApiResponse<Void>> deleteGame(@PathVariable Long id) {
        log.info("Eliminazione gioco ID: {}", id);
        
        gameService.deleteGame(id);
        return noContent();
    }
    
    @GetMapping("/search")
    @Operation(
        summary = "Ricerca giochi",
        description = "Ricerca giochi con criteri multipli"
    )
    public ResponseEntity<ApiResponse<PagedResponse<GameSummaryDTO>>> searchGames(
            @Valid GameSearchCriteria criteria,
            @PageableDefault(size = 20, sort = "popularityScore", direction = Sort.Direction.DESC) 
            Pageable pageable) {
        
        log.debug("Ricerca giochi con criteri: {}", criteria);
        
        PagedResponse<GameSummaryDTO> games = gameService.searchGames(criteria, pageable);
        return success(games);
    }
    
    @GetMapping("/search/title")
    @Operation(
        summary = "Ricerca giochi per titolo",
        description = "Ricerca giochi che contengono il testo specificato nel titolo"
    )
    public ResponseEntity<ApiResponse<PagedResponse<GameSummaryDTO>>> searchGamesByTitle(
            @RequestParam @Parameter(description = "Testo da cercare nel titolo") String title,
            @PageableDefault(size = 20) Pageable pageable) {
        
        log.debug("Ricerca giochi per titolo: {}", title);
        
        PagedResponse<GameSummaryDTO> games = gameService.findGamesByTitle(title, pageable);
        return success(games);
    }
    
    @GetMapping("/genre/{genreId}")
    @Operation(
        summary = "Giochi per genere",
        description = "Ottieni tutti i giochi di un genere specifico"
    )
    public ResponseEntity<ApiResponse<PagedResponse<GameSummaryDTO>>> getGamesByGenre(
            @PathVariable Long genreId,
            @PageableDefault(size = 20) Pageable pageable) {
        
        log.debug("Ricerca giochi per genere ID: {}", genreId);
        
        PagedResponse<GameSummaryDTO> games = gameService.findGamesByGenre(genreId, pageable);
        return success(games);
    }
    
    @GetMapping("/platform/{platformId}")
    @Operation(
        summary = "Giochi per piattaforma",
        description = "Ottieni tutti i giochi di una piattaforma specifica"
    )
    public ResponseEntity<ApiResponse<PagedResponse<GameSummaryDTO>>> getGamesByPlatform(
            @PathVariable Long platformId,
            @PageableDefault(size = 20) Pageable pageable) {
        
        log.debug("Ricerca giochi per piattaforma ID: {}", platformId);
        
        PagedResponse<GameSummaryDTO> games = gameService.findGamesByPlatform(platformId, pageable);
        return success(games);
    }
    
    @GetMapping("/developer/{developerId}")
    @Operation(
        summary = "Giochi per sviluppatore",
        description = "Ottieni tutti i giochi di uno sviluppatore specifico"
    )
    public ResponseEntity<ApiResponse<PagedResponse<GameSummaryDTO>>> getGamesByDeveloper(
            @PathVariable Long developerId,
            @PageableDefault(size = 20) Pageable pageable) {
        
        log.debug("Ricerca giochi per sviluppatore ID: {}", developerId);
        
        PagedResponse<GameSummaryDTO> games = gameService.findGamesByDeveloper(developerId, pageable);
        return success(games);
    }
    
    @GetMapping("/recommendations")
    @PreAuthorize("isAuthenticated()")
    @SecurityRequirement(name = "bearerAuth")
    @Operation(
        summary = "Raccomandazioni personalizzate",
        description = "Ottieni raccomandazioni di giochi basate sui gusti dell'utente"
    )
    public ResponseEntity<ApiResponse<PagedResponse<GameSummaryDTO>>> getRecommendations(
            @PageableDefault(size = 10) Pageable pageable) {
        
        // L'ID utente sarà estratto dal SecurityContext nel service
        PagedResponse<GameSummaryDTO> recommendations = gameService.getRecommendationsForCurrentUser(pageable);
        return success(recommendations);
    }
    
    @GetMapping("/{id}/similar")
    @Operation(
        summary = "Giochi simili",
        description = "Ottieni giochi simili a quello specificato"
    )
    public ResponseEntity<ApiResponse<List<GameSummaryDTO>>> getSimilarGames(
            @PathVariable Long id,
            @RequestParam(defaultValue = "5") @Parameter(description = "Numero massimo di giochi da restituire") int limit) {
        
        log.debug("Ricerca giochi simili a ID: {}, limite: {}", id, limit);
        
        List<GameSummaryDTO> similarGames = gameService.getSimilarGames(id, limit);
        return success(similarGames);
    }
    
    @GetMapping("/statistics")
    @Operation(
        summary = "Statistiche giochi",
        description = "Ottieni statistiche generali sui giochi"
    )
    public ResponseEntity<ApiResponse<GameStatisticsDTO>> getGameStatistics() {
        GameStatisticsDTO statistics = gameService.getGameStatistics();
        return success(statistics);
    }
}
```

### 6.1.3 User Controller

**UserController.java**:
```java
package mc.videogiochi.controller;

import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.security.SecurityRequirement;
import io.swagger.v3.oas.annotations.tags.Tag;
import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import mc.videogiochi.dto.request.*;
import mc.videogiochi.dto.response.*;
import mc.videogiochi.service.UserService;
import mc.videogiochi.service.SecurityService;
import org.springframework.data.domain.Pageable;
import org.springframework.data.web.PageableDefault;
import org.springframework.http.ResponseEntity;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.validation.BindingResult;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/v1/users")
@RequiredArgsConstructor
@Slf4j
@Tag(name = "Users", description = "API per la gestione degli utenti")
public class UserController extends BaseController {
    
    private final UserService userService;
    private final SecurityService securityService;
    
    @PostMapping("/register")
    @Operation(
        summary = "Registrazione utente",
        description = "Registra un nuovo utente nel sistema"
    )
    public ResponseEntity<ApiResponse<UserDTO>> registerUser(
            @Valid @RequestBody UserRegistrationRequest request,
            BindingResult bindingResult) {
        
        validateRequest(bindingResult);
        
        log.info("Registrazione nuovo utente: {}", request.getUsername());
        
        UserDTO createdUser = userService.createUser(request);
        return created(createdUser);
    }
    
    @GetMapping("/profile")
    @PreAuthorize("isAuthenticated()")
    @SecurityRequirement(name = "bearerAuth")
    @Operation(
        summary = "Profilo utente corrente",
        description = "Ottieni il profilo dell'utente attualmente autenticato"
    )
    public ResponseEntity<ApiResponse<UserProfileDTO>> getCurrentUserProfile() {
        Long currentUserId = securityService.getCurrentUserId();
        UserProfileDTO profile = userService.findUserProfile(currentUserId);
        return success(profile);
    }
    
    @PutMapping("/profile")
    @PreAuthorize("isAuthenticated()")
    @SecurityRequirement(name = "bearerAuth")
    @Operation(
        summary = "Aggiorna profilo",
        description = "Aggiorna il profilo dell'utente corrente"
    )
    public ResponseEntity<ApiResponse<UserDTO>> updateCurrentUserProfile(
            @Valid @RequestBody UserUpdateRequest request,
            BindingResult bindingResult) {
        
        validateRequest(bindingResult);
        
        Long currentUserId = securityService.getCurrentUserId();
        log.info("Aggiornamento profilo utente ID: {}", currentUserId);
        
        UserDTO updatedUser = userService.updateUser(currentUserId, request);
        return success(updatedUser, "Profilo aggiornato con successo");
    }
    
    @PostMapping("/change-password")
    @PreAuthorize("isAuthenticated()")
    @SecurityRequirement(name = "bearerAuth")
    @Operation(
        summary = "Cambia password",
        description = "Cambia la password dell'utente corrente"
    )
    public ResponseEntity<ApiResponse<Void>> changePassword(
            @Valid @RequestBody ChangePasswordRequest request,
            BindingResult bindingResult) {
        
        validateRequest(bindingResult);
        
        Long currentUserId = securityService.getCurrentUserId();
        log.info("Cambio password per utente ID: {}", currentUserId);
        
        userService.changePassword(currentUserId, request);
        return success(null, "Password cambiata con successo");
    }
    
    @GetMapping("/{id}")
    @Operation(
        summary = "Profilo utente pubblico",
        description = "Ottieni il profilo pubblico di un utente"
    )
    public ResponseEntity<ApiResponse<UserProfileDTO>> getUserProfile(@PathVariable Long id) {
        log.debug("Richiesta profilo pubblico utente ID: {}", id);
        
        UserProfileDTO profile = userService.findUserProfile(id);
        return success(profile);
    }
    
    // ==================== Admin Operations ====================
    
    @GetMapping
    @PreAuthorize("hasRole('ADMIN')")
    @SecurityRequirement(name = "bearerAuth")
    @Operation(
        summary = "Lista utenti (Admin)",
        description = "Ottieni lista paginata di tutti gli utenti. Richiede ruolo ADMIN"
    )
    public ResponseEntity<ApiResponse<PagedResponse<UserDTO>>> getAllUsers(
            @PageableDefault(size = 20) Pageable pageable) {
        
        log.debug("Richiesta lista utenti (Admin)");
        
        PagedResponse<UserDTO> users = userService.findAllUsers(pageable);
        return success(users);
    }
    
    @PutMapping("/{id}")
    @PreAuthorize("hasRole('ADMIN')")
    @SecurityRequirement(name = "bearerAuth")
    @Operation(
        summary = "Aggiorna utente (Admin)",
        description = "Aggiorna i dati di un utente. Richiede ruolo ADMIN"
    )
    public ResponseEntity<ApiResponse<UserDTO>> updateUser(
            @PathVariable Long id,
            @Valid @RequestBody UserUpdateRequest request,
            BindingResult bindingResult) {
        
        validateRequest(bindingResult);
        
        log.info("Aggiornamento utente ID: {} (Admin)", id);
        
        UserDTO updatedUser = userService.updateUser(id, request);
        return success(updatedUser, "Utente aggiornato con successo");
    }
    
    @DeleteMapping("/{id}")
    @PreAuthorize("hasRole('ADMIN')")
    @SecurityRequirement(name = "bearerAuth")
    @Operation(
        summary = "Disattiva utente (Admin)",
        description = "Disattiva un utente. Richiede ruolo ADMIN"
    )
    public ResponseEntity<ApiResponse<Void>> deactivateUser(@PathVariable Long id) {
        log.info("Disattivazione utente ID: {} (Admin)", id);
        
        userService.deactivateUser(id);
        return success(null, "Utente disattivato con successo");
    }
    
    @PostMapping("/{id}/roles/{roleName}")
    @PreAuthorize("hasRole('ADMIN')")
    @SecurityRequirement(name = "bearerAuth")
    @Operation(
        summary = "Aggiungi ruolo (Admin)",
        description = "Aggiungi un ruolo a un utente. Richiede ruolo ADMIN"
    )
    public ResponseEntity<ApiResponse<Void>> addRoleToUser(
            @PathVariable Long id,
            @PathVariable String roleName) {
        
        log.info("Aggiunta ruolo {} all'utente ID: {} (Admin)", roleName, id);
        
        userService.addRoleToUser(id, roleName);
        return success(null, "Ruolo aggiunto con successo");
    }
    
    @DeleteMapping("/{id}/roles/{roleName}")
    @PreAuthorize("hasRole('ADMIN')")
    @SecurityRequirement(name = "bearerAuth")
    @Operation(
        summary = "Rimuovi ruolo (Admin)",
        description = "Rimuovi un ruolo da un utente. Richiede ruolo ADMIN"
    )
    public ResponseEntity<ApiResponse<Void>> removeRoleFromUser(
            @PathVariable Long id,
            @PathVariable String roleName) {
        
        log.info("Rimozione ruolo {} dall'utente ID: {} (Admin)", roleName, id);
        
        userService.removeRoleFromUser(id, roleName);
        return success(null, "Ruolo rimosso con successo");
    }
}
```

### 6.1.4 Authentication Controller

**AuthController.java**:
```java
package mc.videogiochi.controller;

import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.tags.Tag;
import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import mc.videogiochi.dto.request.LoginRequest;
import mc.videogiochi.dto.response.ApiResponse;
import mc.videogiochi.dto.response.JwtResponse;
import mc.videogiochi.service.AuthenticationService;
import org.springframework.http.ResponseEntity;
import org.springframework.validation.BindingResult;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/v1/auth")
@RequiredArgsConstructor
@Slf4j
@Tag(name = "Authentication", description = "API per autenticazione e autorizzazione")
public class AuthController extends BaseController {
    
    private final AuthenticationService authenticationService;
    
    @PostMapping("/login")
    @Operation(
        summary = "Login utente",
        description = "Autentica un utente e restituisce un JWT token"
    )
    public ResponseEntity<ApiResponse<JwtResponse>> login(
            @Valid @RequestBody LoginRequest request,
            BindingResult bindingResult) {
        
        validateRequest(bindingResult);
        
        log.info("Tentativo di login per utente: {}", request.getUsername());
        
        JwtResponse jwtResponse = authenticationService.authenticate(request);
        return success(jwtResponse, "Login effettuato con successo");
    }
    
    @PostMapping("/logout")
    @Operation(
        summary = "Logout utente",
        description = "Effettua il logout dell'utente corrente"
    )
    public ResponseEntity<ApiResponse<Void>> logout(
            @RequestHeader("Authorization") String authHeader) {
        
        log.info("Logout utente");
        
        authenticationService.logout(authHeader);
        return success(null, "Logout effettuato con successo");
    }
    
    @PostMapping("/refresh")
    @Operation(
        summary = "Refresh token",
        description = "Rinnova il JWT token"
    )
    public ResponseEntity<ApiResponse<JwtResponse>> refreshToken(
            @RequestHeader("Authorization") String authHeader) {
        
        log.debug("Richiesta refresh token");
        
        JwtResponse jwtResponse = authenticationService.refreshToken(authHeader);
        return success(jwtResponse, "Token rinnovato con successo");
    }
    
    @PostMapping("/verify-email")
    @Operation(
        summary = "Verifica email",
        description = "Verifica l'indirizzo email dell'utente"
    )
    public ResponseEntity<ApiResponse<Void>> verifyEmail(
            @RequestParam String token) {
        
        log.info("Verifica email con token");
        
        authenticationService.verifyEmail(token);
        return success(null, "Email verificata con successo");
    }
    
    @PostMapping("/forgot-password")
    @Operation(
        summary = "Password dimenticata",
        description = "Invia email per reset password"
    )
    public ResponseEntity<ApiResponse<Void>> forgotPassword(
            @RequestParam String email) {
        
        log.info("Richiesta reset password per email: {}", email);
        
        authenticationService.forgotPassword(email);
        return success(null, "Email di reset inviata");
    }
    
    @PostMapping("/reset-password")
    @Operation(
        summary = "Reset password",
        description = "Reimposta la password con il token ricevuto via email"
    )
    public ResponseEntity<ApiResponse<Void>> resetPassword(
            @RequestParam String token,
            @RequestParam String newPassword) {
        
        log.info("Reset password con token");
        
        authenticationService.resetPassword(token, newPassword);
        return success(null, "Password reimpostata con successo");
    }
}
```

## 6.2 Configurazione Sicurezza

> **Pattern implementato**: Implementiamo l'autenticazione JWT e l'autorizzazione basata sui ruoli. Vedi [Sicurezza Web](../knowledge/04-sicurezza-web.md) per i dettagli teorici.

### 6.2.1 Security Configuration

> **Configurazione centrale**: Questa classe configura tutta la sicurezza dell'applicazione: autenticazione, autorizzazione, CORS e gestione delle sessioni.

**SecurityConfig.java**:
```java
package mc.videogiochi.config;

import lombok.RequiredArgsConstructor;
import mc.videogiochi.security.*;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpMethod;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.AuthenticationProvider;
import org.springframework.security.authentication.dao.DaoAuthenticationProvider;
import org.springframework.security.config.annotation.authentication.configuration.AuthenticationConfiguration;
import org.springframework.security.config.annotation.method.configuration.EnableMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configurers.AbstractHttpConfigurer;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.CorsConfigurationSource;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;

import java.util.Arrays;
import java.util.List;

@Configuration
@EnableWebSecurity
@EnableMethodSecurity(prePostEnabled = true)
@RequiredArgsConstructor
public class SecurityConfig {
    
    private final UserDetailsService userDetailsService;
    private final JwtAuthenticationEntryPoint jwtAuthenticationEntryPoint;
    private final JwtAccessDeniedHandler jwtAccessDeniedHandler;
    private final JwtAuthenticationFilter jwtAuthenticationFilter;
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(12);
    }
    
    @Bean
    public AuthenticationProvider authenticationProvider() {
        DaoAuthenticationProvider authProvider = new DaoAuthenticationProvider();
        authProvider.setUserDetailsService(userDetailsService);
        authProvider.setPasswordEncoder(passwordEncoder());
        return authProvider;
    }
    
    @Bean
    public AuthenticationManager authenticationManager(
            AuthenticationConfiguration config) throws Exception {
        return config.getAuthenticationManager();
    }
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(AbstractHttpConfigurer::disable)
            .cors(cors -> cors.configurationSource(corsConfigurationSource()))
            .sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .exceptionHandling(exceptions -> exceptions
                .authenticationEntryPoint(jwtAuthenticationEntryPoint)
                .accessDeniedHandler(jwtAccessDeniedHandler)
            )
            .authorizeHttpRequests(authz -> authz
                // Endpoint pubblici
                .requestMatchers("/api/v1/auth/**").permitAll()
                .requestMatchers("/api/v1/users/register").permitAll()
                .requestMatchers(HttpMethod.GET, "/api/v1/games/**").permitAll()
                .requestMatchers(HttpMethod.GET, "/api/v1/genres/**").permitAll()
                .requestMatchers(HttpMethod.GET, "/api/v1/platforms/**").permitAll()
                .requestMatchers(HttpMethod.GET, "/api/v1/developers/**").permitAll()
                .requestMatchers(HttpMethod.GET, "/api/v1/publishers/**").permitAll()
                
                // Swagger/OpenAPI
                .requestMatchers("/swagger-ui/**", "/v3/api-docs/**", "/swagger-ui.html").permitAll()
                
                // Actuator endpoints
                .requestMatchers("/actuator/health", "/actuator/info").permitAll()
                .requestMatchers("/actuator/**").hasRole("ADMIN")
                
                // Endpoint che richiedono autenticazione
                .requestMatchers(HttpMethod.POST, "/api/v1/games/**").hasAnyRole("ADMIN", "MODERATOR")
                .requestMatchers(HttpMethod.PUT, "/api/v1/games/**").hasAnyRole("ADMIN", "MODERATOR")
                .requestMatchers(HttpMethod.DELETE, "/api/v1/games/**").hasRole("ADMIN")
                
                .requestMatchers("/api/v1/users/profile").authenticated()
                .requestMatchers("/api/v1/users/change-password").authenticated()
                .requestMatchers("/api/v1/lists/**").authenticated()
                
                // Admin endpoints
                .requestMatchers("/api/v1/admin/**").hasRole("ADMIN")
                .requestMatchers(HttpMethod.GET, "/api/v1/users").hasRole("ADMIN")
                .requestMatchers(HttpMethod.PUT, "/api/v1/users/{id}").hasRole("ADMIN")
                .requestMatchers(HttpMethod.DELETE, "/api/v1/users/{id}").hasRole("ADMIN")
                
                // Tutto il resto richiede autenticazione
                .anyRequest().authenticated()
            )
            .authenticationProvider(authenticationProvider())
            .addFilterBefore(jwtAuthenticationFilter, UsernamePasswordAuthenticationFilter.class);
        
        return http.build();
    }
    
    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration configuration = new CorsConfiguration();
        configuration.setAllowedOriginPatterns(List.of("*"));
        configuration.setAllowedMethods(Arrays.asList("GET", "POST", "PUT", "DELETE", "OPTIONS"));
        configuration.setAllowedHeaders(Arrays.asList("*"));
        configuration.setAllowCredentials(true);
        configuration.setMaxAge(3600L);
        
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/api/**", configuration);
        return source;
    }
}
```

### 6.2.2 JWT Components

> **Concetto chiave**: JWT (JSON Web Token) permette autenticazione stateless, ideale per API REST. Vedi [Sicurezza Web](../knowledge/04-sicurezza-web.md) per approfondimenti sui token JWT.

**JwtTokenProvider.java**:
```java
package mc.videogiochi.security;

import io.jsonwebtoken.*;
import io.jsonwebtoken.security.Keys;
import lombok.extern.slf4j.Slf4j;
import mc.videogiochi.exception.SecurityException;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.stereotype.Component;

import javax.crypto.SecretKey;
import java.util.Date;

@Component
@Slf4j
public class JwtTokenProvider {
    
    private final SecretKey jwtSecret;
    private final long jwtExpirationMs;
    
    public JwtTokenProvider(
            @Value("${app.security.jwt.secret}") String jwtSecret,
            @Value("${app.security.jwt.expiration}") long jwtExpirationMs) {
        this.jwtSecret = Keys.hmacShaKeyFor(jwtSecret.getBytes());
        this.jwtExpirationMs = jwtExpirationMs;
    }
    
    public String generateToken(Authentication authentication) {
        UserDetails userPrincipal = (UserDetails) authentication.getPrincipal();
        Date expiryDate = new Date(System.currentTimeMillis() + jwtExpirationMs);
        
        return Jwts.builder()
                .setSubject(userPrincipal.getUsername())
                .setIssuedAt(new Date())
                .setExpiration(expiryDate)
                .signWith(jwtSecret, SignatureAlgorithm.HS512)
                .compact();
    }
    
    public String generateTokenFromUsername(String username) {
        Date expiryDate = new Date(System.currentTimeMillis() + jwtExpirationMs);
        
        return Jwts.builder()
                .setSubject(username)
                .setIssuedAt(new Date())
                .setExpiration(expiryDate)
                .signWith(jwtSecret, SignatureAlgorithm.HS512)
                .compact();
    }
    
    public String getUsernameFromToken(String token) {
        Claims claims = Jwts.parserBuilder()
                .setSigningKey(jwtSecret)
                .build()
                .parseClaimsJws(token)
                .getBody();
        
        return claims.getSubject();
    }
    
    public Date getExpirationDateFromToken(String token) {
        Claims claims = Jwts.parserBuilder()
                .setSigningKey(jwtSecret)
                .build()
                .parseClaimsJws(token)
                .getBody();
        
        return claims.getExpiration();
    }
    
    public boolean validateToken(String token) {
        try {
            Jwts.parserBuilder()
                .setSigningKey(jwtSecret)
                .build()
                .parseClaimsJws(token);
            return true;
        } catch (MalformedJwtException e) {
            log.error("Token JWT malformato: {}", e.getMessage());
        } catch (ExpiredJwtException e) {
            log.error("Token JWT scaduto: {}", e.getMessage());
        } catch (UnsupportedJwtException e) {
            log.error("Token JWT non supportato: {}", e.getMessage());
        } catch (IllegalArgumentException e) {
            log.error("JWT claims string è vuoto: {}", e.getMessage());
        } catch (Exception e) {
            log.error("Errore validazione token JWT: {}", e.getMessage());
        }
        return false;
    }
    
    public boolean isTokenExpired(String token) {
        Date expiration = getExpirationDateFromToken(token);
        return expiration.before(new Date());
    }
    
    public long getExpirationMs() {
        return jwtExpirationMs;
    }
}
```

**JwtAuthenticationFilter.java**:
```java
package mc.videogiochi.security;

import jakarta.servlet.FilterChain;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.web.authentication.WebAuthenticationDetailsSource;
import org.springframework.stereotype.Component;
import org.springframework.util.StringUtils;
import org.springframework.web.filter.OncePerRequestFilter;

import java.io.IOException;

@Component
@RequiredArgsConstructor
@Slf4j
public class JwtAuthenticationFilter extends OncePerRequestFilter {
    
    private final JwtTokenProvider jwtTokenProvider;
    private final UserDetailsService userDetailsService;
    
    @Override
    protected void doFilterInternal(
            HttpServletRequest request,
            HttpServletResponse response,
            FilterChain filterChain) throws ServletException, IOException {
        
        try {
            String jwt = getJwtFromRequest(request);
            
            if (StringUtils.hasText(jwt) && jwtTokenProvider.validateToken(jwt)) {
                String username = jwtTokenProvider.getUsernameFromToken(jwt);
                
                UserDetails userDetails = userDetailsService.loadUserByUsername(username);
                
                UsernamePasswordAuthenticationToken authentication = 
                    new UsernamePasswordAuthenticationToken(
                        userDetails, null, userDetails.getAuthorities());
                
                authentication.setDetails(
                    new WebAuthenticationDetailsSource().buildDetails(request));
                
                SecurityContextHolder.getContext().setAuthentication(authentication);
                
                log.debug("Utente autenticato: {}", username);
            }
        } catch (Exception e) {
            log.error("Errore durante l'autenticazione JWT: {}", e.getMessage());
        }
        
        filterChain.doFilter(request, response);
    }
    
    private String getJwtFromRequest(HttpServletRequest request) {
        String bearerToken = request.getHeader("Authorization");
        if (StringUtils.hasText(bearerToken) && bearerToken.startsWith("Bearer ")) {
            return bearerToken.substring(7);
        }
        return null;
    }
}
```

### 6.2.3 Exception Handlers

**JwtAuthenticationEntryPoint.java**:
```java
package mc.videogiochi.security;

import com.fasterxml.jackson.databind.ObjectMapper;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import lombok.extern.slf4j.Slf4j;
import mc.videogiochi.dto.response.ApiResponse;
import org.springframework.http.MediaType;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.web.AuthenticationEntryPoint;
import org.springframework.stereotype.Component;

import java.io.IOException;

@Component
@Slf4j
public class JwtAuthenticationEntryPoint implements AuthenticationEntryPoint {
    
    private final ObjectMapper objectMapper = new ObjectMapper();
    
    @Override
    public void commence(
            HttpServletRequest request,
            HttpServletResponse response,
            AuthenticationException authException) throws IOException {
        
        log.error("Errore di autenticazione: {}", authException.getMessage());
        
        response.setContentType(MediaType.APPLICATION_JSON_VALUE);
        response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
        
        ApiResponse<Void> apiResponse = ApiResponse.error(
            "Accesso non autorizzato. Token JWT mancante o non valido.");
        apiResponse.setPath(request.getRequestURI());
        
        objectMapper.writeValue(response.getOutputStream(), apiResponse);
    }
}
```

**JwtAccessDeniedHandler.java**:
```java
package mc.videogiochi.security;

import com.fasterxml.jackson.databind.ObjectMapper;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import lombok.extern.slf4j.Slf4j;
import mc.videogiochi.dto.response.ApiResponse;
import org.springframework.http.MediaType;
import org.springframework.security.access.AccessDeniedException;
import org.springframework.security.web.access.AccessDeniedHandler;
import org.springframework.stereotype.Component;

import java.io.IOException;

@Component
@Slf4j
public class JwtAccessDeniedHandler implements AccessDeniedHandler {
    
    private final ObjectMapper objectMapper = new ObjectMapper();
    
    @Override
    public void handle(
            HttpServletRequest request,
            HttpServletResponse response,
            AccessDeniedException accessDeniedException) throws IOException {
        
        log.error("Accesso negato: {}", accessDeniedException.getMessage());
        
        response.setContentType(MediaType.APPLICATION_JSON_VALUE);
        response.setStatus(HttpServletResponse.SC_FORBIDDEN);
        
        ApiResponse<Void> apiResponse = ApiResponse.error(
            "Accesso negato. Privilegi insufficienti per questa operazione.");
        apiResponse.setPath(request.getRequestURI());
        
        objectMapper.writeValue(response.getOutputStream(), apiResponse);
    }
}
```

Questo documento continua con l'implementazione completa della sicurezza, includendo i servizi di autenticazione, la gestione delle sessioni, e tutti gli altri componenti necessari per un sistema di sicurezza robusto e scalabile.