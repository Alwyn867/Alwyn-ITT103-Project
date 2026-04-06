

# ALWYN
# Best Buy Retail Store - Point of Sale (POS) System
# Course: ITT103 - Programming Techniques
# Spring 2026

import datetime

# Each product stored as: { name: {"price": float, "stock": int} }

product_catalog = {
    "rice":          {"price": 850.00,  "stock": 40},
    "flour":         {"price": 620.00,  "stock": 35},
    "sugar":         {"price": 490.00,  "stock": 28},
    "cooking oil":   {"price": 1350.00, "stock": 20},
    "milk":          {"price": 520.00,  "stock": 50},
    "bread":         {"price": 450.00,  "stock": 30},
    "eggs":          {"price": 600.00,  "stock": 25},
    "butter":        {"price": 780.00,  "stock": 18},
    "chicken":       {"price": 1200.00, "stock": 15},
    "soap":          {"price": 350.00,  "stock": 45},
    "toothpaste":    {"price": 475.00,  "stock": 22},
    "tissue paper":  {"price": 390.00,  "stock": 60},
    "detergent":     {"price": 950.00,  "stock": 12},
    "juice":         {"price": 310.00,  "stock": 38},
    "water":         {"price": 150.00,  "stock": 100},
}

# The shopping cart holds items being purchased in the current transaction
# Format: [ {"name": str, "qty": int, "unit_price": float} ]
shopping_cart = []

TAX_RATE = 0.10           # 10% sales tax
DISCOUNT_THRESHOLD = 5000  # subtotal must exceed this for discount
DISCOUNT_RATE = 0.05       # 5% discount
LOW_STOCK_LIMIT = 5        # warn if stock drops below this


# This is for the helper functions 

def get_valid_int(prompt):
    """Keep asking user until they enter a valid whole number."""
    while True:
        raw = input(prompt)
        try:
            value = int(raw)
            if value <= 0:
                print("  >> Quantity must be a positive number. Try again.")
                continue
            return value
        except ValueError:
            print("  >> That is not a valid number. Please enter a whole number.")


def get_valid_float(prompt):
    """Keep asking user until they enter a valid monetary amount."""
    while True:
        raw = input(prompt)
        try:
            value = float(raw)
            if value < 0:
                print("  >> Amount cannot be negative. Try again.")
                continue
            return value
        except ValueError:
            print("  >> Invalid amount. Please enter a numeric value.")


def display_separator():
    print("-" * 55)

# This for the product display 

def show_product_catalog():
    """Prints the full product list with prices and available stock."""
    print("\n" + "=" * 55)
    print("           BEST BUY - PRODUCT CATALOG")
    print("=" * 55)
    print(f"  {'Product':<18} {'Price ($)':<12} {'In Stock':<10}")
    display_separator()

    for name, info in product_catalog.items():
        tag = ""
        if info["stock"] < LOW_STOCK_LIMIT:
            tag = " ** LOW STOCK **"
        print(f"  {name:<18} {info['price']:>8.2f}     {info['stock']:>4}{tag}")

    display_separator()


# --------------- Low Stock Alerts ---------------

def check_low_stock():
    """Scan the catalog and warn the cashier about items running low."""
    alerts = []
    for name, info in product_catalog.items():
        if info["stock"] < LOW_STOCK_LIMIT and info["stock"] > 0:
            alerts.append((name, info["stock"]))
        elif info["stock"] == 0:
            alerts.append((name, 0))

    if len(alerts) > 0:
        print("\n  *** LOW STOCK ALERTS ***")
        for item_name, qty_left in alerts:
            if qty_left == 0:
                print(f"  - {item_name}: OUT OF STOCK")
            else:
                print(f"  - {item_name}: only {qty_left} remaining")
        print()


# --------------- Cart Operations ---------------

