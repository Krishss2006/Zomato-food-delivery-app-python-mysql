# Food Delivery System

This Python script implements a simple food delivery system with MySQL database integration. Users can view a menu, place orders, and make payments either online or via cash on delivery (COD).

## Features

- Display menu with food items and prices.
- Allow users to select items and place orders.
- Process payments (COD or online) and update order status.
- Log user orders in the database.
- User account creation with balance management.

## Requirements

- Python 3.x
- MySQL database server
- MySQL Connector/Python library (`mysql-connector-python`)
- Ensure you have MySQL installed and running locally.

## Setup

1. Clone the repository.
2. Install dependencies: `pip install mysql-connector-python`.
3. Set up your MySQL database and update the `mysql_password` variable in the script.
4. Run the script: `python food_delivery.py`.

## Usage

- View the menu and enter the item numbers to place orders.
- Choose payment options: Cash on Delivery (COD) or Online.
- For online payment, select your bank and follow the instructions.
- Orders will be logged in the database.

## Additional Notes

- You can expand the system with features like order tracking, delivery status, etc.
- Feel free to contribute and enhance the system further!

# - Join [Telegram link](https://t.me/pythonbykrishss) now!
# - Subscribe [YouTube link](https://youtube.com/@krishsscodes?si=EM7A7U_er-WGZZE_) now!



