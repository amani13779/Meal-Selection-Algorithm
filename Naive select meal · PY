def naive_select_meal(meals, max_calories, max_prep_time, diet_type="", unwanted_ingredients=None):
    if unwanted_ingredients is None:
        unwanted_ingredients = set()
    best_meal = None
    best_score = -1
    for meal in meals:
        if meal['calories'] > max_calories or meal['prep_time'] > max_prep_time:
            continue
        if diet_type and meal['diet_type'] != diet_type:
            continue
        has_unwanted = any(ingredient in unwanted_ingredients for ingredient in meal['ingredients'])
        if has_unwanted:
            continue
        if meal['protein'] > best_score:
            best_score = meal['protein']
            best_meal = meal
    return best_meal


if __name__ == "__main__":
    meals = [
        {'name': 'Grilled Chicken', 'calories': 400, 'protein': 25, 'prep_time': 30, 'diet_type': 'keto', 'ingredients': ['chicken', 'spices']},
        {'name': 'Vegan Bowl', 'calories': 350, 'protein': 15, 'prep_time': 15, 'diet_type': 'vegan', 'ingredients': ['beans', 'rice', 'vegetables']},
        {'name': 'Grilled Chicken Salad', 'calories': 450, 'protein': 30, 'prep_time': 15, 'diet_type': 'keto', 'ingredients': ['chicken', 'lettuce', 'tomato']},
    ]

    max_calories = int(input("Enter max calories: "))
    max_prep_time = int(input("Enter max preparation time (minutes): "))
    diet_type = input("Enter diet type (leave blank if no preference): ").strip()
    unwanted_input = input("Enter unwanted ingredients (comma-separated): ").strip()
    unwanted_ingredients = set(i.strip() for i in unwanted_input.split(',')) if unwanted_input else set()

    result = naive_select_meal(meals, max_calories, max_prep_time, diet_type, unwanted_ingredients)

    if result:
        print("Recommended meal:", result['name'])
    else:
        print("No suitable meal found.")
