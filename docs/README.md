# Libreria Videogiochi - Documentazione Progetto

## Panoramica del Progetto

Questo progetto implementa una libreria di videogiochi simile a IMDb, progettata per praticare concetti avanzati di sviluppo software come:

- **Pattern MVC** (Model-View-Controller)
- **IoC** (Inversion of Control) e Dependency Injection
- **DTO** (Data Transfer Objects)
- **Query dinamiche** e ottimizzazione database
- **Paginazione** e performance
- **Best practices** di sviluppo

## Struttura della Documentazione

1. [Analisi dei Requisiti](./01-analisi-requisiti.md)
2. [Architettura del Sistema](./02-architettura-sistema.md)
3. [Design del Database](./03-design-database.md)
4. [Implementazione Backend](./04-implementazione-backend.md)
5. [Implementazione Frontend](./05-implementazione-frontend.md)
6. [Sicurezza e Autenticazione](./06-sicurezza-autenticazione.md)
7. [Testing e Quality Assurance](./07-testing-qa.md)
8. [Deployment e DevOps](./08-deployment-devops.md)
9. [Ottimizzazioni e Performance](./09-ottimizzazioni-performance.md)
10. [Manutenzione e Evoluzione](./10-manutenzione-evoluzione.md)

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

## Prossimi Passi

Per iniziare lo sviluppo, consultare la documentazione nell'ordine indicato, partendo dall'[Analisi dei Requisiti](./01-analisi-requisiti.md).

Per approfondire i concetti teorici, consultare la cartella `knowledge/` che contiene le dispense dettagliate su tutti gli argomenti trattati.