```python
# Created by Krihna
## Telegram link join now (t.me/pythonbykrishss)
## Youtube link suubscribe now (https://youtube.com/@krishsscodes?si=EM7A7U_er-WGZZE_)

from time import sleep
from mysql.connector import connect, Error
from sys import exit

# Don not forget to enter you MySQL Password
mysql_password = "YOUR_PASSWORD_HERE"

def create_database_and_tables():
    try:
        # Connect to the database server and create necessary tables if they don't exist
        conn = connect(host="localhost", user="root", password="M@ihunkri$hn@06")
        cursor = conn.cursor()
        cursor.execute("CREATE DATABASE IF NOT EXISTS food_delivery")  # Create food_delivery database if not exists
        cursor.execute("USE food_delivery")  # Select food_delivery database
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS records (
                account_no VARCHAR(20) PRIMARY KEY,
                name VARCHAR(100),
                pin INT,
                balance FLOAT
            )
        """)  # Create 'records' table to store user details
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS food_items (
                food_name VARCHAR(100) PRIMARY KEY,
                price FLOAT
            )
        """)  # Create 'food_items' table to store food items and prices
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS orders (
                order_id INT AUTO_INCREMENT PRIMARY KEY,
                account_no VARCHAR(20),
                items TEXT,
                total_amt FLOAT,
                delivery_address VARCHAR(255),
                order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                FOREIGN KEY (account_no) REFERENCES records(account_no)
            )
        """)  # Create 'orders' table to store order details
        conn.commit()
        cursor.close()
        conn.close()
    except Error as e:
        print(f"Error: {e}")
        exit()

def get_connection():
    global mysql_password
    try:
        # Connect to the database
        return connect(host="localhost", user="root", password=mysql_password, database="food_delivery")
    except Error as e:
        print(f"Error: {e}")
        exit()

def ensure_food_items():
    # Dictionary containing food items and their prices 
    items = {
        "Pizza": 8, "Cheetos": 4, "Popcorn": 5, 
        "Coke": 2, "Pepsi": 2, "Chips": 3,
        "Butter Chicken": 10, "Paneer Tikka": 8, "Samosa": 2, 
        "Biryani": 9, "Masala Dosa": 6, "Burger": 7, 
        "French Fries": 3, "Tacos": 4, "Pasta": 7
    }  # You could add more items to the dictionary, and they will be added to the database
    
    conn = get_connection()
    cursor = conn.cursor()
    for item, price in items.items():
        cursor.execute("INSERT IGNORE INTO food_items (food_name, price) VALUES (%s, %s)", (item, price))
    conn.commit()
    
    cursor.execute("SELECT food_name, price FROM food_items")
    data = cursor.fetchall()
    cursor.close()
    conn.close()
    
    food_dict = {food: price for food, price in data}
    return food_dict

def menu(items):
    # Display the menu with food items and prices
    print("\n\n\n------ Welcome to Foodie Area ------")
    print("\n              Menu ")
    print("            ----------")
    print("        {:<15}{:<10}".format("Food", "Price"))
    print("      ---------       --------")
    for n, (item, price) in enumerate(items.items(), 1):
        print(f"   {str(n)+'.':<3} {item:<15}  ${float(price):<10}")

def place_order(items, orders):
    global total_amt
    total_amt = float(total_amt)
    ch = ""
    print("\nPlace your order by entering item number.")
    while ch.lower() != 'd':
        order = input("Enter the item number or 'D' to finish: ")
        if order.lower() == 'd':
            break
        elif order.isdigit() and int(order) in range(1, len(items) + 1):
            # Add the selected item to the order list
            item_number = int(order)
            item = list(items.keys())[item_number - 1]
            price = items[item]
            orders.append(item)
            total_amt += price
        else:
            print("Invalid item number. Please enter a valid item number.")
            continue
    print("\nYour order is:", ", ".join(orders))
    print(f"\nTotal price: ${total_amt}")
    print("Adding $1 delivery charge")
    total_amt += 1
    print(f"Your Total Amount is ${total_amt}\n")


def cod_payment(total_amt):
    # Process Cash on Delivery payment
    print("Your order has been placed")
    sleep(1)
    print("Payment method is Cash on Delivery")
    print("Your payment will be delivered in 10 minutes")
    sleep(2)
    exit_program()

def online_payment(total_amt, banks):
    # Process online payment
    print("Enter your bank.")
    for i, bank in enumerate(banks, 1):
        print(f"{i}. {bank}")
    while True:
        bank = input("(1-7)--> ")
        if bank not in [str(i) for i in range(1, 8)]:
            print("Invalid choice")
        else:
            print(f"You selected {banks[int(bank) - 1]}")
            process_payment(total_amt)
            break

def process_payment(total_amt):
    # Process payment after verifying user account
    conn = get_connection() 
    cursor = conn.cursor()
    while True:
        acc_no = input("Enter your account number: ")
        cursor.execute(f"SELECT name, account_no, pin FROM records WHERE account_no='{acc_no}'")
        data = cursor.fetchone()
        if data:
            print(f"Account Number: {data[1]}")
            print(f"Account Holder Name: {data[0]}")
            input(f"Press 'Enter' to pay ${total_amt}")
            if verify_pin(acc_no, cursor):
                cursor.execute(f"UPDATE records SET balance = balance - {total_amt} WHERE account_no = '{acc_no}'")
                # Save order details to the database
                cursor.execute("INSERT INTO orders (account_no, items, total_amt) VALUES (%s, %s, %s)", (acc_no, ', '.join(orders), total_amt))
                conn.commit()
                print("Your order has been placed")
                print("Will be delivered in 10 minutes")
                conn.close()
                exit_program()
        else:
            print("Account Not Found. Create an account first.")
            if input("Enter 'Y' to create account or 'N' to try again: ").lower() == 'y':
                create_account(cursor, conn)
                conn.commit()
            else:
                continue

def create_account(cursor, conn):
    # Create a new user account
    name = input("Enter your name: ")
    account_no = input("Enter your new account number: ")
    pin = input("Enter your PIN: ")
    balance = float(input("Enter your initial balance: "))
    cursor.execute("INSERT INTO records (account_no, name, pin, balance) VALUES (%s, %s, %s, %s)", 
                   (account_no, name, pin, balance))
    conn.commit()
    print("Account created successfully.")

def verify_pin(acc_no, cursor):
    # Verify PIN entered by the user
    pin = input("Enter your PIN: ")
    cursor.execute(f"SELECT pin FROM records WHERE account_no='{acc_no}'")
    data = cursor.fetchone()
    if int(pin) == data[0]:
        return True
    else:
        print("Invalid PIN Entered")
        return verify_pin(acc_no, cursor)

def payment_modes(orders, total_amt, banks):
    # Display payment options and process accordingly
    while True:
        print("\t1. Cash on Delivery")
        print("\t2. Online")
        print("\t3. Cancel Order\n")
        ch = input("(1-3)--> ")
        if ch == "1":
            cod_payment(total_amt)
        elif ch == "2":
            online_payment(total_amt, banks)
        elif ch == "3":
            print("Your order has been Cancelled")
            sleep(1)
            exit_program()
        else:
            print("Invalid Choice")

def exit_program():
    # Exit the program
    print("\n----- Thanks for using our service -----")
    input("Press 'Enter' to exit the app completely")
    print("Bye")
    sleep(2)
    exit()

if __name__ == "__main__":
    banks = ["HDFC Bank", "Bank of Baroda", "Bank of India", "State Bank of India", "ICIC Bank", "Central Bank of India", "Kotak Bank"]
    orders = []  # List to store user's orders
    total_amt = 0  # Variable to store total amount of the order
    create_database_and_tables() 
    food_items = ensure_food_items()
    menu(food_items)
    place_order(food_items, orders)
    payment_modes(orders, total_amt, banks)
```
