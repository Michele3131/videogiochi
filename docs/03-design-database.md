# 3. Design del Database

## 3.1 Panoramica del Modello Dati

### 3.1.1 Principi di Design

Il design del database segue i seguenti principi:

- **Normalizzazione**: Fino alla 3NF per ridurre ridondanza
- **Integrità Referenziale**: Vincoli FK per consistenza dati
- **Performance**: Indici strategici per query frequenti
- **Scalabilità**: Struttura che supporta crescita dati
- **Flessibilità**: Estensibilità per nuove funzionalità
- **Sicurezza**: Controllo accessi e audit trail

### 3.1.2 Tecnologie Utilizzate

- **RDBMS**: MySQL 8.0+
- **ORM**: Hibernate/JPA
- **Migration**: Flyway
- **Connection Pool**: HikariCP
- **Monitoring**: MySQL Performance Schema

## 3.2 Modello Entità-Relazione

### 3.2.1 Diagramma ER Concettuale

```
┌─────────────┐       ┌─────────────┐       ┌─────────────┐
│    User     │       │    Game     │       │ Developer   │
│             │       │             │       │             │
│ + id        │       │ + id        │       │ + id        │
│ + username  │       │ + title     │       │ + name      │
│ + email     │       │ + desc      │       │ + founded   │
│ + password  │       │ + release   │       │ + website   │
└─────────────┘       └─────────────┘       └─────────────┘
       │                     │                     │
       │                     │                     │
       │              ┌─────────────┐               │
       │              │ Publisher   │               │
       │              │             │               │
       │              │ + id        │               │
       │              │ + name      │               │
       │              │ + founded   │               │
       │              └─────────────┘               │
       │                     │                     │
       │                     │                     │
┌─────────────┐       ┌─────────────┐       ┌─────────────┐
│UserGameList │       │   Genre     │       │  Platform   │
│             │       │             │       │             │
│ + user_id   │       │ + id        │       │ + id        │
│ + game_id   │       │ + name      │       │ + name      │
│ + list_type │       │ + desc      │       │ + type      │
│ + rating    │       └─────────────┘       └─────────────┘
│ + review    │              │                     │
└─────────────┘              │                     │
                      ┌─────────────┐       ┌─────────────┐
                      │ GameGenre   │       │GamePlatform │
                      │             │       │             │
                      │ + game_id   │       │ + game_id   │
                      │ + genre_id  │       │ + platform  │
                      └─────────────┘       └─────────────┘
```

### 3.2.2 Entità Principali

#### User (Utenti)
```sql
CREATE TABLE users (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    birth_date DATE,
    avatar_url VARCHAR(500),
    bio TEXT,
    is_active BOOLEAN DEFAULT TRUE,
    email_verified BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    last_login TIMESTAMP,
    
    INDEX idx_username (username),
    INDEX idx_email (email),
    INDEX idx_active (is_active),
    INDEX idx_created_at (created_at)
);
```

#### Game (Videogiochi)
```sql
CREATE TABLE games (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    title VARCHAR(200) NOT NULL,
    description TEXT,
    release_date DATE,
    developer_id BIGINT,
    publisher_id BIGINT,
    cover_image_url VARCHAR(500),
    trailer_url VARCHAR(500),
    average_rating DECIMAL(3,2) DEFAULT 0.00,
    total_ratings INT DEFAULT 0,
    metacritic_score INT,
    esrb_rating ENUM('E', 'E10+', 'T', 'M', 'AO', 'RP'),
    price DECIMAL(8,2),
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    FOREIGN KEY (developer_id) REFERENCES developers(id),
    FOREIGN KEY (publisher_id) REFERENCES publishers(id),
    
    INDEX idx_title (title),
    INDEX idx_release_date (release_date),
    INDEX idx_developer (developer_id),
    INDEX idx_publisher (publisher_id),
    INDEX idx_rating (average_rating),
    INDEX idx_active (is_active),
    FULLTEXT idx_search (title, description)
);
```

