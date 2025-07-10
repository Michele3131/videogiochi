# 5. Sicurezza e Autenticazione

## 5.1 Concetti Fondamentali di Sicurezza

### 5.1.1 Autenticazione vs Autorizzazione

**Autenticazione** (Authentication): Processo di verifica dell'identità di un utente.
- "Chi sei?"
- Verifica credenziali (username/password, token, certificati)
- Stabilisce l'identità dell'utente

**Autorizzazione** (Authorization): Processo di verifica dei permessi di un utente autenticato.
- "Cosa puoi fare?"
- Controllo accessi basato su ruoli/permessi
- Determina le azioni consentite

### 5.1.2 Principi di Sicurezza

#### CIA Triad
1. **Confidentiality (Riservatezza)**: I dati devono essere accessibili solo agli utenti autorizzati
2. **Integrity (Integrità)**: I dati devono rimanere accurati e non modificati da utenti non autorizzati
3. **Availability (Disponibilità)**: I sistemi devono essere accessibili quando necessario

#### Defense in Depth
```java
// Esempio di sicurezza a livelli multipli
@RestController
@RequestMapping("/api/v1/admin")
@PreAuthorize("hasRole('ADMIN')") // Livello 1: Autorizzazione a livello controller
public class AdminController {
    
    @PostMapping("/users/{userId}/deactivate")
    @PreAuthorize("hasAuthority('USER_MANAGEMENT')") // Livello 2: Autorizzazione specifica
    public ResponseEntity<ApiResponse> deactivateUser(
            @PathVariable Long userId,
            @Valid @RequestBody DeactivationRequest request,
            Authentication authentication) {
        
        // Livello 3: Validazione business logic
        if (!userService.canDeactivateUser(userId, getCurrentUserId(authentication))) {
            throw new SecurityException("Non autorizzato a disattivare questo utente");
        }
        
        // Livello 4: Audit logging
        auditService.logUserDeactivation(userId, getCurrentUserId(authentication));
        
        userService.deactivateUser(userId, request.getReason());
        return ResponseEntity.ok(ApiResponse.success("Utente disattivato con successo"));
    }
}
```

## 5.2 Spring Security

### 5.2.1 Architettura di Spring Security

#### Security Filter Chain
```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity(prePostEnabled = true)
public class SecurityConfig {
    
    private final JwtAuthenticationEntryPoint jwtAuthenticationEntryPoint;
    private final JwtAccessDeniedHandler jwtAccessDeniedHandler;
    private final JwtTokenProvider jwtTokenProvider;
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(12); // Strength 12 per sicurezza elevata
    }
    
    @Bean
    public AuthenticationManager authenticationManager(
            AuthenticationConfiguration config) throws Exception {
        return config.getAuthenticationManager();
    }
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            // Disabilita CSRF per API REST
            .csrf(csrf -> csrf.disable())
            
            // Configurazione CORS
            .cors(cors -> cors.configurationSource(corsConfigurationSource()))
            
            // Gestione sessioni stateless
            .sessionManagement(session -> 
                session.sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            )
            
            // Gestione eccezioni
            .exceptionHandling(exceptions -> exceptions
                .authenticationEntryPoint(jwtAuthenticationEntryPoint)
                .accessDeniedHandler(jwtAccessDeniedHandler)
            )
            
            // Configurazione autorizzazioni
            .authorizeHttpRequests(authz -> authz
                // Endpoint pubblici
                .requestMatchers("/api/v1/auth/**").permitAll()
                .requestMatchers("/api/v1/games/search").permitAll()
                .requestMatchers("/api/v1/games/{id}").permitAll()
                .requestMatchers(HttpMethod.GET, "/api/v1/games").permitAll()
                
                // Endpoint per utenti autenticati
                .requestMatchers("/api/v1/users/profile").hasRole("USER")
                .requestMatchers("/api/v1/users/lists/**").hasRole("USER")
                .requestMatchers("/api/v1/games/*/rate").hasRole("USER")
                
                // Endpoint amministrativi
                .requestMatchers("/api/v1/admin/**").hasRole("ADMIN")
                .requestMatchers(HttpMethod.POST, "/api/v1/games").hasRole("ADMIN")
                .requestMatchers(HttpMethod.PUT, "/api/v1/games/**").hasRole("ADMIN")
                .requestMatchers(HttpMethod.DELETE, "/api/v1/games/**").hasRole("ADMIN")
                
                // Endpoint per moderatori
                .requestMatchers("/api/v1/moderation/**").hasAnyRole("MODERATOR", "ADMIN")
                
                // Tutti gli altri endpoint richiedono autenticazione
                .anyRequest().authenticated()
            )
            
            // Aggiunta filtro JWT
            .addFilterBefore(
                new JwtAuthenticationFilter(jwtTokenProvider),
                UsernamePasswordAuthenticationFilter.class
            );
        
        return http.build();
    }
    
    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration configuration = new CorsConfiguration();
        
        // Origini consentite (configurabile via properties)
        configuration.setAllowedOriginPatterns(Arrays.asList(
            "http://localhost:3000", // Frontend development
            "http://localhost:5173", // Vite dev server
            "https://videogames-app.com" // Production
        ));
        
        configuration.setAllowedMethods(Arrays.asList(
            "GET", "POST", "PUT", "DELETE", "PATCH", "OPTIONS"
        ));
        
        configuration.setAllowedHeaders(Arrays.asList(
            "Authorization", "Content-Type", "X-Requested-With",
            "Accept", "Origin", "Access-Control-Request-Method",
            "Access-Control-Request-Headers"
        ));
        
        configuration.setExposedHeaders(Arrays.asList(
            "Access-Control-Allow-Origin", "Access-Control-Allow-Credentials"
        ));
        
        configuration.setAllowCredentials(true);
        configuration.setMaxAge(3600L); // Cache preflight per 1 ora
        
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", configuration);
        
        return source;
    }
}
```

