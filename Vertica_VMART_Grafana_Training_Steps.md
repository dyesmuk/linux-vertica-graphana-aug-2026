# Vertica VMart → Grafana Training Lab
## Sequential Setup and First Visualization

> **Scope:** This guide documents only the steps completed in the discussion so far.
>
> **Assumption:** All required software installations are already complete:
> - Windows 11
> - WSL2
> - Grafana installed natively in WSL2
> - Docker Desktop
> - Vertica CE running in Docker
> - DBeaver
>
> The goal is to get realistic VMart data into Vertica and display that live Vertica data in Grafana.

---


---

# 0. Important Note: Why This Lab Uses `molo17/vertica-ce`

This training environment uses the Docker image:

```text
molo17/vertica-ce
```

Docker Hub:

https://hub.docker.com/r/molo17/vertica-ce

The image currently used in this lab is:

```text
molo17/vertica-ce:24.1.0-0
```

It can be pulled with:

```bash
docker pull molo17/vertica-ce:24.1.0-0
```

Docker Hub currently lists the `24.1.0-0` tag for this repository.  
Reference:

https://hub.docker.com/r/molo17/vertica-ce

## Why are we using this image?

The training material originally expects a Vertica Community Edition environment.

However, the Vertica product ownership has changed. OpenText announced the divestiture of Vertica to Rocket Software on February 2, 2026, and Rocket Software completed the acquisition on May 11, 2026.

References:

- Rocket Software announcement: https://www.rocketsoftware.com/en-us/news/rocket-software-acquire-vertica-analytics-database-platform-opentext
- Rocket Software acquisition completion: https://www.rocketsoftware.com/en-us/news/rocket-software-completes-acquisition-vertica
- OpenText announcement: https://investors.opentext.com/press-releases/press-releases-details/2026/OpenText-to-Divest-Vertica-for-US150-million/default.aspx

For this training lab, the practical workaround is therefore to use the available Docker image:

```text
molo17/vertica-ce:24.1.0-0
```

This allows trainees to run a self-contained Vertica CE-style training environment locally using Docker Desktop rather than depending on the historical official CE distribution/download workflow.

> **Training note:** The `molo17/vertica-ce` image is a third-party Docker Hub image, not an official Rocket Software Docker image. Use it specifically as the lab workaround described here.

The rest of this guide assumes that the image has already been pulled and that the container is running as:

```text
vertica-ce
```

with Vertica exposed on:

```text
localhost:5433
```


# 1. Target Architecture

The lab uses this arrangement:

```text
Windows 11
│
├── WSL2
│   └── Grafana
│       └── http://localhost:3000
│
├── Docker Desktop
│   └── Vertica CE container
│       └── demo database
│           └── VMart sample data
│
└── DBeaver
```

The important data path is:

```text
Vertica VMart
      │
      │ SQL
      ▼
Grafana Vertica Data Source
      │
      ▼
Grafana Explore / Dashboard
      │
      ▼
Time-series visualization
```

---

# 2. Verify the Vertica Container

From the WSL terminal:

```bash
docker ps --format "table {{.Names}}\t{{.Ports}}"
```

Expected result should include something similar to:

```text
NAMES        PORTS
vertica-ce   0.0.0.0:5433->5433/tcp, [::]:5433->5433/tcp
```

The important point is:

```text
5433 -> 5433
```

Vertica's port 5433 is exposed by Docker.

---

# 3. Connect to Vertica from WSL

Use:

```bash
docker exec -it vertica-ce /opt/vertica/bin/vsql -U dbadmin -d demo
```

Expected:

```text
Welcome to vsql, the Vertica Analytic Database interactive terminal.
```

You should then see:

```text
dbadmin@...=>
```

---

# 4. Verify Vertica Version

Inside `vsql`:

```sql
SELECT version();
```

In this lab the result was:

```text
Vertica Analytic Database v24.1.0-0
```

---

# 5. Verify the Database

Inside `vsql`:

```sql
\l
```

The lab initially showed:

```text
name | user_name
-----+----------
demo | dbadmin
```

There was only one database:

```text
demo
```

---

# 6. Check Whether the Database Contains Tables

Inside `vsql`:

```sql
\dt
```

Initially the result was:

```text
No relations found.
```

A more explicit check is:

```sql
SELECT table_schema, table_name
FROM v_catalog.tables
WHERE is_system_table = false
ORDER BY table_schema, table_name;
```

Initially:

```text
(0 rows)
```

Therefore the `demo` database was empty of user tables.

---

