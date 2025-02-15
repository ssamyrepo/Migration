

## **Alternative Approaches for SQL Benchmarking & Comparison**
### **1. Using Oracle SQL Performance Analyzer (SPA)**
- **Oracle SPA** can capture SQL workloads before migration and replay them after migration.
- **Benefits**:
  - Compares **execution plans** and **performance metrics**.
  - Identifies **regressions in query performance**.
  - Allows testing on **actual production-like data**.

**Steps**:
1. Capture SQL workload on the **on-prem database**:
   ```sql
   EXEC DBMS_SQLTUNE.CAPTURE_SQL_WORKLOAD(start_time => SYSDATE, capture_mode => 'ALL', capture_name => 'PRE_MIGRATION');
   ```
2. Export the workload and import it into **AWS Oracle RDS/EC2**.
3. Replay workload in **AWS database**:
   ```sql
   EXEC DBMS_SQLTUNE.REPLAY_SQL_WORKLOAD(workload_name => 'PRE_MIGRATION');
   ```
4. Compare performance metrics using:
   ```sql
   SELECT * FROM DBA_SQL_PERFORMANCE_ANALYZER;
   ```

---

### **2. SQL Workload Capture & Replay (SWR & AWR)**
- **Steps**:
  - Use **SQL Workload Repository (SWR)** to capture SQL statistics.
  - Replay in AWS Oracle to compare query runtimes.

**Capture on on-prem DB:**
```sql
EXEC DBMS_WORKLOAD_CAPTURE.START_CAPTURE(name => 'PRE_MIGRATION_WORKLOAD');
-- Run workload
EXEC DBMS_WORKLOAD_CAPTURE.END_CAPTURE;
```
**Transfer & Replay in AWS Oracle:**
```sql
EXEC DBMS_WORKLOAD_REPLAY.PREPARE_REPLAY();
EXEC DBMS_WORKLOAD_REPLAY.START_REPLAY();
```

---

### **3. AWS RDS Performance Insights & CloudWatch**
- If using **Amazon RDS for Oracle**, leverage **Performance Insights** for **real-time SQL analytics**.
- **Steps**:
  1. Enable **Performance Insights** in RDS settings.
  2. Capture queries executed before and after migration.
  3. Use **CloudWatch Metrics** to track **CPU utilization, disk IOPS, query latency**.

---

### **4. Advanced Scripting with Python (Enhanced Version)**
Instead of running individual queries, use **parallel execution and result logging**:

#### **Optimized Python Script**
```python
import cx_Oracle
import pandas as pd
import time
from concurrent.futures import ThreadPoolExecutor

# Configurations
ON_PREM_DB = {"user": "on_prem_user", "password": "on_prem_pass", "dsn": "on_prem_host"}
AWS_DB = {"user": "aws_user", "password": "aws_pass", "dsn": "aws_host"}

SCHEMA_NAME = "DM"
QUERY_LIMIT = 100  # Limit for performance testing

# Extract SQL queries
def fetch_queries(connection):
    cursor = connection.cursor()
    query = f"""
        SELECT sql_text FROM v$sql
        WHERE parsing_schema_name = :schema_name AND ROWNUM <= :limit
    """
    cursor.execute(query, schema_name=SCHEMA_NAME, limit=QUERY_LIMIT)
    queries = [row[0] for row in cursor.fetchall()]
    cursor.close()
    return queries

# Execute query and measure execution time
def execute_query(connection, query):
    cursor = connection.cursor()
    start = time.time()
    try:
        cursor.execute(query)
        result = cursor.fetchall()
    except cx_Oracle.DatabaseError as e:
        result = None
    return time.time() - start, result

# Benchmark comparison
def run_benchmark():
    with cx_Oracle.connect(**ON_PREM_DB) as on_prem_conn, cx_Oracle.connect(**AWS_DB) as aws_conn:
        queries = fetch_queries(on_prem_conn)
        
        results = []
        with ThreadPoolExecutor(max_workers=5) as executor:
            for query in queries:
                on_prem_time, _ = executor.submit(execute_query, on_prem_conn, query).result()
                aws_time, _ = executor.submit(execute_query, aws_conn, query).result()
                
                results.append({"query": query, "on_prem_time": on_prem_time, "aws_time": aws_time})
        
        df = pd.DataFrame(results)
        df.to_csv("sql_benchmark_results.csv", index=False)
        print("Benchmark results saved.")

# Run Benchmark
run_benchmark()
```
---
## **Comparison of Methods**
| Approach  | Pros  | Cons  |
|-----------|--------|------|
| **SPA (SQL Performance Analyzer)** | Most **accurate**, captures execution plans | Needs **Enterprise Edition** |
| **SWR & AWR Reports** | Can **replay full workloads** | Requires **DBA access** |
| **AWS RDS Performance Insights** | Real-time tracking, **no manual setup** | AWS-specific, lacks historical replay |
| **Python Benchmarking Script** | **Simple, automated** execution | Only works for **individual queries**, not full workloads |

\
