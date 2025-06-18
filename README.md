CREATE DATABASE IF NOT EXISTS atm_db;

USE atm_db;

CREATE TABLE IF NOT EXISTS accounts (
    acc_no INT PRIMARY KEY,
    name VARCHAR(100),
    pin VARCHAR(100), -- store hashed PINs for security
    balance DECIMAL(10,2)
);
import mysql.connector
import bcrypt

# Connect to MySQL
conn = mysql.connector.connect(
    host="localhost",
    user="your_mysql_user",
    password="your_mysql_password",
    database="atm_db"
)
cursor = conn.cursor()

# Hash PIN
def hash_pin(pin):
    return bcrypt.hashpw(pin.encode(), bcrypt.gensalt())

# Verify PIN
def verify_pin(pin, hashed):
    return bcrypt.checkpw(pin.encode(), hashed.encode())

# Login
def login():
    acc_no = input("Enter Account Number: ")
    pin = input("Enter PIN: ")
    cursor.execute("SELECT pin FROM accounts WHERE acc_no = %s", (acc_no,))
    result = cursor.fetchone()
    if result and verify_pin(pin, result[0]):
        print("Login successful.")
        return acc_no
    else:
        print("Invalid PIN or account.")
        return None

# Balance Inquiry
def balance_inquiry(acc_no):
    cursor.execute("SELECT balance FROM accounts WHERE acc_no = %s", (acc_no,))
    print("Current Balance: ₹", cursor.fetchone()[0])

# Deposit
def deposit(acc_no):
    try:
        amount = float(input("Enter deposit amount: "))
        cursor.execute("UPDATE accounts SET balance = balance + %s WHERE acc_no = %s", (amount, acc_no))
        conn.commit()
        print("Amount deposited successfully.")
        balance_inquiry(acc_no)
    except:
        print("Invalid input.")

# Withdrawal
def withdraw(acc_no):
    try:
        amount = float(input("Enter withdrawal amount: "))
        cursor.execute("SELECT balance FROM accounts WHERE acc_no = %s", (acc_no,))
        balance = cursor.fetchone()[0]
        if balance >= amount:
            cursor.execute("UPDATE accounts SET balance = balance - %s WHERE acc_no = %s", (amount, acc_no))
            conn.commit()
            print("Withdrawal successful.")
        else:
            print("Insufficient balance.")
        balance_inquiry(acc_no)
    except:
        print("Invalid input.")

# Fund Transfer
def transfer(acc_no):
    try:
        target = input("Enter target account number: ")
        amount = float(input("Enter amount to transfer: "))

        cursor.execute("SELECT balance FROM accounts WHERE acc_no = %s", (acc_no,))
        sender_balance = cursor.fetchone()[0]

        cursor.execute("SELECT * FROM accounts WHERE acc_no = %s", (target,))
        if cursor.fetchone() is None:
            print("Target account does not exist.")
            return

        if sender_balance >= amount:
            cursor.execute("UPDATE accounts SET balance = balance - %s WHERE acc_no = %s", (amount, acc_no))
            cursor.execute("UPDATE accounts SET balance = balance + %s WHERE acc_no = %s", (amount, target))
            conn.commit()
            print("Transfer successful.")
        else:
            print("Insufficient balance.")
        balance_inquiry(acc_no)
    except:
        print("Invalid input.")

# Main Menu
def main():
    acc_no = login()
    if not acc_no:
        return
    while True:
        print("\n1. Balance Inquiry\n2. Deposit\n3. Withdraw\n4. Fund Transfer\n5. Exit")
        choice = input("Choose an option: ")
        if choice == '1':
            balance_inquiry(acc_no)
        elif choice == '2':
            deposit(acc_no)
        elif choice == '3':
            withdraw(acc_no)
        elif choice == '4':
            transfer(acc_no)
        elif choice == '5':
            break
        else:
            print("Invalid choice.")

main()
cursor.close()
conn.close()
def create_account():
    acc_no = int(input("Account number: "))
    name = input("Name: ")
    pin = input("4-digit PIN: ")
    balance = float(input("Initial Balance: "))
    hashed_pin = hash_pin(pin).decode()

    cursor.execute("INSERT INTO accounts (acc_no, name, pin, balance) VALUES (%s, %s, %s, %s)",
                   (acc_no, name, hashed_pin, balance))
    conn.commit()
    print("Account created.")

