# JesiJoshRider
Inventory Monitoring System_ Group 7_ COC-FB2-BSIT1-02-P


from datetime import date, datetime

users = {
    "admin": {"password": "admin123", "role": "Admin / Inventory Manager"},
    "staff": {"password": "staff123", "role": "Staff / Viewer"}
}
inventory = []
CATEGORIES = ["Drinks", "Snacks", "Instant Food", "Bread", "Siopao", "Ice Cream", "Cigarettes", "Essentials"]

def print_header(title):
    print("="*72)
    print(title.center(72))
    print("="*72)

def print_border():
    print("+" + "-"*70 + "+")

def is_numbers_only(s):
    return s.isdigit()

def is_letters_only(s):
    return all(c.isalpha() or c.isspace() for c in s) and s.strip() != ""

def is_decimal_number(s):
    try:
        return float(s) >= 0
    except:
        return False

def parse_date(date_str):
    try:
        return datetime.strptime(date_str, "%Y-%m-%d").date()
    except:
        return None

def get_product_status(product):
    qty = product['quantity']
    exp = product['exp_date']
    today = date.today()
    if qty == 0:
        return "OUT OF STOCK"
    if exp:
        if exp < today:
            return "EXPIRED"
        if exp == today:
            return "EXPIRES TODAY"
        if (exp - today).days <= 7:
            return f"{(exp - today).days} day(s) left"
    if qty <= 5:
        return "LOW STOCK"
    return "OK"

def login():
    print_header("INVENTORY SYSTEM LOGIN")
    print_border()
    attempts = 0
    while attempts < 3:
        u = input("Username: ").strip()
        p = input("Password: ").strip()
        if u in users and users[u]["password"] == p:
            print(f"\nLogin successful. Welcome, {u}! Role: {users[u]['role']}\n")
            return u, users[u]["role"]
        else:
            attempts += 1
            print(f"Invalid login. Attempts left: {3-attempts}")
    print("Too many failed attempts. Exiting.")
    exit()

def dashboard():
    print_header("INVENTORY MONITORING DASHBOARD")
    print_border()
    total = len(inventory)
    out_stock = sum(1 for p in inventory if p['quantity'] == 0)
    low_stock = sum(1 for p in inventory if 1 <= p['quantity'] <= 5)
    expired = sum(1 for p in inventory if p['exp_date'] and p['exp_date'] < date.today())
    expiring = sum(1 for p in inventory if p['exp_date'] and 0 <= (p['exp_date'] - date.today()).days <= 7)
    total_value = sum(p['price'] * p['quantity'] for p in inventory)
    print(f"| Total products : {total}")
    print(f"| Current inventory value : P{total_value:.2f}")
    print(f"| Out of stock : {out_stock}")
    print(f"| Low stock (1-5) : {low_stock}")
    print(f"| Expired : {expired}")
    print(f"| Expiring within 7 days : {expiring}")
    print_border()

def add_product():
    print_header("ADD NEW PRODUCT - MONITORING")
    print_border()
    while True:
        code = input("Product code: ").strip()
        if not is_numbers_only(code):
            print("ERROR: Numbers only.")
            continue
        if any(p['id'] == int(code) for p in inventory):
            print("ERROR: Code already exists.")
            continue
        break
    while True:
        name = input("Product name: ").strip()
        if not is_letters_only(name):
            print("ERROR: Letters only.")
            continue
        break
    print("Available Categories:")
    print("+" + "-"*50 + "+")
    for i, c in enumerate(CATEGORIES, 1):
        print(f"| {i}. {c:<44} |")
    print("+" + "-"*50 + "+")
    while True:
        cat_input = input(f"Choose category (1-{len(CATEGORIES)}): ").strip()
        if is_numbers_only(cat_input) and 1 <= int(cat_input) <= len(CATEGORIES):
            category = CATEGORIES[int(cat_input)-1]
            break
        print("ERROR: Invalid category. Choose 1-8.")
    while True:
        supplier = input("Supplier: ").strip()
        if not is_letters_only(supplier):
            print("ERROR: Letters only.")
            continue
        break
    while True:
        cost = input("Cost price: ").strip()
        if not is_decimal_number(cost):
            print("ERROR: Invalid.")
            continue
        break
    while True:
        price = input("Selling price: ").strip()
        if not is_decimal_number(price) or float(price) < float(cost):
            print("ERROR: Price must be >= cost.")
            continue
        break
    while True:
        qty = input("Initial stock: ").strip()
        if not is_numbers_only(qty) or int(qty) == 0:
            print("ERROR: Stock must be > 0.")
            continue
        break
    while True:
        exp_str = input("Expiry date (YYYY-MM-DD) or blank: ").strip()
        if exp_str == "":
            exp_date = None
            break
        exp_date = parse_date(exp_str)
        if not exp_date:
            print("ERROR: Use YYYY-MM-DD.")
            continue
        if exp_date < date.today():
            print("DENIED: Cannot add expired product. Enter future date.")
            continue
        break
    inventory.append({'id': int(code), 'name': name, 'category': category, 'supplier': supplier, 'cost': float(cost), 'price': float(price), 'quantity': int(qty), 'exp_date': exp_date})
    print(f"\nSUCCESS: {name} added.")
    print_border()

