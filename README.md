# US-Patent-Document-Template
Location to store standard templates for various types of patents. Open to all to clone or add more templates.
import sqlite3
from datetime import datetime, timedelta

# Database setup
db_connection = sqlite3.connect('sync_log.db')
cursor = db_connection.cursor()

# Create tables if they don't exist
cursor.execute('''
    CREATE TABLE IF NOT EXISTS sync_log (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        start_time TIMESTAMP,
        end_time TIMESTAMP
    )
''')

cursor.execute('''
    CREATE TABLE IF NOT EXISTS last_sync_time (
        id INTEGER PRIMARY KEY CHECK (id = 1), -- ensures only one record
        last_start_time TIMESTAMP
    )
''')
db_connection.commit()

def start_sync():
    """Logs start time, checks last sync, executes sync logic, and logs end time."""
    # Get the current system time as the start time
    start_time = datetime.now()

    # Check for the last sync time in the last_sync_time table
    cursor.execute("SELECT last_start_time FROM last_sync_time WHERE id = 1")
    last_sync = cursor.fetchone()
    
    if last_sync:
        last_sync_time = last_sync[0]
        last_sync_time = datetime.strptime(last_sync_time, "%Y-%m-%d %H:%M:%S.%f")
        time_difference = start_time - last_sync_time
        print(f"Time since last sync: {time_difference}")
    else:
        print("This is the first sync or no previous sync record found.")

    # Log the start time in sync_log history
    cursor.execute("INSERT INTO sync_log (start_time) VALUES (?)", (start_time,))
    db_connection.commit()

    # Update last_sync_time table with the new start time
    cursor.execute("INSERT OR REPLACE INTO last_sync_time (id, last_start_time) VALUES (1, ?)", (start_time,))
    db_connection.commit()

    # Simulate sync operation (replace this with your actual sync logic)
    print("Executing sync operation...")

    # Log the end time in sync_log history
    end_time = datetime.now()
    cursor.execute("UPDATE sync_log SET end_time = ? WHERE start_time = ?", (end_time, start_time))
    db_connection.commit()
    print("Sync completed.")

try:
    # Run the sync
    start_sync()
finally:
    # Close the database connection
    db_connection.close()
