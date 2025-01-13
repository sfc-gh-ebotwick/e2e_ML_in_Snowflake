# e2e_ML_in_Snowflake

Snowflake ML demo showcasing Model Monitoring and A/B testing between two Snow ML Models predicting Mortgage Loan Repayments

# Demo Notebooks showcasing an end-to-end ML worfklow in Snowflake including the following components
- Use Feature Store to track engineered features
    - Store feature defintions in feature store for reproducible computation of ML features
- Train two SnowML Models
    - Xgboost with tree booster
    - Xgboost with linear booster
- Register both models in Snowflake model registry
    - Explore model registry capabilities such as metadata tracking, inference, and explainability
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
