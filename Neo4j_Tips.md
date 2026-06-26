# Tips and Tricks
## Q-1 How do you connect to SQL / python / RDBMS from NEO4j
-  Connecting Neo4j with SQL / RDBMS
  .  ETL Pipelines
  .  Extract data from SQL (e.g., PostgreSQL, MySQL, Oracle).
  .  Transform into graph structures.
  .  Load into Neo4j using LOAD CSV, neo4j-admin import, or APOC procedures.
```
CALL apoc.load.jdbc(
  'jdbc:mysql://localhost:3306/mydb?user=root&password=pass',
  'SELECT * FROM customers'
) YIELD row
MERGE (c:Customer {id: row.id})
SET c.name = row.name;

```

   
-  Connecting Neo4j with Python
  ```
from neo4j import GraphDatabase

uri = "bolt://localhost:7687"
driver = GraphDatabase.driver(uri, auth=("neo4j", "password"))

def get_customers(tx):
    result = tx.run("MATCH (c:Customer) RETURN c.name AS name")
    return [record["name"] for record in result]

with driver.session() as session:
    customers = session.read_transaction(get_customers)
    print(customers)

  ```
-  Hybrid Approach (SQL → Python → Neo4j)
```
import psycopg2
from neo4j import GraphDatabase

# Connect to PostgreSQL
pg_conn = psycopg2.connect("dbname=test user=postgres password=secret")

# Connect to Neo4j
neo_driver = GraphDatabase.driver("bolt://localhost:7687", auth=("neo4j", "password"))

cursor = pg_conn.cursor()
cursor.execute("SELECT id, name FROM customers")

with neo_driver.session() as session:
    for row in cursor.fetchall():
        session.run("MERGE (c:Customer {id:$id}) SET c.name=$name", id=row[0], name=row[1])


```
-  