### 5.2.2 Custom UserDetailsService

#### Implementazione UserDetails
```java
@Data
@Builder
public class UserPrincipal implements UserDetails {
    
    private Long id;
    private String username;
    private String email;
    private String password;
    private Set<UserRole> roles;
    private boolean enabled;
    private boolean accountNonExpired;
    private boolean accountNonLocked;
    private boolean credentialsNonExpired;
    private LocalDateTime lastLoginAt;
    
    // Conversione da Entity
    public static UserPrincipal from(User user) {
        return UserPrincipal.builder()
            .id(user.getId())
            .username(user.getUsername())
            .email(user.getEmail())
            .password(user.getPassword())
            .roles(user.getRoles())
            .enabled(user.isActive())
            .accountNonExpired(!user.isExpired())
            .accountNonLocked(!user.isLocked())
            .credentialsNonExpired(!user.isPasswordExpired())
            .lastLoginAt(user.getLastLoginAt())
            .build();
    }
    
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        Set<GrantedAuthority> authorities = new HashSet<>();
        
        // Aggiunta ruoli
        for (UserRole role : roles) {
            authorities.add(new SimpleGrantedAuthority("ROLE_" + role.name()));
            
            // Aggiunta permessi specifici basati sui ruoli
            authorities.addAll(getPermissionsForRole(role));
        }
        
        return authorities;
    }
    
    private Set<GrantedAuthority> getPermissionsForRole(UserRole role) {
        Set<GrantedAuthority> permissions = new HashSet<>();
        
        switch (role) {
            case ADMIN:
                permissions.add(new SimpleGrantedAuthority("USER_MANAGEMENT"));
                permissions.add(new SimpleGrantedAuthority("GAME_MANAGEMENT"));
                permissions.add(new SimpleGrantedAuthority("SYSTEM_CONFIGURATION"));
                permissions.add(new SimpleGrantedAuthority("AUDIT_ACCESS"));
                // Fall through per includere permessi di livello inferiore
                
            case MODERATOR:
                permissions.add(new SimpleGrantedAuthority("CONTENT_MODERATION"));
                permissions.add(new SimpleGrantedAuthority("USER_SUSPENSION"));
                // Fall through
                
            case USER:
                permissions.add(new SimpleGrantedAuthority("PROFILE_MANAGEMENT"));
                permissions.add(new SimpleGrantedAuthority("LIST_MANAGEMENT"));
                permissions.add(new SimpleGrantedAuthority("RATING_SUBMISSION"));
                break;
                
            default:
                // Nessun permesso per ruoli non riconosciuti
                break;
        }
        
        return permissions;
    }
    
    @Override
    public boolean isAccountNonExpired() {
        return accountNonExpired;
    }
    
    @Override
    public boolean isAccountNonLocked() {
        return accountNonLocked;
    }
    
    @Override
    public boolean isCredentialsNonExpired() {
        return credentialsNonExpired;
    }
    
    @Override
    public boolean isEnabled() {
        return enabled;
    }
}
```

#### UserDetailsService Implementation
```java
@Service
@Transactional(readOnly = true)
public class CustomUserDetailsService implements UserDetailsService {
    
    private final UserRepository userRepository;
    private final LoginAttemptService loginAttemptService;
    
    public CustomUserDetailsService(
            UserRepository userRepository,
            LoginAttemptService loginAttemptService) {
        this.userRepository = userRepository;
        this.loginAttemptService = loginAttemptService;
    }
    
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        // Controllo tentativi di login
        if (loginAttemptService.isBlocked(username)) {
            throw new AccountLockedException(
                "Account temporaneamente bloccato per troppi tentativi di login falliti"
            );
        }
        
        // Ricerca utente per username o email
        User user = userRepository.findByUsernameOrEmail(username, username)
            .orElseThrow(() -> new UsernameNotFoundException(
                "Utente non trovato: " + username
            ));
        
        // Controlli aggiuntivi
        validateUserAccount(user);
        
        return UserPrincipal.from(user);
    }
    
    private void validateUserAccount(User user) {
        if (!user.isActive()) {
            throw new DisabledException("Account disattivato");
        }
        
        if (user.isLocked()) {
            throw new AccountLockedException("Account bloccato");
        }
        
        if (user.isExpired()) {
            throw new AccountExpiredException("Account scaduto");
        }
        
        if (user.isPasswordExpired()) {
            throw new CredentialsExpiredException("Password scaduta");
        }
        
        if (!user.isEmailVerified()) {
            throw new EmailNotVerifiedException("Email non verificata");
        }
    }
}
```

