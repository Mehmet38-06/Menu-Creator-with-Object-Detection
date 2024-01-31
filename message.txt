import cv2
import numpy as np


class Dish:
    def __init__(self, name, color, shape, price):
        self.name = name
        self.color = color
        self.shape = shape
        self.price = price


class Menu:
    def __init__(self, dishes):
        self.dishes = dishes
        self.starter = None
        self.main_course = None
        self.snack = None
        self.dessert = None

    def check_order(self):
        if not self.starter or not self.main_course:
            print("You need to choose at least one starter and one main course!")
            return False
        else:
            return True

    def calculate_total(self):
        total = 0
        for dish in self.dishes:
            total += dish.price
        return total


class MenuDetector:
    def __init__(self, image_path):
        self.image_path = image_path
        self.menu = Menu([])
        self.colors = {"red": "starter", "green": "snack", "blue": "main_course", "purple": "dessert"}

    def detect_order(self):
        colors, shapes = self.detect_objects()

        # Print color and shape information
        for color, shape in shapes.items():
            print(f"Detected {color} object with shape: {shape}")

        self.parse_selection(colors, shapes)
        self.validate_order()
        self.show_order()

    def detect_objects(self):
        colors = {}
        shapes = {}
        image = cv2.imread(self.image_path, cv2.IMREAD_COLOR)
        hsv = cv2.cvtColor(image, cv2.COLOR_BGR2HSV)

        # Set a minimum contour area threshold
        some_min_area_threshold = 100

        # Custom color detection logic
        for color_name, hue_range in self.custom_color_ranges().items():
            mask = cv2.inRange(hsv, hue_range[0], hue_range[1])
            contours, hierarchy = cv2.findContours(mask, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)

            # Filter out small contours
            contours = [cnt for cnt in contours if cv2.contourArea(cnt) > some_min_area_threshold]

            # Store all contours for each color
            colors[color_name] = contours

        # Custom shape detection logic
        for color, contour_list in colors.items():
            for contour in contour_list:
                shapes[color] = self.get_shape(contour)
                break  # Only consider the first contour (largest), you can modify as needed

        return colors, shapes

    def custom_color_ranges(self):
        # Define custom color ranges for each food group
        custom_ranges = {
            "red": (np.array([160, 50, 50]), np.array([180, 255, 255])),
            "green": (np.array([35, 100, 100]), np.array([75, 255, 255])),
            "blue": (np.array([75, 100, 100]), np.array([130, 255, 255])),
            "purple": (np.array([125, 50, 50]), np.array([155, 255, 255])),
        }
        return custom_ranges

    def get_shape(self, contour):
        peri = cv2.arcLength(contour, True)
        approx = cv2.approxPolyDP(contour, 0.02 * peri, True)

        if len(approx) == 3:
            return "triangle"
        elif len(approx) == 4:
            return "rectangle"
        else:
            return "circle"

    def parse_selection(self, colors, shapes):
        for color, contour_list in colors.items():
            for contour in contour_list:
                print(f"Detected color: {color}, shape: {self.get_shape(contour)}")
                dish_name = self.get_dish_name(color, self.get_shape(contour))
                self.add_dish_to_menu(dish_name, self.colors[color])

    def get_dish_name(self, color, shape):
        # Define mapping between color-shape combinations and dish names (you can customize this)
        dish_map = {
            "red-triangle": "Soup",
            "red-rectangle": "Cheese Platter",
            "red-circle": "Garlic Bread",
            "green-triangle": "Crispy Chicken",
            "green-rectangle": "Fish_&_Chips",  # Replace space with underscore
            "green-circle": "Omelet",
            "blue-triangle": "Meatballs",
            "blue-rectangle": "Casseroles",
            "blue-circle": "Fajitas",
            "purple-triangle": "Souffle",
            "purple-rectangle": "Tiramisu",
            "purple-circle": "Cheesecake"
        }
        return dish_map.get(f"{color}-{shape}", "Unknown Dish")  # Use get() with a default value

    def add_dish_to_menu(self, dish_name, category):
        # Create a Dish object with price (you can adjust prices as needed)
        dish = Dish(dish_name, category, None, {
            "Soup": 25,
            "Cheese Platter": 30,
            "Garlic Bread": 15,
            "Crispy Chicken": 40,
            "Fish_&_Chips": 35,
            "Omelet": 10,
            "Meatballs": 45,
            "Casseroles": 20,
            "Fajitas": 25,
            "Souffle": 50,
            "Tiramisu": 30,
            "Cheesecake": 15
        }.get(dish_name, 0))  # Use get() with a default value

        # Add the dish to the menu and respective category
        self.menu.dishes.append(dish)
        setattr(self.menu, category, dish)  # Set the dish as the attribute of the corresponding category

    def validate_order(self):
        # Check for duplicate colors in the menu
        seen_colors = set()
        for dish in self.menu.dishes:
            if dish.color in seen_colors:
                print("You cannot choose more than one object of the same color!")
                return
            seen_colors.add(dish.color)

        if not self.menu.check_order():
            return  # Already printed a warning message within check_order

        # Check for excessive number of dishes
        if len(self.menu.dishes) > 3:
            print("You can only choose up to 3 dishes!")
            return

        # Check if at least one dish is selected
        if len(self.menu.dishes) == 0:
            print("You need to choose at least one dish!")
            return

        # Check if only one dish is selected
        if len(self.menu.dishes) == 1:
            print("You need to choose more than one dish!")
            return

    def show_order(self):
        print("Your order is:")
        for dish in self.menu.dishes:
            print(f"- {dish.name}")

        if self.menu.check_order():
            confirmation = input("Do you confirm your order? (y/n): ")
            if confirmation.lower() == "y":
                total = self.menu.calculate_total()
                print(f"Your food is prepared successfully.")
                print(f"Total amount to pay: {total} TL")
            else:
                print("Order cancelled.")


# Example usage
detector = MenuDetector(r"C:\Users\OSMAN\Desktop\Projects\karisik2.png")
detector.detect_order()