#### Developer (Sviluppatori)
```sql
CREATE TABLE developers (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL UNIQUE,
    description TEXT,
    founded_year YEAR,
    headquarters VARCHAR(100),
    website VARCHAR(200),
    logo_url VARCHAR(500),
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    INDEX idx_name (name),
    INDEX idx_founded_year (founded_year),
    INDEX idx_active (is_active)
);
```

#### Publisher (Editori)
```sql
CREATE TABLE publishers (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL UNIQUE,
    description TEXT,
    founded_year YEAR,
    headquarters VARCHAR(100),
    website VARCHAR(200),
    logo_url VARCHAR(500),
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    INDEX idx_name (name),
    INDEX idx_founded_year (founded_year),
    INDEX idx_active (is_active)
);
```

#### UserGameList (Liste Utente)
```sql
CREATE TABLE user_game_lists (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT NOT NULL,
    game_id BIGINT NOT NULL,
    list_type ENUM('PLAYED', 'WISHLIST', 'PLAYING', 'DROPPED', 'CUSTOM') NOT NULL,
    rating TINYINT CHECK (rating >= 1 AND rating <= 10),
    review TEXT,
    completion_date DATE,
    play_time_hours INT,
    priority ENUM('LOW', 'MEDIUM', 'HIGH') DEFAULT 'MEDIUM',
    notes TEXT,
    is_favorite BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (game_id) REFERENCES games(id) ON DELETE CASCADE,
    
    UNIQUE KEY uk_user_game_list (user_id, game_id, list_type),
    INDEX idx_user_list_type (user_id, list_type),
    INDEX idx_game_rating (game_id, rating),
    INDEX idx_completion_date (completion_date),
    INDEX idx_favorite (is_favorite)
);
```

### 3.2.3 Entità di Supporto

#### Genre (Generi)
```sql
CREATE TABLE genres (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50) NOT NULL UNIQUE,
    description TEXT,
    parent_id BIGINT,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    FOREIGN KEY (parent_id) REFERENCES genres(id),
    INDEX idx_name (name),
    INDEX idx_parent (parent_id),
    INDEX idx_active (is_active)
);
```

#### Platform (Piattaforme)
```sql
CREATE TABLE platforms (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50) NOT NULL UNIQUE,
    type ENUM('CONSOLE', 'PC', 'MOBILE', 'HANDHELD', 'VR') NOT NULL,
    manufacturer VARCHAR(50),
    release_date DATE,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    INDEX idx_name (name),
    INDEX idx_type (type),
    INDEX idx_manufacturer (manufacturer),
    INDEX idx_active (is_active)
);
```

#### GameGenre (Relazione Many-to-Many)
```sql
CREATE TABLE game_genres (
    game_id BIGINT NOT NULL,
    genre_id BIGINT NOT NULL,
    
    PRIMARY KEY (game_id, genre_id),
    FOREIGN KEY (game_id) REFERENCES games(id) ON DELETE CASCADE,
    FOREIGN KEY (genre_id) REFERENCES genres(id) ON DELETE CASCADE,
    
    INDEX idx_genre_games (genre_id)
);
```

#### GamePlatform (Relazione Many-to-Many)
```sql
CREATE TABLE game_platforms (
    game_id BIGINT NOT NULL,
    platform_id BIGINT NOT NULL,
    release_date DATE,
    price DECIMAL(8,2),
    
    PRIMARY KEY (game_id, platform_id),
    FOREIGN KEY (game_id) REFERENCES games(id) ON DELETE CASCADE,
    FOREIGN KEY (platform_id) REFERENCES platforms(id) ON DELETE CASCADE,
    
    INDEX idx_platform_games (platform_id),
    INDEX idx_release_date (release_date)
);
```

### 3.2.4 Entità per Funzionalità Avanzate

#### CustomList (Liste Personalizzate)
```sql
CREATE TABLE custom_lists (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT NOT NULL,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    is_public BOOLEAN DEFAULT FALSE,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    
    INDEX idx_user_name (user_id, name),
    INDEX idx_public (is_public),
    INDEX idx_active (is_active)
);
```

