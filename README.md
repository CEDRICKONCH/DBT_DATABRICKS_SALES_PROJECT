# 📊 dbt Databricks Sales Project

> **An enterprise-grade data transformation pipeline demonstrating advanced dbt, Jinja templating, YAML configuration, and multi-environment deployment strategies**

<div align="center">

[![dbt](https://img.shields.io/badge/dbt-core%201.11.7-orange?style=for-the-badge)](https://www.getdbt.com/)
[![Databricks](https://img.shields.io/badge/Databricks-1.10.9-blue?style=for-the-badge)](https://www.databricks.com/)
[![Python](https://img.shields.io/badge/Python-3.11+-green?style=for-the-badge)](https://www.python.org/)
[![License](https://img.shields.io/badge/License-Portfolio%20Project-brightgreen?style=for-the-badge)](#)

**[🎯 Compétences](#compétences) • [🏗️ Architecture](#architecture) • [🎓 Apprentissages](#apprentissages-clés)**

</div>

---

## 📌 Vue d'ensemble

Ce projet démontre une **maîtrise approfondie** des technologies modernes de data engineering:

- ✨ **Jinja Templating** avancé pour génération dynamique de SQL
- 🔧 **YAML Configuration** pour orchestration et contracts de données
- ✅ **Data Quality Testing** complets et personnalisés
- 🚀 **Multi-Environment Deployment** (Dev → Staging → Production)
- 📈 **Architecture Medallion** (Bronze-Silver-Gold) professionnelle

---

## 💼 Compétences Démontrées

### 🎯 **1. Jinja Templating Avancé**

#### **A. Macros Personnalisées**

**Fichier:** `macros/multiply.sql`
```jinja
{%- macro multiply(col1, col2) -%}
    {{ col1 }} * {{ col2 }}
{%- endmacro -%}
```

**Utilisation réelle:**
```sql
-- models/silver/silver_salesinfo.sql
SELECT
    sales_id,
    {{ multiply('quantity', 'unit_price') }} AS calculated_gross_amount,
    gross_amount,
    payment_method
FROM {{ ref('bronze_sales') }}
```

**Compétence démontrée:**
- Création de macros réutilisables pour éviter la duplication de code
- Application des macros dans plusieurs modèles
- Gestion de la logique métier centralisée

---

#### **B. Schémas Dynamiques avec Conditionnels**

**Fichier:** `macros/generate_schema.sql`
```jinja
{% macro generate_schema_name(custom_schema_name, node) -%}
    {%- set default_schema = target.schema -%}
    {%- if custom_schema_name is none -%}
        {{ default_schema }}
    {%- else -%}
        {{ custom_schema_name | trim }}
    {%- endif -%}
{% endmacro %}
```

**Compétence démontrée:**
- Logique conditionnelle en Jinja (`{% if %}` statements)
- Utilisation de variables Jinja (`{%- set -%}`)
- Filtres de template (`| trim`)
- Gestion dynamique des schémas selon l'environnement

---

#### **C. Références Dynamiques et Dépendances**

**Exemple - Silver Layer:**
```sql
-- models/silver/silver_salesinfo.sql
WITH sales AS (
    SELECT * FROM {{ ref('bronze_sales') }}  -- Dépendance explicite
),
products AS (
    SELECT * FROM {{ ref('bronze_product') }}  -- DAG management
),
customer AS (
    SELECT * FROM {{ ref('bronze_customer') }}
)
```

**Compétence démontrée:**
- Utilisation de `{{ ref() }}` pour dépendances explicites
- Construction du DAG (Directed Acyclic Graph)
- Gestion des dépendances croisées entre modèles
- Réduction des risques de dépendances circulaires

---

#### **D. Sources et Contrats de Données**

**Fichier:** `models/source/source.yml`
```jinja
sources:
  - name: source
    database: '{{ target.catalog }}'  # Jinja variable
    schema: source
    tables:
      - name: fact_sales
      - name: dim_customer
```

**Compétence démontrée:**
- Utilisation de variables Jinja dans YAML
- Template variables pour multi-catalog support
- Source abstraction pour flexibilité

---

### 🔧 **2. YAML Configuration - Orchestration Complète**

#### **A. Configuration Projet (dbt_project.yml)**

```yaml
name: 'sales_project_dbt_databricks'
version: '1.0.0'
profile: 'sales_project_dbt_databricks'

# Organisation des chemins
model-paths: ["models"]
test-paths: ["tests"]
macro-paths: ["macros"]
analysis-paths: ["analyses"]
snapshot-paths: ["snapshots"]

# Configuration par couche
models:
  sales_project_dbt_databricks:
    bronze:
      +materialized: table
      +schema: bronze
    silver:
      +materialized: table
      +schema: silver
    gold:
      +materialized: table
      +schema: gold
```

**Compétence démontrée:**
- Configuration structurée d'un projet dbt
- Organisation des paths pour clarity
- Configuration multi-couche cohérente
- Spécification de matérialisation par layer

---

#### **B. Contrats de Données (source.yml)**

```yaml
sources:
  - name: source
    database: '{{ target.catalog }}'
    schema: source
    tables:
      - name: fact_sales
      - name: fact_returns
      - name: dim_customer
      - name: dim_product
      - name: dim_date
      - name: dim_store
      - name: items
```

**Compétence démontrée:**
- Définition explicite des sources de données
- Data contracts documentation
- Spécification du catalogue et schéma
- Coupling loose avec les sources externes

---

#### **C. Propriétés & Data Quality (properties.yml)**

```yaml
version: 2

models:
  - name: bronze_date
    config:
      materialized: view
      schema: bronze

  - name: bronze_sales
    columns:
      - name: sales_id
        data_tests:
          - unique
          - not_null
      - name: gross_amount
        data_tests:
          - generic_non_negative

  - name: bronze_store
    columns:
      - name: store_sk
        data_tests:
          - unique
          - not_null
      - name: store_name
        data_tests:
          - accepted_values:
              values: ['MegaMart Manhattan', 'MegaMart Brooklyn', ...]
              config:
                severity: warn
```

**Compétence démontrée:**
- Tests génériques (unique, not_null, accepted_values)
- Configuration de tests avec arguments
- Severity levels pour production resilience
- Column-level data quality specification
- YAML structure pour tests maintenables

---

### ✅ **3. Data Quality Testing - Stratégie Complète**

#### **A. Tests Génériques Intégrés**

| Test | Colonne | Couche | Purpose |
|------|---------|--------|---------|
| `unique` | bronze_sales.sales_id | Bronze | Pas de doublons en source |
| `not_null` | bronze_sales.sales_id | Bronze | Clés primaires obligatoires |
| `unique` | bronze_store.store_sk | Bronze | Dimension unique |
| `accepted_values` | bronze_store.store_name | Bronze | Whitelist de valeurs |
| `generic_non_negative` | bronze_sales.gross_amount | Bronze | Validation métier custom |

**Exemple de configuration:**
```yaml
- name: bronze_sales
  columns:
    - name: sales_id
      data_tests:
        - unique          # ← Détecte les doublons
        - not_null        # ← Détecte les valeurs manquantes
    - name: gross_amount
      data_tests:
        - generic_non_negative  # ← Custom test
```

---

#### **B. Tests Personnalisés (Custom Macros)**

**Fichier:** `tests/generic/generic_non_negative.sql`
```sql
{% test generic_non_negative(model, column_name) %}

SELECT
    *
FROM
    {{ model }}
WHERE
    {{ column_name }} < 0

{% endtest %}
```

**Utilisation:**
```yaml
models:
  - name: bronze_sales
    columns:
      - name: gross_amount
        data_tests:
          - generic_non_negative
      - name: net_amount
        data_tests:
          - generic_non_negative
```

**Compétence démontrée:**
- Création de tests génériques réutilisables
- Test de logique métier (montants > 0)
- Paramétrage du test avec `column_name`
- Application à plusieurs modèles/colonnes

---

#### **C. Tests Spécifiques aux Modèles**

**Fichier:** `tests/non_negative_test.sql`
```sql
SELECT
    *
FROM
    {{ ref('bronze_sales') }}
WHERE
    gross_amount < 0 AND net_amount < 0
```

**Compétence démontrée:**
- Tests d'intégrité croisée (multiple colonnes)
- Logique métier complexe en tests
- Isolation des tests critiques
- Détection d'anomalies combinées

---

#### **D. Stratégie d'Exécution des Tests**

```bash
# Exécuter tous les tests
dbt test

# Tests par modèle spécifique
dbt test -s bronze_sales

# Tests avec détail des échecs
dbt test --store-failures

# Tests avant déploiement
dbt test --select state:modified+
```

**Compétence démontrée:**
- Sélection intelligente des tests
- Failure reporting pour debugging
- Integration dans pipelines CI/CD
- Test de changements seulement (state:modified+)

---

### 🚀 **4. Multi-Environment Deployment Strategy**

#### **A. Configuration des Targets (profiles.yml)**

**Fichier:** `profiles.yml.example`

```yaml
sales_project_dbt_databricks:
  outputs:
    # 🟢 DEVELOPMENT - Local development & testing
    dev:
      type: databricks
      catalog: dev_catalog
      schema: dbt_dev
      host: my-workspace.cloud.databricks.com
      http_path: /sql/1.0/warehouses/dev_warehouse_id
      token: dev_pat_token
      threads: 4
      timeout_seconds: 300
    # 🔴 PRODUCTION - Live data transformations
    prod:
      type: databricks
      catalog: prod_catalog
      schema: dbt_prod
      host: my-workspace.cloud.databricks.com
      http_path: /sql/1.0/warehouses/prod_warehouse_id
      token: prod_pat_token
      threads: 8      # Higher concurrency for production
      timeout_seconds: 600

  # Target par défaut
  target: dev
```

**Compétence démontrée:**
- Configuration multi-environnement
- Isolation par catalog et schema
- Gestion des credentials sécurisée (tokens)
- Tuning per-environment (threads, timeout)

---

#### **B. Flux de Déploiement Dev → Prod**

```
┌─────────────────────┐
│   LOCAL DEVELOPMENT │
│   Target: dev       │
│   - dbt run         │
│   - dbt test        │
│   - Validation      │
└──────────┬──────────┘
           │ git push
           ▼
┌─────────────────────┐
│  STAGING/TESTING    │
│  Target: staging    │
│  - Full validation  │
│  - Integration test │
│  - UAT              │
└──────────┬──────────┘
           │ PR approved
           ▼
┌─────────────────────┐
│    PRODUCTION       │
│  Target: prod       │
│  - Full pipeline    │
│  - Monitor & Alert  │
│  - Audit trail      │
└─────────────────────┘
```

---

#### **C. Sélection de Target**

```bash
# ✅ Utiliser développement (défaut)
dbt run --target dev

# Exécuter sur staging
dbt run --target staging

# Déployer en production
dbt run --target prod

# Ou via variable d'environnement
export DBT_TARGET=prod
dbt run
```

**Compétence démontrée:**
- Switching facile entre environnements
- CLI flags pour sélection dynamique
- Variables d'environnement pour CI/CD
- Reproducibilité cross-environment

---

#### **D. Gestion des Credentials (Securité)**

**Best Practices implémentées:**

```yaml
# ✅ Fichier example fourni
profiles.yml.example  → Versionné dans Git

# ❌ Fichier réel ignoré
profiles.yml          → Dans .gitignore
~/.dbt/profiles.yml   → Unique per developer
```

**.gitignore:**
```
# Credentials & sensitive files
profiles.yml
.env
*.secret
dbt_packages/
target/
```

**Compétence démontrée:**
- Sécurisation des credentials
- Version control best practices
- Separation config/secrets
- Per-environment token management

---

### 📈 **5. Architecture Medallion - Couches de Données**

#### **Bronze Layer** 🥉 - Raw Data Ingestion

```sql
-- models/bronze/bronze_sales.sql
{{ config(materialized='view')}}

SELECT *
FROM {{ source('source', 'fact_sales') }}
```

**Caractéristiques:**
- Minimal transformations
- Direct mapping from sources
- Views for accessibility
- Source abstractions

**Compétence:** Source definition, view materialization

---

#### **Silver Layer** 🥈 - Cleaned & Transformed

```sql
-- models/silver/silver_salesinfo.sql
WITH sales AS (
    SELECT
        sales_id,
        product_sk,
        customer_sk,
        {{ multiply('quantity', 'unit_price') }} AS calculated_gross_amount,
        gross_amount,
        payment_method
    FROM {{ ref('bronze_sales') }}
),

products AS (
    SELECT product_sk, category
    FROM {{ ref('bronze_product') }}
),

customer AS (
    SELECT customer_sk, gender
    FROM {{ ref('bronze_customer') }}
),

joined_query AS (
    SELECT
        sales.sales_id,
        sales.gross_amount,
        sales.payment_method,
        products.category,
        customer.gender
    FROM sales
    JOIN products ON sales.product_sk = products.product_sk
    JOIN customer ON sales.customer_sk = customer.customer_sk
)

SELECT
    category,
    gender,
    SUM(gross_amount) AS total_sales
FROM joined_query
GROUP BY category, gender
ORDER BY total_sales DESC
```

**Compétences démontrées:**
- Multi-model joins (3-way join)
- Macro utilization (multiply)
- CTEs pour clarity
- Aggregation & grouping
- Business logic implementation

---

#### **Gold Layer** 🥇 - Business Ready Analytics

```sql
-- models/gold/source_gold_items.sql
WITH dedup_query AS (
    SELECT
        *,
        ROW_NUMBER() OVER (PARTITION BY id ORDER BY updated DESC) AS deduplication_id
    FROM {{ source('source', 'items') }}
)

SELECT
    id,
    name,
    category,
    updated
FROM dedup_query
WHERE deduplication_id = 1
```

**Compétences démontrées:**
- Window functions (ROW_NUMBER)
- Deduplication logic
- SCD Type 2 pattern
- Final aggregation layer

---

## 🏗️ Architecture Complète

```
sales_project_dbt_databricks/
│
├── 📄 dbt_project.yml              ← Configuration principale
│
├── 📁 models/                      ← Transformations SQL
│   ├── source/
│   │   └── source.yml              ← Data contracts
│   ├── bronze/
│   │   ├── bronze_*.sql            ← 6 modèles raw
│   │   └── properties.yml          ← Tests & configs
│   ├── silver/
│   │   └── silver_salesinfo.sql    ← Transformed & joined
│   └── gold/
│       └── source_gold_items.sql   ← Business-ready
│
├── 📁 macros/                      ← Jinja reusable logic
│   ├── multiply.sql                ← Custom calculation
│   └── generate_schema.sql         ← Dynamic schemas
│
├── 📁 tests/                       ← Data quality
│   ├── generic/
│   │   └── generic_non_negative.sql ← Reusable test
│   └── non_negative_test.sql       ← Specific test
│
├── 📁 analyses/                    ← Ad-hoc queries
│   └── target_variables.sql
│
└── 📁 snapshots/                   ← SCD tracking
    └── gold_items.yml
```

---

## 🎓 Apprentissages Clés

### **Jinja Templating**
- ✅ Macros réutilisables pour éviter la duplication
- ✅ Template variables et conditionnels
- ✅ Filtres de template pour transformation de strings
- ✅ Reference management avec `ref()` et `source()`

### **YAML Configuration**
- ✅ Multi-couche architecture definition
- ✅ Data contracts dans `source.yml`
- ✅ Data quality rules déclaratives
- ✅ Configuration hiérarchique par couche

### **Data Quality Testing**
- ✅ Tests génériques standardisés
- ✅ Tests personnalisés métier-spécifiques
- ✅ Sélection intelligente des tests
- ✅ Failure reporting pour debugging

### **Multi-Environment Deployment**
- ✅ Profiles.yml pour target management
- ✅ Dev/Staging/Prod séparation
- ✅ Credentials management sécurisé
- ✅ Environment-specific tuning

### **Architecture Medallion**
- ✅ Bronze: Raw data staging
- ✅ Silver: Cleaned & validated transformations
- ✅ Gold: Business-ready aggregations

---

## 🚀 Installation & Utilisation

### **Prérequis**
```bash
Python 3.11+
Databricks workspace
dbt-core >= 1.11.7
dbt-databricks >= 1.10.9
```

### **Setup**
```bash
# 1. Clone
git clone <repo-url>
cd dbt_databricks_sales-project

# 2. Virtual env
python -m venv .venv
source .venv/bin/activate

# 3. Install
pip install -e ".[dev]"

# 4. Configure
cp profiles.yml.example ~/.dbt/profiles.yml
# Edit with your Databricks credentials

# 5. Verify
cd sales_project_dbt_databricks
dbt debug
```

### **Exécution**
```bash
# Development
dbt run --target dev
dbt test --target dev

# Staging
dbt run --target staging
dbt test --target staging

# Production (with caution!)
dbt run --target prod
dbt test --target prod
```

---

## 📊 Statistiques du Projet

| Métrique | Valeur |
|----------|--------|
| **Modèles** | 8 (6 bronze, 1 silver, 1 gold) |
| **Tests** | 5+ (génériques + personnalisés) |
| **Macros** | 2 (multiply, generate_schema) |
| **Sources** | 7 tables |
| **Couches** | 3 (Medallion) |
| **Environments** | 3 (dev, staging, prod) |

---

## 📝 License

Portfolio project for demonstration purposes.

---

<div align="center">

**Construit avec ❤️ en utilisant dbt + Databricks**

[⬆ Back to top](#-dbt-databricks-sales-project)

</div>
