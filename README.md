# e2e_ML_in_Snowflake

Snowflake ML demo showcasing Model Monitoring and A/B testing between two Snow ML Models predicting Mortgage Loan Repayments (Optimized for Snowflake Container Runtime)

# Demo Notebook showcasing an end-to-end ML worfklow in Snowflake including the following components
- Use Feature Store to track engineered features
    - Store feature defintions in feature store for reproducible computation of ML features
- Train two SnowML Models
    - Baseline XGboost
    - XGboost with optimal hyper-parameters identified via Snowflake ML distributed HPO methods
- Register both models in Snowflake model registry
    - Explore model registry capabilities such as metadata tracking, inference, and explainability
    - Compare model metrics on train/test set to identify any issues of model performance or overfitting
    - Tag the best performing model version as 'default' version
- Set up Model Monitor to track 1 year of predicted and actual loan repayments
    - Compute performance metrics such a F1, Precision, Recall
    - Inspect model drift (i.e. how much has the average predicted repayment rate changed day-to-day)
    - Compare models side-by-side to understand which model should be used in production
    - Identify and understand data issues
- Track data and model lineage throughout
    - View and understand
      - The origin of the data used for computed features
      - The data used for model training
      - The available model versions being monitored
- Additional components also include
    - Distribtued GPU model training example
    - SPCS deployment for inference
        - [WIP] REST API scoring example 


INSTRUCTIONS:

To run this workload - log into snowflake and run the following SQL code:

```sql
-- Using ACCOUNTADMIN, create a new role for this exercise and grant to applicable users
USE ROLE ACCOUNTADMIN;
SET USERNAME = (SELECT CURRENT_USER());
SELECT $USERNAME;
CREATE OR REPLACE ROLE E2E_SNOW_MLOPS_ROLE;
GRANT ROLE E2E_SNOW_MLOPS_ROLE to USER identifier($USERNAME);


-- Create a warehouse
CREATE WAREHOUSE IF NOT EXISTS E2E_SNOW_MLOPS_WH WITH WAREHOUSE_SIZE='MEDIUM';

-- Create Database (if it doesnt already exist)
CREATE DATABASE IF NOT EXISTS E2E_SNOW_MLOPS_DB;

-- Create Schema (if it doesnt already exist)
CREATE SCHEMA IF NOT EXISTS MLOPS_SCHEMA;


-- Create compute pool
CREATE COMPUTE POOL IF NOT EXISTS MLOPS_COMPUTE_POOL
  MIN_NODES = 1
  MAX_NODES = 1
  INSTANCE_FAMILY = CPU_X64_M;

-- Create the API integration with Github
CREATE OR REPLACE API INTEGRATION GITHUB_INTEGRATION_E2E_SNOW_MLOPS
    api_provider = git_https_api
    api_allowed_prefixes = ('https://github.com/sfc-gh-ebotwick/')
    enabled = true
    comment='Git integration with Snowflake Demo Github Repository.';

-- Create the integration with the Github demo repository
CREATE GIT REPOSITORY GITHUB_REPO_E2E_SNOW_MLOPS
	ORIGIN = 'https://github.com/sfc-gh-ebotwick/e2e_ML_in_Snowflake' 
	API_INTEGRATION = 'GITHUB_INTEGRATION_E2E_SNOW_MLOPS' 
	COMMENT = 'Github Repository ';

-- Fetch most recent files from Github repository
ALTER GIT REPOSITORY GITHUB_REPO_E2E_SNOW_MLOPS FETCH;

-- Copy notebook into snowflake
CREATE OR REPLACE NOTEBOOK E2E_SNOW_MLOPS_DB.MLOPS_SCHEMA.TRAIN_DEPLOY_MONITOR_ML 
FROM '@E2E_SNOW_MLOPS_DB.MLOPS_SCHEMA.GITHUB_REPO_E2E_SNOW_MLOPS/branches/main/' 
MAIN_FILE = 'train_deploy_monitor_ML_in_snowflake.ipynb' QUERY_WAREHOUSE = E2E_SNOW_MLOPS_WH
IDLE_AUTO_SHUTDOWN_TIME_SECONDS = 3600;

ALTER NOTEBOOK E2E_SNOW_MLOPS_DB.MLOPS_SCHEMA.TRAIN_DEPLOY_MONITOR_ML ADD LIVE VERSION FROM LAST;


GRANT USAGE ON COMPUTE POOL MLOPS_COMPUTE_POOL to ROLE E2E_SNOW_MLOPS_ROLE;


```