## 5.3 JWT (JSON Web Tokens)

### 5.3.1 Struttura e Implementazione JWT

#### JWT Token Provider
```java
@Component
public class JwtTokenProvider {
    
    private static final Logger logger = LoggerFactory.getLogger(JwtTokenProvider.class);
    
    @Value("${app.jwt.secret}")
    private String jwtSecret;
    
    @Value("${app.jwt.access-token-expiration:900000}") // 15 minuti
    private long accessTokenExpiration;
    
    @Value("${app.jwt.refresh-token-expiration:604800000}") // 7 giorni
    private long refreshTokenExpiration;
    
    @Value("${app.jwt.issuer:videogames-app}")
    private String issuer;
    
    private Key getSigningKey() {
        byte[] keyBytes = Decoders.BASE64.decode(jwtSecret);
        return Keys.hmacShaKeyFor(keyBytes);
    }
    
    public String generateAccessToken(UserPrincipal userPrincipal) {
        Date now = new Date();
        Date expiryDate = new Date(now.getTime() + accessTokenExpiration);
        
        return Jwts.builder()
            .setSubject(userPrincipal.getId().toString())
            .setIssuer(issuer)
            .setIssuedAt(now)
            .setExpiration(expiryDate)
            .claim("username", userPrincipal.getUsername())
            .claim("email", userPrincipal.getEmail())
            .claim("roles", userPrincipal.getRoles().stream()
                .map(Enum::name)
                .collect(Collectors.toList()))
            .claim("type", "access")
            .signWith(getSigningKey(), SignatureAlgorithm.HS512)
            .compact();
    }
    
    public String generateRefreshToken(UserPrincipal userPrincipal) {
        Date now = new Date();
        Date expiryDate = new Date(now.getTime() + refreshTokenExpiration);
        
        return Jwts.builder()
            .setSubject(userPrincipal.getId().toString())
            .setIssuer(issuer)
            .setIssuedAt(now)
            .setExpiration(expiryDate)
            .claim("username", userPrincipal.getUsername())
            .claim("type", "refresh")
            .signWith(getSigningKey(), SignatureAlgorithm.HS512)
            .compact();
    }
    
    public String generateEmailVerificationToken(Long userId, String email) {
        Date now = new Date();
        Date expiryDate = new Date(now.getTime() + 86400000); // 24 ore
        
        return Jwts.builder()
            .setSubject(userId.toString())
            .setIssuer(issuer)
            .setIssuedAt(now)
            .setExpiration(expiryDate)
            .claim("email", email)
            .claim("type", "email_verification")
            .signWith(getSigningKey(), SignatureAlgorithm.HS512)
            .compact();
    }
    
    public String generatePasswordResetToken(Long userId, String email) {
        Date now = new Date();
        Date expiryDate = new Date(now.getTime() + 3600000); // 1 ora
        
        return Jwts.builder()
            .setSubject(userId.toString())
            .setIssuer(issuer)
            .setIssuedAt(now)
            .setExpiration(expiryDate)
            .claim("email", email)
            .claim("type", "password_reset")
            .signWith(getSigningKey(), SignatureAlgorithm.HS512)
            .compact();
    }
    
    public Long getUserIdFromToken(String token) {
        Claims claims = Jwts.parserBuilder()
            .setSigningKey(getSigningKey())
            .build()
            .parseClaimsJws(token)
            .getBody();
        
        return Long.parseLong(claims.getSubject());
    }
    
    public String getUsernameFromToken(String token) {
        Claims claims = Jwts.parserBuilder()
            .setSigningKey(getSigningKey())
            .build()
            .parseClaimsJws(token)
            .getBody();
        
        return claims.get("username", String.class);
    }
    
    public String getTokenType(String token) {
        Claims claims = Jwts.parserBuilder()
            .setSigningKey(getSigningKey())
            .build()
            .parseClaimsJws(token)
            .getBody();
        
        return claims.get("type", String.class);
    }
    
    public boolean validateToken(String token) {
        try {
            Jwts.parserBuilder()
                .setSigningKey(getSigningKey())
                .build()
                .parseClaimsJws(token);
            return true;
        } catch (SecurityException ex) {
            logger.error("Invalid JWT signature: {}", ex.getMessage());
        } catch (MalformedJwtException ex) {
            logger.error("Invalid JWT token: {}", ex.getMessage());
        } catch (ExpiredJwtException ex) {
            logger.error("Expired JWT token: {}", ex.getMessage());
        } catch (UnsupportedJwtException ex) {
            logger.error("Unsupported JWT token: {}", ex.getMessage());
        } catch (IllegalArgumentException ex) {
            logger.error("JWT claims string is empty: {}", ex.getMessage());
        }
        return false;
    }
    
    public boolean isTokenExpired(String token) {
        try {
            Claims claims = Jwts.parserBuilder()
                .setSigningKey(getSigningKey())
                .build()
                .parseClaimsJws(token)
                .getBody();
            
            return claims.getExpiration().before(new Date());
        } catch (Exception e) {
            return true;
        }
    }
    
    public Date getExpirationDateFromToken(String token) {
        Claims claims = Jwts.parserBuilder()
            .setSigningKey(getSigningKey())
            .build()
            .parseClaimsJws(token)
            .getBody();
        
        return claims.getExpiration();
    }
}
```

