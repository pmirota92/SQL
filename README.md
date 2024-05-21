# Examples of Solutions to SQL Tasks Using Python

This Jupyter Notebook provides examples of how to use SQL queries within Python to analyze transaction data stored in an Excel file. The notebook demonstrates how to extract, transform, and load the data into an SQLite database and perform various SQL queries to answer specific business questions.

## Datasets

The data is contained in the `Transactions.xlsx` file, which includes the following sheets:
- **Trans_Line**: Transaction lines, detailing individual items in each transaction.
- **Trans_Header**: Transaction headers, summarizing each transaction.
- **Store**: Information about stores.
- **Item**: Information about items in inventory.
- **Description**: A description of the tables.

## Analysis Tasks

We will prepare SQL queries to answer the following questions:
1. **Share of transactions by sales channel in stream A that contain at least two products**: Only sales lines are considered, excluding returns.
2. **Brand with the highest share of returns in Poland**.
3. **Product with the highest percentage margin in stream C from July 5-15, 2019**: Only sales lines are considered, excluding returns.
4. **Margin that the PL country would achieve without price reductions**.
5. **Share of all transactions in July in stream A in individual countries that include both category B and K inventories**: Only sales lines are considered, excluding returns.

## Steps to Load Data and Perform Analysis

### 1. Import Libraries

```python
import pandas as pd
import sqlite3
```

### 2. Load Data from Excel

```python
TransLine = pd.read_excel('Transactions.xlsx', sheet_name='Trans_Line', header=0).dropna()
TransHeader = pd.read_excel('Transactions.xlsx', sheet_name='Trans_Header', header=0).dropna()
Store = pd.read_excel('Transactions.xlsx', sheet_name='Store', header=0).dropna()
Item = pd.read_excel('Transactions.xlsx', sheet_name='Item', header=0).dropna()
```

### 3. Create SQLite Database and Tables

```python
db_conn = sqlite3.connect("TransNew1.db")
c = db_conn.cursor()

# Create Store Table
c.execute("""
    CREATE TABLE Store (
        store_no TEXT NOT NULL,
        Stream TEXT,
        Region INTEGER,
        Channel TEXT,
        Country TEXT,
        PRIMARY KEY(store_no)
    );
""")

# Create Item Table
c.execute("""
    CREATE TABLE Item (
        item_no TEXT,
        Category_Code TEXT,
        Brand_Code TEXT,
        PRIMARY KEY(item_no)
    );
""")

# Create TransHeader Table
c.execute("""
    CREATE TABLE TransHeader (
        store_no TEXT NOT NULL,
        transaction_no TEXT NOT NULL,
        status TEXT NOT NULL,
        date TEXT NOT NULL
    );
""")

# Create TransLine Table
c.execute("""
    CREATE TABLE TransLine (
        store_no TEXT NOT NULL,
        transaction_no TEXT NOT NULL,
        item_no TEXT NOT NULL,
        quantity INTEGER NOT NULL,
        net_amount REAL NOT NULL,
        cost_amount REAL NOT NULL,
        discount_amount REAL NOT NULL
    );
""")
```

### 4. Insert Data into Tables

```python
Store.to_sql('Store', db_conn, if_exists='append', index=False)
Item.to_sql('Item', db_conn, if_exists='append', index=False)
TransHeader.to_sql('TransHeader', db_conn, if_exists='append', index=False)
TransLine.to_sql('TransLine', db_conn, if_exists='append', index=False)
```

### 5. Execute SQL Queries

#### a) Share of transactions by sales channel in stream A that contain at least two products

```python
zad_a = ("""
    SELECT Store.Channel,
        (SELECT COUNT(DISTINCT TransLine.transaction_no)
         FROM TransLine
         JOIN Store ON TransLine.store_no = Store.store_no
         WHERE Store.Stream = 'A' AND TransLine.quantity >= 2
         GROUP BY Store.Channel) AS Over2,
        COUNT(DISTINCT TransLine.transaction_no) AS TransCountTotal
    FROM TransLine
    JOIN Store ON TransLine.store_no = Store.store_no
    WHERE Store.Stream = 'A'
    GROUP BY Store.Channel
""")
pd.read_sql(zad_a, db_conn)
```

## Additional Queries

The notebook can be extended to include SQL queries for the other tasks mentioned. Each query will follow a similar pattern, joining relevant tables and applying necessary conditions to filter and aggregate the data.

---

This README provides a brief overview of how to set up and analyze transaction data using SQL within a Python environment. For detailed code and further analysis, refer to the provided Jupyter Notebook.
