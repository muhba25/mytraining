# Training MySQL Oktober 2025
## 1. Replication GTID for Disaster Recovery

enable GTID_MODE on config file server DC & DRC 
```
nano /etc/my.cnf

--- ADD ---
enforce_gtid_consistency=ON
gtid_mode=ON

```

Restart Service MySQL
```
systemctl restart mysqld

```

On Slave
```
mysql> show variables like '%gtid_mode%';show variables like '%enforce_gtid_consistency%';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| gtid_mode     | ON    |
+---------------+-------+
1 row in set (0.01 sec)

+--------------------------+-------+
| Variable_name            | Value |
+--------------------------+-------+
| enforce_gtid_consistency | ON    |
+--------------------------+-------+
1 row in set (0.00 sec)

mysql> start slave;
Query OK, 0 rows affected, 2 warnings (0.00 sec)

mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for source to send event
                  Master_Host: 192.168.170.134
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: binlog-innodbcluster2.000001
          Read_Master_Log_Pos: 504
               Relay_Log_File: innodbcluster3-relay-bin.000002
                Relay_Log_Pos: 744
        Relay_Master_Log_File: binlog-innodbcluster2.000001
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB:
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 0
                   Last_Error:
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 504
              Relay_Log_Space: 963
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File:
           Master_SSL_CA_Path:
              Master_SSL_Cert:
            Master_SSL_Cipher:
               Master_SSL_Key:
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error:
               Last_SQL_Errno: 0
               Last_SQL_Error:
  Replicate_Ignore_Server_Ids:
             Master_Server_Id: 134
                  Master_UUID: 97f0dde3-ab69-11f0-b830-0050562e94ff
             Master_Info_File: mysql.slave_master_info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Replica has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind:
      Last_IO_Error_Timestamp:
     Last_SQL_Error_Timestamp:
               Master_SSL_Crl:
           Master_SSL_Crlpath:
           Retrieved_Gtid_Set: 97f0dde3-ab69-11f0-b830-0050562e94ff:1
            Executed_Gtid_Set: 97f0dde3-ab69-11f0-b830-0050562e94ff:1
                Auto_Position: 1
         Replicate_Rewrite_DB:
                 Channel_Name:
           Master_TLS_Version:
       Master_public_key_path:
        Get_master_public_key: 0
            Network_Namespace:
1 row in set, 1 warning (0.01 sec)

```

On Master
```
mysql> show master status\G
*************************** 1. row ***************************
             File: binlog-innodbcluster2.000001
         Position: 504
     Binlog_Do_DB:
 Binlog_Ignore_DB:
Executed_Gtid_Set: 97f0dde3-ab69-11f0-b830-0050562e94ff:1
1 row in set (0.00 sec)

mysql> CREATE TABLE employees (
    employee_id INT PRIMARY KEY AUTO_INCREMENT,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    hourly_pay DECIMAL(10, 2),
    hire_date DATE
);
Query OK, 1 row affected (0.00 sec)

mysql> INSERT INTO db_test2.employees (first_name, last_name, email, hire_date, salary) VALUES ('orang1', 'oke1', 'orang1.oke1@example.com', '2023-01-15', 60000.00);
Query OK, 1 row affected (0.00 sec)

mysql> select * from db_test2.employees;
+-------------+------------+-----------+--------------------------+------------+----------+
| employee_id | first_name | last_name | email                    | hire_date  | salary   |
+-------------+------------+-----------+--------------------------+------------+----------+
|           1 | John       | Doe       | john.doe@example.com     | 2023-01-15 | 60000.00 |
|           3 | Jane       | Smith     | jane.smith@example.com   | 2022-05-20 | 75000.00 |
|           4 | Peter      | Jones     | peter.jones@example.com  | 2024-03-10 | 55000.00 |
|           6 | John       | Doel      | john.doel@example.com    | 2023-01-15 | 60000.00 |
|           7 | Johny      | yDoe      | john.ydoe@example.com    | 2023-01-15 | 60000.00 |
|           8 | Johnyy     | yDoel     | john.ydoel@example.com   | 2023-01-15 | 60000.00 |
|           9 | Johnt      | Doet      | john.doet@example.com    | 2023-01-15 | 60000.00 |
|          10 | Johnll     | Doelll    | john.doelll@example.com  | 2023-01-15 | 60000.00 |
|          11 | Johnll1    | Doelll    | john1.doelll@example.com | 2023-01-15 | 60000.00 |
|          12 | orang1     | oke1      | orang1.oke1@example.com  | 2023-01-15 | 60000.00 |
+-------------+------------+-----------+--------------------------+------------+----------+
10 rows in set (0.00 sec)

mysql> show master status\G
*************************** 1. row ***************************
             File: binlog-innodbcluster2.000001
         Position: 847
     Binlog_Do_DB:
 Binlog_Ignore_DB:
Executed_Gtid_Set: 97f0dde3-ab69-11f0-b830-0050562e94ff:1-2
1 row in set (0.00 sec)

```