### 5.3.2 JWT Authentication Filter

```java
public class JwtAuthenticationFilter extends OncePerRequestFilter {
    
    private static final Logger logger = LoggerFactory.getLogger(JwtAuthenticationFilter.class);
    private static final String AUTHORIZATION_HEADER = "Authorization";
    private static final String BEARER_PREFIX = "Bearer ";
    
    private final JwtTokenProvider jwtTokenProvider;
    private final CustomUserDetailsService userDetailsService;
    
    public JwtAuthenticationFilter(
            JwtTokenProvider jwtTokenProvider,
            CustomUserDetailsService userDetailsService) {
        this.jwtTokenProvider = jwtTokenProvider;
        this.userDetailsService = userDetailsService;
    }
    
    @Override
    protected void doFilterInternal(
            HttpServletRequest request,
            HttpServletResponse response,
            FilterChain filterChain) throws ServletException, IOException {
        
        try {
            String token = extractTokenFromRequest(request);
            
            if (token != null && jwtTokenProvider.validateToken(token)) {
                // Verifica che sia un access token
                String tokenType = jwtTokenProvider.getTokenType(token);
                if (!"access".equals(tokenType)) {
                    logger.warn("Invalid token type: {}", tokenType);
                    filterChain.doFilter(request, response);
                    return;
                }
                
                String username = jwtTokenProvider.getUsernameFromToken(token);
                
                if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {
                    UserDetails userDetails = userDetailsService.loadUserByUsername(username);
                    
                    if (userDetails != null) {
                        UsernamePasswordAuthenticationToken authentication =
                            new UsernamePasswordAuthenticationToken(
                                userDetails, null, userDetails.getAuthorities()
                            );
                        
                        authentication.setDetails(
                            new WebAuthenticationDetailsSource().buildDetails(request)
                        );
                        
                        SecurityContextHolder.getContext().setAuthentication(authentication);
                        
                        // Log successful authentication
                        logger.debug("Successfully authenticated user: {}", username);
                    }
                }
            }
        } catch (Exception ex) {
            logger.error("Could not set user authentication in security context", ex);
            // Non interrompere la catena di filtri, lascia che altri gestiscano l'errore
        }
        
        filterChain.doFilter(request, response);
    }
    
    private String extractTokenFromRequest(HttpServletRequest request) {
        String bearerToken = request.getHeader(AUTHORIZATION_HEADER);
        
        if (StringUtils.hasText(bearerToken) && bearerToken.startsWith(BEARER_PREFIX)) {
            return bearerToken.substring(BEARER_PREFIX.length());
        }
        
        return null;
    }
    
    @Override
    protected boolean shouldNotFilter(HttpServletRequest request) throws ServletException {
        String path = request.getRequestURI();
        
        // Skip JWT filter per endpoint pubblici
        return path.startsWith("/api/v1/auth/") ||
               path.equals("/api/v1/games") ||
               path.startsWith("/api/v1/games/search") ||
               (path.matches("/api/v1/games/\\d+") && "GET".equals(request.getMethod()));
    }
}
```

## 5.4 Gestione Sessioni e Refresh Tokens

### 5.4.1 Refresh Token Management

#### Entity per Refresh Token
```java
@Entity
@Table(name = "refresh_tokens")
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class RefreshToken {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, unique = true)
    private String token;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id", nullable = false)
    private User user;
    
    @Column(name = "expires_at", nullable = false)
    private LocalDateTime expiresAt;
    
    @Column(name = "created_at", nullable = false)
    private LocalDateTime createdAt;
    
    @Column(name = "last_used_at")
    private LocalDateTime lastUsedAt;
    
    @Column(name = "revoked")
    private boolean revoked = false;
    
    @Column(name = "device_info")
    private String deviceInfo;
    
    @Column(name = "ip_address")
    private String ipAddress;
    
    @PrePersist
    protected void onCreate() {
        createdAt = LocalDateTime.now();
    }
    
    public boolean isExpired() {
        return LocalDateTime.now().isAfter(expiresAt);
    }
    
    public boolean isValid() {
        return !revoked && !isExpired();
    }
}
```

