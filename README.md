# ETL-WetterDaten Pipeline

Dieses Projekt ist eine robuste ETL-Pipeline (Extract, Transform, Load) zur automatisierten Erfassung von Wetterdaten. Sie nutzt Apache Airflow (verwaltet über die Astronomer Astro CLI), um täglich aktuelle Wetterdaten von der Open-Meteo API abzurufen. Die Rohdaten werden anschließend in Python transformiert und zuverlässig in einer PostgreSQL-Datenbank gespeichert. Das gesamte Setup läuft containerisiert mit Docker auf einem Hostinger VPS, was eine einfache Bereitstellung, Reproduzierbarkeit und Skalierbarkeit gewährleistet. Es dient als ideale Grundlage für weiterführende Datenanalysen oder Machine-Learning-Modelle.

---

## Tech Stack

Für die Umsetzung dieser Pipeline wird ein modernes und bewährtes Data-Engineering-Setup verwendet:

- **Workflow Orchestrierung:** Apache Airflow (bereitgestellt via Astronomer Astro CLI & Astro Runtime)
- **Programmiersprache:** Python (unter Verwendung der deklarativen TaskFlow API mit `@task`-Dekoratoren)
- **Datenbank:** PostgreSQL 13 (dient sowohl als Airflow Metadata DB als auch als Zielspeicher für die Wetterdaten)
- **Externe Datenquelle:** Open-Meteo API (integriert über den Airflow `HttpHook`)
- **Containerisierung:** Docker & Docker Compose (gewährleistet isolierte, reproduzierbare und portable Umgebungen)
- **Infrastruktur & Zugriff:** Hostinger VPS (Ubuntu Linux) mit sicherem SSH-Port-Forwarding für den lokalen UI-Zugriff

---

## Architekturdiagramm

Das folgende Diagramm veranschaulicht den Datenfluss und die Infrastruktur der Pipeline:

```mermaid
graph TD
    %% Externe Dienste
    API[("Open-Meteo API<br/>(Externe Datenquelle)")]
    DB[("PostgreSQL Datenbank<br/>Tabelle: weather_data")]

    %% Airflow Tasks
    Extract["📥 Task: extract_weather_data<br/>(HttpHook)"]
    Transform["⚙️ Task: transform_weather_data<br/>(Python Dict)"]
    Load["📤 Task: load_weather_data<br/>(PostgresHook)"]

    %% Airflow Komponenten
    subgraph "Hostinger VPS (Astro Runtime / Docker)"
        Scheduler["Airflow Scheduler"]
        Webserver["Airflow Webserver UI"]
        
        Scheduler -.->|Triggert täglich| Extract
        Webserver -.->|Überwacht & Visualisiert| Scheduler
    end

    %% Datenfluss
    API -->|HTTP GET Request| Extract
    Extract -->|JSON Response| Transform
    Transform -->|Bereinigtes Dictionary| Load
    Load -->|SQL INSERT| DB

    %% Styling
    classDef api fill:#f9f,stroke:#333,stroke-width:2px;
    classDef db fill:#bbf,stroke:#333,stroke-width:2px;
    classDef task fill:#dfd,stroke:#333,stroke-width:1px;
    classDef airflow fill:#ffe4b5,stroke:#333,stroke-width:2px;
    
    class API api;
    class DB db;
    class Extract,Transform,Load task;
    class Scheduler,Webserver airflow;
