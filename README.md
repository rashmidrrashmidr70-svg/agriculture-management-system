# agriculture-management-system
An Agriculture Management System (AMS)—often called Farm Management Software—is a comprehensive digital platform designed to plan, monitor, and optimize every stage of farming operations. By replacing fragmented paper records and guesswork with centralized, real-time data, it empowers farmers, agronomists, and agribusinesses to boost c
import sqlite3
from datetime import datetime


class AgricultureManagementSystem:

    def __init__(self, db_name="farm_data.db"):
        self.conn = sqlite3.connect(db_name)
        self.create_tables()

    def create_tables(self):
        cursor = self.conn.cursor()
        # Crops & Field Table
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS crops (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                crop_name TEXT NOT NULL,
                field_area_acres REAL NOT NULL,
                planting_date TEXT NOT NULL,
                status TEXT DEFAULT 'Growing',
                expected_yield_kg REAL
            )
        """)
        # Inventory Table (Seeds, Fertilizers, Harvest)
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS inventory (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                item_name TEXT NOT NULL,
                category TEXT NOT NULL,
                quantity REAL NOT NULL,
                unit TEXT NOT NULL
            )
        """)
        # Financial Ledger Table
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS finances (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                crop_id INTEGER,
                transaction_type TEXT NOT NULL,
                description TEXT,
                amount REAL NOT NULL,
                date TEXT NOT NULL,
                FOREIGN KEY (crop_id) REFERENCES crops(id)
            )
        """)
        self.conn.commit()

    def add_crop(self, name, acres, expected_yield):
        date_now = datetime.now().strftime("%Y-%m-%d")
        cursor = self.conn.cursor()
        cursor.execute(
            "INSERT INTO crops (crop_name, field_area_acres, planting_date, expected_yield_kg) VALUES (?, ?, ?, ?)",
            (name, acres, date_now, expected_yield),
        )
        self.conn.commit()
        print(f"\n[SUCCESS] Added crop '{name}' ({acres} acres) to field records.")

    def view_crops(self):
        cursor = self.conn.cursor()
        cursor.execute("SELECT * FROM crops")
        rows = cursor.fetchall()
        print("\n--- ACTIVE CROPS & FIELDS ---")
        if not rows:
            print("No crops recorded yet.")
        for row in rows:
            print(
                f"ID: {row[0]} | Crop: {row[1]} | Area: {row[2]} acres | Planted: {row[3]} | Status: {row[4]} | Est. Yield: {row[5]} kg"
            )

    def update_inventory(self, item, category, qty, unit):
        cursor = self.conn.cursor()
        cursor.execute("SELECT id, quantity FROM inventory WHERE item_name = ?", (item,))
        existing = cursor.fetchone()
        if existing:
            new_qty = existing[1] + qty
            cursor.execute("UPDATE inventory SET quantity = ? WHERE id = ?", (new_qty, existing[0]))
        else:
            cursor.execute(
                "INSERT INTO inventory (item_name, category, quantity, unit) VALUES (?, ?, ?, ?)",
                (item, category, qty, unit),
            )
        self.conn.commit()
        print(f"\n[SUCCESS] Inventory updated: {item} ({qty:+} {unit})")

    def view_inventory(self):
        cursor = self.conn.cursor()
        cursor.execute("SELECT * FROM inventory")
        rows = cursor.fetchall()
        print("\n--- FARM INVENTORY ---")
        if not rows:
            print("Inventory is empty.")
        for row in rows:
            print(f"ID: {row[0]} | Item: {row[1]} ({row[2]}) | Stock: {row[3]} {row[4]}")

    def record_transaction(self, crop_id, t_type, desc, amount):
        date_now = datetime.now().strftime("%Y-%m-%d")
        cursor = self.conn.cursor()
        cursor.execute(
            "INSERT INTO finances (crop_id, transaction_type, description, amount, date) VALUES (?, ?, ?, ?, ?)",
            (crop_id, t_type, desc, amount, date_now),
        )
        self.conn.commit()
        print(f"\n[SUCCESS] Recorded {t_type}: ₹{amount:.2f} ({desc})")

    def financial_summary(self):
        cursor = self.conn.cursor()
        cursor.execute("SELECT SUM(amount) FROM finances WHERE transaction_type = 'Expense'")
        expenses = cursor.fetchone()[0] or 0.0
        cursor.execute("SELECT SUM(amount) FROM finances WHERE transaction_type = 'Revenue'")
        revenue = cursor.fetchone()[0] or 0.0
        profit = revenue - expenses

        print("\n--- FARM FINANCIAL SUMMARY ---")
        print(f"Total Revenue : ₹{revenue:,.2f}")
        print(f"Total Expenses: ₹{expenses:,.2f}")
        print(f"Net Profit    : ₹{profit:,.2f}")


def main():
    ams = AgricultureManagementSystem()

    while True:
        print("\n==========================================")
        print("      AGRICULTURE MANAGEMENT SYSTEM       ")
        print("==========================================")
        print("1. Add New Crop Planting")
        print("2. View All Crops")
        print("3. Add / Use Inventory (Seeds, Fertilizer)")
        print("4. View Inventory")
        print("5. Record Expense or Harvest Sale")
        print("6. View Financial Profit/Loss Report")
        print("7. Exit")

        choice = input("\nSelect an option (1-7): ")

        if choice == "1":
            name = input("Enter Crop Name (e.g., Paddy, Ragi, Sugarcane): ")
            acres = float(input("Enter Field Area (in acres): "))
            y = float(input("Expected Yield (in kg): "))
            ams.add_crop(name, acres, y)

        elif choice == "2":
            ams.view_crops()

        elif choice == "3":
            item = input("Item Name (e.g., Urea, Hybrid Seeds): ")
            cat = input("Category (Seeds / Fertilizer / Pesticide / Harvest): ")
            qty = float(input("Quantity (use negative number to deduct stock, e.g., -10): "))
            unit = input("Unit (kg, liters, bags): ")
            ams.update_inventory(item, cat, qty, unit)

        elif choice == "4":
            ams.view_inventory()

        elif choice == "5":
            ams.view_crops()
            cid = int(input("Enter Crop ID associated with this transaction: "))
            t_type = input("Type ('Expense' or 'Revenue'): ").capitalize()
            desc = input("Description (e.g., Drip irrigation setup / Market sale): ")
            amt = float(input("Amount (₹): "))
            ams.record_transaction(cid, t_type, desc, amt)

        elif choice == "6":
            ams.financial_summary()

        elif choice == "7":
            print("Exiting Agriculture Management System. Goodbye!")
            break
        else:
            print("Invalid choice. Please try again.")


if __name__ == "__main__":
    main()
