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