#### Refresh Token Service
```java
@Service
@Transactional
public class RefreshTokenService {
    
    private final RefreshTokenRepository refreshTokenRepository;
    private final JwtTokenProvider jwtTokenProvider;
    private final UserRepository userRepository;
    
    @Value("${app.jwt.refresh-token-expiration:604800000}") // 7 giorni
    private long refreshTokenExpiration;
    
    public RefreshToken createRefreshToken(
            Long userId, String deviceInfo, String ipAddress) {
        
        User user = userRepository.findById(userId)
            .orElseThrow(() -> new ResourceNotFoundException("Utente non trovato"));
        
        // Revoca eventuali refresh token esistenti per questo dispositivo
        revokeRefreshTokensForUserAndDevice(userId, deviceInfo);
        
        // Genera nuovo refresh token
        UserPrincipal userPrincipal = UserPrincipal.from(user);
        String tokenValue = jwtTokenProvider.generateRefreshToken(userPrincipal);
        
        RefreshToken refreshToken = RefreshToken.builder()
            .token(tokenValue)
            .user(user)
            .expiresAt(LocalDateTime.now().plusSeconds(refreshTokenExpiration / 1000))
            .deviceInfo(deviceInfo)
            .ipAddress(ipAddress)
            .build();
        
        return refreshTokenRepository.save(refreshToken);
    }
    
    public Optional<RefreshToken> findByToken(String token) {
        return refreshTokenRepository.findByToken(token);
    }
    
    public RefreshToken verifyExpiration(RefreshToken token) {
        if (token.isExpired()) {
            refreshTokenRepository.delete(token);
            throw new TokenRefreshException(
                "Refresh token scaduto. Effettua nuovamente il login"
            );
        }
        
        if (token.isRevoked()) {
            throw new TokenRefreshException(
                "Refresh token revocato. Effettua nuovamente il login"
            );
        }
        
        // Aggiorna last used
        token.setLastUsedAt(LocalDateTime.now());
        return refreshTokenRepository.save(token);
    }
    
    public void revokeRefreshToken(String token) {
        refreshTokenRepository.findByToken(token)
            .ifPresent(refreshToken -> {
                refreshToken.setRevoked(true);
                refreshTokenRepository.save(refreshToken);
            });
    }
    
    public void revokeAllRefreshTokensForUser(Long userId) {
        List<RefreshToken> userTokens = refreshTokenRepository.findByUserIdAndRevokedFalse(userId);
        userTokens.forEach(token -> token.setRevoked(true));
        refreshTokenRepository.saveAll(userTokens);
    }
    
    public void revokeRefreshTokensForUserAndDevice(Long userId, String deviceInfo) {
        List<RefreshToken> deviceTokens = refreshTokenRepository
            .findByUserIdAndDeviceInfoAndRevokedFalse(userId, deviceInfo);
        deviceTokens.forEach(token -> token.setRevoked(true));
        refreshTokenRepository.saveAll(deviceTokens);
    }
    
    @Scheduled(fixedRate = 3600000) // Ogni ora
    public void cleanupExpiredTokens() {
        LocalDateTime now = LocalDateTime.now();
        int deletedCount = refreshTokenRepository.deleteByExpiresAtBefore(now);
        
        if (deletedCount > 0) {
            logger.info("Cleaned up {} expired refresh tokens", deletedCount);
        }
    }
    
    public List<RefreshTokenInfo> getUserActiveSessions(Long userId) {
        List<RefreshToken> activeTokens = refreshTokenRepository
            .findByUserIdAndRevokedFalseAndExpiresAtAfter(userId, LocalDateTime.now());
        
        return activeTokens.stream()
            .map(this::mapToRefreshTokenInfo)
            .collect(Collectors.toList());
    }
    
    private RefreshTokenInfo mapToRefreshTokenInfo(RefreshToken token) {
        return RefreshTokenInfo.builder()
            .id(token.getId())
            .deviceInfo(token.getDeviceInfo())
            .ipAddress(token.getIpAddress())
            .createdAt(token.getCreatedAt())
            .lastUsedAt(token.getLastUsedAt())
            .expiresAt(token.getExpiresAt())
            .build();
    }
}
```

### 5.4.2 Authentication Service