On Slave
```
mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for source to send event
                  Master_Host: 192.168.170.134
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: binlog-innodbcluster2.000001
          Read_Master_Log_Pos: 847
               Relay_Log_File: innodbcluster3-relay-bin.000002
                Relay_Log_Pos: 1087
        Relay_Master_Log_File: binlog-innodbcluster2.000001
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB:
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 0
                   Last_Error:
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 847
              Relay_Log_Space: 1306
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File:
           Master_SSL_CA_Path:
              Master_SSL_Cert:
            Master_SSL_Cipher:
               Master_SSL_Key:
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error:
               Last_SQL_Errno: 0
               Last_SQL_Error:
  Replicate_Ignore_Server_Ids:
             Master_Server_Id: 134
                  Master_UUID: 97f0dde3-ab69-11f0-b830-0050562e94ff
             Master_Info_File: mysql.slave_master_info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Replica has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind:
      Last_IO_Error_Timestamp:
     Last_SQL_Error_Timestamp:
               Master_SSL_Crl:
           Master_SSL_Crlpath:
           Retrieved_Gtid_Set: 97f0dde3-ab69-11f0-b830-0050562e94ff:1-2
            Executed_Gtid_Set: 97f0dde3-ab69-11f0-b830-0050562e94ff:1-2
                Auto_Position: 1
         Replicate_Rewrite_DB:
                 Channel_Name:
           Master_TLS_Version:
       Master_public_key_path:
        Get_master_public_key: 0
            Network_Namespace:
1 row in set, 1 warning (0.01 sec)

mysql> select * from db_test2.employees;
+-------------+------------+-----------+--------------------------+------------+----------+
| employee_id | first_name | last_name | email                    | hire_date  | salary   |
+-------------+------------+-----------+--------------------------+------------+----------+
|           1 | John       | Doe       | john.doe@example.com     | 2023-01-15 | 60000.00 |
|           3 | Jane       | Smith     | jane.smith@example.com   | 2022-05-20 | 75000.00 |
|           4 | Peter      | Jones     | peter.jones@example.com  | 2024-03-10 | 55000.00 |
|           6 | John       | Doel      | john.doel@example.com    | 2023-01-15 | 60000.00 |
|           7 | Johny      | yDoe      | john.ydoe@example.com    | 2023-01-15 | 60000.00 |
|           8 | Johnyy     | yDoel     | john.ydoel@example.com   | 2023-01-15 | 60000.00 |
|           9 | Johnt      | Doet      | john.doet@example.com    | 2023-01-15 | 60000.00 |
|          10 | Johnll     | Doelll    | john.doelll@example.com  | 2023-01-15 | 60000.00 |
|          11 | Johnll1    | Doelll    | john1.doelll@example.com | 2023-01-15 | 60000.00 |
|          12 | orang1     | oke1      | orang1.oke1@example.com  | 2023-01-15 | 60000.00 |
+-------------+------------+-----------+--------------------------+------------+----------+
10 rows in set (0.00 sec)

```

## Using Replication Filter for specific schema to switchover / drill

