import time
import random

# ANSI escape codes for colors
class bcolors:
    HEADER = '\033[95m'
    OKBLUE = '\033[94m'
    OKCYAN = '\033[96m'
    OKGREEN = '\033[92m'
    WARNING = '\033[93m'
    FAIL = '\033[91m'
    ENDC = '\033[0m'
    BOLD = '\033[1m'
    UNDERLINE = '\033[4m'

class MenuItem:
    """Represents an item on a restaurant's menu."""
    def __init__(self, name, price, description=""):
        self.name = name
        self.price = price
        self.description = description

    def __str__(self):
        return f"{self.name} - ${self.price:.2f}"

class Restaurant:
    """Represents a restaurant with a menu."""
    def __init__(self, name, location):
        self.name = name
        self.location = location
        self.menu = []
        self.orders = []

    def add_menu_item(self, item):
        self.menu.append(item)

    def display_menu(self):
        print(f"\n--- {bcolors.HEADER}{self.name} Menu{bcolors.ENDC} ---")
        for i, item in enumerate(self.menu, 1):
            print(f"{i}. {item}")
        print("--------------------")

    def receive_order(self, order):
        self.orders.append(order)
        print(f"{bcolors.OKCYAN}Restaurant '{self.name}' received a new order.{bcolors.ENDC}")

class Customer:
    """Represents a customer of the delivery app."""
    def __init__(self, name, address):
        self.name = name
        self.address = address

    def __str__(self):
        return f"Customer: {self.name}, Address: {self.address}"

class DeliveryDriver:
    """Represents a delivery driver."""
    def __init__(self, name, current_location):
        self.name = name
        self.current_location = current_location
        self.current_order = None
        self.is_available = True

    def assign_order(self, order):
        if self.is_available:
            self.current_order = order
            self.is_available = False
            print(f"{bcolors.OKBLUE}Driver {self.name} assigned to order #{order.order_id}.{bcolors.ENDC}")
            return True
        return False

    def complete_delivery(self):
        if self.current_order:
            self.current_order.update_status("Delivered")
            print(f"{bcolors.OKGREEN}Driver {self.name} has delivered order #{self.current_order.order_id}.{bcolors.ENDC}")
            self.current_order = None
            self.is_available = True

class Order:
    """Represents a customer's order."""
    ORDER_STATUSES = ["Pending", "Accepted by Restaurant", "Preparing", "Out for Delivery", "Delivered", "Cancelled"]

    def __init__(self, customer, restaurant):
        self.order_id = random.randint(1000, 9999)
        self.customer = customer
        self.restaurant = restaurant
        self.items = []
        self.total_cost = 0.0
        self.status = "Pending"

    def add_item(self, item, quantity=1):
        self.items.append({"item": item, "quantity": quantity})
        self.calculate_total()

    def calculate_total(self):
        self.total_cost = sum(item['item'].price * item['quantity'] for item in self.items)

    def update_status(self, new_status):
        if new_status in self.ORDER_STATUSES:
            self.status = new_status
            print(f"{bcolors.OKCYAN}Order #{self.order_id} status updated to: {self.status}{bcolors.ENDC}")
        else:
            print(f"{bcolors.FAIL}Invalid order status.{bcolors.ENDC}")

    def display_summary(self):
        print(f"\n--- {bcolors.BOLD}Order #{self.order_id} Summary{bcolors.ENDC} ---")
        print(f"Customer: {self.customer.name}")
        print(f"Restaurant: {self.restaurant.name}")
        print("Items:")
        for entry in self.items:
            print(f"  - {entry['item'].name} (x{entry['quantity']})")
        print(f"Total Cost: ${self.total_cost:.2f}")
        print(f"Status: {self.status}")
        print("--------------------------")