def add_to_cart():
    """Prompt cashier for product name and quantity, then add to the cart."""
    show_product_catalog()

    item_name = input("\n  Enter product name to add: ").strip().lower()

    # check if the product exists
    if item_name not in product_catalog:
        print(f"  >> '{item_name}' is not in our catalog.")
        return

    available = product_catalog[item_name]["stock"]
    if available == 0:
        print(f"  >> Sorry, '{item_name}' is currently out of stock.")
        return

    qty = get_valid_int(f"  Enter quantity (available: {available}): ")

    if qty > available:
        print(f"  >> Not enough stock. Only {available} unit(s) of '{item_name}' available.")
        return

    # see if this item is already in the cart; if so, update the quantity
    found = False
    for entry in shopping_cart:
        if entry["name"] == item_name:
            # make sure combined qty doesn't exceed stock
            new_total = entry["qty"] + qty
            if new_total > available:
                print(f"  >> Cannot add {qty} more. You already have {entry['qty']} in the cart "
                      f"and only {available} in stock.")
                return
            entry["qty"] = new_total
            found = True
            break

    if not found:
        shopping_cart.append({
            "name": item_name,
            "qty": qty,
            "unit_price": product_catalog[item_name]["price"]
        })

    # reduce stock right away so it stays accurate
    product_catalog[item_name]["stock"] -= qty
    line_total = product_catalog[item_name]["price"] * qty
    print(f"  >> Added {qty} x {item_name} (${line_total:.2f}) to cart.")

    # warn if stock is now low
    if product_catalog[item_name]["stock"] < LOW_STOCK_LIMIT:
        remaining = product_catalog[item_name]["stock"]
        if remaining == 0:
            print(f"  *** ALERT: '{item_name}' is now OUT OF STOCK ***")
        else:
            print(f"  *** ALERT: '{item_name}' stock is low - {remaining} left ***")


def remove_from_cart():
    """Remove an item from the cart and restore its stock."""
    if len(shopping_cart) == 0:
        print("  >> The cart is empty. Nothing to remove.")
        return

    view_cart()
    item_name = input("\n  Enter the product name to remove: ").strip().lower()

    index_to_remove = -1
    for i in range(len(shopping_cart)):
        if shopping_cart[i]["name"] == item_name:
            index_to_remove = i
            break

    if index_to_remove == -1:
        print(f"  >> '{item_name}' is not in your cart.")
        return

    removed = shopping_cart.pop(index_to_remove)
    # put the stock back
    product_catalog[item_name]["stock"] += removed["qty"]
    print(f"  >> Removed {removed['qty']} x {item_name} from the cart.")


def view_cart():
    """Display all items currently in the shopping cart."""
    if len(shopping_cart) == 0:
        print("\n  >> Your cart is empty.")
        return

    print("\n" + "=" * 55)
    print("              SHOPPING CART")
    print("=" * 55)
    print(f"  {'Item':<18} {'Qty':<6} {'Unit $':<10} {'Total $':<10}")
    display_separator()

    running_total = 0.0
    for entry in shopping_cart:
        line_total = entry["qty"] * entry["unit_price"]
        running_total += line_total
        print(f"  {entry['name']:<18} {entry['qty']:<6} {entry['unit_price']:>8.2f}   {line_total:>8.2f}")

    display_separator()
    print(f"  {'Cart Subtotal:':<36} ${running_total:>8.2f}")
    print("=" * 55)


# This is for the paymnent and check out

def calculate_subtotal():
    """Sum up all line totals in the cart."""
    total = 0.0
    for entry in shopping_cart:
        total += entry["qty"] * entry["unit_price"]
    return total


def checkout():
    """Handle the full checkout: subtotal, discount, tax, payment, receipt."""
    if len(shopping_cart) == 0:
        print("  >> Cart is empty. Add some items before checking out.")
        return False

    view_cart()

    subtotal = calculate_subtotal()

    # apply discount if subtotal is over the threshold
    discount_amount = 0.0
    if subtotal > DISCOUNT_THRESHOLD:
        discount_amount = subtotal * DISCOUNT_RATE
        print(f"\n  ** You qualify for a {DISCOUNT_RATE * 100:.0f}% discount! **")
        print(f"     Discount: -${discount_amount:.2f}")

    discounted_subtotal = subtotal - discount_amount
    tax = discounted_subtotal * TAX_RATE
    total_due = discounted_subtotal + tax

    print(f"\n  Subtotal:       ${subtotal:>10.2f}")
    if discount_amount > 0:
        print(f"  Discount (5%):  -${discount_amount:>9.2f}")
    print(f"  Sales Tax (10%): ${tax:>9.2f}")
    display_separator()
    print(f"  TOTAL DUE:       ${total_due:>9.2f}")
    display_separator()

    # this our payment loop -- cashier enters amount received
    while True:
        amount_paid = get_valid_float("\n  Enter amount received from customer: $")
        if amount_paid < total_due:
            shortage = total_due - amount_paid
            print(f"  >> Insufficient payment. Still need ${shortage:.2f} more.")
        else:
            break

    change = amount_paid - total_due

    # generate and print the receipt
    print_receipt(subtotal, discount_amount, tax, total_due, amount_paid, change)

    # clear the cart after successful checkout
    shopping_cart.clear()
    return True


