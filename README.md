import sqlite3
import tkinter as tk
from tkinter import ttk
from tkinter import messagebox
from os import path
import subprocess

# Database functions
def connect_db():
    try:
        filename = path.abspath(__file__)
        dbdir = filename.rstrip('FilmsSQLDatabase.py')  # Ensure this works correctly
        dbpath = path.join(dbdir, 'movies.db')
        print(f"Database path: {dbpath}")  # Debugging line to see where it's trying to connect

        conn = sqlite3.connect(dbpath)
        cursor = conn.cursor()
        return conn, cursor
    except sqlite3.Error as error:
        print("Tekkis viga andmebaasiga ühendamisel või päringute teostamisel:", error)
        return None, None

def load_data_from_db(tree, search_query=""):
    # Clear previous data in the Treeview
    for item in tree.get_children():
        tree.delete(item)

    # Connect to the database
    conn, cursor = connect_db()
    if not conn:
        messagebox.showerror("Viga", "Andmebaasi ühendus ebaõnnestus!")
        return
    
    # Create a query to fetch data from the database
    query = "SELECT title, director, release_year, genre, duration, rating, language, country, description FROM movies"
    if search_query:
        query += " WHERE title LIKE ?"
        cursor.execute(query, ('%' + search_query + '%',))
    else:
        cursor.execute(query)
    
    rows = cursor.fetchall()

    # Insert fetched data into the Treeview
    for row in rows:
        tree.insert("", "end", values=row)

    conn.close()

# Insert data into the database (from the other code you provided)
def insert_data():
    conn, cursor = connect_db()
    if not conn:
        messagebox.showerror("Viga", "Andmebaasi ühendus ebaõnnestus!")
        return

    title = entries["Pealkiri"].get()
    director = entries["Režissöör"].get()
    release_year = entries["Aasta"].get()
    genre = entries["Žanr"].get()
    duration = entries["Kestus"].get()
    rating = entries["Reiting"].get()
    language = entries["Keel"].get()
    country = entries["Riik"].get()
    description = entries["Kirjeldus"].get()

    if not validate_data(title, release_year, rating):
        return

    try:
        cursor.execute(insert_into, (title, director, int(release_year), genre, int(duration), float(rating), language, country, description))
        conn.commit()
        messagebox.showinfo("Edu", "Andmed sisestatud edukalt!")
    except sqlite3.Error as error:
        messagebox.showerror("Viga", f"Viga andmete sisestamisel: {error}")
    finally:
        conn.close()

def validate_data(title, release_year, rating):
    if not title:
        messagebox.showerror("Viga", "Pealkiri on kohustuslik!")
        return False
    if not release_year.isdigit():
        messagebox.showerror("Viga", "Aasta peab olema arv!")
        return False
    if rating and (not rating.replace('.', '', 1).isdigit() or not (0 <= float(rating) <= 10)):
        messagebox.showerror("Viga", "Reiting peab olema vahemikus 0 kuni 10!")
        return False
    return True

# Insert into SQL template
insert_into = """ 
INSERT INTO movies (title, director, release_year, genre, duration, rating, language, country, description) VALUES
('The Shawshank Redemption', 'Frank Darabont', 1994, 'Drama', 142, 9.3, 'English', 'USA', 'Two imprisoned men bond over a number of years.'),
('The Godfather', 'Francis Ford Coppola', 1972, 'Crime, Drama', 175, 9.2, 'English', 'USA', 'The aging patriarch of an organized crime dynasty transfers control of his empire to his reluctant son.')
;"""  # Example of some data you might already have

# Tkinter interface
def on_search():
    search_query = search_entry.get()
    load_data_from_db(tree, search_query)

def switch_to_insert_window():
    # Open the insert data window
    subprocess.run(["python", "01.py"])  # Ensure 01.py is your data input script

# Create the main Tkinter window for displaying data
root = tk.Tk()
root.title("Filmid")

# Frame for scrollable content
frame = tk.Frame(root)
frame.pack(pady=20, fill=tk.BOTH, expand=True)

scrollbar = tk.Scrollbar(frame)
scrollbar.pack(side=tk.RIGHT, fill=tk.Y)

# Treeview to display the movie data
tree = ttk.Treeview(frame, yscrollcommand=scrollbar.set, columns=("title", "director", "year", "genre", "duration", "rating", "language", "country", "description"), show="headings")
tree.pack(fill=tk.BOTH, expand=True)

scrollbar.config(command=tree.yview)

# Set column headings
tree.heading("title", text="Pealkiri")
tree.heading("director", text="Režissöör")
tree.heading("year", text="Aasta")
tree.heading("genre", text="Žanr")
tree.heading("duration", text="Kestus")
tree.heading("rating", text="Reiting")
tree.heading("language", text="Keel")
tree.heading("country", text="Riik")
tree.heading("description", text="Kirjeldus")

# Set column widths
tree.column("title", width=150)
tree.column("director", width=100)
tree.column("year", width=60)
tree.column("genre", width=100)
tree.column("duration", width=60)
tree.column("rating", width=60)
tree.column("language", width=80)
tree.column("country", width=80)
tree.column("description", width=200)

# Add search functionality
search_frame = tk.Frame(root)
search_frame.pack(pady=10)

search_label = tk.Label(search_frame, text="Otsi filmi pealkirja järgi:")
search_label.pack(side=tk.LEFT)

search_entry = tk.Entry(search_frame)
search_entry.pack(side=tk.LEFT, padx=10)

search_button = tk.Button(search_frame, text="Otsi", command=on_search)
search_button.pack(side=tk.LEFT)

# Button to switch to the data insertion window
open_button = tk.Button(root, text="Lisa andmeid", command=switch_to_insert_window)
open_button.pack(pady=20)

# Load and display data from the database
load_data_from_db(tree)

# Start the Tkinter event loop
root.mainloop()