scenario : we have 2 schema database (db_test1 & db_test2) but we just want db_test1 schema switchover to master, so we can use replica filter for this scenario
```
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| db_test1           |
| db_test2           |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
6 rows in set (0.00 sec)

mysql> use db_test1; show tables; select * from users;
Database changed
+--------------------+
| Tables_in_db_test1 |
+--------------------+
| users              |
+--------------------+
1 row in set (0.00 sec)

+----+----------+----------------------+-------------------+
| id | name     | email                | registration_date |
+----+----------+----------------------+-------------------+
|  1 | John Doe | john.doe@example.com | 2025-10-18        |
+----+----------+----------------------+-------------------+
1 row in set (0.00 sec)

mysql> use db_test2; show tables; select * from employees;
Database changed
+--------------------+
| Tables_in_db_test2 |
+--------------------+
| employees          |
+--------------------+
1 row in set (0.01 sec)

+-------------+------------+-----------+--------------------------+------------+----------+
| employee_id | first_name | last_name | email                    | hire_date  | salary   |
+-------------+------------+-----------+--------------------------+------------+----------+
|           1 | John       | Doe       | john.doe@example.com     | 2023-01-15 | 60000.00 |
|           3 | Jane       | Smith     | jane.smith@example.com   | 2022-05-20 | 75000.00 |
|           4 | Peter      | Jones     | peter.jones@example.com  | 2024-03-10 | 55000.00 |
|           6 | John       | Doel      | john.doel@example.com    | 2023-01-15 | 60000.00 |
|           7 | Johny      | yDoe      | john.ydoe@example.com    | 2023-01-15 | 60000.00 |
|           8 | Johnyy     | yDoel     | john.ydoel@example.com   | 2023-01-15 | 60000.00 |
|           9 | Johnt      | Doet      | john.doet@example.com    | 2023-01-15 | 60000.00 |
|          10 | Johnll     | Doelll    | john.doelll@example.com  | 2023-01-15 | 60000.00 |
|          11 | Johnll1    | Doelll    | john1.doelll@example.com | 2023-01-15 | 60000.00 |
|          12 | orang1     | oke1      | orang1.oke1@example.com  | 2023-01-15 | 60000.00 |
+-------------+------------+-----------+--------------------------+------------+----------+
10 rows in set (0.00 sec)


```

On Slave : Disable super_read_only & read_only and Verify Slave Status Running
```
mysql> show variables like '%read_only%';
+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| innodb_read_only      | OFF   |
| read_only             | ON    |
| super_read_only       | ON    |
| transaction_read_only | OFF   |
+-----------------------+-------+
4 rows in set (0.01 sec)

mysql> set global super_read_only=0;set global read_only=0;
Query OK, 0 rows affected (0.00 sec)

mysql> show variables like '%read_only%';
+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| innodb_read_only      | OFF   |
| read_only             | OFF   |
| super_read_only       | OFF   |
| transaction_read_only | OFF   |
+-----------------------+-------+
4 rows in set (0.01 sec)

mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for source to send event
                  Master_Host: 192.168.170.134
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: binlog-innodbcluster2.000001
          Read_Master_Log_Pos: 2304
               Relay_Log_File: innodbcluster3-relay-bin.000002
                Relay_Log_Pos: 2544
        Relay_Master_Log_File: binlog-innodbcluster2.000001
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB:
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 0
                   Last_Error:
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 2304
              Relay_Log_Space: 2763
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File:
           Master_SSL_CA_Path:
              Master_SSL_Cert:
            Master_SSL_Cipher:
               Master_SSL_Key:
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error:
               Last_SQL_Errno: 0
               Last_SQL_Error:
  Replicate_Ignore_Server_Ids:
             Master_Server_Id: 134
                  Master_UUID: 97f0dde3-ab69-11f0-b830-0050562e94ff
             Master_Info_File: mysql.slave_master_info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Replica has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind:
      Last_IO_Error_Timestamp:
     Last_SQL_Error_Timestamp:
               Master_SSL_Crl:
           Master_SSL_Crlpath:
           Retrieved_Gtid_Set: 97f0dde3-ab69-11f0-b830-0050562e94ff:1-8
            Executed_Gtid_Set: 97f0dde3-ab69-11f0-b830-0050562e94ff:1-8
                Auto_Position: 1
         Replicate_Rewrite_DB:
                 Channel_Name:
           Master_TLS_Version:
       Master_public_key_path:
        Get_master_public_key: 0
            Network_Namespace:
1 row in set, 1 warning (0.01 sec)

```