#### CustomListGame (Giochi in Liste Personalizzate)
```sql
CREATE TABLE custom_list_games (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    custom_list_id BIGINT NOT NULL,
    game_id BIGINT NOT NULL,
    position INT,
    notes TEXT,
    added_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    FOREIGN KEY (custom_list_id) REFERENCES custom_lists(id) ON DELETE CASCADE,
    FOREIGN KEY (game_id) REFERENCES games(id) ON DELETE CASCADE,
    
    UNIQUE KEY uk_list_game (custom_list_id, game_id),
    INDEX idx_list_position (custom_list_id, position)
);
```

#### UserRole (Ruoli Utente)
```sql
CREATE TABLE roles (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50) NOT NULL UNIQUE,
    description TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE user_roles (
    user_id BIGINT NOT NULL,
    role_id BIGINT NOT NULL,
    assigned_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    PRIMARY KEY (user_id, role_id),
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (role_id) REFERENCES roles(id) ON DELETE CASCADE
);
```

#### AuditLog (Log delle Operazioni)
```sql
CREATE TABLE audit_logs (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT,
    entity_type VARCHAR(50) NOT NULL,
    entity_id BIGINT NOT NULL,
    operation ENUM('CREATE', 'UPDATE', 'DELETE') NOT NULL,
    old_values JSON,
    new_values JSON,
    ip_address VARCHAR(45),
    user_agent TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE SET NULL,
    
    INDEX idx_user_operation (user_id, operation),
    INDEX idx_entity (entity_type, entity_id),
    INDEX idx_created_at (created_at)
);
```

## 3.3 Strategie di Indicizzazione

### 3.3.1 Indici Primari

**Chiavi Primarie**:
- Tutte le tabelle hanno chiave primaria AUTO_INCREMENT
- Tipo BIGINT per supportare grandi volumi
- Clustered index automatico su PK

### 3.3.2 Indici Secondari

**Indici per Performance Query**:
```sql
-- Ricerca giochi per titolo (case-insensitive)
CREATE INDEX idx_games_title_ci ON games (title);

-- Ricerca giochi per range di date
CREATE INDEX idx_games_release_rating ON games (release_date, average_rating);

-- Query liste utente per tipo
CREATE INDEX idx_user_lists_type_date ON user_game_lists (user_id, list_type, created_at);

-- Statistiche per sviluppatore
CREATE INDEX idx_games_dev_rating ON games (developer_id, average_rating, release_date);

-- Ricerca full-text
CREATE FULLTEXT INDEX idx_games_fulltext ON games (title, description);
```

**Indici Composti per Query Complesse**:
```sql
-- Ricerca avanzata giochi
CREATE INDEX idx_games_search_complex ON games (
    is_active, release_date, average_rating, developer_id
);

-- Performance dashboard utente
CREATE INDEX idx_user_stats ON user_game_lists (
    user_id, list_type, rating, completion_date
);

-- Raccomandazioni basate su genere
CREATE INDEX idx_recommendations ON game_genres (
    genre_id, game_id
) INCLUDE (games.average_rating, games.total_ratings);
```

### 3.3.3 Indici per Integrità Referenziale

```sql
-- Supporto per CASCADE DELETE efficiente
CREATE INDEX idx_user_game_lists_user ON user_game_lists (user_id);
CREATE INDEX idx_user_game_lists_game ON user_game_lists (game_id);
CREATE INDEX idx_custom_list_games_list ON custom_list_games (custom_list_id);
CREATE INDEX idx_custom_list_games_game ON custom_list_games (game_id);
```

## 3.4 Ottimizzazioni per Performance

### 3.4.1 Partitioning Strategy

**Partitioning per Audit Log**:
```sql
CREATE TABLE audit_logs (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT,
    entity_type VARCHAR(50) NOT NULL,
    entity_id BIGINT NOT NULL,
    operation ENUM('CREATE', 'UPDATE', 'DELETE') NOT NULL,
    old_values JSON,
    new_values JSON,
    ip_address VARCHAR(45),
    user_agent TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
)
PARTITION BY RANGE (YEAR(created_at)) (
    PARTITION p2023 VALUES LESS THAN (2024),
    PARTITION p2024 VALUES LESS THAN (2025),
    PARTITION p2025 VALUES LESS THAN (2026),
    PARTITION p_future VALUES LESS THAN MAXVALUE
);
```