def view_inventory():
    print_header("CURRENT INVENTORY / SHELF")
    print_border()
    if not inventory:
        print("| No products in inventory. |")
        print_border()
        return
    print(f"| {'CODE':<5} | {'PRODUCT':<10} | {'CATEGORY':<12} | {'QTY':<4} | {'EXPIRY':<10} | {'STATUS'} |")
    print("+" + "-"*70 + "+")
    for p in inventory:
        exp = p['exp_date'].strftime("%Y-%m-%d") if p['exp_date'] else "N/A"
        print(f"| {p['id']:<5} | {p['name']:<10} | {p['category']:<12} | {p['quantity']:<4} | {exp:<10} | {get_product_status(p)} |")
    print_border()

def find_product():
    print_header("FIND PRODUCT")
    print_border()
    key = input("Enter code or name: ").strip().lower()
    found = [p for p in inventory if str(p['id']) == key or key in p['name'].lower()]
    if not found:
        print("| No product found. |")
    else:
        for p in found:
            print(f"| {p['id']} - {p['name']} - {p['quantity']} pcs - {get_product_status(p)} | Supplier: {p['supplier']} |")
    print_border()

def category_view():
    print_header("VIEW BY CATEGORY")
    print("+" + "-"*50 + "+")
    for i, c in enumerate(CATEGORIES, 1):
        print(f"| {i}. {c:<44} |")
    print("+" + "-"*50 + "+")
    choice = input(f"Choose (1-{len(CATEGORIES)}): ").strip()
    if not is_numbers_only(choice) or not 1 <= int(choice) <= len(CATEGORIES):
        print("Invalid.")
        return
    cat = CATEGORIES[int(choice)-1]
    filtered = [p for p in inventory if p['category'] == cat]
    print_border()
    if not filtered:
        print(f"| No products in {cat} |")
        print_border()
        return
    print(f"| CATEGORY: {cat} |")
    print_border()
    for p in filtered:
        print(f"| {p['id']} | {p['name']} | {p['quantity']} | {get_product_status(p)} | Supplier: {p['supplier']} |")
    print_border()

def update_product():
    print_header("UPDATE PRODUCT - MONITORING")
    print_border()
    code = input("Enter product code to update: ").strip()
    if not is_numbers_only(code):
        print("Invalid code.")
        return
    prod = next((p for p in inventory if p['id'] == int(code)), None)
    if not prod:
        print("Product not found.")
        return
    print(f"| Updating {prod['name']} (Qty: {prod['quantity']}) |")
    print("| NOTE: Stock cannot be set to 0 here. Use Stock In/Out. |")
    print_border()
    new_name = input(f"New name (blank to keep {prod['name']}): ").strip()
    if new_name != "":
        if not is_letters_only(new_name):
            print("ERROR: Letters only.")
            return
        prod['name'] = new_name
    new_supplier = input(f"New supplier (blank to keep {prod['supplier']}): ").strip()
    if new_supplier != "":
        if not is_letters_only(new_supplier):
            print("ERROR: Letters only.")
            return
        prod['supplier'] = new_supplier
    new_exp = input(f"New expiry (current {prod['exp_date']}, blank to keep): ").strip()
    if new_exp != "":
        exp_date = parse_date(new_exp)
        if not exp_date:
            print("ERROR: Invalid format.")
            return
        if exp_date < date.today():
            print("DENIED: Cannot set expired date.")
            return
        prod['exp_date'] = exp_date
    print("Product updated. Quantity unchanged.")
    print_border()

def stock_in():
    print_header("STOCK IN - ADD STOCK")
    print_border()
    code = input("Enter product code: ").strip()
    prod = next((p for p in inventory if p['id'] == int(code)), None)
    if not prod:
        print("Product not found.")
        return
    qty = input(f"Quantity to ADD to {prod['name']}: ").strip()
    if not is_numbers_only(qty) or int(qty) == 0:
        print("ERROR: Must be > 0.")
        return
    prod['quantity'] += int(qty)
    print(f"| Stock updated. New quantity: {prod['quantity']} |")
    print_border()

def stock_out():
    print_header("STOCK OUT - DEDUCT STOCK")
    print_border()
    code = input("Enter product code: ").strip()
    prod = next((p for p in inventory if p['id'] == int(code)), None)
    if not prod:
        print("Product not found.")
        return
    if prod['quantity'] == 0:
        print("Out of stock.")
        return
    qty = input(f"Quantity to DEDUCT from {prod['name']} ({prod['quantity']} avail): ").strip()
    if not is_numbers_only(qty) or int(qty) == 0:
        print("ERROR: Must be > 0.")
        return
    if int(qty) > prod['quantity']:
        print("Cannot deduct more than available.")
        return
    prod['quantity'] -= int(qty)
    print(f"| Stock adjusted. New quantity: {prod['quantity']} |")
    print_border()