```java
@Service
@Transactional
public class AuthService {
    
    private final AuthenticationManager authenticationManager;
    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;
    private final JwtTokenProvider jwtTokenProvider;
    private final RefreshTokenService refreshTokenService;
    private final EmailService emailService;
    private final LoginAttemptService loginAttemptService;
    
    public LoginResponse login(LoginRequest request, HttpServletRequest httpRequest) {
        String username = request.getUsername();
        
        // Controllo tentativi di login
        if (loginAttemptService.isBlocked(username)) {
            throw new AccountLockedException(
                "Account temporaneamente bloccato per troppi tentativi falliti"
            );
        }
        
        try {
            // Autenticazione
            Authentication authentication = authenticationManager.authenticate(
                new UsernamePasswordAuthenticationToken(
                    username, request.getPassword()
                )
            );
            
            UserPrincipal userPrincipal = (UserPrincipal) authentication.getPrincipal();
            
            // Genera tokens
            String accessToken = jwtTokenProvider.generateAccessToken(userPrincipal);
            
            String deviceInfo = extractDeviceInfo(httpRequest);
            String ipAddress = getClientIpAddress(httpRequest);
            
            RefreshToken refreshToken = refreshTokenService.createRefreshToken(
                userPrincipal.getId(), deviceInfo, ipAddress
            );
            
            // Aggiorna ultimo login
            updateLastLogin(userPrincipal.getId(), ipAddress);
            
            // Reset tentativi di login
            loginAttemptService.loginSucceeded(username);
            
            return LoginResponse.builder()
                .accessToken(accessToken)
                .refreshToken(refreshToken.getToken())
                .tokenType("Bearer")
                .expiresIn(jwtTokenProvider.getAccessTokenExpiration() / 1000)
                .user(UserDTO.from(userPrincipal))
                .build();
                
        } catch (BadCredentialsException ex) {
            loginAttemptService.loginFailed(username);
            throw new BadCredentialsException("Credenziali non valide");
        }
    }
    
    public LoginResponse refreshToken(RefreshTokenRequest request) {
        String requestRefreshToken = request.getRefreshToken();
        
        return refreshTokenService.findByToken(requestRefreshToken)
            .map(refreshTokenService::verifyExpiration)
            .map(RefreshToken::getUser)
            .map(user -> {
                UserPrincipal userPrincipal = UserPrincipal.from(user);
                String newAccessToken = jwtTokenProvider.generateAccessToken(userPrincipal);
                
                return LoginResponse.builder()
                    .accessToken(newAccessToken)
                    .refreshToken(requestRefreshToken)
                    .tokenType("Bearer")
                    .expiresIn(jwtTokenProvider.getAccessTokenExpiration() / 1000)
                    .user(UserDTO.from(userPrincipal))
                    .build();
            })
            .orElseThrow(() -> new TokenRefreshException(
                "Refresh token non valido"
            ));
    }
    
    public void logout(String refreshToken) {
        if (StringUtils.hasText(refreshToken)) {
            refreshTokenService.revokeRefreshToken(refreshToken);
        }
    }
    
    public void logoutAllDevices(Long userId) {
        refreshTokenService.revokeAllRefreshTokensForUser(userId);
    }
    
    public UserDTO register(UserRegistrationRequest request) {
        // Validazione
        validateRegistrationRequest(request);
        
        // Creazione utente
        User user = User.builder()
            .username(request.getUsername())
            .email(request.getEmail())
            .password(passwordEncoder.encode(request.getPassword()))
            .firstName(request.getFirstName())
            .lastName(request.getLastName())
            .roles(Set.of(UserRole.USER))
            .active(true)
            .emailVerified(false)
            .build();
        
        User savedUser = userRepository.save(user);
        
        // Invio email di verifica
        sendEmailVerification(savedUser);
        
        return UserDTO.from(savedUser);
    }
    
    public void sendEmailVerification(User user) {
        String token = jwtTokenProvider.generateEmailVerificationToken(
            user.getId(), user.getEmail()
        );
        
        emailService.sendEmailVerification(user.getEmail(), user.getFirstName(), token);
    }
    
    public void verifyEmail(String token) {
        if (!jwtTokenProvider.validateToken(token)) {
            throw new InvalidTokenException("Token di verifica non valido o scaduto");
        }
        
        String tokenType = jwtTokenProvider.getTokenType(token);
        if (!"email_verification".equals(tokenType)) {
            throw new InvalidTokenException("Tipo di token non valido");
        }
        
        Long userId = jwtTokenProvider.getUserIdFromToken(token);
        User user = userRepository.findById(userId)
            .orElseThrow(() -> new ResourceNotFoundException("Utente non trovato"));
        
        if (user.isEmailVerified()) {
            throw new ValidationException("Email già verificata");
        }
        
        user.setEmailVerified(true);
        user.setEmailVerifiedAt(LocalDateTime.now());
        userRepository.save(user);
    }
    
    public void forgotPassword(ForgotPasswordRequest request) {
        User user = userRepository.findByEmail(request.getEmail())
            .orElseThrow(() -> new ResourceNotFoundException(
                "Nessun account associato a questa email"
            ));
        
        String token = jwtTokenProvider.generatePasswordResetToken(
            user.getId(), user.getEmail()
        );
        
        emailService.sendPasswordReset(user.getEmail(), user.getFirstName(), token);
    }
    
    public void resetPassword(ResetPasswordRequest request) {
        String token = request.getToken();
        
        if (!jwtTokenProvider.validateToken(token)) {
            throw new InvalidTokenException("Token di reset non valido o scaduto");
        }
        
        String tokenType = jwtTokenProvider.getTokenType(token);
        if (!"password_reset".equals(tokenType)) {
            throw new InvalidTokenException("Tipo di token non valido");
        }
        
        Long userId = jwtTokenProvider.getUserIdFromToken(token);
        User user = userRepository.findById(userId)
            .orElseThrow(() -> new ResourceNotFoundException("Utente non trovato"));
        
        // Aggiorna password
        user.setPassword(passwordEncoder.encode(request.getNewPassword()));
        user.setPasswordChangedAt(LocalDateTime.now());
        userRepository.save(user);
        
        // Revoca tutti i refresh token
        refreshTokenService.revokeAllRefreshTokensForUser(userId);
    }
    
    private void validateRegistrationRequest(UserRegistrationRequest request) {
        // Controllo username duplicato
        if (userRepository.existsByUsername(request.getUsername())) {
            throw new DuplicateResourceException(
                "Username già in uso: " + request.getUsername()
            );
        }
        
        // Controllo email duplicata
        if (userRepository.existsByEmail(request.getEmail())) {
            throw new DuplicateResourceException(
                "Email già registrata: " + request.getEmail()
            );
        }
    }
    
    private void updateLastLogin(Long userId, String ipAddress) {
        userRepository.updateLastLogin(userId, LocalDateTime.now(), ipAddress);
    }
    
    private String extractDeviceInfo(HttpServletRequest request) {
        String userAgent = request.getHeader("User-Agent");
        // Parsing semplificato del User-Agent
        return userAgent != null ? userAgent.substring(0, Math.min(userAgent.length(), 255)) : "Unknown";
    }
    
    private String getClientIpAddress(HttpServletRequest request) {
        String xForwardedFor = request.getHeader("X-Forwarded-For");
        if (StringUtils.hasText(xForwardedFor)) {
            return xForwardedFor.split(",")[0].trim();
        }
        
        String xRealIp = request.getHeader("X-Real-IP");
        if (StringUtils.hasText(xRealIp)) {
            return xRealIp;
        }
        
        return request.getRemoteAddr();
    }
}
```