# 7. Check for the Vertica VMart Example Files

Exit `vsql`:

```text
\q
```

You should return to:

```text
vaman@VAMUX:/mnt/d/Projects$
```

Now check the example directory:

```bash
docker exec -it vertica-ce ls -l /opt/vertica/examples
```

Expected:

```text
VMart_Schema
```

Check its contents:

```bash
docker exec -it vertica-ce ls -l /opt/vertica/examples/VMart_Schema
```

The supplied package contains, among other files:

```text
01_load_vmart_schema.sh
02_vmart_etl.sql
vmart_define_schema.sql
vmart_load_data.sql
vmart_load_data_datadir.sql
vmart_count_data.sql
vmart_queries.sql
vmart_query_01.sql
...
vmart_schema_drop.sql
vmart_gen
vmart_gen.cpp
Time.txt
Time_custom.txt
```

---

# 8. Understand the VMart Loader

Read the README:

```bash
docker exec -it vertica-ce cat /opt/vertica/examples/VMart_Schema/README
```

The important point is that:

```text
vmart_gen
```

is the VMart data generator.

The supplied loader script:

```text
01_load_vmart_schema.sh
```

performs the overall setup:

```text
Drop old VMart schema
        ↓
Generate data
        ↓
Create schema/tables
        ↓
Load data
        ↓
Run ETL
        ↓
Confirm successful load
```

---

# 9. Inspect the VMart Loader

Run:

```bash
docker exec -it vertica-ce cat /opt/vertica/examples/VMart_Schema/01_load_vmart_schema.sh
```

Important environment variables in this lab are:

```text
VERTICA_DB_USER=dbadmin
VMART_DIR=/opt/vertica/examples/VMart_Schema
VMART_ETL_SQL=02_vmart_etl.sql
VMART_CONFIRM_LOAD_SCHEMA=public
VMART_CONFIRM_LOAD_TABLE=vmart_load_success
```

Verify them with:

```bash
docker exec -it vertica-ce bash -lc 'echo "VERTICA_DB_USER=$VERTICA_DB_USER"; echo "VMART_DIR=$VMART_DIR"; echo "VMART_ETL_SQL=$VMART_ETL_SQL"; echo "VMART_CONFIRM_LOAD_SCHEMA=$VMART_CONFIRM_LOAD_SCHEMA"; echo "VMART_CONFIRM_LOAD_TABLE=$VMART_CONFIRM_LOAD_TABLE"'
```

Expected:

```text
VERTICA_DB_USER=dbadmin
VMART_DIR=/opt/vertica/examples/VMart_Schema
VMART_ETL_SQL=02_vmart_etl.sql
VMART_CONFIRM_LOAD_SCHEMA=public
VMART_CONFIRM_LOAD_TABLE=vmart_load_success
```

---

# 10. Verify the Default Database Used by the Loader

The loader uses:

```text
vsql -U dbadmin
```

rather than explicitly specifying `-d demo`.

Therefore verify the default database:

```bash
docker exec -it vertica-ce bash -lc '/opt/vertica/bin/vsql -U dbadmin -Atc "SELECT current_database();"'
```

Expected:

```text
demo
```

This confirms the supplied VMart loader will operate on the `demo` database in this lab.

---

# 11. Load VMart

Run this from the WSL/Linux shell:

```bash
docker exec -it vertica-ce bash -lc 'cd /opt/vertica/examples/VMart_Schema && ./01_load_vmart_schema.sh'
```

Do NOT run this command from inside `vsql`.

The loader generates the following significant amounts of data in this setup:

```text
store_sales_fact        5,000,000 rows
online_sales_fact       5,000,000 rows
store_orders_fact         300,000 rows
inventory_fact            300,000 rows
customer_dimension         50,000 rows
employee_dimension        10,000 rows
product_dimension            500 rows
store_dimension               50 rows
promotion_dimension          100 rows
vendor_dimension              50 rows
warehouse_dimension          100 rows
shipping_dimension           100 rows
online_page_dimension      1,000 rows
call_center_dimension        200 rows
```

The VMart generation in this lab used:

```text
years = 2003 to 2027
```

The loader should progress through stages similar to:

```text
Dropping old schema ...
Generating data ...
Creating schema ...
Loading files ...
Running ETL ...
Confirm successful load
```

---

# 12. Normal Messages During the Initial Drop

On a completely empty database, you may see messages such as:

```text
Schema "online_sales" does not exist
Schema "store" does not exist
Table "inventory_fact" does not exist
...
```

