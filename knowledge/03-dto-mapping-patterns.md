# 3. Data Transfer Objects (DTO) e Pattern di Mapping

## 3.1 Filosofia dei DTO

I **Data Transfer Objects (DTO)** implementano il principio di **separazione delle responsabilità** tra rappresentazione interna (entità) e rappresentazione esterna (API). Risolvono il problema dell'**accoppiamento strutturale** che si crea esponendo direttamente le entità di dominio.

**Principi Fondamentali:**
- **Isolamento**: Ciò che è ottimale per la persistenza non è ottimale per la comunicazione
- **Stabilità**: L'interfaccia API rimane stabile indipendentemente dai cambiamenti interni
- **Performance**: Ottimizzazione della struttura per il trasferimento dati

```java
// Entità: Persistenza e integrità
@Entity
public class Game {
    private Long id;
    private String title;
    @ManyToOne(fetch = FetchType.LAZY)
    private Developer developer;
}

// DTO: Comunicazione ottimizzata
public class GameSummaryDTO {
    private String title;
    private String developerName; // Denormalizzato
    private boolean discountEligible; // Pre-calcolato
}
```
## 3.2 Tipi di DTO

### Response DTO (Output)
- **Summary DTO**: Informazioni essenziali per liste (id, title, price)
- **Detail DTO**: Vista completa con dati derivati e aggregati
- **Stats DTO**: Metriche e dati aggregati per reporting

### Request DTO (Input)
- **Create DTO**: Campi obbligatori con validazioni stringenti
- **Update DTO**: Campi opzionali per modifiche parziali
- **Search DTO**: Parametri di ricerca tipizzati

### DTO per Paginazione
Gestione di grandi dataset con parametri di navigazione e metadati di risposta.

```java
// Paginazione Request
public class PageRequest {
    private int page = 0;
    private int size = 20;
    private String sortBy = "id";
    private String sortDir = "asc";
}

// Paginazione Response
public class PageResponse<T> {
    private List<T> content;
    private int totalPages;
    private long totalElements;
    private boolean hasNext;
}
```
```

## 3.3 Pattern di Mapping

### Mapping Manuale
**Controllo completo** sulla trasformazione con logica di business personalizzata.

**Vantaggi**: Trasparenza, controllo granulare, debugging facilitato
**Svantaggi**: Verbosità, manutenzione manuale, rischio di inconsistenza

```java
@Service
public class GameMapper {
    public GameDTO toDTO(Game game) {
        return GameDTO.builder()
            .title(game.getTitle())
            .developerName(game.getDeveloper().getName())
            .discountEligible(game.isEligibleForDiscount())
            .build();
    }
}
```

### MapStruct - Mapping Automatico
**Generazione di codice compile-time** con convenzioni automatiche.

**Vantaggi**: Produttività, sicurezza compile-time, performance ottimizzate
**Principi**: Convenzione su configurazione, null safety, composizione

```java
@Mapper(componentModel = "spring", uses = {DeveloperMapper.class})
public interface GameMapper {
    @Mapping(target = "developerName", source = "developer.name")
    GameDTO toDTO(Game game);
    
    List<GameDTO> toDTOList(List<Game> games);
}
```

## 3.4 Best Practices

**Principi Fondamentali:**
- **Separazione delle Responsabilità**: DTO per trasferimento, Entity per persistenza, Mapper per trasformazione
- **Immutabilità**: DTO immutabili con pattern Builder
- **Validazione Stratificata**: Sintassi, semantica e contesto
- **Naming Convention**: `*DTO` per response, `*Request` per input

```java
// DTO Immutabile
@Value
@Builder
public class GameDTO {
    Long id;
    String title;
    BigDecimal price;
    LocalDate releaseDate;
}

// Request con Validazione
@Data
public class GameCreateRequest {
    @NotBlank @Size(max = 255)
    private String title;
    
    @NotNull @DecimalMin("0.01")
    private BigDecimal price;
    
    @PastOrPresent
    private LocalDate releaseDate;
}
```

**Strategie Avanzate:**
- **Projection**: Selezione di campi specifici per performance
- **EntityGraph**: Controllo del lazy loading
- **Caching**: Per DTO costosi e dati aggregati
- **Versionamento**: Gestione dell'evoluzione API

I DTO forniscono un livello di astrazione essenziale tra il dominio interno e l'interfaccia esterna, garantendo stabilità, performance e manutenibilità.