class DeliveryApp:
    """The main application class that orchestrates everything."""
    def __init__(self):
        self.restaurants = []
        self.drivers = []
        self.customers = []
        self.active_orders = []

    def setup(self):
        """Initial setup with sample data."""
        # Add Restaurants and Menus
        pizza_place = Restaurant("Pizza Palace", "123 Pizza St")
        pizza_place.add_menu_item(MenuItem("Margherita Pizza", 12.99, "Classic cheese and tomato"))
        pizza_place.add_menu_item(MenuItem("Pepperoni Pizza", 14.99))
        pizza_place.add_menu_item(MenuItem("Garlic Bread", 5.99))

        burger_joint = Restaurant("Burger Barn", "456 Burger Ave")
        burger_joint.add_menu_item(MenuItem("Classic Burger", 9.50, "Beef patty, lettuce, tomato"))
        burger_joint.add_menu_item(MenuItem("Cheese Burger", 10.50))
        burger_joint.add_menu_item(MenuItem("Fries", 3.50))

        sushi_house = Restaurant("Sushi Station", "789 Rice Rd")
        sushi_house.add_menu_item(MenuItem("California Roll", 8.00))
        sushi_house.add_menu_item(MenuItem("Spicy Tuna Roll", 9.00))
        sushi_house.add_menu_item(MenuItem("Miso Soup", 2.50))

        self.restaurants.extend([pizza_place, burger_joint, sushi_house])

        # Add Drivers
        self.drivers.append(DeliveryDriver("Dave", "Downtown"))
        self.drivers.append(DeliveryDriver("Diana", "Uptown"))
        self.drivers.append(DeliveryDriver("Bob", "Midtown"))

        # Add a default customer
        self.customers.append(Customer("John Doe", "555 Main St"))

    def list_restaurants(self):
        print(f"\n--- {bcolors.HEADER}Available Restaurants{bcolors.ENDC} ---")
        for i, r in enumerate(self.restaurants, 1):
            print(f"{i}. {r.name} ({r.location})")
        print("--------------------------")

    def find_available_driver(self):
        """Finds a driver who is currently available."""
        available_drivers = [d for d in self.drivers if d.is_available]
        return random.choice(available_drivers) if available_drivers else None

    def simulate_delivery_process(self, order):
        """Simulates the entire lifecycle of an order."""
        print(f"\n{bcolors.BOLD}--- Starting Delivery Simulation for Order #{order.order_id} ---{bcolors.ENDC}")
        
        # 1. Restaurant accepts the order
        time.sleep(2)
        order.restaurant.receive_order(order)
        order.update_status("Accepted by Restaurant")
        
        # 2. Restaurant prepares the food
        time.sleep(3)
        order.update_status("Preparing")
        
        # 3. Assign a driver
        time.sleep(2)
        driver = self.find_available_driver()
        if driver:
            driver.assign_order(order)
            order.update_status("Out for Delivery")
        else:
            print(f"{bcolors.WARNING}No available drivers at the moment. Order is waiting.{bcolors.ENDC}")
            order.update_status("Waiting for Driver")
            return # End simulation if no driver

        # 4. Driver delivers the food
        delivery_time = random.randint(5, 10)
        print(f"Delivery in progress... Estimated time: {delivery_time} seconds.")
        time.sleep(delivery_time)
        driver.complete_delivery()
        
        print(f"{bcolors.OKGREEN}{bcolors.BOLD}--- Simulation Complete! Enjoy your meal! ---{bcolors.ENDC}")

    def run(self):
        """Main loop for the application CLI."""
        print(f"{bcolors.HEADER}{bcolors.BOLD}Welcome to the Python Delivery App!{bcolors.ENDC}")
        current_customer = self.customers[0] # Use the default customer

        while True:
            print("\nWhat would you like to do?")
            print("1. View Restaurants")
            print("2. Place an Order")
            print("3. Exit")

            choice = input("Enter your choice: ")

            if choice == '1':
                self.list_restaurants()
            
            elif choice == '2':
                self.list_restaurants()
                try:
                    res_choice = int(input("Select a restaurant by number: ")) - 1
                    if not 0 <= res_choice < len(self.restaurants):
                        print(f"{bcolors.FAIL}Invalid selection.{bcolors.ENDC}")
                        continue
                    
                    chosen_restaurant = self.restaurants[res_choice]
                    new_order = Order(current_customer, chosen_restaurant)
                    
                    # Add items to order
                    while True:
                        chosen_restaurant.display_menu()
                        item_choice = input("Select an item by number to add to your order (or 'done' to finish): ")
                        if item_choice.lower() == 'done':
                            break
                        
                        try:
                            item_index = int(item_choice) - 1
                            if 0 <= item_index < len(chosen_restaurant.menu):
                                selected_item = chosen_restaurant.menu[item_index]
                                new_order.add_item(selected_item)
                                print(f"{bcolors.OKGREEN}Added {selected_item.name} to your order.{bcolors.ENDC}")
                            else:
                                print(f"{bcolors.FAIL}Invalid item number.{bcolors.ENDC}")
                        except ValueError:
                            print(f"{bcolors.FAIL}Invalid input. Please enter a number or 'done'.{bcolors.ENDC}")

                    if not new_order.items:
                        print(f"{bcolors.WARNING}Order cancelled because no items were added.{bcolors.ENDC}")
                        continue

                    # Confirm and process order
                    new_order.display_summary()
                    confirm = input("Confirm your order? (yes/no): ")
                    if confirm.lower() == 'yes':
                        self.active_orders.append(new_order)
                        print(f"{bcolors.OKGREEN}Order confirmed! Processing delivery...{bcolors.ENDC}")
                        self.simulate_delivery_process(new_order)
                    else:
                        print(f"{bcolors.WARNING}Order cancelled.{bcolors.ENDC}")

                except (ValueError, IndexError):
                    print(f"{bcolors.FAIL}Invalid input. Please try again.{bcolors.ENDC}")

            elif choice == '3':
                print("Thank you for using the Python Delivery App!")
                break
            else:
                print(f"{bcolors.FAIL}Invalid choice. Please enter a number from the menu.{bcolors.ENDC}")


if __name__ == "__main__":
    app = DeliveryApp()
    app.setup()
    app.run()
    app.stop()