## 5.5 Protezione da Attacchi

### 5.5.1 Rate Limiting e Brute Force Protection

#### Login Attempt Service
```java
@Service
public class LoginAttemptService {
    
    private static final int MAX_ATTEMPTS = 5;
    private static final int BLOCK_DURATION_MINUTES = 15;
    
    private final LoadingCache<String, Integer> attemptsCache;
    private final LoadingCache<String, LocalDateTime> blockCache;
    
    public LoginAttemptService() {
        this.attemptsCache = Caffeine.newBuilder()
            .expireAfterWrite(Duration.ofMinutes(BLOCK_DURATION_MINUTES))
            .build(key -> 0);
            
        this.blockCache = Caffeine.newBuilder()
            .expireAfterWrite(Duration.ofMinutes(BLOCK_DURATION_MINUTES))
            .build(key -> null);
    }
    
    public void loginSucceeded(String key) {
        attemptsCache.invalidate(key);
        blockCache.invalidate(key);
    }
    
    public void loginFailed(String key) {
        int attempts = attemptsCache.get(key) + 1;
        attemptsCache.put(key, attempts);
        
        if (attempts >= MAX_ATTEMPTS) {
            blockCache.put(key, LocalDateTime.now());
        }
    }
    
    public boolean isBlocked(String key) {
        LocalDateTime blockTime = blockCache.getIfPresent(key);
        return blockTime != null && 
               LocalDateTime.now().isBefore(blockTime.plusMinutes(BLOCK_DURATION_MINUTES));
    }
    
    public int getAttempts(String key) {
        return attemptsCache.get(key);
    }
    
    public int getRemainingAttempts(String key) {
        return Math.max(0, MAX_ATTEMPTS - getAttempts(key));
    }
}
```

#### Rate Limiting Filter
```java
@Component
public class RateLimitingFilter implements Filter {
    
    private final LoadingCache<String, Integer> requestCounts;
    private final LoadingCache<String, LocalDateTime> lastRequestTime;
    
    @Value("${app.rate-limit.requests-per-minute:60}")
    private int requestsPerMinute;
    
    public RateLimitingFilter() {
        this.requestCounts = Caffeine.newBuilder()
            .expireAfterWrite(Duration.ofMinutes(1))
            .build(key -> 0);
            
        this.lastRequestTime = Caffeine.newBuilder()
            .expireAfterWrite(Duration.ofMinutes(1))
            .build(key -> LocalDateTime.now());
    }
    
    @Override
    public void doFilter(
            ServletRequest request, 
            ServletResponse response, 
            FilterChain chain) throws IOException, ServletException {
        
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        HttpServletResponse httpResponse = (HttpServletResponse) response;
        
        String clientIp = getClientIpAddress(httpRequest);
        String key = clientIp + ":" + httpRequest.getRequestURI();
        
        if (isRateLimited(key)) {
            httpResponse.setStatus(HttpStatus.TOO_MANY_REQUESTS.value());
            httpResponse.setContentType("application/json");
            
            ApiResponse errorResponse = ApiResponse.error(
                "Troppi tentativi. Riprova più tardi.",
                "RATE_LIMIT_EXCEEDED"
            );
            
            ObjectMapper mapper = new ObjectMapper();
            httpResponse.getWriter().write(mapper.writeValueAsString(errorResponse));
            return;
        }
        
        chain.doFilter(request, response);
    }
    
    private boolean isRateLimited(String key) {
        LocalDateTime now = LocalDateTime.now();
        LocalDateTime lastRequest = lastRequestTime.get(key);
        
        if (Duration.between(lastRequest, now).toMinutes() >= 1) {
            requestCounts.put(key, 1);
            lastRequestTime.put(key, now);
            return false;
        }
        
        int currentCount = requestCounts.get(key) + 1;
        requestCounts.put(key, currentCount);
        lastRequestTime.put(key, now);
        
        return currentCount > requestsPerMinute;
    }
    
    private String getClientIpAddress(HttpServletRequest request) {
        String xForwardedFor = request.getHeader("X-Forwarded-For");
        if (StringUtils.hasText(xForwardedFor)) {
            return xForwardedFor.split(",")[0].trim();
        }
        return request.getRemoteAddr();
    }
}
```

