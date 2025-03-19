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

  --Create network rule and api integration to install packages from pypi
CREATE NETWORK RULE IF NOT EXISTS pypi_network_rule
  MODE = EGRESS
  TYPE = HOST_PORT
  VALUE_LIST = ('pypi.org', 'pypi.python.org', 'pythonhosted.org',  'files.pythonhosted.org');

CREATE EXTERNAL ACCESS INTEGRATION IF NOT EXISTS pypi_access_integration
  ALLOWED_NETWORK_RULES = (pypi_network_rule)
  ENABLED = true;

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


GRANT OWNERSHIP ON COMPUTE POOL MLOPS_COMPUTE_POOL TO ROLE E2E_SNOW_MLOPS_ROLE;

