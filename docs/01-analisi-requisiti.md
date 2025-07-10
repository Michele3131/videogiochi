# 1. Analisi dei Requisiti

## 1.1 Introduzione

La Libreria Videogiochi è un'applicazione web che permette agli utenti di gestire la propria collezione di videogiochi, simile a come IMDb gestisce i film. L'obiettivo è creare un sistema completo che dimostri l'implementazione di best practices e pattern di sviluppo moderni.

## 1.2 Stakeholder

### 1.2.1 Utenti Finali
- **Giocatori Casual**: Utenti che vogliono tenere traccia dei giochi giocati
- **Giocatori Hardcore**: Utenti che necessitano di funzionalità avanzate di catalogazione
- **Collezionisti**: Utenti interessati a statistiche dettagliate e gestione avanzata

### 1.2.2 Amministratori
- **Content Manager**: Gestione del catalogo videogiochi
- **System Administrator**: Manutenzione del sistema

## 1.3 Requisiti Funzionali

### 1.3.1 Gestione Utenti (RF-01)

#### RF-01.1 Registrazione
- **Descrizione**: Gli utenti possono creare un nuovo account
- **Input**: Email, username, password, conferma password
- **Validazioni**:
  - Email valida e univoca
  - Username univoco (3-20 caratteri)
  - Password sicura (min 8 caratteri, maiuscole, minuscole, numeri)
- **Output**: Account creato, email di conferma inviata
- **Priorità**: Alta

#### RF-01.2 Autenticazione
- **Descrizione**: Gli utenti possono accedere al sistema
- **Input**: Username/Email e password
- **Validazioni**: Credenziali corrette, account attivo
- **Output**: Sessione autenticata, redirect alla dashboard
- **Priorità**: Alta

#### RF-01.3 Gestione Profilo
- **Descrizione**: Gli utenti possono modificare le proprie informazioni
- **Funzionalità**:
  - Modifica dati personali
  - Cambio password
  - Upload avatar
  - Impostazioni privacy
- **Priorità**: Media

### 1.3.2 Catalogo Videogiochi (RF-02)

#### RF-02.1 Visualizzazione Catalogo
- **Descrizione**: Visualizzazione paginata di tutti i videogiochi
- **Funzionalità**:
  - Lista con anteprima (titolo, immagine, anno, rating)
  - Paginazione configurabile (10, 25, 50, 100 elementi)
  - Ordinamento multiplo (titolo, anno, rating, data aggiunta)
- **Priorità**: Alta

#### RF-02.2 Ricerca Avanzata
- **Descrizione**: Sistema di ricerca con filtri multipli
- **Filtri disponibili**:
  - Testo libero (titolo, descrizione)
  - Genere/Categoria
  - Anno di pubblicazione (range)
  - Sviluppatore/Publisher
  - Piattaforma
  - Rating (range)
- **Funzionalità**:
  - Combinazione di filtri con operatori AND/OR
  - Salvataggio ricerche frequenti
  - Suggerimenti automatici
- **Priorità**: Alta

#### RF-02.3 Dettaglio Videogioco
- **Descrizione**: Visualizzazione completa delle informazioni
- **Informazioni mostrate**:
  - Dati base (titolo, anno, genere, piattaforme)
  - Descrizione dettagliata
  - Screenshots e trailer
  - Sviluppatore e publisher
  - Recensioni utenti
  - Statistiche (tempo medio di gioco, difficoltà)
- **Priorità**: Alta

### 1.3.3 Liste Personali (RF-03)

#### RF-03.1 Lista "Giocati"
- **Descrizione**: Gestione dei videogiochi completati
- **Funzionalità**:
  - Aggiunta/rimozione giochi
  - Valutazione (1-10 stelle)
  - Recensione personale
  - Data di completamento
  - Tempo di gioco
  - Note private
- **Priorità**: Alta

#### RF-03.2 Wishlist
- **Descrizione**: Lista dei giochi da giocare
- **Funzionalità**:
  - Aggiunta/rimozione giochi
  - Priorità (alta, media, bassa)
  - Note e promemoria
  - Notifiche per sconti/uscite
