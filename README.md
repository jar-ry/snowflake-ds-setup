# Snowflake Data Science Setup - Retail CLV Regression Demo

## Disclaimer

**All opinions, code, and implementation details in this repository are my own personal views and do not represent the views, policies, or recommendations of my employer or any organisation I am affiliated with.**

This project is provided for educational and demonstration purposes only.

## Overview

This is the setup repository for a four-part blog series on building ML pipelines on Snowflake. It creates the Snowflake environment and synthetic data used by all downstream implementations.

| Downstream Repo | Blog | Approach |
|-----------------|------|----------|
| [snowflake-ds-01-notebooks](https://github.com/jar-ry/snowflake-ds-01-notebooks) | Snowflake Notebooks | Interactive development in Snowflake UI |
| [snowflake-ds-02-ml-jobs-notebook](https://github.com/jar-ry/snowflake-ds-02-ml-jobs-notebook) | ML Jobs with `@remote` | Local IDE + remote compute |
| [snowflake-ds-03-ml-jobs-framework](https://github.com/jar-ry/snowflake-ds-03-ml-jobs-framework) | ML Jobs Framework | Modular Python + `submit_directory` |

## Prerequisites

- [Miniconda](https://docs.conda.io/en/latest/miniconda.html) or [Anaconda](https://www.anaconda.com/download)
- Snowflake account with ACCOUNTADMIN privileges (for initial setup), **or** a custom role with the grants listed below
- Git

### Required Privileges (if not using ACCOUNTADMIN)

The setup notebook needs a role with the following grants. Replace `<DEMO_ROLE>`, `<DB>`, `<SCHEMA>`, and `<WH>` with your values.

**Account-level:**

```sql
GRANT CREATE ROLE ON ACCOUNT TO ROLE <YOUR_SETUP_ROLE>;
GRANT CREATE WAREHOUSE ON ACCOUNT TO ROLE <YOUR_SETUP_ROLE>;
GRANT CREATE DATABASE ON ACCOUNT TO ROLE <YOUR_SETUP_ROLE>;
GRANT CREATE COMPUTE POOL ON ACCOUNT TO ROLE <YOUR_SETUP_ROLE>;
GRANT CREATE INTEGRATION ON ACCOUNT TO ROLE <YOUR_SETUP_ROLE>;
GRANT CREATE NETWORK RULE ON ACCOUNT TO ROLE <YOUR_SETUP_ROLE>;
GRANT EXECUTE MANAGED TASK ON ACCOUNT TO ROLE <YOUR_SETUP_ROLE>;
GRANT EXECUTE TASK ON ACCOUNT TO ROLE <YOUR_SETUP_ROLE>;
```

**After database/warehouse creation (granted to the demo role by the setup notebook):**

```sql
-- Warehouse
GRANT ALL ON WAREHOUSE <WH> TO ROLE <DEMO_ROLE>;

-- Database & schemas
GRANT ALL ON DATABASE <DB> TO ROLE <DEMO_ROLE>;
GRANT ALL ON ALL SCHEMAS IN DATABASE <DB> TO ROLE <DEMO_ROLE>;
GRANT ALL ON FUTURE SCHEMAS IN DATABASE <DB> TO ROLE <DEMO_ROLE>;
GRANT CREATE SCHEMA ON DATABASE <DB> TO ROLE <DEMO_ROLE>;

-- Schema objects
GRANT USAGE ON SCHEMA <DB>.<SCHEMA> TO ROLE <DEMO_ROLE>;
GRANT CREATE TABLE ON SCHEMA <DB>.<SCHEMA> TO ROLE <DEMO_ROLE>;
GRANT CREATE VIEW ON SCHEMA <DB>.<SCHEMA> TO ROLE <DEMO_ROLE>;
GRANT CREATE TAG ON SCHEMA <DB>.<SCHEMA> TO ROLE <DEMO_ROLE>;
GRANT CREATE DYNAMIC TABLE ON SCHEMA <DB>.<SCHEMA> TO ROLE <DEMO_ROLE>;

-- Object permissions (existing + future)
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA <DB>.<SCHEMA> TO ROLE <DEMO_ROLE>;
GRANT SELECT, INSERT, UPDATE, DELETE ON FUTURE TABLES IN SCHEMA <DB>.<SCHEMA> TO ROLE <DEMO_ROLE>;
GRANT SELECT, REFERENCES ON ALL VIEWS IN SCHEMA <DB>.<SCHEMA> TO ROLE <DEMO_ROLE>;
GRANT SELECT, REFERENCES ON FUTURE VIEWS IN SCHEMA <DB>.<SCHEMA> TO ROLE <DEMO_ROLE>;
GRANT SELECT, MONITOR ON ALL DYNAMIC TABLES IN SCHEMA <DB>.<SCHEMA> TO ROLE <DEMO_ROLE>;
GRANT SELECT, MONITOR ON FUTURE DYNAMIC TABLES IN SCHEMA <DB>.<SCHEMA> TO ROLE <DEMO_ROLE>;

-- Tasks
GRANT EXECUTE MANAGED TASK ON ACCOUNT TO ROLE <DEMO_ROLE>;
GRANT EXECUTE TASK ON ACCOUNT TO ROLE <DEMO_ROLE>;
```

## Quick Start

```bash
chmod +x setup_env.sh
./setup_env.sh
conda activate snowflake_ds
```

Create a `connection.json` in the project root:

```json
{
    "account": "your_account_identifier",
    "user": "your_username",
    "password": "your_password",
    "warehouse": "your_warehouse",
    "role": "ACCOUNTADMIN"
}
```

> **Security Note**: `connection.json` is in `.gitignore` and will not be committed.

Then run the setup notebook:

```bash
jupyter lab Step01_Setup.ipynb
```

## What Gets Created

| Object | Name |
|--------|------|
| Database | `RETAIL_REGRESSION_DEMO` |
| Schemas | `DS`, `MODELLING`, `FEATURE_STORE` |
| Warehouse | `RETAIL_REGRESSION_DEMO_WH` |
| Compute Pool | `CLV_MODEL_POOL_CPU` |
| Role | `RETAIL_REGRESSION_DEMO_ROLE` |
| Tables | `CUSTOMERS`, `PURCHASE_BEHAVIOR` (10,000 rows each) |

## Data Model

**CUSTOMERS** -- customer attributes

| Column | Type | Description |
|--------|------|-------------|
| CUSTOMER_ID | INTEGER | Primary key |
| AGE | INTEGER | Customer age (18-75) |
| ANNUAL_INCOME | DECIMAL | Estimated income ($20k-$200k) |
| LOYALTY_TIER | VARCHAR | `low`, `medium`, `high` |
| GENDER | VARCHAR | `male`, `female` |
| STATE | VARCHAR | Australian state/territory |
| TENURE_MONTHS | INTEGER | Months as customer (1-120) |

**PURCHASE_BEHAVIOR** -- transactional features + regression target

| Column | Type | Description |
|--------|------|-------------|
| CUSTOMER_ID | INTEGER | Foreign key |
| AVG_ORDER_VALUE | DECIMAL | Average transaction ($15-$500) |
| PURCHASE_FREQUENCY | DECIMAL | Orders per month (0.1-8) |
| RETURN_RATE | DECIMAL | % items returned (0-30%) |
| LIFETIME_VALUE | DECIMAL | Expected monthly value (regression target) |

## Configuration

Edit the configuration cell in `Step01_Setup.ipynb`:

```python
admin_role = 'ACCOUNTADMIN'
demo_role = 'RETAIL_REGRESSION_DEMO_ROLE'
database_name = 'RETAIL_REGRESSION_DEMO'
schema_name = 'DS'
warehouse_name = 'RETAIL_REGRESSION_DEMO_WH'
warehouse_size = 'SMALL'
num_customers = 10000
```

## Cleanup

To remove Snowflake objects, uncomment and run the cleanup cell at the end of `Step01_Setup.ipynb`.

To remove the conda environment:

```bash
conda deactivate
conda env remove -n snowflake_ds
```
