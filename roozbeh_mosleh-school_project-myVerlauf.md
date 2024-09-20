# Documentation for azubiVerlauf Application

This documentation provides a detailed explanation of the `azubiVerlauf` application, a Python-based GUI program designed to assist apprentices in tracking their progress. The application uses SQLite for data storage and Tkinter for the user interface, with enhancements for better performance, especially in handling background images.

## Table of Contents

1. [Overview](#overview)
2. [Imports and Database Setup](#imports-and-database-setup)
3. [Database Table Creation](#database-table-creation)
4. [User Interface Setup](#user-interface-setup)
   - [Main Window](#main-window)
   - [Background Image](#background-image)
   - [Arrival/Departure Section](#arrivaldeparture-section)
   - [Vocabulary Section](#vocabulary-section)
   - [Topics Section](#topics-section)
   - [Self-Learning Section](#self-learning-section)
   - [Export Section](#export-section)
5. [Event Handling Functions](#event-handling-functions)
   - [Arrival Function](#arrival-function)
   - [Departure Function](#departure-function)
   - [Save Vocabulary Function](#save-vocabulary-function)
   - [Save Topic Function](#save-topic-function)
   - [Save Self-Learning Function](#save-self-learning-function)
   - [Export to CSV Function](#export-to-csv-function)
6. [Main Function](#main-function)
7. [Conclusion](#conclusion)

---

## Overview

The `azubiVerlauf` application assists apprentices in:

- Recording arrival and departure times.
- Managing vocabulary terms and definitions.
- Logging topics and self-learning activities.
- Exporting data to CSV files.

Enhancements have been made for better performance, particularly in dynamic background image resizing.

---

## Imports and Database Setup

```python
import sqlite3
import csv
import tkinter as tk
import datetime
from PIL import Image, ImageTk
```

- **sqlite3**: Connects and interacts with SQLite databases.
- **csv**: Reads and writes CSV files.
- **tkinter**: Creates graphical user interfaces.
- **datetime**: Handles date and time operations.
- **PIL (Pillow)**: Handles image processing, specifically for dynamic resizing.

---

## Database Table Creation

### Database Connection

```python
sqliteDb = 'myVerlauf.db'   # Assign the database
conn = sqlite3.connect(sqliteDb)  # Connect to the database
cursor = conn.cursor()
```

- **sqliteDb**: Database file name.
- **conn**: Establishes a connection to the SQLite database.
- **cursor**: Executes SQL commands.

### Creating Tables

#### eintragung Table

Records arrival and departure times.

```python
create_table_eintragung = """
CREATE TABLE IF NOT EXISTS eintragung (
    id INTEGER PRIMARY KEY AUTOINCREMENT, 
    datum DATETIME DEFAULT CURRENT_TIMESTAMP, 
    kategorie TEXT,
    dozent TEXT,
    notiz TEXT 
);
"""
cursor.execute(create_table_eintragung)
```

#### themen Table

Stores topics, categories, and notes.

```python
create_table_themen = """
CREATE TABLE IF NOT EXISTS themen (
    id INTEGER PRIMARY KEY AUTOINCREMENT, 
    datum DATETIME DEFAULT CURRENT_TIMESTAMP, 
    thema TEXT, 
    kategorie TEXT,
    notiz TEXT
);
"""
cursor.execute(create_table_themen)
```

#### selbstlearn Table

Logs self-learning activities.

```python
create_table_selbstlearn = """
CREATE TABLE IF NOT EXISTS selbstlearn (
    id INTEGER PRIMARY KEY AUTOINCREMENT, 
    datum DATETIME DEFAULT CURRENT_TIMESTAMP, 
    thema TEXT, 
    kategorie TEXT,
    notiz TEXT
);
"""
cursor.execute(create_table_selbstlearn)
```

#### wortschatz Table

Manages vocabulary terms and definitions.

```python
create_table_wortschatz = """
CREATE TABLE IF NOT EXISTS wortschatz (
    id INTEGER PRIMARY KEY AUTOINCREMENT, 
    datum DATETIME DEFAULT CURRENT_TIMESTAMP, 
    title TEXT, 
    definition TEXT, 
    kategorie TEXT,
    notiz TEXT  
);
"""
cursor.execute(create_table_wortschatz)
```

### Commit and Close Connection

```python
conn.commit()
conn.close()
print(f"Datenbank wurde eingerichtet: '{sqliteDb}'")
```

- **Commit**: Saves changes to the database.
- **Close**: Closes the database connection.

Re-establish the connection for upcoming functions:

```python
conn = sqlite3.connect(sqliteDb)
cursor = conn.cursor()
```

---

## User Interface Setup

### Main Window

```python
def hauptFenster():
    haupt_fenster = tk.Tk()
    haupt_fenster.title("azubiVerlauf")
    haupt_fenster.maxsize(4000, 3000)
    haupt_fenster.minsize(800, 1000)
    return haupt_fenster
```

- **Creates**: The main application window.
- **Sets**: Title and size constraints.

### Background Image

```python
def hintergrundBild(haupt_fenster):
    hintergrund_bild_path = "resources/image/myVerlauf-Hauptfenster.png"
    hintergrund_bild = Image.open(hintergrund_bild_path)  # Load image with Pillow

    def resize_background(event):
        # Avoid frequent resizing by cancelling previous resize calls
        if hasattr(resize_background, 'resize_id'):
            haupt_fenster.after_cancel(resize_background.resize_id)
        # Schedule resizing after a delay
        resize_background.resize_id = haupt_fenster.after(100, lambda: _resize(event))

    def _resize(event):
        new_width = event.width
        new_height = event.height
        hintergrund_bild_resized = hintergrund_bild.resize((new_width, new_height), Image.LANCZOS)
        hintergrund_photo = ImageTk.PhotoImage(hintergrund_bild_resized)
        # Update the label with the new image
        hintergrund_label.config(image=hintergrund_photo)
        hintergrund_label.image = hintergrund_photo  # Keep reference to prevent garbage collection

    # Create a Label to hold the background image
    hintergrund_label = tk.Label(haupt_fenster)
    hintergrund_label.place(x=0, y=0, relwidth=1, relheight=1)
    # Bind the resize event to dynamically resize the background image
    haupt_fenster.bind("<Configure>", resize_background)
```

- **Enhancement**: Uses Pillow to dynamically resize the background image when the window size changes.
- **Avoids**: Excessive resizing by introducing a delay using `after_cancel` and `after`.

### Arrival/Departure Section

```python
def an_ab_Abschnitt(haupt_fenster):
    an_ab_fenster = tk.Frame(haupt_fenster, bg="#007A7A")
    an_ab_fenster.place(relx=0.05, rely=0.2, relwidth=0.32, relheight=0.22)
    # Configure grid
    an_ab_fenster.grid_columnconfigure(0, weight=1)
    an_ab_fenster.grid_columnconfigure(1, weight=1)
    # Add widgets
    tk.Label(an_ab_fenster, text="Ankunfts- und Abgang", font=("Helvetica", 13, "bold"), fg="black", bg="#007A7A").grid(...)
    tk.Button(an_ab_fenster, text="Ankunft", font=("Helvetica", 10, "bold"), fg="black", bg="#009999", command=ankunft_function).grid(...)
    tk.Button(an_ab_fenster, text="Abgang", font=("Helvetica", 10, "bold"), fg="black", bg="#009999", command=abgang_func).grid(...)
```

- **Purpose**: Allows users to log arrival and departure times.
- **Components**:
  - **Labels**: For section title.
  - **Buttons**: "Ankunft" (Arrival) and "Abgang" (Departure).
- **Visual Update**: Text color changed to black for better contrast.

### Vocabulary Section

```python
def wortschatz_abschnitt(haupt_fenster):
    global entry_wort_title, entry_definition
    wortschatz_fenster = tk.Frame(haupt_fenster, bg="#007A7A")
    wortschatz_fenster.place(relx=0.05, rely=0.45, relwidth=0.32, relheight=0.22)
    # Configure grid
    wortschatz_fenster.grid_columnconfigure(0, weight=1)
    wortschatz_fenster.grid_columnconfigure(1, weight=1)
    # Add widgets
    tk.Label(wortschatz_fenster, text="Wortschatz", font=("Helvetica", 13, "bold"), fg="black", bg="#007A7A").grid(...)
    tk.Label(wortschatz_fenster, text="Begriff", font=("Helvetica", 10, "bold"), fg="black", bg="#007A7A").grid(...)
    entry_wort_title = tk.Entry(wortschatz_fenster)
    entry_wort_title.grid(...)
    tk.Label(wortschatz_fenster, text="Definition", font=("Helvetica", 10, "bold"), fg="black", bg="#007A7A").grid(...)
    entry_definition = tk.Entry(wortschatz_fenster)
    entry_definition.grid(...)
    tk.Button(wortschatz_fenster, text="Speichern", font=("Helvetica", 10, "bold"), fg="black", bg="#009999", command=wortschatz_speichern).grid(...)
```

- **Purpose**: Allows users to add vocabulary terms and definitions.
- **Components**:
  - **Entry Fields**: For term and definition.
  - **Button**: "Speichern" (Save) to save the entry.
- **Visual Update**: Text color changed to black.

### Topics Section

```python
def themen_abschnitt(haupt_fenster):
    themen_fenster = tk.Frame(haupt_fenster, bg="#007A7A")
    themen_fenster.place(relx=0.4, rely=0.2, relwidth=0.55, relheight=0.225)
    # Configure grid
    themen_fenster.grid_columnconfigure(0, weight=1)
    themen_fenster.grid_columnconfigure(1, weight=1)
    # Add widgets
    tk.Label(themen_fenster, text="Themen", font=("Helvetica", 13, "bold"), fg="black", bg="#007A7A").grid(...)
    tk.Label(themen_fenster, text="Thema", font=("Helvetica", 10, "bold"), fg="black", bg="#007A7A").grid(...)
    entry_thema = tk.Entry(themen_fenster)
    entry_thema.grid(...)
    tk.Label(themen_fenster, text="Kategorie", font=("Helvetica", 10, "bold"), fg="black", bg="#007A7A").grid(...)
    entry_kategorie = tk.Entry(themen_fenster)
    entry_kategorie.grid(...)
    tk.Label(themen_fenster, text="Notiz", font=("Helvetica", 10, "bold"), fg="black", bg="#007A7A").grid(...)
    entry_notiz = tk.Entry(themen_fenster)
    entry_notiz.grid(...)
    tk.Button(themen_fenster, text="Speichern", font=("Helvetica", 10, "bold"), fg="black", bg="#009999", command=lambda: themen_speichern_funktion(entry_thema, entry_kategorie, entry_notiz)).grid(...)
```

- **Purpose**: Enables users to log topics with categories and notes.
- **Components**:
  - **Entry Fields**: For topic, category, and note.
  - **Button**: "Speichern" to save the topic.
- **Visual Update**: Text color changed to black.

### Self-Learning Section

```python
def selbstlern_abschnitt(haupt_fenster):
    selbstlern_fenster = tk.Frame(haupt_fenster, bg="#007A7A")
    selbstlern_fenster.place(relx=0.4, rely=0.45, relwidth=0.55, relheight=0.225)
    # Configure grid
    selbstlern_fenster.grid_columnconfigure(0, weight=1)
    selbstlern_fenster.grid_columnconfigure(1, weight=1)
    # Add widgets
    tk.Label(selbstlern_fenster, text="Selbstlern", font=("Helvetica", 13, "bold"), fg="black", bg="#007A7A").grid(...)
    tk.Label(selbstlern_fenster, text="Thema", font=("Helvetica", 10, "bold"), fg="black", bg="#007A7A").grid(...)
    entry_thema = tk.Entry(selbstlern_fenster)
    entry_thema.grid(...)
    tk.Label(selbstlern_fenster, text="Kategorie", font=("Helvetica", 10, "bold"), fg="black", bg="#007A7A").grid(...)
    entry_kategorie = tk.Entry(selbstlern_fenster)
    entry_kategorie.grid(...)
    tk.Label(selbstlern_fenster, text="Notiz", font=("Helvetica", 10, "bold"), fg="black", bg="#007A7A").grid(...)
    entry_notiz = tk.Entry(selbstlern_fenster)
    entry_notiz.grid(...)
    tk.Button(selbstlern_fenster, text="Speichern", font=("Helvetica", 10, "bold"), fg="black", bg="#009999", command=lambda: selbstlern_speichern_funktion(entry_thema, entry_kategorie, entry_notiz)).grid(...)
```

- **Purpose**: Logs self-learning topics.
- **Components**:
  - Similar to the topics section.
  - **Entry Fields**: For self-learning topic, category, and note.
  - **Button**: "Speichern" to save the entry.
- **Visual Update**: Text color changed to black.

### Export Section

```python
def export_abschnitt(haupt_fenster):
    export_fenster = tk.Frame(haupt_fenster, bg="#007A7A")
    export_fenster.place(relx=0.05, rely=0.7, relwidth=0.9, relheight=0.22)
    # Configure grid
    export_fenster.grid_columnconfigure(0, weight=1)
    export_fenster.grid_columnconfigure(1, weight=1)
    # Add widgets
    tk.Label(export_fenster, text="Export", font=("Helvetica", 13, "bold"), fg="black", bg="#007A7A").grid(...)
    tk.Label(export_fenster, text="Exportieren Ihre Daten in CSV Datei", font=("Helvetica", 10, "bold"), fg="black", bg="#007A7A").grid(...)
    tk.Button(export_fenster, text="Selbstlern Daten Exportieren", font=("Helvetica", 10, "bold"), fg="black", bg="#009999", command=lambda: sqliteDb_to_csv(Selbstlern)).grid(...)
    tk.Button(export_fenster, text="Themen Daten Exportieren", font=("Helvetica", 10, "bold"), fg="black", bg="#009999", command=lambda: sqliteDb_to_csv(themen)).grid(...)
    tk.Button(export_fenster, text="Wortschatz Datei Exportieren", font=("Helvetica", 10, "bold"), fg="black", bg="#009999", command=lambda: sqliteDb_to_csv(wortschatz)).grid(...)
    tk.Button(export_fenster, text="Ankunft/Abgang Daten Exportieren", font=("Helvetica", 10, "bold"), fg="black", bg="#009999", command=lambda: sqliteDb_to_csv(eintragung)).grid(...)
```

- **Purpose**: Provides options to export data to CSV files.
- **Components**:
  - **Buttons**: For exporting each data category.
- **Visual Update**: Text color changed to black.

---

## Event Handling Functions

### Arrival Function

```python
def ankunft_function():
    current_datetime = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    values = {
        'datum': current_datetime,
        'kategorie': 'eingang'
    }
    columns = ', '.join(values.keys())
    placeholders = ', '.join('?' for _ in values)
    query = f"INSERT INTO eintragung ({columns}) VALUES ({placeholders})"
    cursor.execute(query, tuple(values.values()))
    conn.commit()
    print("Daten in 'eintragung' eingefügt:", values)
```

- **Records**: The current date and time as an arrival entry.
- **Inserts**: Into the `eintragung` table with category 'eingang'.

### Departure Function

```python
def abgang_func():
    current_datetime = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    values = {
        'datum': current_datetime,
        'kategorie': 'ausgang'
    }
    columns = ', '.join(values.keys())
    placeholders = ', '.join('?' for _ in values)
    query = f"INSERT INTO eintragung ({columns}) VALUES ({placeholders})"
    cursor.execute(query, tuple(values.values()))
    conn.commit()
    print("Daten in 'eintragung' eingefügt:", values)
```

- **Records**: The current date and time as a departure entry.
- **Inserts**: Into the `eintragung` table with category 'ausgang'.

### Save Vocabulary Function

```python
def wortschatz_speichern():
    wort_title = entry_wort_title.get()
    definition = entry_definition.get()
    wortschatz_speichern_funktion(wort_title, definition)
```

- **Fetches**: Input from the vocabulary entry fields.
- **Calls**: The function to save data to the database.

```python
def wortschatz_speichern_funktion(wort_title, definition):
    with sqlite3.connect(sqliteDb) as conn:
        cursor = conn.cursor()
        cursor.execute("INSERT INTO wortschatz (title, definition) VALUES (?, ?)", (wort_title, definition))
        conn.commit()
```

- **Inserts**: The term and definition into the `wortschatz` table.

### Save Topic Function

```python
def themen_speichern_funktion(entry_thema, entry_kategorie, entry_notiz):
    thema = entry_thema.get()
    kategorie = entry_kategorie.get()
    notiz = entry_notiz.get()
    with sqlite3.connect(sqliteDb) as conn:
        cursor = conn.cursor()
        cursor.execute("INSERT INTO themen (thema, kategorie, notiz) VALUES (?, ?, ?)", (thema, kategorie, notiz))
        conn.commit()
```

- **Fetches**: Inputs from the topics section.
- **Inserts**: Data into the `themen` table.

### Save Self-Learning Function

```python
def selbstlern_speichern_funktion(entry_thema, entry_kategorie, entry_notiz):
    thema = entry_thema.get()
    kategorie = entry_kategorie.get()
    notiz = entry_notiz.get()
    with sqlite3.connect(sqliteDb) as conn:
        cursor = conn.cursor()
        cursor.execute("INSERT INTO selbstlearn (thema, kategorie, notiz) VALUES (?, ?, ?)", (thema, kategorie, notiz))
        conn.commit()
```

- **Fetches**: Inputs from the self-learning section.
- **Inserts**: Data into the `selbstlearn` table.

### Export to CSV Function

```python
def sqliteDb_to_csv(table):
    cursor.execute(f"SELECT * FROM {table}")
    rows = cursor.fetchall()
    with open(f"{table}.csv", 'w', newline='') as file:
        writer = csv.writer(file)
        writer.writerow([i[0] for i in cursor.description])  # Header row
        writer.writerows(rows)
```

- **Exports**: Data from the specified table to a CSV file.
- **Parameters**:
  - **table**: Name of the table to export.

Define table names:

```python
Selbstlern = 'selbstlearn'
wortschatz = 'wortschatz'
themen = 'themen'
eintragung = 'eintragung'
```

---

## Main Function

```python
def main():
    fenster = hauptFenster()
    hintergrundBild(fenster)
    an_ab_Abschnitt(fenster)
    wortschatz_abschnitt(fenster)
    themen_abschnitt(fenster)
    selbstlern_abschnitt(fenster)
    export_abschnitt(fenster)
    fenster.mainloop()
```

- **Initializes**: The main window and adds all UI sections.
- **Starts**: The Tkinter event loop with `mainloop()`.

```python
if __name__ == "__main__":
    main()
```

- **Ensures**: The `main()` function runs when the script is executed directly.

---

## Conclusion

The `azubiVerlauf` application is a comprehensive tool for apprentices to:

- Track arrival and departure times.
- Manage vocabulary and definitions.
- Record topics and self-learning activities.
- Export their data for reporting and analysis.

**Enhancements in the Final Version**:

- **Dynamic Background Resizing**: Improved performance using the Pillow library to handle background image resizing efficiently.
- **Visual Updates**: Changed text colors to black for better readability against the background.
- **Optimized Event Handling**: Reduced unnecessary processing by throttling resize events.

**Notes**:

- Ensure the database file `myVerlauf.db` is accessible and located in the working directory.
- Verify that the image file `myVerlauf-Hauptfenster.png` exists at the specified path (`resources/image/`).
- Install the Pillow library (`PIL`) if not already installed:
  ```bash
  pip install Pillow
  ```
- Consider adding exception handling to manage potential errors, such as missing files or database connection issues.

---

This documentation provides a clear and structured explanation of the updated code, making it easier to understand, maintain, and extend the `azubiVerlauf` application.