### 3.4.2 Denormalizzazione Controllata

**Campi Calcolati per Performance**:
```sql
-- Aggiunta campi denormalizzati alla tabella games
ALTER TABLE games ADD COLUMN (
    total_in_wishlists INT DEFAULT 0,
    total_completed INT DEFAULT 0,
    popularity_score DECIMAL(10,4) DEFAULT 0.0000
);

-- Trigger per mantenere consistenza
DELIMITER //
CREATE TRIGGER update_game_stats_after_list_insert
AFTER INSERT ON user_game_lists
FOR EACH ROW
BEGIN
    IF NEW.list_type = 'WISHLIST' THEN
        UPDATE games SET total_in_wishlists = total_in_wishlists + 1 
        WHERE id = NEW.game_id;
    ELSEIF NEW.list_type = 'PLAYED' THEN
        UPDATE games SET total_completed = total_completed + 1 
        WHERE id = NEW.game_id;
    END IF;
    
    -- Ricalcola popularity score
    UPDATE games SET popularity_score = (
        (total_in_wishlists * 0.3) + 
        (total_completed * 0.5) + 
        (total_ratings * 0.2)
    ) WHERE id = NEW.game_id;
END//
DELIMITER ;
```

### 3.4.3 Caching Strategy a Livello Database

**Query Cache Configuration**:
```sql
-- Configurazione MySQL per query cache
SET GLOBAL query_cache_type = ON;
SET GLOBAL query_cache_size = 268435456; -- 256MB
SET GLOBAL query_cache_limit = 1048576;  -- 1MB per query
```

**Materialized Views (Simulazione)**:
```sql
-- Tabella per statistiche pre-calcolate
CREATE TABLE game_statistics (
    game_id BIGINT PRIMARY KEY,
    total_players INT DEFAULT 0,
    average_rating DECIMAL(3,2) DEFAULT 0.00,
    total_reviews INT DEFAULT 0,
    wishlist_count INT DEFAULT 0,
    completion_rate DECIMAL(5,2) DEFAULT 0.00,
    last_updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    FOREIGN KEY (game_id) REFERENCES games(id) ON DELETE CASCADE,
    INDEX idx_rating (average_rating),
    INDEX idx_popularity (total_players, wishlist_count)
);

-- Stored procedure per aggiornamento statistiche
DELIMITER //
CREATE PROCEDURE UpdateGameStatistics(IN game_id_param BIGINT)
BEGIN
    DECLARE total_players_count INT DEFAULT 0;
    DECLARE avg_rating_calc DECIMAL(3,2) DEFAULT 0.00;
    DECLARE total_reviews_count INT DEFAULT 0;
    DECLARE wishlist_count_calc INT DEFAULT 0;
    DECLARE completion_rate_calc DECIMAL(5,2) DEFAULT 0.00;
    
    -- Calcola statistiche
    SELECT COUNT(DISTINCT user_id) INTO total_players_count
    FROM user_game_lists WHERE game_id = game_id_param;
    
    SELECT AVG(rating), COUNT(rating) INTO avg_rating_calc, total_reviews_count
    FROM user_game_lists 
    WHERE game_id = game_id_param AND rating IS NOT NULL;
    
    SELECT COUNT(*) INTO wishlist_count_calc
    FROM user_game_lists 
    WHERE game_id = game_id_param AND list_type = 'WISHLIST';
    
    -- Aggiorna o inserisce statistiche
    INSERT INTO game_statistics (
        game_id, total_players, average_rating, total_reviews, 
        wishlist_count, completion_rate
    ) VALUES (
        game_id_param, total_players_count, avg_rating_calc, 
        total_reviews_count, wishlist_count_calc, completion_rate_calc
    ) ON DUPLICATE KEY UPDATE
        total_players = total_players_count,
        average_rating = avg_rating_calc,
        total_reviews = total_reviews_count,
        wishlist_count = wishlist_count_calc,
        completion_rate = completion_rate_calc,
        last_updated = CURRENT_TIMESTAMP;
END//
DELIMITER ;
```

