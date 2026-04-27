# Career Recommendation App Database Documentation

## ERD
Field and table names conform to naming standards (plural snake_case tables, snake_case columns).

```mermaid
erDiagram
    users {
        bigint id PK \"AUTO_INCREMENT\"
        varchar(50) first_name
        varchar(50) last_name
        varchar(100) email \"UNIQUE NOT NULL\"
        varchar(255) password \"NOT NULL\"
        enum role \"NOT NULL\"
        boolean active \"DEFAULT true\"
        timestamp created_at \"DEFAULT CURRENT_TIMESTAMP\"
    }
    subjects {
        bigint subject_id PK \"AUTO_INCREMENT\"
        varchar(100) subject_name \"UNIQUE NOT NULL\"
        text description
    }
    marks {
        bigint mark_id PK \"AUTO_INCREMENT\"
        bigint user_id FK
        bigint subject_id FK
        decimal(5,2) mark
        timestamp date_entered \"DEFAULT CURRENT_TIMESTAMP\"
    }
    aps_scores {
        bigint aps_id PK \"AUTO_INCREMENT\"
        bigint user_id FK \"NOT NULL\"
        decimal(4,1) aps_score \"NOT NULL\"
        timestamp calculated_at \"DEFAULT CURRENT_TIMESTAMP\"
    }
    institutions {
        bigint institution_id PK \"AUTO_INCREMENT\"
        varchar(200) name \"NOT NULL\"
        text description
    }
    programs {
        bigint program_id PK \"AUTO_INCREMENT\"
        varchar(100) program_name \"NOT NULL\"
        text description
        int min_aps
        bigint institution_id FK
    }

    users ||--o{ marks : \"has\"
    subjects ||--o{ marks : \"has\"
    users ||--|| aps_scores : \"has (1:1)\"
    institutions ||--o{ programs : \"offers\"
```

## Database Script (MySQL for career_app database)

```sql
-- Create database if not exists
CREATE DATABASE IF NOT EXISTS career_app;
USE career_app;

-- Table: users
CREATE TABLE users (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    email VARCHAR(100) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    role ENUM('LEARNER', 'ADMIN') NOT NULL,
    active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Table: subjects
CREATE TABLE subjects (
    subject_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    subject_name VARCHAR(100) NOT NULL UNIQUE,
    description TEXT
);

-- Table: institutions
CREATE TABLE institutions (
    institution_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(200) NOT NULL,
    description TEXT
);

-- Table: programs
CREATE TABLE programs (
    program_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    program_name VARCHAR(100) NOT NULL,
    description TEXT,
    min_aps INT,
    institution_id BIGINT,
    FOREIGN KEY (institution_id) REFERENCES institutions(institution_id) ON DELETE SET NULL
);

-- Table: marks
CREATE TABLE marks (
    mark_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT,
    subject_id BIGINT,
    mark DECIMAL(5,2),
    date_entered TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (subject_id) REFERENCES subjects(subject_id) ON DELETE CASCADE
);

-- Table: aps_scores
CREATE TABLE aps_scores (
    aps_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT NOT NULL,
    aps_score DECIMAL(4,1) NOT NULL,
    calculated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    UNIQUE KEY unique_user_aps (user_id)
);
```

## Example Tables Populated (5+ records each, relationships maintained)

```sql
-- Insert users (5 records)
INSERT INTO users (first_name, last_name, email, password, role) VALUES
('John', 'Doe', 'john.learner@example.com', 'hashed_pass1', 'LEARNER'),
('Jane', 'Smith', 'jane.learner@example.com', 'hashed_pass2', 'LEARNER'),
('Admin', 'User', 'admin@example.com', 'hashed_admin', 'ADMIN'),
('Bob', 'Johnson', 'bob.learner@example.com', 'hashed_pass3', 'LEARNER'),
('Alice', 'Brown', 'alice.learner@example.com', 'hashed_pass4', 'LEARNER'),
('Charlie', 'Davis', 'charlie.learner@example.com', 'hashed_pass5', 'LEARNER');

-- Insert subjects (6 records)
INSERT INTO subjects (subject_name, description) VALUES
('English', 'Home Language'),
('Mathematics', 'Core Mathematics'),
('Physical Sciences', 'Physics and Chemistry'),
('Life Sciences', 'Biology'),
('Accounting', 'Financial Accounting'),
('History', 'South African History');

-- Insert institutions (3 records)
INSERT INTO institutions (name, description) VALUES
('University of Cape Town', 'Top university in South Africa'),
('Wits University', 'University of the Witwatersrand'),
('University of Pretoria', 'Leading research university');

-- Insert programs (6 records, linked to institutions)
INSERT INTO programs (program_name, description, min_aps, institution_id) VALUES
('BSc Computer Science', 'Bachelor of Science in Computer Science', 35, 1),
('BA Law', 'Bachelor of Arts in Law', 30, 1),
('MBChB Medicine', 'Bachelor of Medicine and Surgery', 40, 2),
('BEng Mechanical', 'Bachelor of Engineering Mechanical', 38, 2),
('BCom Accounting', 'Bachelor of Commerce Accounting', 32, 3),
('BEd Foundation Phase', 'Bachelor of Education Foundation', 28, 3);

-- Insert marks for users 1-3, subjects 1-3 (10+ records)
INSERT INTO marks (user_id, subject_id, mark) VALUES
(1, 1, 78.50), (1, 2, 65.00), (1, 3, 72.25), (1, 4, 80.00),
(2, 1, 85.75), (2, 2, 70.50), (2, 3, 68.00),
(3, 1, 92.00), (3, 2, 88.25), (3, 3, 90.50);

-- Insert aps_scores for users 1-3 (calculated roughly from marks)
INSERT INTO aps_scores (user_id, aps_score) VALUES
(1, 28.5),
(2, 30.2),
(3, 35.8);
```

**Note:** Relationships created as specified: FK constraints with CASCADE/SET NULL where appropriate. Sample data ensures referential integrity (e.g., marks reference existing users/subjects, aps_scores unique per user, programs reference institutions). Passwords shown as 'hashed_pass*' for demo; use BCrypt in production.

To generate PDF:
1. Install pandoc if not present: `winget install JohnMacFarlane.pandoc`
2. Run: `pandoc career_app_database_documentation.md -o career_app_database.pdf --pdf-engine=weasyprint` (or use browser: open MD in VSCode/Markdown preview, print to PDF).