- **Priorità**: Media

#### RF-03.3 Liste Personalizzate
- **Descrizione**: Creazione di liste tematiche
- **Funzionalità**:
  - Creazione liste con nome e descrizione
  - Aggiunta/rimozione giochi
  - Condivisione liste (pubbliche/private)
  - Ordinamento personalizzato
- **Priorità**: Bassa

### 1.3.4 Gestione Contenuti (RF-04)

#### RF-04.1 CRUD Videogiochi
- **Descrizione**: Gestione completa del catalogo (solo admin)
- **Operazioni**:
  - Creazione nuovo videogioco
  - Modifica informazioni esistenti
  - Eliminazione (soft delete)
  - Gestione immagini e media
- **Validazioni**:
  - Campi obbligatori
  - Formati file supportati
  - Dimensioni massime
- **Priorità**: Alta

#### RF-04.2 Gestione Sviluppatori/Publisher
- **Descrizione**: Anagrafica aziende del settore
- **Informazioni**:
  - Nome, descrizione, logo
  - Anno di fondazione
  - Sede principale
  - Sito web e social
- **Priorità**: Media

### 1.3.5 Funzionalità Avanzate (RF-05)

#### RF-05.1 Raccomandazioni
- **Descrizione**: Sistema di suggerimenti personalizzati
- **Algoritmi**:
  - Basato su giochi simili
  - Basato su preferenze utente
  - Collaborative filtering
- **Priorità**: Bassa

#### RF-05.2 Statistiche
- **Descrizione**: Dashboard con analytics personali
- **Metriche**:
  - Giochi completati per anno/mese
  - Generi preferiti
  - Tempo totale di gioco
  - Trend valutazioni
- **Priorità**: Bassa

## 1.4 Requisiti Non Funzionali

### 1.4.1 Performance (RNF-01)
- **Tempo di risposta**: < 2 secondi per operazioni standard
- **Throughput**: Supporto per 100 utenti concorrenti
- **Scalabilità**: Architettura scalabile orizzontalmente

### 1.4.2 Sicurezza (RNF-02)
- **Autenticazione**: Sistema robusto con session management
- **Autorizzazione**: Controllo accessi basato su ruoli
- **Protezione dati**: Crittografia password, HTTPS obbligatorio
- **Validazione input**: Sanitizzazione contro XSS, SQL Injection

### 1.4.3 Usabilità (RNF-03)
- **Responsive design**: Supporto mobile e desktop
- **Accessibilità**: Conformità WCAG 2.1 AA
- **Internazionalizzazione**: Supporto multilingua (IT, EN)

### 1.4.4 Affidabilità (RNF-04)
- **Disponibilità**: 99.5% uptime
- **Backup**: Backup automatici giornalieri
- **Recovery**: RTO < 4 ore, RPO < 1 ora

### 1.4.5 Manutenibilità (RNF-05)
- **Codice**: Copertura test > 80%
- **Documentazione**: API documentate con OpenAPI
- **Logging**: Sistema di logging strutturato
- **Monitoring**: Metriche applicative e infrastrutturali

## 1.5 Vincoli Tecnici

### 1.5.1 Tecnologie Obbligatorie
- **Backend**: Spring Boot 3.x, Java 21
- **Database**: MySQL 8.x
- **Frontend**: Thymeleaf, Bootstrap 5
- **Build**: Maven 3.x

### 1.5.2 Vincoli Architetturali
- **Pattern**: MVC con separazione netta dei layer
- **Dependency Injection**: Uso estensivo di Spring IoC
- **ORM**: JPA/Hibernate per persistenza
- **REST API**: Esposizione servizi RESTful

## 1.6 Casi d'Uso Principali

### 1.6.1 UC-01: Registrazione Nuovo Utente
**Attore**: Visitatore
**Precondizioni**: Nessuna
**Flusso principale**:
1. L'utente accede alla pagina di registrazione
2. Compila il form con i dati richiesti
3. Il sistema valida i dati
4. Il sistema crea l'account
5. Il sistema invia email di conferma
6. L'utente conferma l'email
7. L'account viene attivato

