import json
import os
from datetime import datetime

class Product:
    def __init__(self, product_id, name, price, stock):
        self.product_id = product_id
        self.name = name
        self.price = price
        self.stock = stock
    def __str__(self):
        return f"ID: {self.product_id} | {self.name} - ${self.price:.2f} (Stock: {self.stock})"

class CartItem:
    def __init__(self, product, quantity):
        self.product = product
        self.quantity = quantity
    
    def get_total_price(self):
        return self.product.price * self.quantity
    
    def __str__(self):
        return f"{self.product.name} x{self.quantity} - ${self.get_total_price():.2f}"

class ShoppingCart:
    def __init__(self):
        self.items = []
        self.total = 0.0
    
    def add_item(self, product, quantity):
        # Check if product already exists in cart
        for item in self.items:
            if item.product.product_id == product.product_id:
                item.quantity += quantity
                self.update_total()
                return
        
        # Add new item
        new_item = CartItem(product, quantity)
        self.items.append(new_item)
        self.update_total()
    
    def remove_item(self, product_id):
        self.items = [item for item in self.items if item.product.product_id != product_id]
        self.update_total()
    
    def update_quantity(self, product_id, new_quantity):
        for item in self.items:
            if item.product.product_id == product_id:
                if new_quantity <= 0:
                    self.remove_item(product_id)
                else:
                    item.quantity = new_quantity
                    self.update_total()
                return
        print("Item not found in cart!")
    
    def update_total(self):
        self.total = sum(item.get_total_price() for item in self.items)
    
    def clear_cart(self):
        self.items = []
        self.total = 0.0
    
    def __str__(self):
        if not self.items:
            return "Your cart is empty."
        
        cart_str = "=== SHOPPING CART ===\n"
        for i, item in enumerate(self.items, 1):
            cart_str += f"{i}. {item}\n"
        cart_str += f"\nTotal: ${self.total:.2f}"
        return cart_str

class ShoppingSystem:
    def __init__(self):
        self.products = []
        self.cart = ShoppingCart()
        self.load_products()
    
    def load_products(self):
        """Load products from products.json or create default products"""
        filename = "products.json"
        if os.path.exists(filename):
            try:
                with open(filename, 'r') as f:
                    data = json.load(f)
                    for p in data:
                        product = Product(p['id'], p['name'], p['price'], p['stock'])
                        self.products.append(product)
            except:
                self.create_default_products()
        else:
            self.create_default_products()
    
    def create_default_products(self):
        default_products = [
            {"id": 1, "name": "Laptop", "price": 999.99, "stock": 10},
            {"id": 2, "name": "Mouse", "price": 25.50, "stock": 50},
            {"id": 3, "name": "Keyboard", "price": 89.99, "stock": 30},
            {"id": 4, "name": "Monitor", "price": 299.99, "stock": 15},
            {"id": 5, "name": "Headphones", "price": 79.99, "stock": 25}
        ]
        
        self.products = []
        for p in default_products:
            product = Product(p['id'], p['name'], p['price'], p['stock'])
            self.products.append(product)
        
        self.save_products()
    
    def save_products(self):
        """Save products to JSON file"""
        data = []
        for product in self.products:
            data.append({
                "id": product.product_id,
                "name": product.name,
                "price": product.price,
                "stock": product.stock
            })
        
        with open("products.json", "w") as f:
            json.dump(data, f, indent=2)
    
    def display_products(self):
        print("\n=== AVAILABLE PRODUCTS ===")
        for product in self.products:
            print(product)
    
    def display_cart(self):
        print("\n" + str(self.cart))
    
    def add_to_cart(self):
        self.display_products()
        try:
            product_id = int(input("\nEnter product ID to add: "))
            quantity = int(input("Enter quantity: "))
            
            product = next((p for p in self.products if p.product_id == product_id), None)
            if not product:
                print("Product not found!")
                return
            
            if quantity > product.stock:
                print(f"Only {product.stock} items available in stock!")
                return
            
            self.cart.add_item(product, quantity)
            print(f"Added {quantity} x {product.name} to cart!")
            
            # Update stock
            product.stock -= quantity
            self.save_products()
            
        except ValueError:
            print("Please enter valid numbers!")
    
    def remove_from_cart(self):
        if not self.cart.items:
            print("Cart is empty!")
            return
        
        self.display_cart()
        try:
            product_id = int(input("\nEnter product ID to remove: "))
            self.cart.remove_item(product_id)
            print("Item removed from cart!")
            
            # Restore stock
            product = next((p for p in self.products if p.product_id == product_id), None)
            if product:
                removed_qty = next((item.quantity for item in self.cart.items if item.product.product_id == product_id), 0)
                product.stock += removed_qty
                self.save_products()
                
        except ValueError:
            print("Please enter a valid product ID!")
    
    def checkout(self):
        if not self.cart.items:
            print("Cart is empty! Nothing to checkout.")
            return
        
        print("\n=== CHECKOUT ===")
        self.display_cart()
        
        print("\nThank you for your purchase!")
        print(f"Order placed at: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")
        print(f"Total Amount: ${self.cart.total:.2f}")
        
        # Clear cart after successful checkout
        self.cart.clear_cart()
        print("Cart has been cleared.")
    
    def run(self):
        while True:
            print("\n" + "="*40)
            print("     🛒 SHOPPING CART SYSTEM 🛒")
            print("="*40)
            print("1. View Products")
            print("2. View Cart")
            print("3. Add to Cart")
            print("4. Remove from Cart")
            print("5. Checkout")
            print("6. Exit")
            print("-"*40)
            
            choice = input("Enter your choice (1-6): ").strip()
            
            if choice == '1':
                self.display_products()
            elif choice == '2':
                self.display_cart()
            elif choice == '3':
                self.add_to_cart()
            elif choice == '4':
                self.remove_from_cart()
            elif choice == '5':
                self.checkout()
            elif choice == '6':
                print("\nThank you for shopping with us! 👋")
                break
            else:
                print("Invalid choice! Please try again.")

# Run the application
if __name__ == "__main__":
    shop = ShoppingSystem()
    shop.run()
