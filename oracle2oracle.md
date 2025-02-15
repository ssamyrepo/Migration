
### Prerequisites:
1. **Python 3.x** installed.
2. **cx_Oracle** library for Oracle database connectivity (`pip install cx_Oracle`).
3. **Pandas** for data comparison and reporting (`pip install pandas`).
4. **Oracle databases**:
   - On-prem Oracle database.
   - Oracle database on AWS EC2 (ensure connectivity via VPN or direct connection).

---

### Script: `oracle_migration_benchmark.py`

```python
import cx_Oracle
import pandas as pd
import time

# Database connection details
ON_PREM_DB_CONFIG = {
    "user": "on_prem_user",
    "password": "on_prem_password",
    "dsn": "on_prem_host:port/service_name"
}

AWS_EC2_DB_CONFIG = {
    "user": "aws_user",
    "password": "aws_password",
    "dsn": "aws_host:port/service_name"
}

# List of SQL queries to test
SQL_QUERIES = [
    "SELECT * FROM employees WHERE department_id = 10",
    "SELECT COUNT(*) FROM orders WHERE order_date > TO_DATE('2023-01-01', 'YYYY-MM-DD')",
    "SELECT customer_id, SUM(order_total) FROM orders GROUP BY customer_id",
    # Add more queries as needed
]

# Function to execute a query and measure execution time
def execute_query(connection, query):
    cursor = connection.cursor()
    start_time = time.time()
    cursor.execute(query)
    result = cursor.fetchall()
    execution_time = time.time() - start_time
    cursor.close()
    return result, execution_time

# Function to compare query results
def compare_results(on_prem_result, aws_result):
    on_prem_df = pd.DataFrame(on_prem_result)
    aws_df = pd.DataFrame(aws_result)
    return on_prem_df.equals(aws_df)

# Main function to run benchmarks
def run_benchmarks():
    # Connect to on-prem and AWS databases
    on_prem_conn = cx_Oracle.connect(**ON_PREM_DB_CONFIG)
    aws_conn = cx_Oracle.connect(**AWS_EC2_DB_CONFIG)

    # Initialize results storage
    benchmark_results = []

    # Execute each query on both databases
    for query in SQL_QUERIES:
        print(f"Testing query: {query}")

        # Execute on on-prem database
        on_prem_result, on_prem_time = execute_query(on_prem_conn, query)
        print(f"On-prem execution time: {on_prem_time:.4f} seconds")

        # Execute on AWS EC2 database
        aws_result, aws_time = execute_query(aws_conn, query)
        print(f"AWS EC2 execution time: {aws_time:.4f} seconds")

        # Compare results
        results_match = compare_results(on_prem_result, aws_result)
        print(f"Results match: {results_match}")

        # Store results
        benchmark_results.append({
            "query": query,
            "on_prem_time": on_prem_time,
            "aws_time": aws_time,
            "results_match": results_match
        })

    # Close database connections
    on_prem_conn.close()
    aws_conn.close()

    # Generate and save benchmark report
    report_df = pd.DataFrame(benchmark_results)
    report_df.to_csv("oracle_migration_benchmark_report.csv", index=False)
    print("Benchmark report saved to 'oracle_migration_benchmark_report.csv'")

    # Print summary
    print("\nBenchmark Summary:")
    print(report_df)

# Run the benchmarks
if __name__ == "__main__":
    run_benchmarks()
```

---

### How It Works:
1. **Database Connection**:
   - The script connects to both the on-prem Oracle database and the Oracle database on AWS EC2 using the `cx_Oracle` library.

2. **Query Execution**:
   - It executes a list of predefined SQL queries on both databases.
   - Measures the execution time for each query.

3. **Result Comparison**:
   - Compares the query results from the on-prem and AWS databases to ensure data consistency.

4. **Reporting**:
   - Generates a CSV report with the following details:
     - Query executed.
     - Execution time on on-prem database.
     - Execution time on AWS EC2 database.
     - Whether the results match.

5. **Output**:
   - The report is saved as `oracle_migration_benchmark_report.csv`.

---

### Example Output:
The CSV report will look like this:

| query                                                | on_prem_time | aws_time | results_match |
|------------------------------------------------------|--------------|----------|---------------|
| SELECT * FROM employees WHERE department_id = 10      | 0.1234       | 0.1456   | True          |
| SELECT COUNT(*) FROM orders WHERE order_date > ...    | 0.5678       | 0.5890   | True          |
| SELECT customer_id, SUM(order_total) FROM orders ... | 1.2345       | 1.3456   | True          |

---

### Customization:
1. **Add More Queries**:
   - Add additional SQL queries to the `SQL_QUERIES` list to test more scenarios.

2. **Performance Metrics**:
   - Add more metrics like CPU usage, memory usage, or I/O operations by integrating with monitoring tools.

3. **Error Handling**:
   - Add error handling for database connection issues or query execution failures.

4. **Automation**:
   - Schedule this script to run periodically using a task scheduler (e.g., cron jobs or AWS Lambda).

---

### Running the Script:
1. Install dependencies:
   ```bash
   pip install cx_Oracle pandas
   ```

2. Update the database connection details in the `ON_PREM_DB_CONFIG` and `AWS_EC2_DB_CONFIG` dictionaries.

3. Run the script:
   ```bash
   python oracle_migration_benchmark.py
   ```

4. Check the generated report: `oracle_migration_benchmark_report.csv`.

   ### How to Apply This to Oracle to AWS Migration:
1. **Assess and Categorize Workloads**:
   - Identify and categorize Oracle databases by workload type (e.g., OLTP, OLAP, reporting).
   - Prioritize databases based on business criticality and complexity.

2. **Develop Custom Replication Tools**:
   - Build or use existing tools (e.g., AWS DMS) to replicate data bidirectionally between Oracle and AWS databases.
   - Ensure the replication process handles schema differences and data conflicts.

3. **Automate Benchmark Testing**:
   - Use tools like **Apache JMeter**, **Gatling**, or custom scripts to simulate workloads and measure performance.
   - Validate RTO, RPO, and scalability during each phase of the migration.

4. **Leverage AWS Services**:
   - Use **Amazon RDS** or **Aurora** for relational workloads and **DynamoDB** for NoSQL use cases.
   - Implement **AWS Lambda** for automated data transformation and validation.

5. **Monitor and Optimize**:
   - Use **CloudWatch** and **Prometheus** for real-time monitoring.
   - Continuously optimize database configurations and scaling policies post-migration.

### Resources:
- **AWS Database Migration Service (DMS)**: [AWS DMS Documentation](https://aws.amazon.com/dms/)
- **Capital One Case Study**: [Capital One Migrates to AWS](https://aws.amazon.com/solutions/case-studies/capital-one/)
- **Automated Benchmark Testing Tools**: [Apache JMeter](https://jmeter.apache.org/), [Gatling](https://gatling.io/)

---
