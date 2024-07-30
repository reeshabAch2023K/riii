import datetime

# Read furniture from file
def read_furniture_file(filename="BRJfurniture.txt"):
    furniture_list = []
    try:
        with open(filename, 'r') as file:
            for line in file:
                parts = line.strip().split(", ")
                if len(parts) == 5:
                    try:
                        furniture_list.append({
                            "id": int(parts[0]),
                            "manufacturer": parts[1],
                            "name": parts[2],
                            "quantity": int(parts[3]),
                            "price": float(parts[4].replace('$', ''))
                        })
                    except ValueError:
                        print(f"Skipping malformed line: {line.strip()}")
                else:
                    print(f"Skipping malformed line: {line.strip()}")
    except FileNotFoundError:
        print("Furniture file not found.")
    except Exception as e:
        print(f"Error reading furniture file: {e}")
    return furniture_list

# Save furniture to file
def save_furniture_file(furniture_list, filename="BRJfurniture.txt"):
    try:
        with open(BRJfurniture, 'w') as file:
            for item in furniture_list:
                file.write(f"{item['id']}, {item['manufacturer']}, {item['name']}, {item['quantity']}, ${item['price']}\n")
    except Exception as e:
        print(f"Error saving furniture file: {e}")

# Display available furniture
def display_furniture(furniture_list):
    print("Available Furniture:")
    for item in furniture_list:
        print(f"{item['id']}: {item['name']} by {item['manufacturer']}, Quantity: {item['quantity']}, Price: ${item['price']}")

# Handle order
def handle_order(furniture_list):
    try:
        furniture_id = int(input("Enter the furniture ID to order: "))
        quantity = int(input("Enter the quantity to order: "))
        for item in furniture_list:
            if item['id'] == furniture_id:
                item['quantity'] += quantity
                amount = item['price'] * quantity
                generate_invoice({
                    "type": "order",
                    "id": furniture_id,
                    "manufacturer": item['manufacturer'],
                    "name": item['name'],
                    "quantity": quantity,
                    "employee": input("Enter your name: "),
                    "date": datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
                    "amount": amount
                })
                print("Order processed successfully!")
                save_furniture_file(furniture_list)
                return
        print("Furniture ID not found")
    except ValueError:
        print("Invalid input. Please enter numeric values for ID and quantity.")

# Handle sale
def handle_sale(furniture_list):
    try:
        customer_name = input("Enter customer name: ")
        total_amount = 0
        items_sold = []
        while True:
            furniture_id = int(input("Enter the furniture ID to sell: "))
            quantity = int(input("Enter the quantity to sell: "))
            for item in furniture_list:
                if item['id'] == furniture_id:
                    if item['quantity'] >= quantity:
                        item['quantity'] -= quantity
                        amount = item['price'] * quantity
                        total_amount += amount
                        items_sold.append(f"{quantity} of {item['name']} by {item['manufacturer']}")
                        print(f"Added {quantity} of {item['name']} to the sale.")
                        break
                    else:
                        print("Insufficient stock")
                        break
            else:
                print("Furniture ID not found")
            
            more = input("Add more furniture? (yes/no): ")
            if more.lower() != 'yes':
                break
        
        shipping_cost = 50
        vat = 0.13
        total_with_vat = total_amount * (1 + vat)
        total_to_pay = total_with_vat + shipping_cost
        
        generate_invoice({
            "type": "sale",
            "customer": customer_name,
            "items_sold": ', '.join(items_sold),
            "total_amount": total_amount,
            "vat": total_amount * vat,
            "shipping_cost": shipping_cost,
            "total_to_pay": total_to_pay,
            "date": datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        })
        save_furniture_file(furniture_list)
        print("Sale processed successfully!")
    except ValueError:
        print("Invalid input. Please enter numeric values for ID and quantity.")

# Generate invoice
def generate_invoice(details):
    filename = f"invoice_{datetime.datetime.now().strftime('%Y%m%d_%H%M%S')}.txt"
    try:
        with open(filename, 'w') as file:
            for key, value in details.items():
                file.write(f"{key}: {value}\n")
        print(f"Invoice generated: {filename}")
    except Exception as e:
        print(f"Error generating invoice: {e}")

# Main loop
def main():
    furniture_list = read_furniture_file()
    while True:
        display_furniture(furniture_list)
        transaction_type = input("Enter transaction type (order/sell): ").strip().lower()
        if transaction_type == 'order':
            handle_order(furniture_list)
        elif transaction_type == 'sell':
            handle_sale(furniture_list)
        else:
            print("Invalid transaction type")
        
        continue_program = input("Do you want to continue? (yes/no): ").strip().lower()
        if continue_program != 'yes':
            break

if __name__ == "__main__":
    main()
 
 