On Master : Verify No Slave Status and Set Slave for db_test1 schema with Replication Filter
```
mysql> show slave status\G
Empty set, 1 warning (0.01 sec)

mysql> show variables like '%read_only%';
+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| innodb_read_only      | OFF   |
| read_only             | OFF   |
| super_read_only       | OFF   |
| transaction_read_only | OFF   |
+-----------------------+-------+
4 rows in set (0.00 sec)

mysql> CHANGE MASTER TO MASTER_HOST='192.168.170.135', MASTER_USER='repl', MASTER_PASSWORD='Tes250299!!!', MASTER_PORT=3306, MASTER_AUTO_POSITION = 1;
Query OK, 0 rows affected, 8 warnings (0.03 sec)

mysql> CHANGE REPLICATION FILTER REPLICATE_DO_DB = (db_test1);
Query OK, 0 rows affected (0.00 sec)

mysql> start slave;
Query OK, 0 rows affected, 1 warning (0.02 sec)

mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for source to send event
                  Master_Host: 192.168.170.135
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: binlog-innodbcluster3.000001
          Read_Master_Log_Pos: 2361
               Relay_Log_File: innodbcluster2-relay-bin.000002
                Relay_Log_Pos: 456
        Relay_Master_Log_File: binlog-innodbcluster3.000001
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: db_test1
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 0
                   Last_Error:
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 2361
              Relay_Log_Space: 675
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File:
           Master_SSL_CA_Path:
              Master_SSL_Cert:
            Master_SSL_Cipher:
               Master_SSL_Key:
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error:
               Last_SQL_Errno: 0
               Last_SQL_Error:
  Replicate_Ignore_Server_Ids:
             Master_Server_Id: 135
                  Master_UUID: 37cf40f4-ab68-11f0-9657-0050563c05df
             Master_Info_File: mysql.slave_master_info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Replica has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind:
      Last_IO_Error_Timestamp:
     Last_SQL_Error_Timestamp:
               Master_SSL_Crl:
           Master_SSL_Crlpath:
           Retrieved_Gtid_Set:
            Executed_Gtid_Set: 97f0dde3-ab69-11f0-b830-0050562e94ff:1-8
                Auto_Position: 1
         Replicate_Rewrite_DB:
                 Channel_Name:
           Master_TLS_Version:
       Master_public_key_path:
        Get_master_public_key: 0
            Network_Namespace:
1 row in set, 1 warning (0.00 sec)

```

On Slave : Insert Data on Users Table\
On Master : Check whether the data in the Users Table has been replicated  
```
mysql> INSERT INTO db_test1.users VALUES (NULL, 'Andy', 'Andy@example.com', '2025-10-18');
Query OK, 1 row affected (0.00 sec)

mysql> select * from db_test1.users;
+----+----------+----------------------+-------------------+
| id | name     | email                | registration_date |
+----+----------+----------------------+-------------------+
|  1 | John Doe | john.doe@example.com | 2025-10-18        |
|  2 | Andy     | Andy@example.com     | 2025-10-18        |
+----+----------+----------------------+-------------------+
2 rows in set (0.00 sec)

```

## 2. Pengenalan MySQL 9 sebagai inovasi rilis dalam pengembangan AI