These occurred in this lab because the loader first attempts to remove an old VMart installation.

If the database is empty, these messages are expected and are not a problem.

The important thing is that the later stages complete successfully:

```text
Generating data ...
Creating schema ...
Loading files ...
Running ETL ...
Confirm successful load
```

---

# 13. Verify the VMart Tables

Reconnect:

```bash
docker exec -it vertica-ce /opt/vertica/bin/vsql -U dbadmin -d demo
```

Then:

```sql
SELECT table_schema, table_name
FROM v_catalog.tables
WHERE is_system_table = false
ORDER BY table_schema, table_name;
```

Expected tables in this lab:

```text
online_sales | call_center_dimension
online_sales | online_page_dimension
online_sales | online_sales_fact
public       | customer_dimension
public       | date_dimension
public       | employee_dimension
public       | inventory_fact
public       | product_dimension
public       | promotion_dimension
public       | shipping_dimension
public       | vendor_dimension
public       | vmart_load_success
public       | warehouse_dimension
store        | store_dimension
store        | store_orders_fact
store        | store_sales_fact
```

Total:

```text
16 rows
```

---

# 14. Verify Actual Sales Data

Inside `vsql`:

```sql
SELECT
    COUNT(*) AS number_of_sales,
    SUM(sales_dollar_amount) AS total_sales
FROM store.store_sales_fact;
```

In this lab the result was:

```text
number_of_sales | total_sales
----------------+------------
5000000         | 1363723977
```

This proves that the VMart sales data is actually loaded.

---

# 15. Inspect the Sales Table Before Writing Queries

Use:

```sql
\d store.store_sales_fact
```

The important columns are:

```text
date_key
product_key
product_version
store_key
promotion_key
customer_key
employee_key
pos_transaction_number
sales_quantity
sales_dollar_amount
cost_dollar_amount
gross_profit_dollar_amount
transaction_type
transaction_time
tender_type
store_sales_date
store_sales_datetime
```

Also inspect:

```sql
\d public.date_dimension
```

The important columns include:

```text
date_key
date
calendar_year
calendar_month_name
calendar_year_month
...
```

The foreign-key relationship is:

```text
store.store_sales_fact.date_key
        ↓
public.date_dimension.date_key
```

---

# 16. Test a Sales-by-Date Query in Vertica

A first query can use the date dimension:

```sql
SELECT
    d.date,
    SUM(s.sales_dollar_amount) AS sales
FROM store.store_sales_fact s
JOIN public.date_dimension d
    ON s.date_key = d.date_key
GROUP BY d.date
ORDER BY d.date
LIMIT 20;
```

This produced results such as:

```text
date        | sales
------------+--------
2003-01-01  | 35980
2003-01-02  | 6531
2003-01-03  | 56003
2003-01-04  | 69503
2003-01-05  | 281057
...
```

---

# 17. Important Troubleshooting: Column Name

Do NOT use:

```sql
s.sale_date_key
```

The actual column is:

```text
s.date_key
```

The schema inspection with:

```sql
\d store.store_sales_fact
```

is the authoritative way to determine the actual columns.

---

# 18. Simpler Query for Grafana

`store_sales_fact` already contains:

```text
store_sales_date
```

Therefore a simple Grafana time-series query can avoid the date-dimension join:

```sql
SELECT
    store_sales_date AS time,
    SUM(sales_dollar_amount) AS sales
FROM store.store_sales_fact
GROUP BY store_sales_date
ORDER BY store_sales_date
LIMIT 20;
```

This produced:

```text
time        | sales
------------+--------
2003-01-01  | 35980
2003-01-02  | 6531
2003-01-03  | 56003
...
```

---

# 19. Verify Grafana

Exit `vsql`:

```text
\q
```

From WSL:

```bash
grafana-server -v
```

In this lab:

```text
Version 13.2.0
```

You may see a deprecation warning saying that:

```text
grafana-server
```

will eventually be replaced by:

```text
grafana server
```

The version command still worked successfully in this lab.

Check the service:

```bash
sudo systemctl status grafana-server --no-pager
```

Expected:

```text
Active: active (running)
```

Check port 3000:

```bash
ss -lntp | grep 3000
```

Expected:

```text
LISTEN ... *:3000
```

---

# 20. Install the Vertica Grafana Plugin

Check whether the plugin is installed:

```bash
grafana cli plugins ls | grep -i vertica
```

If there is no result, install it:

```bash
sudo grafana cli plugins install vertica-grafana-datasource
```

Then verify:

```bash
grafana cli plugins ls | grep -i vertica
```

In this lab the installed plugin was:

```text
vertica-grafana-datasource v3.2.1
```

Restart Grafana:

```bash
sudo systemctl restart grafana-server
```

Verify:

```bash
sudo systemctl status grafana-server --no-pager
```

It should show:

```text
Active: active (running)
```

---

# 21. Troubleshooting: Grafana Plugin Directory Permission

This command:

```bash
ls -la /var/lib/grafana/plugins | grep -i vertica
```

may produce:

```text
Permission denied
```

when run as the normal WSL user.

That does not mean the plugin is missing.

Use:

```bash
grafana cli plugins ls | grep -i vertica
```

instead.

---

# 22. Configure the Grafana Vertica Data Source

Open:

```text
http://localhost:3000
```

In Grafana:

```text
Connections
    ↓
Data sources
    ↓
Add new data source
    ↓
Vertica
```

Create the data source.

Recommended name:

```text
Vertica-VMart
```

Use:

```text
Host:     localhost:5433
Database: demo
User:     dbadmin
Password: <your Vertica dbadmin password>
SSL Mode: none
```

The critical value is:

```text
localhost:5433
```

because Docker has published Vertica's port 5433.

---

# 23. Test the Grafana Connection

Click:

```text
Save & test
```

A successful result should say something similar to:

```text
Successfully connected to Vertica Analytic Database v24.1.0-0
```

At this point:

```text
Grafana → Vertica
```

connectivity is proven.

---

# 24. First Grafana Query Using Explore

In Grafana:

```text
Explore
```

Select the data source:

```text
Vertica-VMart
```

Use SQL/code mode.

Set:

```text
Format as: Time Series
```

Enter:

```sql
SELECT
    store_sales_date AS time,
    SUM(sales_dollar_amount) AS sales
FROM store.store_sales_fact
WHERE $__timeFilter(store_sales_date)
GROUP BY store_sales_date
ORDER BY store_sales_date;
```

Click:

```text
Run query
```

---

# 25. Important Grafana Macro

The following is a Grafana macro:

```text
$__timeFilter(store_sales_date)
```

It makes the SQL depend on the Grafana time picker.

Conceptually:

```text
Grafana time range
       ↓
$__timeFilter(...)
       ↓
SQL WHERE condition
       ↓
Vertica
       ↓
Filtered data
       ↓
Grafana visualization
```

This is preferable to hard-coding a date range in the dashboard query.

---

# 26. Troubleshooting: "Data outside time range"

You may see:

```text
Data outside time range
```

even though the table below the graph contains data.

This does NOT necessarily mean the query failed.

It can simply mean:

```text
Query returned data
        ↓
Returned timestamps are outside
Grafana's currently selected time range
```

For example, VMart contains historical dates beginning in 2003, while Grafana may initially be displaying the current date/time range.

---

# 27. Fix the Grafana Time Range

In Explore, open the time-range control in the upper-right.

Select a custom/absolute time range such as:

```text
From:
2003-01-01 00:00:00

To:
2003-01-31 23:59:59
```

Apply the range and run the query again.

The January 2003 data should then appear in the graph.

---

# 28. Successful Result

The successful result should show a time-series graph with daily sales.

For example:

```text
Sales
300K ┤        ╭╮
250K ┤    ╭───╯╰╮
200K ┤ ╭──╯      ╰──╮
150K ┤─╯             ╰─
100K ┤
     └────────────────────
       Jan 1        Jan 31
```

The table response should contain rows similar to:

```text
time                    sales
----------------------  ------
2003-01-01 05:30:00     35980
2003-01-02 05:30:00      6531
2003-01-03 05:30:00     56003
2003-01-04 05:30:00     69503
...
```

The `05:30:00` displayed by Grafana is the timezone representation of the date value in the Grafana UI.

---

# 29. Final Working Pipeline

At this point the complete pipeline is working:

```text
                    Windows 11
                         │
        ┌────────────────┴────────────────┐
        │                                 │
      WSL2                           Docker Desktop
        │                                 │
        ▼                                 ▼
    Grafana 13.2                    Vertica CE 24.1
        │                                 │
        │ Vertica plugin                  │
        │                                 │
        └────────── localhost:5433 ───────┘
                         │
                         ▼
                       demo
                         │
                         ▼
                       VMart
                         │
                         ▼
              store.store_sales_fact
                         │
                         ▼
                       SQL
                         │
                         ▼
                Grafana Explore
                         │
                         ▼
                 Time Series Graph
```