## 3.5 Gestione Dati e Migration

### 3.5.1 Flyway Migration Scripts

**V1__Initial_Schema.sql**:
```sql
-- Creazione schema iniziale
CREATE TABLE users (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Altre tabelle...
```

**V2__Add_User_Profile_Fields.sql**:
```sql
-- Aggiunta campi profilo utente
ALTER TABLE users ADD COLUMN (
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    birth_date DATE,
    avatar_url VARCHAR(500),
    bio TEXT
);
```

**V3__Add_Game_Statistics.sql**:
```sql
-- Aggiunta statistiche giochi
ALTER TABLE games ADD COLUMN (
    total_in_wishlists INT DEFAULT 0,
    total_completed INT DEFAULT 0,
    popularity_score DECIMAL(10,4) DEFAULT 0.0000
);

-- Creazione indici per performance
CREATE INDEX idx_games_popularity ON games (popularity_score DESC);
```

### 3.5.2 Data Seeding

**Dati di Base**:
```sql
-- Inserimento ruoli di sistema
INSERT INTO roles (name, description) VALUES 
('ADMIN', 'Amministratore del sistema'),
('USER', 'Utente standard'),
('MODERATOR', 'Moderatore contenuti');

-- Inserimento generi base
INSERT INTO genres (name, description) VALUES 
('Action', 'Giochi d\'azione'),
('Adventure', 'Giochi d\'avventura'),
('RPG', 'Giochi di ruolo'),
('Strategy', 'Giochi strategici'),
('Simulation', 'Simulatori'),
('Sports', 'Giochi sportivi'),
('Racing', 'Giochi di corse'),
('Puzzle', 'Giochi puzzle');

-- Inserimento piattaforme
INSERT INTO platforms (name, type, manufacturer) VALUES 
('PlayStation 5', 'CONSOLE', 'Sony'),
('Xbox Series X', 'CONSOLE', 'Microsoft'),
('Nintendo Switch', 'HANDHELD', 'Nintendo'),
('PC', 'PC', 'Various'),
('iOS', 'MOBILE', 'Apple'),
('Android', 'MOBILE', 'Google');
```

## 3.6 Sicurezza del Database

### 3.6.1 Controllo Accessi

**Utenti Database**:
```sql
-- Utente applicazione (limitato)
CREATE USER 'videogiochi_app'@'%' IDENTIFIED BY 'secure_password';
GRANT SELECT, INSERT, UPDATE, DELETE ON videogiochi.* TO 'videogiochi_app'@'%';

-- Utente read-only per reporting
CREATE USER 'videogiochi_readonly'@'%' IDENTIFIED BY 'readonly_password';
GRANT SELECT ON videogiochi.* TO 'videogiochi_readonly'@'%';

-- Utente backup
CREATE USER 'videogiochi_backup'@'localhost' IDENTIFIED BY 'backup_password';
GRANT SELECT, LOCK TABLES, SHOW VIEW ON videogiochi.* TO 'videogiochi_backup'@'localhost';
```

### 3.6.2 Crittografia Dati Sensibili

**Configurazione SSL**:
```sql
-- Forzare connessioni SSL
ALTER USER 'videogiochi_app'@'%' REQUIRE SSL;

-- Configurazione crittografia a riposo
-- (configurazione my.cnf)
[mysqld]
innodb_encrypt_tables=ON
innodb_encrypt_log=ON
innodb_encryption_threads=4
```

### 3.6.3 Audit e Monitoring

**Configurazione Audit Log**:
```sql
-- Abilitazione audit plugin
INSTALL PLUGIN audit_log SONAME 'audit_log.so';

-- Configurazione audit
SET GLOBAL audit_log_policy = ALL;
SET GLOBAL audit_log_format = JSON;
SET GLOBAL audit_log_file = 'audit.log';
```

