# Vehicle-Rental-System
import tkinter as tk
from tkinter import ttk, messagebox
import sqlite3
from datetime import datetime, timedelta


class VehicleRentalApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Vehicle Rental System")
        self.root.geometry("600x500")
        self.create_database()
        self.setup_login()

    def create_database(self):
        conn = sqlite3.connect('vehicle_rental.db')
        c = conn.cursor()

        # Create users table
        c.execute('''CREATE TABLE IF NOT EXISTS users
                    (id INTEGER PRIMARY KEY,
                     username TEXT UNIQUE,
                     password TEXT,
                     role TEXT)''')

        # Create vehicles table
        c.execute('''CREATE TABLE IF NOT EXISTS vehicles
                    (id INTEGER PRIMARY KEY,
                     type TEXT,
                     model TEXT,
                     number TEXT,
                     status TEXT,
                     base_price REAL,
                     description TEXT)''')

        # Create bookings table
        c.execute('''CREATE TABLE IF NOT EXISTS bookings
                    (id INTEGER PRIMARY KEY,
                     vehicle_id INTEGER,
                     customer_id INTEGER,
                     start_date TEXT,
                     end_date TEXT,
                     total_price REAL,
                     status TEXT,
                     payment_status TEXT)''')

        # Insert vehicle details with prices for 1 day
        vehicle_data = [
            ("Scooty", "Honda Activa", "KA01AB1234", "available", 1000, "2-seater, great for city travel"),
            ("Bike", "Yamaha FZ", "KA02CD5678", "available", 1000, "Stylish bike for adventure"),
            ("Car", "Honda City", "KA03EF9101", "available", 3000, "Comfortable family car"),
            ("Truck", "Tata 407", "KA04GH1122", "available", 10000, "Heavy-duty truck for transportation"),
            ("Tractor", "Mahindra 475", "KA05IJ2233", "available", 2000, "Powerful tractor for farming")
        ]

        c.executemany("INSERT OR IGNORE INTO vehicles (type, model, number, status, base_price, description) VALUES (?, ?, ?, ?, ?, ?)", vehicle_data)

        conn.commit()
        conn.close()

    def setup_login(self):
        self.login_frame = ttk.Frame(self.root, padding=20, style="TFrame")
        self.login_frame.pack(pady=20)

        login_container = ttk.LabelFrame(self.login_frame, text="Login", padding=10)
        login_container.grid(padx=20, pady=20)

        ttk.Label(login_container, text="Username:", style="TLabel").grid(row=0, column=0, padx=5, pady=5)
        self.username_entry = ttk.Entry(login_container)
        self.username_entry.grid(row=0, column=1, padx=5, pady=5)

        ttk.Label(login_container, text="Password:", style="TLabel").grid(row=1, column=0, padx=5, pady=5)
        self.password_entry = ttk.Entry(login_container, show="*")
        self.password_entry.grid(row=1, column=1, padx=5, pady=5)

        ttk.Button(login_container, text="Login", command=self.login).grid(row=2, column=0, columnspan=2, pady=10)
        ttk.Button(login_container, text="Register", command=self.show_registration).grid(row=3, column=0, columnspan=2, pady=5)

    def show_registration(self):
        self.login_frame.pack_forget()
        self.reg_frame = ttk.Frame(self.root, padding=20, style="TFrame")
        self.reg_frame.pack(pady=20)

        reg_container = ttk.LabelFrame(self.reg_frame, text="Register", padding=10)
        reg_container.grid(padx=20, pady=20)

        ttk.Label(reg_container, text="Username:", style="TLabel").grid(row=0, column=0, padx=5, pady=5)
        self.reg_username = ttk.Entry(reg_container)
        self.reg_username.grid(row=0, column=1, padx=5, pady=5)

        ttk.Label(reg_container, text="Password:", style="TLabel").grid(row=1, column=0, padx=5, pady=5)
        self.reg_password = ttk.Entry(reg_container, show="*")
        self.reg_password.grid(row=1, column=1, padx=5, pady=5)

        ttk.Label(reg_container, text="Confirm Password:", style="TLabel").grid(row=2, column=0, padx=5, pady=5)
        self.reg_confirm = ttk.Entry(reg_container, show="*")
        self.reg_confirm.grid(row=2, column=1, padx=5, pady=5)

        ttk.Button(reg_container, text="Register", command=self.register_user).grid(row=3, column=0, columnspan=2, pady=10)
        ttk.Button(reg_container, text="Back to Login", command=self.back_to_login).grid(row=4, column=0, columnspan=2, pady=5)

    def register_user(self):
        username = self.reg_username.get()
        password = self.reg_password.get()
        confirm = self.reg_confirm.get()

        if not all([username, password, confirm]):
            messagebox.showerror("Error", "All fields are required!")
            return

        if password != confirm:
            messagebox.showerror("Error", "Passwords do not match!")
            return

        conn = sqlite3.connect('vehicle_rental.db')
        c = conn.cursor()

        try:
            c.execute("INSERT INTO users (username, password, role) VALUES (?, ?, ?)", (username, password, 'customer'))
            conn.commit()
            messagebox.showinfo("Success", "Registration Successful!")
            self.back_to_login()
        except sqlite3.IntegrityError:
            messagebox.showerror("Error", "Username already exists.")

        conn.close()

    def back_to_login(self):
        self.reg_frame.pack_forget()
        self.login_frame.pack(pady=20)

    def login(self):
        username = self.username_entry.get()
        password = self.password_entry.get()

        conn = sqlite3.connect('vehicle_rental.db')
        c = conn.cursor()

        c.execute("SELECT password, id FROM users WHERE username = ?", (username,))
        result = c.fetchone()

        if result and result[0] == password:
            self.current_user_id = result[1]
            self.login_frame.pack_forget()
            self.setup_main_application()
            self.main_frame.pack(fill=tk.BOTH, expand=True)
        else:
            messagebox.showerror("Error", "Invalid credentials!")

        conn.close()

    def setup_main_application(self):
        self.main_frame = ttk.Frame(self.root, padding=20, style="TFrame")
        self.notebook = ttk.Notebook(self.main_frame)
        self.notebook.pack(pady=10, expand=True)

        self.setup_customer_tabs()

    def setup_customer_tabs(self):
        browse_tab = ttk.Frame(self.notebook)
        bookings_tab = ttk.Frame(self.notebook)
        rent_tab = ttk.Frame(self.notebook)
        logout_tab = ttk.Frame(self.notebook)

        self.notebook.add(browse_tab, text="Browse Vehicles")
        self.notebook.add(bookings_tab, text="My Bookings")
        self.notebook.add(rent_tab, text="Rent Vehicle")
        self.notebook.add(logout_tab, text="Logout")

        self.setup_vehicle_browsing(browse_tab)
        self.setup_customer_bookings(bookings_tab)
        self.setup_rent_vehicle(rent_tab)
        self.setup_logout(logout_tab)

    def setup_vehicle_browsing(self, tab):
        ttk.Label(tab, text="Select Vehicle Type:").pack(pady=10)

        self.vehicle_type_var = tk.StringVar()
        self.vehicle_type_dropdown = ttk.Combobox(tab, textvariable=self.vehicle_type_var, state="readonly")
        self.vehicle_type_dropdown['values'] = ["Scooty", "Bike", "Car", "Truck", "Tractor"]
        self.vehicle_type_dropdown.pack(pady=5)

        ttk.Button(tab, text="Search", command=self.load_vehicle_list).pack(pady=5)
        self.vehicle_listbox = tk.Listbox(tab)
        self.vehicle_listbox.pack(pady=10, fill=tk.BOTH)

    def setup_customer_bookings(self, tab):
        ttk.Label(tab, text="Your Bookings:").pack(pady=10)
        self.bookings_list = tk.Listbox(tab)
        self.bookings_list.pack(fill=tk.BOTH)
        self.load_bookings()

    def setup_rent_vehicle(self, tab):
        ttk.Label(tab, text="Rent Vehicle functionality will go here.").pack(pady=20)

    def setup_logout(self, tab):
        ttk.Button(tab, text="Logout", command=self.logout).pack(pady=20)

    def load_vehicle_list(self):
        vehicle_type = self.vehicle_type_var.get()
        self.vehicle_listbox.delete(0, tk.END)

        vehicle_prices = {
            "Scooty": 1000,
            "Bike": 1000,
            "Car": 3000,
            "Truck": 10000,
            "Tractor": 2000
        }

        conn = sqlite3.connect('vehicle_rental.db')
        c = conn.cursor()
        c.execute("SELECT id, model FROM vehicles WHERE type = ? AND status = 'available'", (vehicle_type,))
        vehicles = c.fetchall()

        for vehicle in vehicles:
            self.vehicle_listbox.insert(tk.END, f"{vehicle[1]} - ₹{vehicle_prices[vehicle_type]} per day")
        conn.close()

    def load_bookings(self):
        self.bookings_list.delete(0, tk.END)
        conn = sqlite3.connect('vehicle_rental.db')
        c = conn.cursor()
        c.execute("SELECT b.start_date, b.end_date, b.total_price, v.model FROM bookings b JOIN vehicles v ON b.vehicle_id = v.id WHERE b.customer_id = ?", (self.current_user_id,))
        bookings = c.fetchall()

        for booking in bookings:
            # Parse date with the format 'YYYY-MM-DD' if there is no time included
            start_datetime = datetime.strptime(booking[0], "%Y-%m-%d")
            end_datetime = datetime.strptime(booking[1], "%Y-%m-%d")
            booking_info = f"{booking[3]} | Start: {start_datetime.strftime('%Y-%m-%d')} | End: {end_datetime.strftime('%Y-%m-%d')} | Total: ₹{booking[2]}"
            self.bookings_list.insert(tk.END, booking_info)

        conn.close()

    def logout(self):
        self.main_frame.pack_forget()
        self.setup_login()
        self.login_frame.pack(pady=20)


if __name__ == "__main__":
    root = tk.Tk()
    app = VehicleRentalApp(root)
    root.mainloop()

