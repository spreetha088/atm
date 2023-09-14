import mysql.connector

# Establish a database connection
con = mysql.connector.connect(
    host='localhost',
    user='root',
    password='',
    database='atm_machine'
)

cursor = con.cursor()

print('''Select your choice:
         1. Deposit
         2. Withdraw
         3. Balance
         4. Change PIN
         5. Exit''')
def deposit():
    try:
        account_no = int(input("Enter your account number: "))
        pin = int(input("Enter your pin: "))
        amount = float(input("Enter amount to deposit: "))

        cursor.execute("UPDATE atm SET balance = balance + %s WHERE account_no = %s AND pin = %s",
               (amount, account_no, pin))

        con.commit()
        print("Amount deposited successfully.")
    except mysql.connector.Error as err:
        print(f"Error: {err}")

def withdraw():
    try:
        account_no = int(input("Enter your account number: "))
        pin = int(input("Enter your pin: "))
        amount = float(input("Enter amount to withdraw: "))

        # Check if there's enough balance
    
        cursor.execute("UPDATE atm SET balance = balance - %s, deposit = %s WHERE account_no = %s AND pin = %s",
               (amount, amount, account_no, pin))

        result = cursor.fetchone()

        if result:
            balance = result[0]
            if balance >= amount:
                # Update account balance
                cursor.execute("UPDATE view atm balance = balance - %s, deposit = %s WHERE account_no = %s AND pin = %s",
                               (amount, amount, account_no, pin))

                con.commit()
                print("Amount withdrawn successfully.")
            else:
                print("Insufficient funds.")
        else:
            print("Account not found or incorrect PIN.")
    except mysql.connector.Error as err:
        print(f"Error: {err}")

def change_pin():
    try:
        account_no = int(input("Enter your account number: "))
        pin = int(input("Enter your current pin: "))
        new_pin = int(input("Enter the new pin: "))

        # Update PIN
        cursor.execute("UPDATE atm SET pin = %s WHERE account_no = %s AND pin = %s",
                       (new_pin, account_no, pin))

        con.commit()
        print("PIN changed successfully.")
    except mysql.connector.Error as err:
        print(f"Error: {err}")

def balance():
    try:
        account_no = int(input("Enter your account number: "))
        pin = int(input("Enter your pin: "))

        cursor.execute("SELECT balance FROM atm WHERE account_no = %s AND pin = %s",
                       (account_no, pin))
        result = cursor.fetchone()

        if result:
            print(f"Your balance is ${result[0]}")
        else:
            print("Account not found or incorrect PIN.")
    except mysql.connector.Error as err:
        print(f"Error: {err}")

while True:
    try:
        choice = int(input("Enter your choice: "))
        if choice == 1:
            deposit()
        elif choice == 2:
            withdraw()
        elif choice == 3:
            balance()
        elif choice == 4:
            change_pin()
        elif choice == 5:
            break
        else:
            print("Invalid choice. Please enter a number between 1 and 5.")
    except ValueError:
        print("Invalid input. Please enter a valid number.")

# Close the database connection when done
con.close()
