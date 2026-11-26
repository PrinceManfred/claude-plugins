---
name: food-tracking
description: Log foods and track nutrition with the Custom Food Tracker connector. Use when the user says what they ate or drank, describes a meal or snack, asks about calories, protein or other nutrients, sets or changes a nutrition goal, saves a custom food or recipe, or wants to see a day, a week, or averages of what they've eaten.
---

The Custom Food Tracker connector keeps the user's custom foods, what they eat each day, and their daily nutrition goals.

## Logging

- Log everything the user ate in one message or meal with a single `log_food` call, one entry in `items` per food, rather than one call per food.
- Make all of a day's corrections in one `update_logged_food` call, and remove a day's items in one `delete_logged_food` call.
- To find the ids of items logged earlier, for example in another conversation, use `list_logged_foods`. It shows the user nothing.
- Use `get_daily_nutrition_total` when the user wants to see the day.

## Goals

The user's nutrition goals are saved on the server.

- When the user states a goal or changes one, save it with `set_nutrition_targets`. Save only goals they stated, and don't invent any.
- `log_food`, `update_logged_food`, `delete_logged_food`, `backfill_day_total`, `get_daily_nutrition_total`, `get_daily_nutrition_totals_by_date_range`, `get_average_daily_nutrition_totals_by_date_range` and `get_average_daily_nutrition_totals_by_day_of_week` compare their totals against the saved goals on their own, so don't pass `targets` to them.
- Pass `targets` to a read tool only to compare against different goals for one call, such as a goal the user is considering.

## Views

Those eight tools show their result to the user as an interactive view, with each total against the saved goals.

- Leave `verbose` off, which is the default. You then get a summary while the user sees everything in the view.
- Set `verbose: true` only when a detailed analysis the user asked for needs more than the summary, such as comparing specific days, finding patterns across days, or breaking a day down by food or nutrient. Never set it just to show or restate the data, because the user already sees it.
- Don't repeat numbers the view already shows. Answer what the user asked, or add a short comment.
- `get_custom_food` and `get_logged_food` show the user a Nutrition Facts label of the one food, so don't restate its values.
