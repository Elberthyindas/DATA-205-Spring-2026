### Ingestion Process

- **Load raw files**: The ingestion script loads raw source files from **/data/raw/**, verifies each file exists, and reads them using efficient readers (CSV or Parquet).  
- **Standardize schema**: It standardizes column names and types (for example, converting wage and employment fields to numeric and year fields to integers).  
- **Deduplicate and validate**: The script removes exact duplicates and drops rows missing key identifiers.  
- **Normalize geography**: It normalizes geographic fields by trimming ZIP and county strings and applies a **ZIP→county crosswalk** to assign awards to counties.  
- **Create derived fields**: The script creates basic derived fields used downstream, such as **annual_hours = WKHP × WKWN**, **log_wage**, and a simplified **education_tier**.  
- **Handle missing values**: It flags or imputes a small number of missing values using documented rules.  
- **Logging and checks**: The ingestion step writes short logs and summary checks (row counts, missing value counts, min/max ranges) so you can confirm the load worked correctly.  
- **Save outputs**: Finally, the cleaned raw tables are saved to **/data/processed/** in Parquet format with a timestamped filename and a small CSV sample is exported for quick inspection.