def supplier_report():
    print_header("SUPPLIER INFORMATION REPORT")
    print_border()
    if not inventory:
        print("| No products. No suppliers to show. |")
        print_border()
        return
    suppliers = {}
    for p in inventory:
        s = p['supplier']
        if s not in suppliers:
            suppliers[s] = []
        suppliers[s].append(p)
    print(f"| Total Suppliers: {len(suppliers)} |")
    print_border()
    print(f"| {'SUPPLIER':<15} | {'PRODUCTS':<8} | {'TOTAL STOCK':<11} | {'PRODUCT LIST'} |")
    print("+" + "-"*70 + "+")
    for supplier, products in suppliers.items():
        total_stock = sum(p['quantity'] for p in products)
        prod_count = len(products)
        prod_names = ", ".join(p['name'] for p in products)
        print(f"| {supplier:<15} | {prod_count:<8} | {total_stock:<11} | {prod_names} |")
    print_border()

def expiry_report():
    print_header("EXPIRY MONITORING REPORT")
    print_border()
    today = date.today()
    expired = [p for p in inventory if p['exp_date'] and p['exp_date'] < today]
    expiring = [p for p in inventory if p['exp_date'] and 0 <= (p['exp_date'] - today).days <= 7]
    if not expired and not expiring:
        print("| No expired or expiring products. |")
        print_border()
        return
    if expired:
        print("| --- EXPIRED PRODUCTS --- |")
        for p in expired:
            print(f"| {p['id']} | {p['name']} | Expired {p['exp_date']} | {p['quantity']} pcs | Supplier: {p['supplier']} |")
    if expiring:
        print("| --- EXPIRING WITHIN 7 DAYS --- |")
        for p in expiring:
            days = (p['exp_date'] - today).days
            print(f"| {p['id']} | {p['name']} | {p['exp_date']} | {days} days left | Supplier: {p['supplier']} |")
    print_border()

def main_menu(current_role):
    while True:
        print_header("INVENTORY MONITORING SYSTEM")
        print(f"Logged in as: {current_role}".center(72))
        print("+----------+-------------------------------+")
        print(f"| {'OPTION':<8} | {'ACTION':<29} |")
        print("+----------+-------------------------------+")
        if current_role == "Admin / Inventory Manager":
            print(f"| {'1':<8} | {'Dashboard':<29} |")
            print(f"| {'2':<8} | {'Add Product':<29} |")
            print(f"| {'3':<8} | {'View Inventory / Shelf':<29} |")
            print(f"| {'4':<8} | {'Find Product':<29} |")
            print(f"| {'5':<8} | {'View by Category':<29} |")
            print(f"| {'6':<8} | {'Update Product':<29} |")
            print(f"| {'7':<8} | {'Stock In':<29} |")
            print(f"| {'8':<8} | {'Stock Out':<29} |")
            print(f"| {'9':<8} | {'Supplier Information':<29} |")
            print(f"| {'10':<8} | {'Expiry Report':<29} |")
            print(f"| {'11':<8} | {'Logout / Exit':<29} |")
            print("+----------+-------------------------------+")
            choice = input("Choose an option (1-11): ").strip()
            if choice == "1":
                dashboard()
            elif choice == "2":
                add_product()
            elif choice == "3":
                view_inventory()
            elif choice == "4":
                find_product()
            elif choice == "5":
                category_view()
            elif choice == "6":
                update_product()
            elif choice == "7":
                stock_in()
            elif choice == "8":
                stock_out()
            elif choice == "9":
                supplier_report()
            elif choice == "10":
                expiry_report()
            elif choice == "11":
                print("Logging out. Goodbye!")
                break
            else:
                print("Invalid choice.")
        else:
            print(f"| {'1':<8} | {'Dashboard':<29} |")
            print(f"| {'2':<8} | {'View Inventory / Shelf':<29} |")
            print(f"| {'3':<8} | {'Find Product':<29} |")
            print(f"| {'4':<8} | {'View by Category':<29} |")
            print(f"| {'5':<8} | {'Supplier Information':<29} |")
            print(f"| {'6':<8} | {'Expiry Report':<29} |")
            print(f"| {'7':<8} | {'Logout / Exit':<29} |")
            print("+----------+-------------------------------+")
            choice = input("Choose an option (1-7): ").strip()
            if choice == "1":
                dashboard()
            elif choice == "2":
                view_inventory()
            elif choice == "3":
                find_product()
            elif choice == "4":
                category_view()
            elif choice == "5":
                supplier_report()
            elif choice == "6":
                expiry_report()
            elif choice == "7":
                print("Logging out. Goodbye!")
                break
            else:
                print("Invalid choice.")

if __name__ == "__main__":
    user, role = login()
    main_menu(role)
