# Proposed Solution

I would automate the reporting process using an Airflow DAG and a simple Python script.

The DAG would run automatically on a schedule (daily or weekly), retrieve the database credentials securely from Vault, and connect to PostgreSQL using a dedicated read-only reporting user with SELECT permissions only.

The Python script would execute the required SQL queries and export the results as CSV files. The files would then be saved to a shared location accessible by the BI teams.

If the job fails or the database becomes unavailable, the DAG should fail gracefully, log the error, and send an email or Teams alert for investigation.

Example:
try:
    connect_to_database()
    run_queries()
    export_csv()
except Exception:
    send_alert()
    raise