The first complete live-data exercise is therefore:

```text
Vertica VMart
     ↓
store.store_sales_fact
     ↓
SQL aggregation by date
     ↓
Grafana Vertica Data Source
     ↓
Grafana Explore
     ↓
Time Series visualization
```

---

# 30. Commands — Copy/Paste Sequence

For a trainee who wants the shortest executable sequence after installations are already complete:

## A. Check Vertica

```bash
docker ps --format "table {{.Names}}\t{{.Ports}}"
```

```bash
docker exec -it vertica-ce /opt/vertica/bin/vsql -U dbadmin -d demo
```

Inside `vsql`:

```sql
SELECT version();
```

```sql
SELECT table_schema, table_name
FROM v_catalog.tables
WHERE is_system_table = false
ORDER BY table_schema, table_name;
```

If VMart is not loaded, exit:

```text
\q
```

## B. Load VMart

```bash
docker exec -it vertica-ce bash -lc 'cd /opt/vertica/examples/VMart_Schema && ./01_load_vmart_schema.sh'
```

## C. Verify data

```bash
docker exec -it vertica-ce /opt/vertica/bin/vsql -U dbadmin -d demo
```

Then:

```sql
SELECT
    COUNT(*) AS number_of_sales,
    SUM(sales_dollar_amount) AS total_sales
FROM store.store_sales_fact;
```

Expected in this lab:

```text
5000000 | 1363723977
```

Exit:

```text
\q
```

## D. Check Grafana

```bash
grafana-server -v
```

```bash
sudo systemctl status grafana-server --no-pager
```

```bash
ss -lntp | grep 3000
```

## E. Check/install plugin

```bash
grafana cli plugins ls | grep -i vertica
```

If missing:

```bash
sudo grafana cli plugins install vertica-grafana-datasource
```

Then:

```bash
sudo systemctl restart grafana-server
```

## F. Configure Grafana

Open:

```text
http://localhost:3000
```

Create:

```text
Data source: Vertica
Name:        Vertica-VMart
Host:        localhost:5433
Database:    demo
User:        dbadmin
Password:    <your password>
SSL Mode:    none
```

Click:

```text
Save & test
```

## G. Run the first Grafana query

Go to:

```text
Explore
```

Select:

```text
Vertica-VMart
```

Format:

```text
Time Series
```

Query:

```sql
SELECT
    store_sales_date AS time,
    SUM(sales_dollar_amount) AS sales
FROM store.store_sales_fact
WHERE $__timeFilter(store_sales_date)
GROUP BY store_sales_date
ORDER BY store_sales_date;
```

Set the time range to:

```text
2003-01-01 00:00:00
to
2003-01-31 23:59:59
```

Click:

```text
Run query
```

You should see the January 2003 sales graph.

---

# 31. Common Problems and Quick Fixes

| Problem | Likely cause | Fix |
|---|---|---|
| `No relations found` | VMart not loaded | Run `01_load_vmart_schema.sh` |
| `docker exec` interpreted as SQL | Still inside `vsql` | Type `\q` first |
| `demo->` prompt | Incomplete SQL statement | Press `Ctrl+C`, then `\q` |
| `exit` gives SQL syntax error | `exit` is not a vsql command | Use `\q` |
| `s.sale_date_key does not exist` | Wrong column name | Use `s.date_key` |
| Grafana cannot connect | Port/network/plugin issue | Verify Docker port 5433 and plugin |
| Vertica plugin not listed | Plugin not installed | Install `vertica-grafana-datasource` |
| Permission denied under `/var/lib/grafana/plugins` | Normal user lacks directory access | Use `grafana cli plugins ls` |
| `Data outside time range` | Grafana time picker excludes returned data | Select January 2003/custom range |
| Grafana service not running | Service stopped | `sudo systemctl restart grafana-server` |
| `localhost:5433` connection fails | Docker port not published | Check `docker ps` for `5433->5433` |

---

# 32. What Has Been Achieved

At the end of this procedure, the trainee has demonstrated:

```text
1. Vertica CE is running
2. Docker exposes Vertica on port 5433
3. VMart sample data is loaded
4. Vertica SQL can query the VMart data
5. Grafana is running on port 3000
6. Grafana has the Vertica datasource plugin
7. Grafana connects successfully to Vertica
8. Grafana executes SQL against Vertica
9. Grafana receives live Vertica data
10. Grafana displays that data as a time-series graph
```

This is the completed scope of the first **Vertica → Grafana live-data visualization** exercise.