Example Backend Code
```
import os
import json
from flask import Flask, request, jsonify
import mysql.connector
import PyPDF2
from sentence_transformers import SentenceTransformer
import numpy as np
from flask_cors import CORS
from openai import OpenAI
from dotenv import load_dotenv

# Load environment variables from .env file
load_dotenv()

app = Flask(__name__)
# Allow requests from your frontend development server
CORS(app, origins=["http://ip-addresss:5175", "http://localhost:5175"])

# --- AI & Model Setup ---

# Initialize the OpenRouter client
try:
    client = OpenAI(
      base_url="https://openrouter.ai/api/v1",
      api_key=os.getenv("OPENROUTER_API_KEY"),
      default_headers={
        "HTTP-Referer": os.getenv("OPENROUTER_REFERRER"),
      },
    )
except Exception as e:
    print(f"Failed to initialize OpenRouter client: {e}")
    client = None

# Load local embedding model (for existing PDF search functionality)
model = SentenceTransformer("all-MiniLM-L6-v2")


# --- Database Configurations ---

db_conf_vector = {
    'host': 'localhost',
    'user': 'root',
    'password': 'password',
    'database': 'vector_app'
}

# --- Helper Functions ---

def get_db_schema(db_config):
    """Connects to a database and retrieves the CREATE TABLE statements for all tables."""
    schema_info = ""
    try:
        conn = mysql.connector.connect(**db_config)
        cur = conn.cursor()
        cur.execute("SHOW TABLES")
        tables = [table[0] for table in cur.fetchall()]
        for table_name in tables:
            cur.execute(f"SHOW CREATE TABLE `{table_name}`")
            schema_info += cur.fetchone()[1] + ";\n\n"
        cur.close()
        conn.close()
        return schema_info
    except Exception as e:
        print(f"Error getting DB schema: {e}")
        return None

def format_results_as_markdown(cursor):
    """Formats SQL query results into a Markdown table string for nice frontend display."""
    columns = [col[0] for col in cursor.description]
    rows = cursor.fetchall()
    if not rows:
        return "The query returned no results."
    header = "| " + " | ".join(columns) + " |"
    separator = "| " + " | ".join(["---"] * len(columns)) + " |"
    body = "\n".join(["| " + " | ".join(map(str, row)) + " |" for row in rows])
    return f"{header}\n{separator}\n{body}"

def cosine_similarity(vec1, vec2):
    v1 = np.array(vec1)
    v2 = np.array(vec2)
    return float(np.dot(v1, v2) / (np.linalg.norm(v1) * np.linalg.norm(v2)))

def extract_text_from_pdf(file):
    reader = PyPDF2.PdfReader(file)
    text = ""
    for page in reader.pages:
        text += page.extract_text() or ""
    return text.strip()

def embed_text(text: str):
    return model.encode(text).tolist()


# --- API Endpoints ---

@app.route('/ask', methods=['POST'])
def ask_sql_bot():
    """The new endpoint that converts natural language to SQL and executes it."""
    if not client:
        return jsonify({"error": "OpenRouter client is not initialized. Check your API key and referrer in the .env file."}), 500
        
    data = request.json
    query = data.get("query")
    if not query:
        return jsonify({"error": "Missing query"}), 400

    # 1. Get the database schema to provide as context to the LLM
    schema = get_db_schema(db_conf_vector)
    if not schema:
        return jsonify({"error": "Could not retrieve database schema. Check 'db_conf_vector' config."}), 500

    # 2. Craft the prompt for the Text-to-SQL model
    system_prompt = f"""
    You are an expert SQL assistant. Given the following MySQL database schema, write a valid MySQL query to answer the user's question.
    - Only output the raw SQL query.
    - Do not add any explanation, comments, or formatting like ```sql.

    Schema:
    {schema}
    """

    try:
        # 3. Call OpenRouter to get the SQL query
        completion = client.chat.completions.create(
          model="deepseek/deepseek-chat",
          messages=[
            {"role": "system", "content": system_prompt},
            {"role": "user", "content": query},
          ],
          temperature=0.0, # Use low temperature for deterministic SQL
        )
        sql_query = completion.choices[0].message.content.strip().replace('`', '')

        # 4. **CRITICAL SECURITY CHECK**: Only allow SELECT statements.
        if not sql_query.lower().lstrip().startswith("select"):
            raise ValueError("For security reasons, only SELECT statements are allowed.")

        # 5. Execute the generated query against the 'office' database
        conn = mysql.connector.connect(**db_conf_vector)
        cur = conn.cursor()
        cur.execute(sql_query)

        # 6. Format and return the result as Markdown
        result_markdown = format_results_as_markdown(cur)
        cur.close()
        conn.close()

        return jsonify({"answer": result_markdown})

    except ValueError as ve:
        print(f"Validation Error: {ve}")
        return jsonify({"answer": f"I cannot process this request. {str(ve)}"})
    except mysql.connector.Error as err:
        print(f"MySQL Error: {err}")
        return jsonify({"answer": f"There was an error executing the SQL query: {err}"})
    except Exception as e:
        print(f"An unexpected error occurred: {e}")
        return jsonify({"error": "An unexpected error occurred on the server."}), 500


@app.route('/search', methods=['POST'])
def search_and_answer():
    """
    Performs vector search on documents and uses OpenRouter AI to answer based on the context.
    This is a Retrieval-Augmented Generation (RAG) implementation.
    """
    if not client:
        return jsonify({"error": "OpenRouter client is not initialized."}), 500
        
    data = request.json
    query = data.get("query")
    if not query:
        return jsonify({"error": "Missing query"}), 400

    # Step 1: Retrieve relevant documents using vector search.
    query_vec = embed_text(query)
    try:
        conn = mysql.connector.connect(**db_conf_vector)
        cur = conn.cursor(dictionary=True)
        sql = "SELECT filename, content, VECTOR_TO_STRING(embedding) AS vec_str FROM documents"
        cur.execute(sql)
        rows = cur.fetchall()
        cur.close()
        conn.close()
    except Exception as e:
        return jsonify({"error": str(e)}), 500

    results = []
    for row in rows:
        try:
            emb = [float(x) for x in row["vec_str"].strip("[]").split(",")]
            score = cosine_similarity(query_vec, emb)
            row["similarity"] = score
            del row["vec_str"]
            results.append(row)
        except Exception as e:
            print("Parse error:", e)
    
    results = sorted(results, key=lambda x: x["similarity"], reverse=True)
    
    if not results or results[0]["similarity"] < 0.5:
        return jsonify({"answer": "I couldn't find any relevant information in the uploaded documents to answer your question."})

    top_doc = results[0]
    context = top_doc['content']
    source_filename = top_doc['filename']

    # Step 2: Generate an answer using the retrieved context (RAG).
    system_prompt = f"""You are a helpful assistant. Answer the user's question based *only* on the following context provided from a PDF document.
    If the answer is not found within the context, state that you cannot find the answer in the document.
    Do not use any prior knowledge.

    Context from '{source_filename}':
    ---
    {context}
    ---
    """
    try:
        completion = client.chat.completions.create(
          model="mistralai/mistral-7b-instruct",
          messages=[
            {"role": "system", "content": system_prompt},
            {"role": "user", "content": query},
          ],
        )
        answer = completion.choices[0].message.content.strip()
        
        # Return the answer, source, and context as a structured object
        return jsonify({
            "answer": answer,
            "source": source_filename,
            "context": context
        })

    except Exception as e:
        print(f"An unexpected error occurred with OpenRouter: {e}")
        return jsonify({"error": "An unexpected error occurred while generating the answer."}), 500


@app.route('/upload', methods=['POST'])
def upload_pdf():
    if 'file' not in request.files:
        return jsonify({"error": "No file uploaded"}), 400
    file = request.files['file']
    filename = file.filename
    text = extract_text_from_pdf(file)
    if not text:
        return jsonify({"error": "No text found in PDF"}), 400
    embedding = embed_text(text)
    vec_str = "[" + ",".join(str(x) for x in embedding) + "]"
    try:
        conn = mysql.connector.connect(**db_conf_vector)
        cur = conn.cursor()
        sql = "INSERT INTO documents (filename, content, embedding) VALUES (%s, %s, STRING_TO_VECTOR(%s))"
        cur.execute(sql, (filename, text, vec_str))
        conn.commit()
        doc_id = cur.lastrowid
        cur.close()
        conn.close()
    except Exception as e:
        return jsonify({"error": str(e)}), 500
    return jsonify({"id": doc_id, "filename": filename, "chars": len(text)})


if __name__ == "__main__":
    app.run(debug=True)

```

Example Frontend Code
```
import React, { useState, useEffect, useRef } from "react";
// Use ESM imports to resolve dependencies directly from a CDN
import ReactMarkdown from 'https://esm.sh/react-markdown';
import remarkGfm from 'https://esm.sh/remark-gfm';

// Since the original components were not provided, these are simple
// functional equivalents to make the component runnable.
const Button = ({ children, ...props }) => (
  <button className="px-4 py-2 bg-blue-600 text-white font-semibold rounded-lg shadow-md hover:bg-blue-700 disabled:bg-gray-400 disabled:cursor-not-allowed transition-colors duration-200" {...props}>
    {children}
  </button>
);
const Card = ({ children, ...props }) => <div className="bg-white rounded-2xl shadow-xl" {...props}>{children}</div>;
const CardContent = ({ children, ...props }) => <div className="p-6 space-y-4" {...props}>{children}</div>;


export default function App() {
  const [file, setFile] = useState(null);
  const [chatHistory, setChatHistory] = useState([
    { sender: "bot", text: "Hello! Ask a question about the 'office' database or search uploaded PDFs." }
  ]);
  const [query, setQuery] = useState("");
  const [loading, setLoading] = useState(false);
  const [actionType, setActionType] = useState(null); // 'ask' or 'search'
  const chatEndRef = useRef(null);

  // NOTE: Replace with your actual backend IP or domain if different
  const API_URL = "http://192.168.112.39:9000";

  // Auto-scroll to the latest message
  useEffect(() => {
    chatEndRef.current?.scrollIntoView({ behavior: "smooth" });
  }, [chatHistory]);

  // Upload PDF (Existing functionality, kept for completeness)
  const handleUpload = async () => {
    if (!file) {
      setChatHistory(prev => [...prev, { sender: 'system', text: 'Please select a PDF file to upload first.'}]);
      return;
    }
    const formData = new FormData();
    formData.append("file", file);

    setLoading(true);
    setActionType('upload');
    try {
      const res = await fetch(`${API_URL}/upload`, {
        method: "POST",
        body: formData,
      });
      const data = await res.json();
      if (!res.ok) throw new Error(data.error || 'Upload failed');
      setChatHistory((prev) => [
        ...prev,
        { sender: "system", text: `Successfully uploaded ${data.filename}. You can now use the 'Search PDFs' button.` },
      ]);
    } catch (err) {
      setChatHistory((prev) => [...prev, { sender: "system", text: `Upload Error: ${err.message}` }]);
    } finally {
      setLoading(false);
      setActionType(null);
      setFile(null);
      document.querySelector('input[type="file"]').value = '';
    }
  };

  // Ask a question to the Text-to-SQL Bot
  const handleAsk = async () => {
    if (!query.trim()) return;

    setChatHistory((prev) => [...prev, { sender: "user", text: query }]);
    setLoading(true);
    setActionType('ask');
    setQuery("");

    try {
      const res = await fetch(`${API_URL}/ask`, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ query }),
      });
      
      const data = await res.json();
      if (!res.ok) throw new Error(data.error || `Request failed with status ${res.status}`);
      
      setChatHistory((prev) => [...prev, { sender: "bot", text: data.answer || "I received an empty response." }]);
    } catch (err) {
      console.error("Ask error:", err);
      setChatHistory((prev) => [...prev, { sender: "bot", text: `Sorry, something went wrong: ${err.message}` }]);
    } finally {
      setLoading(false);
      setActionType(null);
    }
  };

  // Search uploaded PDFs using the new RAG endpoint
  const handleSearch = async () => {
    if (!query.trim()) return;

    setChatHistory((prev) => [...prev, { sender: "user", text: query }]);
    setLoading(true);
    setActionType('search');
    setQuery("");

    try {
      const res = await fetch(`${API_URL}/search`, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ query }),
      });
      
      const data = await res.json();
      if (!res.ok) throw new Error(data.error || `Request failed with status ${res.status}`);
      
      let botResponse;
      if (data.answer && data.source && data.context) {
          // Truncate context for display to avoid overwhelming the chat.
          const contextSnippet = data.context.length > 500 
              ? `${data.context.substring(0, 500)}...` 
              : data.context;

          botResponse = `
${data.answer}

---
*Source: \`${data.source}\`*

**Retrieved Context Snippet:**
> ${contextSnippet}
`;
      } else {
          botResponse = data.answer || "I received an empty or incomplete response from the server.";
      }
      setChatHistory((prev) => [...prev, { sender: "bot", text: botResponse }]);
    } catch (err) {
      console.error("Search error:", err);
      setChatHistory((prev) => [...prev, { sender: "bot", text: `Sorry, something went wrong while searching: ${err.message}` }]);
    } finally {
      setLoading(false);
      setActionType(null);
    }
  };

  const handleKeyPress = (event) => {
    if (event.key === 'Enter' && !loading && query.trim()) {
      handleAsk(); // Default 'Enter' action is to ask the database
    }
  };

  return (
    <div className="min-h-screen flex flex-col items-center justify-center bg-gray-100 p-4 font-sans">
      <Card className="w-full max-w-3xl">
        <CardContent>
          <h1 className="text-3xl font-bold text-center text-gray-800 mb-4">AI Database & Document Assistant</h1>
          
          <div className="flex items-center space-x-2 p-3 bg-gray-50 rounded-lg border">
            <span className="text-sm font-medium text-gray-600">PDF Tools:</span>
            <input
              type="file"
              accept=".pdf"
              onChange={(e) => setFile(e.target.files[0])}
              className="flex-1 text-sm text-gray-700 file:mr-4 file:py-2 file:px-4 file:rounded-full file:border-0 file:text-sm file:font-semibold file:bg-blue-50 file:text-blue-700 hover:file:bg-blue-100"
            />
            <Button onClick={handleUpload} disabled={loading || !file}>
              {loading && actionType === 'upload' ? "Uploading..." : "Upload PDF"}
            </Button>
          </div>

          <div className="h-[32rem] overflow-y-auto border rounded-lg p-4 bg-white space-y-4">
            {chatHistory.map((msg, idx) => (
              <div key={idx} className={`flex ${msg.sender === "user" ? "justify-end" : "justify-start"}`}>
                <div
                  className={`py-2 px-4 rounded-2xl max-w-xl whitespace-pre-wrap ${
                    msg.sender === "user"
                      ? "bg-blue-500 text-white"
                      : msg.sender === "bot"
                      ? "bg-gray-200 text-gray-800 prose prose-sm max-w-none"
                      : "bg-yellow-100 text-yellow-800 text-xs italic w-full text-center"
                  }`}
                >
                  {msg.sender === "bot" ? (
                     <ReactMarkdown remarkPlugins={[remarkGfm]}>{msg.text}</ReactMarkdown>
                  ) : (
                    msg.text
                  )}
                </div>
              </div>
            ))}
             <div ref={chatEndRef} />
          </div>

          <div className="flex space-x-2">
            <input
              type="text"
              value={query}
              onChange={(e) => setQuery(e.target.value)}
              onKeyPress={handleKeyPress}
              placeholder="Ask database or search PDFs..."
              className="flex-1 border rounded-lg p-3 focus:ring-2 focus:ring-blue-500 focus:outline-none transition"
              disabled={loading}
            />
            <Button onClick={handleAsk} disabled={loading || !query.trim()}>
              {loading && actionType === 'ask' ? "Asking..." : "Ask Database"}
            </Button>
            <Button onClick={handleSearch} disabled={loading || !query.trim()} className="px-4 py-2 bg-green-600 text-white font-semibold rounded-lg shadow-md hover:bg-green-700 disabled:bg-gray-400 disabled:cursor-not-allowed transition-colors duration-200">
              {loading && actionType === 'search' ? "Searching..." : "Search PDFs"}
            </Button>
          </div>
        </CardContent>
      </Card>
       <footer className="text-center text-gray-500 text-sm mt-4">
        Powered by OpenRouter and MySQL.
      </footer>
    </div>
  );
}

```