## 3.7 Backup e Recovery

### 3.7.1 Strategia di Backup

**Script Backup Completo**:
```bash
#!/bin/bash
# backup_full.sh

DATE=$(date +"%Y%m%d_%H%M%S")
BACKUP_DIR="/backups/mysql"
DB_NAME="videogiochi"

# Backup completo
mysqldump --single-transaction --routines --triggers \
  --user=videogiochi_backup --password=backup_password \
  $DB_NAME > $BACKUP_DIR/full_backup_$DATE.sql

# Compressione
gzip $BACKUP_DIR/full_backup_$DATE.sql

# Rimozione backup vecchi (>30 giorni)
find $BACKUP_DIR -name "full_backup_*.sql.gz" -mtime +30 -delete
```

**Script Backup Incrementale**:
```bash
#!/bin/bash
# backup_incremental.sh

DATE=$(date +"%Y%m%d_%H%M%S")
BACKUP_DIR="/backups/mysql/incremental"

# Backup binary log
mysqlbinlog --read-from-remote-server --host=localhost \
  --user=videogiochi_backup --password=backup_password \
  --raw --result-file=$BACKUP_DIR/binlog_$DATE \
  mysql-bin.000001
```

### 3.7.2 Procedure di Recovery

**Recovery Point-in-Time**:
```bash
#!/bin/bash
# recovery_pit.sh

# 1. Restore da backup completo
mysql -u root -p videogiochi < /backups/mysql/full_backup_20240101_000000.sql

# 2. Apply binary logs fino al punto desiderato
mysqlbinlog --stop-datetime="2024-01-15 10:30:00" \
  /backups/mysql/incremental/binlog_* | mysql -u root -p videogiochi
```

## 3.8 Monitoring e Performance Tuning

### 3.8.1 Metriche Chiave

**Query Performance**:
```sql
-- Abilitazione Performance Schema
SET GLOBAL performance_schema = ON;

-- Query più lente
SELECT 
    DIGEST_TEXT,
    COUNT_STAR,
    AVG_TIMER_WAIT/1000000000 as avg_time_sec,
    MAX_TIMER_WAIT/1000000000 as max_time_sec
FROM performance_schema.events_statements_summary_by_digest
ORDER BY AVG_TIMER_WAIT DESC
LIMIT 10;
```

**Utilizzo Indici**:
```sql
-- Indici non utilizzati
SELECT 
    object_schema,
    object_name,
    index_name
FROM performance_schema.table_io_waits_summary_by_index_usage
WHERE index_name IS NOT NULL
  AND count_star = 0
  AND object_schema = 'videogiochi'
ORDER BY object_schema, object_name;
```

### 3.8.2 Ottimizzazioni Configurazione

**my.cnf Ottimizzato**:
```ini
[mysqld]
# Buffer Pool (70-80% della RAM disponibile)
innodb_buffer_pool_size = 2G
innodb_buffer_pool_instances = 8

# Log Files
innodb_log_file_size = 256M
innodb_log_buffer_size = 16M
innodb_flush_log_at_trx_commit = 2

# Query Cache
query_cache_type = 1
query_cache_size = 256M
query_cache_limit = 2M

# Connections
max_connections = 200
max_connect_errors = 10000

# Table Cache
table_open_cache = 4000
table_definition_cache = 2000

# Temporary Tables
tmp_table_size = 256M
max_heap_table_size = 256M

# MyISAM (se utilizzato)
key_buffer_size = 128M

# Binary Logging
log_bin = mysql-bin
binlog_format = ROW
expire_logs_days = 7

# Slow Query Log
slow_query_log = 1
slow_query_log_file = /var/log/mysql/slow.log
long_query_time = 2
log_queries_not_using_indexes = 1
```

Questo design del database fornisce una base solida per l'applicazione, bilanciando normalizzazione, performance e scalabilità, con particolare attenzione alla sicurezza e alla manutenibilità.