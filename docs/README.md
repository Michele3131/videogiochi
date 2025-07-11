# Libreria Videogiochi - Documentazione Progetto

> **Riferimenti teorici**: Per approfondire i pattern architetturali utilizzati, consulta [Pattern Architetturali](../knowledge/01-pattern-architetturali.md) e [DTO Mapping Patterns](../knowledge/03-dto-mapping-patterns.md)

## Panoramica del Progetto

Questo progetto implementa una libreria di videogiochi simile a IMDb, progettata per praticare concetti avanzati di sviluppo software come:

- **Pattern MVC** (Model-View-Controller) - *Vedi [Pattern Architetturali](../knowledge/01-pattern-architetturali.md)*
- **IoC** (Inversion of Control) e Dependency Injection
- **DTO** (Data Transfer Objects) - *Vedi [DTO Mapping Patterns](../knowledge/03-dto-mapping-patterns.md)*
- **Query dinamiche** e ottimizzazione database
- **Paginazione** e performance
- **Best practices** di sviluppo

## Struttura della Documentazione

### Documentazione Implementativa (docs/)
1. [Analisi dei Requisiti](./01-analisi-requisiti.md)
2. [Architettura del Sistema](./02-architettura-sistema.md)
3. [Design del Database](./03-design-database.md)
4. [Implementazione Backend](./04-implementazione-backend.md) - **Codice completo backend**
5. [Service Layer e Mappers](./05-service-layer-mappers.md) - **Logica di business**
6. [Controller e Sicurezza](./06-controller-security.md) - **API REST e autenticazione**
7. [Exception e Validation](./07-exception-validation.md) - **Gestione errori**
8. [Testing, Config e Deployment](./08-testing-config-deployment.md) - **Test e configurazione**
9. [Frontend Implementation](./09-frontend-implementation.md) - **Interfaccia utente**
10. [Frontend Components e Pages](./10-frontend-components-pages.md) - **Componenti UI**

### Documentazione Teorica (knowledge/)
- [Pattern Architetturali](../knowledge/01-pattern-architetturali.md) - MVC, Layered Architecture, Repository Pattern
- [DTO Mapping Patterns](../knowledge/03-dto-mapping-patterns.md) - Strategie di mapping e best practices

## Obiettivi di Apprendimento

### Concetti Teorici da Implementare

- **Separazione delle responsabilità** attraverso il pattern MVC
- **Inversione delle dipendenze** con Spring IoC Container
- **Mapping oggetto-relazionale** con JPA/Hibernate
- **Gestione delle transazioni** e consistenza dei dati
- **Validazione dei dati** a livello di business logic
- **Gestione degli errori** e exception handling
- **Logging e monitoring** dell'applicazione
- **Caching** per migliorare le performance
- **Paginazione** per gestire grandi dataset
- **Query dinamiche** per ricerche flessibili

### Tecnologie Utilizzate

- **Backend**: Spring Boot, Spring Data JPA, Spring Security
- **Database**: MySQL
- **Frontend**: Thymeleaf, Bootstrap, JavaScript
- **Build Tool**: Maven
- **Testing**: JUnit 5, Mockito, TestContainers
- **Documentation**: OpenAPI/Swagger

## Funzionalità Principali

### Gestione Utenti
- Registrazione e autenticazione
- Profili utente personalizzabili
- Gestione delle preferenze

### Catalogo Videogiochi
- Visualizzazione completa del catalogo
- Ricerca avanzata con filtri multipli
- Ordinamento per vari criteri
- Paginazione intelligente

### Liste Personali
- Videogiochi giocati
- Wishlist (da giocare)
- Valutazioni e recensioni
- Liste personalizzate

### Gestione Contenuti
- CRUD completo per videogiochi
- Gestione sviluppatori e publisher
- Categorie e tag
- Upload e gestione immagini

### Funzionalità Avanzate
- Raccomandazioni personalizzate
- Statistiche e analytics
- Export/Import dati
- API REST per integrazioni esterne

## Metodologia di Sviluppo

Il progetto seguirà un approccio incrementale:

1. **Fase 1**: Setup iniziale e modello dati
2. **Fase 2**: Implementazione backend core
3. **Fase 3**: Interfaccia utente base
4. **Fase 4**: Funzionalità avanzate
5. **Fase 5**: Ottimizzazioni e deployment

Ogni fase includerà:
- Analisi e design
- Implementazione
- Testing
- Documentazione
- Review e refactoring

## 🚀 Guida Rapida per Iniziare

### Prerequisiti
- **Java 21** o superiore
- **Maven 3.8+**
- **MySQL 8.0+**
- **IDE** (IntelliJ IDEA, Eclipse, VS Code)

### Installazione e Setup

1. **Clona il repository**
   ```bash
   git clone <repository-url>
   cd videogiochi
   ```

2. **Configura il database MySQL**
   ```sql
   CREATE DATABASE videogiochi;
   CREATE USER 'videogiochi_app'@'localhost' IDENTIFIED BY 'secure_password';
   GRANT ALL PRIVILEGES ON videogiochi.* TO 'videogiochi_app'@'localhost';
   FLUSH PRIVILEGES;
   ```

3. **Configura le variabili d'ambiente** (opzionale)
   ```bash
   export DB_USERNAME=videogiochi_app
   export DB_PASSWORD=secure_password
   export JWT_SECRET=mySecretKey
   ```

4. **Compila e avvia l'applicazione**
   ```bash
   # Compila il progetto
   mvn clean compile
   
   # Avvia l'applicazione
   mvn spring-boot:run
   ```

5. **Accedi all'applicazione**
   - **Frontend**: http://localhost:8080
   - **API Documentation**: http://localhost:8080/api/swagger-ui.html
   - **API Base URL**: http://localhost:8080/api/v1

### Struttura del Progetto
```
src/
├── main/
│   ├── java/mc/videogiochi/
│   │   ├── domain/entity/          # Entità JPA
│   │   ├── repository/             # Repository per accesso dati
│   │   ├── service/                # Logica di business
│   │   ├── controller/             # Controller REST
│   │   ├── dto/                    # Data Transfer Objects
│   │   ├── mapper/                 # MapStruct mappers
│   │   ├── config/                 # Configurazioni
│   │   └── exception/              # Gestione eccezioni
│   └── resources/
│       ├── application.yml         # Configurazione applicazione
│       ├── db/migration/           # Script Flyway
│       └── templates/              # Template Thymeleaf
└── test/                           # Test unitari e integrazione
```

### Credenziali di Default
- **Username**: admin
- **Password**: admin123

### Prossimi Passi

1. **Per sviluppatori principianti**: Inizia con [Implementazione Backend](./04-implementazione-backend.md) per vedere il codice completo
2. **Per approfondimenti teorici**: Consulta [Pattern Architetturali](../knowledge/01-pattern-architetturali.md)
3. **Per comprendere i DTO**: Leggi [DTO Mapping Patterns](../knowledge/03-dto-mapping-patterns.md)

### Supporto e Documentazione
- Tutti i file di codice contengono commenti dettagliati per ogni riga importante
- La documentazione teorica nella cartella `knowledge/` spiega i concetti utilizzati
- Le API sono documentate con Swagger/OpenAPI