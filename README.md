# Meal Selection Algorithm

**CS315 – Algorithm Design & Analysis** · Group 3332 · Semester 462
College of Computer, Department of Computer Science – Qassim University
**Supervisor:** Dr. Rinad Al-Suwaid

---

## Table of Contents

1. [Overview](#1-overview)
2. [Theoretical Analysis](#2-theoretical-analysis)
3. [Empirical Analysis](#3-empirical-analysis)
4. [Conclusion](#4-conclusion)
5. [How to Run](#how-to-run)

---

## 1. Overview

### 1.1 Introduction

This project develops an algorithm that selects the best meal based on user-defined constraints: maximum calories, maximum preparation time, diet type, and unwanted ingredients. The goal is to support healthier and faster food choices.

Two approaches are implemented and compared: a **naive** algorithm and an **optimized** algorithm. The objective is to select the meal with the **highest protein content** without exceeding the calorie and time limits, while respecting dietary restrictions and avoiding unwanted ingredients.

```
Meals Database  →  Filter by calories & prep time  →  Filter by diet & unwanted items  →  Select meal with highest protein
```

### 1.2 Problem Description

Given a set of meals, select the meal satisfying:

- `calories <= max_calories`
- `prep_time <= max_prep_time`
- matches the diet type (if specified)
- does not contain unwanted ingredients
- has the highest protein content among all valid meals

**Sample inputs and outputs**

| Input | Output |
|-------|--------|
| `max_calories=500`, `max_prep_time=30`, `diet_type="Vegetarian"`, `unwanted_ingredients=["peanuts"]` | "Grilled Tofu Salad" with 25g protein |
| `max_calories=800`, `max_prep_time=45` | "Chicken Breast with Rice" with 45g protein |
| `max_calories=400`, `max_prep_time=20`, `diet_type="Vegan"`, `unwanted_ingredients=["cheese"]` | "Quinoa Veggie Bowl" with 20g protein |

---

## 2. Theoretical Analysis

### 2.1 Naive Algorithm

**Rationale:** iterate over every meal and check all constraints one by one.

1. Initialize `best_meal = NIL`, `best_score = -1`.
2. For each meal: skip it if it exceeds the calorie or preparation-time limit.
3. Skip it if a diet type is specified and the meal does not match.
4. Skip it if it contains any unwanted ingredient.
5. If its protein is higher than `best_score`, it becomes the new `best_meal`.
6. Return `best_meal`.

#### 2.1.1 Pseudocode

```text
NAIVE-SELECT-MEAL(meals, max_calories, max_prep_time, diet_type = "", unwanted_ingredients = ∅)
1.  if unwanted_ingredients = ∅ then
2.      unwanted_ingredients ← ∅
3.  best_meal ← NIL
4.  best_score ← -1
5.  for each meal ∈ meals do
6.      if meal.calories > max_calories or meal.prep_time > max_prep_time then
7.          continue
8.      if diet_type ≠ "" and meal.diet_type ≠ diet_type then
9.          continue
10.     has_unwanted ← any ingredient ∈ meal.ingredients such that ingredient ∈ unwanted_ingredients
11.     if has_unwanted then
12.         continue
13.     if meal.protein > best_score then
14.         best_score ← meal.protein
15.         best_meal ← meal
16. return best_meal
```

#### 2.1.2 Analysis

Let **n** = number of meals, **k** = ingredients per meal, **u** = number of unwanted ingredients.

| Step | Cost per meal |
|------|---------------|
| Calories / prep-time check | O(1) |
| Diet type check | O(1) |
| Unwanted ingredients check | O(k × u) worst case (O(k) if `unwanted_ingredients` is a hash set) |
| Best-meal update | O(1) |

**Time complexity:** O(n × k × u), which simplifies to O(n) when k and u are small constants.
**Space complexity:** O(1) extra space (the unwanted set is part of the input).

| Scenario | Time Complexity |
|----------|-----------------|
| Best case | O(n) |
| Average case | O(n × k × u) |
| Worst case | O(n × k × u) |

#### 2.1.3 Example

**Meals**

| # | Name | Calories | Protein | Prep (min) | Diet | Ingredients |
|---|------|----------|---------|------------|------|-------------|
| 1 | Grilled Chicken | 400 | 25g | 30 | keto | chicken, spices |
| 2 | Vegan Bowl | 350 | 15g | 15 | vegan | beans, rice, vegetables |
| 3 | Grilled Chicken Salad | 450 | 30g | 15 | keto | chicken, lettuce, tomato |

**Constraints:** max calories 450 · max prep time 20 · diet `keto` · unwanted `["tomato"]`

1. **Calories & prep time:** Grilled Chicken is excluded (30 min). Vegan Bowl and Grilled Chicken Salad pass.
2. **Diet type:** Vegan Bowl is excluded (`vegan` ≠ `keto`). Grilled Chicken Salad passes.
3. **Unwanted ingredients:** Grilled Chicken Salad contains `tomato` and is excluded.
4. **Selection:** no meals remain.

**Output:** `No suitable meal found.`

---

### 2.2 Optimized Algorithm

**Rationale:** first filter the list by calories and preparation time, producing a smaller subset, then run the remaining checks only on that subset. Ingredient checks use set intersection.

1. If `unwanted_ingredients` is `None`, initialize it to an empty set.
2. Build `filtered_meals` containing only meals within the calorie and time limits.
3. Initialize `best_meal = NIL`, `best_score = -1`.
4. For each filtered meal: skip if the diet type doesn't match, skip if it contains an unwanted ingredient, otherwise update the best meal if its protein is higher.
5. Return `best_meal`.

#### 2.2.1 Pseudocode

```text
OPTIMIZED-SELECT-MEAL(meals, max_calories, max_prep_time, diet_type = "", unwanted_ingredients = ∅)
1.  if unwanted_ingredients = None then
2.      unwanted_ingredients ← ∅
3.  filtered_meals ← { meal ∈ meals | meal.calories ≤ max_calories ∧ meal.prep_time ≤ max_prep_time }
4.  best_meal ← NIL
5.  best_score ← -1
6.  for each meal ∈ filtered_meals do
7.      if diet_type ≠ "" ∧ meal.diet_type ≠ diet_type then
8.          continue
9.      if unwanted_ingredients ∩ meal.ingredients ≠ ∅ then
10.         continue
11.     if meal.protein > best_score then
12.         best_score ← meal.protein
13.         best_meal ← meal
14. return best_meal
```

#### 2.2.2 Analysis

Let **f** = number of meals that pass the first filter.

- Initial filtering: **O(n)**
- Checks on filtered meals: **O(f × min(k, u))** (set intersection)

**Time complexity:** O(n + f × min(k, u))
**Space complexity:** O(f) for the filtered list (O(n + u) in the worst case, counting the unwanted set)

> This analysis assumes `meal.ingredients` and `unwanted_ingredients` are stored as sets.

| Scenario | Time Complexity |
|----------|-----------------|
| Best case (filter removes most meals) | O(n) |
| Average case | O(n + f × min(k, u)) |
| Worst case (f ≈ n) | O(n + n × min(k, u)) |

#### 2.2.3 Example

**Meals**

| # | Name | Calories | Prep (min) | Diet | Ingredients |
|---|------|----------|------------|------|-------------|
| 1 | Chicken Salad | 350 | 10 | LowCarb | chicken, lettuce, tomato, olive oil |
| 2 | Veggie Stir Fry | 250 | 15 | Vegan | tofu, broccoli, carrot, soy sauce |
| 3 | Grilled Chicken | 450 | 25 | LowCarb | chicken, olive oil, garlic |
| 4 | Beef Burger | 700 | 20 | Keto | beef, cheese, lettuce, onion |

**Constraints:** max calories 500 · max prep time 20 · diet `LowCarb` · unwanted `["tomato", "onion"]`

1. **Calories & prep time:** Chicken Salad ✅, Veggie Stir Fry ✅, Grilled Chicken ❌ (25 min), Beef Burger ❌ (700 cal).
2. **Diet type:** Chicken Salad ✅, Veggie Stir Fry ❌.
3. **Unwanted ingredients:** Chicken Salad contains `tomato` ❌.
4. **Result:** no meals remain → `No Meal Found`.

| Stage | Meals left |
|-------|-----------|
| Initial | 4 |
| After calories & prep time | 2 |
| After diet type | 1 |
| After unwanted ingredients | 0 |

---

### 2.3 Comparison

| Aspect | Naive Algorithm | Optimized Algorithm |
|--------|-----------------|---------------------|
| Time complexity | O(n × k × u) | O(n + f × min(k, u)) |
| Space complexity | O(1) | O(f) |
| Advantages | Very simple, no extra data structures | Fewer checks, better for large datasets |
| Disadvantages | Scans and fully checks every meal | Creates a filtered list, slightly more complex |

---

## 3. Empirical Analysis

### 3.1 Naive Algorithm Implementation

Iterates through each meal individually with no preliminary filtering. See [`naive_select_meal.py`](naive_select_meal.py).

```python
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
```

**Sample run**

```text
Enter max calories: 400
Enter max preparation time (minutes): 30
Enter diet type (leave blank if no preference): keto
Enter unwanted ingredients (comma-separated): rice
Recommended meal: Grilled Chicken
```

### 3.2 Optimized Algorithm Implementation

Filters by calories and preparation time first, then checks diet type and unwanted ingredients (set intersection) on the remaining meals. See [`optimized_select_meal.py`](optimized_select_meal.py).

```python
def optimized_select_meal(meals, max_calories, max_prep_time, diet_type="", unwanted_ingredients=None):
    if unwanted_ingredients is None:
        unwanted_ingredients = set()
    filtered_meals = [meal for meal in meals
                      if meal['calories'] <= max_calories and meal['prep_time'] <= max_prep_time]
    best_meal = None
    best_score = -1
    for meal in filtered_meals:
        if diet_type and meal['diet_type'] != diet_type:
            continue
        if unwanted_ingredients & set(meal['ingredients']):
            continue
        if meal['protein'] > best_score:
            best_score = meal['protein']
            best_meal = meal
    return best_meal
```

**Sample run**

```text
Enter max calories: 350
Enter max preparation time (minutes): 15
Enter diet type (leave blank if no preference): vegan
Enter unwanted ingredients (comma-separated): chicken
Recommended meal: Vegan Bowl
```

### 3.3 Performance Analysis

Both algorithms were tested under identical conditions with varying calorie limits, preparation times, diet types, and unwanted ingredients.

**Observations**

- For small datasets (fewer than 10 meals) both performed similarly.
- As the number of meals grew, the optimized algorithm was significantly faster thanks to its initial filtering.
- The naive algorithm slowed down noticeably with hundreds or thousands of meals.

**Memory:** both use similar memory (the meals list plus a few variables). The optimized version uses slightly more because of the filtered list, but the impact is minimal compared to the speed gain.

**Summary of empirical findings**

| Criterion | Naive | Optimized |
|-----------|-------|-----------|
| Speed on small datasets | Fast | Fast |
| Speed on large datasets | Slow | Fast |
| Memory usage | Low | Low |
| Implementation simplicity | Very simple | Moderate |

> Add your charts here: save the two images from the PDF (Execution Time Comparison, Memory Usage Comparison, and Performance and Memory Trend) into an `images/` folder and embed them like this:
>
> `![Execution Time](images/execution_time.png)`

---

## 4. Conclusion

### 4.1 Comparison

We developed and analyzed two methods for selecting the best meal from user-defined criteria. The naive algorithm is easy to implement but scales worse on large datasets because every meal is fully checked. The optimized algorithm filters early and uses set operations, reducing the number of expensive checks and giving better performance as the number of meals grows. Experimental results supported the theoretical analysis.

### 4.2 Final Thoughts

Both analyses agree that the optimized algorithm is more efficient, especially for large datasets. Minor differences between theory and practice are due to hardware limitations and background processes during testing. This project shows how thoughtful optimization can bring significant performance improvements.

---

## How to Run

Requires Python 3.

```bash
python naive_select_meal.py
python optimized_select_meal.py
```

Each script asks for max calories, max preparation time, diet type (optional), and unwanted ingredients (comma-separated), then prints the recommended meal.