**Flussi alternativi**:
- 3a. Dati non validi: mostra errori, torna al punto 2
- 5a. Errore invio email: mostra messaggio, permette reinvio

### 1.6.2 UC-02: Ricerca Videogiochi
**Attore**: Utente autenticato
**Precondizioni**: Utente loggato
**Flusso principale**:
1. L'utente accede alla pagina di ricerca
2. Inserisce criteri di ricerca
3. Il sistema esegue la query
4. Il sistema mostra i risultati paginati
5. L'utente può raffinare la ricerca

**Flussi alternativi**:
- 3a. Nessun risultato: mostra messaggio appropriato
- 4a. Troppi risultati: suggerisce filtri aggiuntivi

### 1.6.3 UC-03: Aggiunta Gioco a Lista
**Attore**: Utente autenticato
**Precondizioni**: Utente loggato, gioco esistente
**Flusso principale**:
1. L'utente visualizza il dettaglio di un gioco
2. Seleziona "Aggiungi a lista"
3. Sceglie la lista di destinazione
4. Opzionalmente aggiunge valutazione/note
5. Il sistema salva l'associazione
6. Il sistema conferma l'operazione

**Flussi alternativi**:
- 3a. Gioco già presente: mostra opzioni di modifica
- 5a. Errore salvataggio: mostra messaggio di errore

## 1.7 Modello dei Dati Concettuale

### 1.7.1 Entità Principali

#### User
- id, username, email, password_hash
- first_name, last_name, birth_date
- avatar_url, bio, created_at, updated_at
- is_active, email_verified, last_login

#### Game
- id, title, description, release_date
- genre, platforms, developer_id, publisher_id
- cover_image_url, screenshots, trailer_url
- average_rating, total_ratings, created_at, updated_at

#### UserGameList
- id, user_id, game_id, list_type
- rating, review, completion_date, play_time
- priority, notes, created_at, updated_at

#### Developer/Publisher
- id, name, description, logo_url
- founded_year, headquarters, website
- created_at, updated_at

### 1.7.2 Relazioni
- User 1:N UserGameList
- Game 1:N UserGameList
- Developer 1:N Game
- Publisher 1:N Game
- User 1:N CustomList
- CustomList 1:N CustomListGame

## 1.8 Prioritizzazione e Roadmap

### 1.8.1 MVP (Minimum Viable Product)
**Durata stimata**: 4-6 settimane
- Registrazione e autenticazione utenti
- Catalogo base con ricerca semplice
- Lista "Giocati" e Wishlist
- CRUD amministrativo base

### 1.8.2 Versione 1.0
**Durata stimata**: 8-10 settimane
- Ricerca avanzata con filtri
- Gestione completa profili
- Liste personalizzate
- API REST documentate

### 1.8.3 Versioni Future
- Sistema di raccomandazioni
- Statistiche avanzate
- Integrazione social
- App mobile

## 1.9 Criteri di Accettazione

Ogni funzionalità deve soddisfare:
1. **Funzionalità**: Comportamento conforme ai requisiti
2. **Performance**: Tempi di risposta entro i limiti
3. **Sicurezza**: Validazioni e controlli implementati
4. **Usabilità**: Interfaccia intuitiva e responsive
5. **Testing**: Copertura test adeguata
6. **Documentazione**: Codice e API documentati

## 1.10 Rischi e Mitigazioni

### 1.10.1 Rischi Tecnici
- **Complessità query**: Mitigazione con caching e ottimizzazioni
- **Scalabilità database**: Mitigazione con indexing e partitioning
- **Sicurezza**: Mitigazione con security audit e penetration testing

### 1.10.2 Rischi di Progetto
- **Scope creep**: Mitigazione con gestione rigorosa dei requisiti
- **Performance**: Mitigazione con testing continuo
- **Manutenibilità**: Mitigazione con code review e refactoring

Questo documento costituisce la base per tutte le fasi successive del progetto e deve essere mantenuto aggiornato durante lo sviluppo.