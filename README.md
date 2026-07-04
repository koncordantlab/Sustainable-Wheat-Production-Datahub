# Sustainable-Wheat-Production-Datahub
## From Field to Graph: GRIP Datahub, a Semantically Rich Graph-Based Framework for Sustainable Wheat Production

This repository contains the ontology schemas and example datasets for the **GRIP Datahub**, a semantically rich knowledge graph for sustainable wheat production.
The datahub integrates nutrient management, disease management, weather, and drought information into a modular RDF/OWL framework.

---

## Repository Structure
```
GRIP_DATAHUB/
├── competency-questions/
│   └── COMPETENCY_QUESTIONS.md
├── data/
│   ├── disease-data/
│   │   ├── disease_data_observations.ttl
│   │   └── disease_data_reference.ttl
│   ├── drought-data/
│   │   ├── drought_data_observations.ttl
│   │   └── drought_data_reference.ttl
│   ├── nutrient-data/
│   │   ├── nm_data_observations.ttl
│   │   └── nm_data_reference.ttl
│   └── weather-data/
│       ├── weather_data_observations.nt
│       └── weather_data_reference.ttl
├── ontology/
│   ├── disease-management/
│   │   ├── modules/
│   │   │   ├── disease.ttl
│   │   │   ├── management.ttl
│   │   │   └── pathogen.ttl
│   │   ├── dm_master.ttl
│   │   └── dm_merged.ttl
│   ├── drought-monitor/
│   │   └── drought_monitor.ttl
│   ├── nutrient-management/
│   │   ├── modules/
│   │   │   ├── nm_environment_revised.ttl
│   │   │   ├── nm_fertilization_revised.ttl
│   │   │   ├── nm_nitrogen_revised.ttl
│   │   │   ├── nm_nutrient_revised.ttl
│   │   │   ├── nm_observation.ttl
│   │   │   └── nm_soil_available_nitrogen_revised.ttl
│   │   ├── nm_master.ttl
│   │   └── nm_merged.ttl
│   └── weather-station/
│       └── weather_station.ttl
├── .gitignore
└── README.md
```
---

## Ontology
The `ontology/` directory holds the schema layer of the GRIP Datahub. It defines the vocabulary and upper-level concepts for four domains: nutrient management, disease management, weather, and drought. These files describe what each domain contains. They do not store the trial records themselves, which live under `data/`.

Each domain follows the same modular pattern. Small, focused modules define the concepts, and two assembly files bring them together:

- A **master** file (`nm_master.ttl`, `dm_master.ttl`) uses `owl:imports` to pull the modules together for editing and reasoning in tools like Protege.
- A **merged** file (`nm_merged.ttl`, `dm_merged.ttl`) flattens the modules into one file and removes the duplicate stubs. This is the file to load into GraphDB, because it avoids `owl:imports` resolution issues.

Every entity carries an `rdfs:label` and `rdfs:comment`, and every property declares its domain and range.

### Nutrient Management Module
Located in `ontology/nutrient-management/`. Six modules model the nutrient hierarchy and the nitrogen-centered concepts that drive nutrient dynamics in wheat.

- **nm_nutrient_revised.ttl**: High-level nutrient classification, covering non-minerals, primary and secondary minerals, micronutrients, and other nutrient forms. Also holds crop species, cultivar, crop response, and nutrient use efficiency.
- **nm_nitrogen_revised.ttl**: Nitrogen demand, supply, and balance, the nitrogen cycle and plant nitrogen dynamics, and the transformation and loss pathways.
- **nm_soil_available_nitrogen_revised.ttl**: Soil nitrogen pools, the processes that supply or deplete them, and the associated measurement properties.
- **nm_environment_revised.ttl**: Environmental and soil factors that influence nutrient behavior, including the 12 USDA soil texture classes as individuals, temperature, moisture, and pH.
- **nm_fertilization_revised.ttl**: Fertilizer sources and their guaranteed analysis, application rates and timings, enhanced efficiency fertilizers, and fertilization management practices.
- **nm_observation.ttl**: The plot observation event that binds a cultivar, site-year, trial, and treatments to measured responses such as yield, protein, and test weight. Also holds maturity group, nitrogen timing split, and previous crop with its category link.
- **nm_master.ttl / nm_merged.ttl**: The master imports the six modules under the `nutrient-management` root. The merged file is the deduplicated single file for GraphDB.

### Disease Management Module
Located in `ontology/disease-management/`. Three modules model pathogens, the diseases they cause, and integrated management.

- **pathogen.ttl**: Pathogen taxonomy (fungal, bacterial, viral) with specific pathogen individuals, plus inoculum reservoirs, dispersal mechanisms, and infection conditions.
- **disease.ttl**: Wheat diseases as individuals, each linked to its causal organism through `caused_by`. Also models the disease triangle (host, pathogen, environment), disease assessment methods (incidence, severity, rating), and per-cultivar per-disease resistance ratings.
- **management.ttl**: The three control pillars, namely chemical control (fungicides with FRAC groups, active ingredients, and application timings), cultural control practices, and host resistance types.
- **dm_master.ttl / dm_merged.ttl**: Assembly files. They bridge to nutrient management through shared stubs such as environment and cultivar. Load the merged file into GraphDB.

### Weather Station Ontology
Located in `ontology/weather-station/weather_station.ttl`. Describes weather station metadata and the structure of daily weather observations such as temperature, precipitation, and snowfall. It provides the schema for the HPRCC weather records under `data/weather-data/`.

### Drought Monitor Ontology
Located in `ontology/drought-monitor/drought_monitor.ttl`. Describes drought observations at state and county level, aligned with the U.S. Drought Monitor categories (D0 to D4). It provides the schema for the drought records under `data/drought-data/`.

---

## Data
The `data/` directory holds the instance-level records that populate the ontology with real observations from field trials, weather stations, and drought monitoring. Each domain is split into two files:

- A **reference** file (`*_reference.ttl`) holds the shared individuals a domain refers to, such as cultivars, locations, and site-years.
- An **observations** file (`*_observations.ttl` or `.nt`) holds the measured records themselves.

### Nutrient Trial Data
`data/nutrient-data/`. Field trial records for nitrogen treatments, grain protein, test weight, and yield components, drawn from the Kansas wheat variety and fertility trials.

- **nm_data_reference.ttl**: Reference individuals for the nutrient trials, including cultivars, locations, and site-years.
- **nm_data_observations.ttl**: The plot-level observation records with their measured values.

### Disease Trial Data
`data/disease-data/`. Field trial records documenting fungicide applications, disease severity and incidence assessments, DON readings, and variety responses for Fusarium head blight and stripe rust.

- **disease_data_reference.ttl**: Reference individuals for the disease trials.
- **disease_data_observations.ttl**: The fungicide application and disease assessment records.

### Weather Data
`data/weather-data/`. Daily weather observations from HPRCC stations in Kansas.

- **weather_data_reference.ttl**: Station reference individuals.
- **weather_data_observations.nt**: The daily observation records, stored as N-Triples because of their volume.

### Drought Data
`data/drought-data/`. State and county drought category observations aligned with the U.S. Drought Monitor.

- **drought_data_reference.ttl**: State and county reference individuals.
- **drought_data_observations.ttl**: The dated drought category records.

---

## Competency Questions
The `competency-questions/` directory holds `COMPETENCY_QUESTIONS.md`, a set of 40 ready-to-run SPARQL queries. They validate the ontology at the schema level and answer real questions against the loaded trial data. Each query is self-contained and includes its prefixes, and the file lists which graphs to load for each group of queries.