### 5.5.2 Input Validation e Sanitization

#### Security Utilities
```java
@Component
public class SecurityUtils {
    
    private static final Pattern SQL_INJECTION_PATTERN = Pattern.compile(
        "(?i)(union|select|insert|update|delete|drop|create|alter|exec|execute|script|javascript|vbscript|onload|onerror)"
    );
    
    private static final Pattern XSS_PATTERN = Pattern.compile(
        "(?i)<script[^>]*>.*?</script>|javascript:|vbscript:|onload=|onerror=|<iframe|<object|<embed"
    );
    
    public static String sanitizeInput(String input) {
        if (input == null) {
            return null;
        }
        
        // Rimozione caratteri di controllo
        String sanitized = input.replaceAll("[\\p{Cntrl}&&[^\\r\\n\\t]]", "");
        
        // Escape HTML
        sanitized = StringEscapeUtils.escapeHtml4(sanitized);
        
        // Trim e normalizzazione spazi
        sanitized = sanitized.trim().replaceAll("\\s+", " ");
        
        return sanitized;
    }
    
    public static boolean containsSqlInjection(String input) {
        if (input == null) {
            return false;
        }
        return SQL_INJECTION_PATTERN.matcher(input).find();
    }
    
    public static boolean containsXss(String input) {
        if (input == null) {
            return false;
        }
        return XSS_PATTERN.matcher(input).find();
    }
    
    public static void validateInput(String input, String fieldName) {
        if (containsSqlInjection(input)) {
            throw new SecurityException(
                "Potenziale SQL injection rilevata nel campo: " + fieldName
            );
        }
        
        if (containsXss(input)) {
            throw new SecurityException(
                "Potenziale XSS rilevato nel campo: " + fieldName
            );
        }
    }
    
    public static String generateSecureRandomString(int length) {
        SecureRandom random = new SecureRandom();
        StringBuilder sb = new StringBuilder(length);
        String chars = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789";
        
        for (int i = 0; i < length; i++) {
            sb.append(chars.charAt(random.nextInt(chars.length())));
        }
        
        return sb.toString();
    }
    
    public static boolean isValidEmail(String email) {
        return email != null && 
               email.matches("^[A-Za-z0-9+_.-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,}$") &&
               email.length() <= 254;
    }
    
    public static boolean isStrongPassword(String password) {
        if (password == null || password.length() < 8) {
            return false;
        }
        
        boolean hasUpper = password.chars().anyMatch(Character::isUpperCase);
        boolean hasLower = password.chars().anyMatch(Character::isLowerCase);
        boolean hasDigit = password.chars().anyMatch(Character::isDigit);
        boolean hasSpecial = password.chars().anyMatch(ch -> "!@#$%^&*()_+-=[]{}|;:,.<>?".indexOf(ch) >= 0);
        
        return hasUpper && hasLower && hasDigit && hasSpecial;
    }
}
```

### 5.5.3 Audit e Logging

#### Audit Service
```java
@Service
@Transactional
public class AuditService {
    
    private final AuditLogRepository auditLogRepository;
    private final ObjectMapper objectMapper;
    
    public void logUserAction(
            Long userId, 
            String action, 
            String resource, 
            Object details,
            HttpServletRequest request) {
        
        try {
            AuditLog auditLog = AuditLog.builder()
                .userId(userId)
                .action(action)
                .resource(resource)
                .details(objectMapper.writeValueAsString(details))
                .ipAddress(getClientIpAddress(request))
                .userAgent(request.getHeader("User-Agent"))
                .timestamp(LocalDateTime.now())
                .build();
            
            auditLogRepository.save(auditLog);
        } catch (Exception e) {
            // Log error but don't fail the main operation
            logger.error("Failed to save audit log", e);
        }
    }
    
    public void logSecurityEvent(
            String eventType, 
            String description, 
            String ipAddress,
            Long userId) {
        
        SecurityEvent event = SecurityEvent.builder()
            .eventType(eventType)
            .description(description)
            .ipAddress(ipAddress)
            .userId(userId)
            .timestamp(LocalDateTime.now())
            .build();
        
        securityEventRepository.save(event);
    }
    
    public void logLoginAttempt(
            String username, 
            boolean successful, 
            String ipAddress,
            String userAgent) {
        
        LoginAttempt attempt = LoginAttempt.builder()
            .username(username)
            .successful(successful)
            .ipAddress(ipAddress)
            .userAgent(userAgent)
            .timestamp(LocalDateTime.now())
            .build();
        
        loginAttemptRepository.save(attempt);
    }
}
```

La sicurezza è un aspetto critico che richiede un approccio a più livelli, dalla validazione dell'input alla gestione delle sessioni, dall'autenticazione all'autorizzazione, fino al monitoraggio e all'audit delle attività.