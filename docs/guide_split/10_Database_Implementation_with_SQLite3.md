# 10. Database Implementation with SQLite3

Lexi will incorporate two SQLite3 databases to manage user-specific data and application usage metrics. SQLite3 is an embedded, serverless, zero-configuration, transactional SQL database engine, making it an ideal choice for local data storage within applications like Lexi. Its simplicity and file-based nature ensure easy deployment and portability, as the entire database is contained within a single file.


### 10.1. Overview of SQLite3 in Lexi

SQLite3 databases will be stored directly alongside the Lexi application, providing a robust yet lightweight solution for persistent data storage. This approach eliminates the need for a separate database server, simplifying the application's architecture and reducing operational overhead. The two distinct databases will serve specific purposes:

**1.User Email Database**: This database will securely store user email addresses, primarily for authentication, communication, or personalized features within Lexi. It will ensure that user contact information is managed efficiently and locally.

**2.Usage Data Database**: This database will collect specific usage data from Lexi when the application is used offline. This data is crucial for understanding user interaction patterns, identifying popular features, and improving the overall user experience. Once an internet connection is available, this usage information will be automatically uploaded to an admin dashboard for centralized viewing and analysis.



### 10.2. Core Concepts for SQLite3 Implementation

Implementing SQLite3 in Python is straightforward, primarily utilizing the built-in sqlite3 module. The fundamental steps involve connecting to a database, creating tables, inserting data, querying data, and closing the connection. For Lexi, these operations will need to be integrated carefully to ensure data integrity and application responsiveness.

**Connecting to a Database**:

To interact with an SQLite3 database, you first need to establish a connection. If the specified database file does not exist, SQLite3 will automatically create it. This connection object is your gateway to performing database operations.

```python
import sqlite3

def get_db_connection(db_name):
    conn = sqlite3.connect(db_name)
    conn.row_factory = sqlite3.Row  # This allows accessing columns by name
    return conn
```

**Creating Tables**:

Once connected, you can create tables to define the structure of your data. It's good practice to check if a table already exists before attempting to create it, using CREATE TABLE IF NOT EXISTS.

```python
def create_tables(conn):
    cursor = conn.cursor()
    # Table for user emails
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS users (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            email TEXT NOT NULL UNIQUE,
            created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
        )
    ''')
    # Table for offline usage data
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS usage_data (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            user_id INTEGER,
            feature_used TEXT NOT NULL,
            timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
            data_payload TEXT, # e.g., JSON string of specific usage details
            uploaded BOOLEAN DEFAULT 0, # 0 for not uploaded, 1 for uploaded
            FOREIGN KEY (user_id) REFERENCES users(id)
        )
    ''')
    conn.commit()
```
**Inserting Data**:

Data can be inserted into tables using SQL INSERT statements. It's crucial to use parameterized queries (e.g., ? placeholders) to prevent SQL injection vulnerabilities.

```python
def insert_user_email(conn, email):
    cursor = conn.cursor()
    try:
        cursor.execute("INSERT INTO users (email) VALUES (?)


, (email))


, (email))"
        cursor.execute("INSERT INTO users (email) VALUES (?)", (email,))
        conn.commit()
        return cursor.lastrowid
    except sqlite3.IntegrityError:
        print(f"Email {email} already exists.")
        return None

def insert_usage_data(conn, user_id, feature_used, data_payload=None):
    cursor = conn.cursor()
    cursor.execute(
        "INSERT INTO usage_data (user_id, feature_used, data_payload) VALUES (?, ?, ?)",
        (user_id, feature_used, data_payload)
    )
    conn.commit()
    return cursor.lastrowid
```

**Querying Data**:

Retrieving data is done using SELECT statements. The conn.row_factory = sqlite3.Row setting is particularly useful as it allows accessing columns by name, similar to a dictionary.

```python
def get_user_by_email(conn, email):
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM users WHERE email = ?", (email,))
    return cursor.fetchone()

def get_unuploaded_usage_data(conn):
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM usage_data WHERE uploaded = 0")
    return cursor.fetchall()
```
**Updating Data**:

To mark usage data as uploaded, an UPDATE statement can be used.

```python
def mark_usage_data_uploaded(conn, data_ids):
    cursor = conn.cursor()
    # Using a placeholder for each ID to prevent SQL injection
    placeholders = ", ".join(["?" for _ in data_ids])
    cursor.execute(f"UPDATE usage_data SET uploaded = 1 WHERE id IN ({placeholders})", data_ids)
    conn.commit()
```
### 10.3.  Integration Points in Drift_mini.py

Given the current structure of Drift_mini.py, the database functionalities will need to be integrated at several key points. The goal is to ensure that database operations are performed efficiently and without disrupting the existing UI and AI logic.

### 1. Application Initialization (main function)**:

Upon application startup, the database connections should be established, and tables should be created if they don't already exist. This ensures that the database is ready for use as soon as Lexi launches.

**•Proposed Location**: Within the main function, early in the execution flow, perhaps after the initial Flet page setup and before model loading.

**•Example Integration**:

### 2. User Email Collection (e.g., during first-time setup or registration):

If Lexi introduces a feature requiring user email input (e.g., for newsletters, account creation, or personalized settings), this is where the insert_user_email function would be called.

**•Proposed Location**: Within a new Flet dialog or form that prompts the user for their email address. This would likely be triggered by a button click or as part of an onboarding flow.

**•Example Integration (Conceptual)**:

### 3. Usage Data Logging (various interaction points):

Every significant user interaction or feature usage within Lexi should trigger a call to log data into the usage_data.db. This includes, but is not limited to:

**•Chat interactions**: Each time a user sends a message or receives a response.

**•PDF processing**: When a PDF is loaded, processed, or summarized.

**•Model changes**: When the user switches between different AI models.

**•Tool usage**: If Lexi integrates external tools or functionalities.

**•Proposed Location**: Within the respective event handlers or functions that manage these interactions. For instance, in the send_message function for chat, or within the PDF loading logic.

**•Example Integration (Conceptual for chat)**:

### 4. Data Upload Mechanism (background task or on app close):

A mechanism is needed to periodically check for unuploaded usage data and send it to the admin dashboard. This could be a background thread that runs at intervals or a process that triggers when the application is closed or an internet connection is detected.

**•Proposed Location**: A separate asynchronous function that can be called periodically (e.g., using asyncio.sleep within a loop) or triggered by network status changes. It should also be considered for execution during graceful application shutdown.

**•Example Integration (Conceptual):**