# Our receipt formatting function(could've made it shorter but wanted to make it look nice)

def print_receipt(subtotal, discount, tax, total_due, paid, change):
    """Format and display a proper receipt to the cashier / customer."""
    now = datetime.datetime.now()
    timestamp = now.strftime("%Y-%m-%d %H:%M:%S")

    print("\n")
    print("=" * 55)
    print("        BEST BUY RETAIL STORE")
    print("        123 Main Street, Kingston")
    print("        Tel: (876) 555-1234")
    print("=" * 55)
    print(f"  Date: {timestamp}")
    print(f"  Cashier: On Duty")
    display_separator()
    print(f"  {'Item':<16} {'Qty':<5} {'Price':<10} {'Total':<10}")
    display_separator()

    for entry in shopping_cart:
        line = entry["qty"] * entry["unit_price"]
        print(f"  {entry['name']:<16} {entry['qty']:<5} ${entry['unit_price']:>7.2f}  ${line:>8.2f}")

    display_separator()
    print(f"  {'Subtotal:':<40} ${subtotal:>8.2f}")

    if discount > 0:
        print(f"  {'Discount (5%):':<40} -${discount:>7.2f}")

    print(f"  {'Sales Tax (10%):':<40} ${tax:>8.2f}")
    
    # --- Line at the total added here ---
    print("-" * 55)
    print(f"  {'TOTAL DUE:':<40} ${total_due:>8.2f}")
    print(f"  {'Amount Paid:':<40} ${paid:>8.2f}")
    print(f"  {'Change:':<40} ${change:>8.2f}")
    print("=" * 55)
    print("       Thank you for shopping at Best Buy!")
    print("            Have a wonderful day!")
    print("=" * 55)
    print()


# --------------- Main Menu ---------------

def display_menu():
    print("\n" + "=" * 55)
    print("     BEST BUY RETAIL STORE - POS SYSTEM")
    print("=" * 55)
    print("  1. View Product Catalog")
    print("  2. Add Item to Cart")
    print("  3. Remove Item from Cart")
    print("  4. View Shopping Cart")
    print("  5. Checkout")
    print("  6. Start New Transaction")
    print("  7. Exit")
    display_separator()


def run_pos_system():
    """Main loop that drives the entire POS system."""
    print("\n" + "*" * 55)
    print("* Welcome to Best Buy Retail Store POS System    *")
    print("*" * 55)

    check_low_stock()

    running = True
    while running:
        display_menu()

        choice = input("  Select an option (1-7): ").strip()

        if choice == "1":
            show_product_catalog()

        elif choice == "2":
            add_to_cart()

        elif choice == "3":
            remove_from_cart()

        elif choice == "4":
            view_cart()

        elif choice == "5":
            result = checkout()
            if result:
                check_low_stock()
                print("  Transaction complete. You can start a new one or exit.")

        elif choice == "6":
            # start a fresh transaction - put any leftover cart items back in stock
            if len(shopping_cart) > 0:
                for entry in shopping_cart:
                    product_catalog[entry["name"]]["stock"] += entry["qty"]
                shopping_cart.clear()
                print("  >> Previous cart cleared. Ready for a new transaction.")
            else:
                print("  >> Cart is already empty. Ready for a new transaction.")

        elif choice == "7":
            # if there are items still in the cart, warn the cashier
            if len(shopping_cart) > 0:
                confirm = input("  >> You have items in the cart. Exit anyway? (y/n): ").strip().lower()
                if confirm != "y":
                    continue
                # restore stock before leaving
                for entry in shopping_cart:
                    product_catalog[entry["name"]]["stock"] += entry["qty"]
                shopping_cart.clear()

            print("\n  Closing POS System. Goodbye!")
            print("=" * 55)
            running = False

        else:
            print("  >> Invalid choice. Please pick a number from 1 to 7.")


# This is the entry point of the program. When we run this script, it will start the POS system.

if __name__ == "__main__":
    run_pos_system()

