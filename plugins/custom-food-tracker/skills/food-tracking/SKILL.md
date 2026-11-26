---
name: food-tracking
description: Log foods and track nutrition with the Custom Food Tracker connector. Use when the user says what they ate or drank, describes a meal or snack, asks about calories, protein or other nutrients, sets or changes a nutrition goal, saves a custom food or recipe, corrects something logged, or wants to see a day, a week, or averages of what they've eaten.
---

The Custom Food Tracker connector keeps the user's custom foods, what they eat each day, and their daily nutrition goals. The user says what they ate, and you log it for the right day: today unless they say otherwise. The server keeps each day's running totals.

The server tracks calories, fat, saturated fat, trans fat, monounsaturated fat, polyunsaturated fat, sodium, potassium, carbohydrates, fiber, sugar, added sugar and protein. Fill in every one you have a value for.

## Quantities

- If the user doesn't say how much they had, ask before logging. With several foods, ask about all the missing quantities at once.
- Don't guess serving sizes, and don't call `log_food` until the quantities are confirmed.

## Where nutrition values come from

1. Values the user gives, typed or from a label photo: use them as given.
2. The user's custom foods: check them with `list_custom_foods` before searching the web. Names can be informal ("my protein shake"), so match loosely.
3. For branded or restaurant foods, the published nutrition data. For generic whole foods, prefer USDA data.

If a value is uncertain or estimated, flag it to the user with an asterisk and a brief note, and put that note in the item's `notes`.

## Logging

- Log everything from one message or meal in a single `log_food` call, one entry in `items` per food, all on one `date`. Never make one call per food. Each item is either:
  - A saved food: `custom_food_id` and `servings`, with no nutrient values.
  - Anything else: a `description` of a single serving, the per-serving nutrient values, and `servings`. Use `servings: 1` when the values already cover the whole amount.
- The server scales by `servings` and returns the items with their ids, what they add up to, and the day so far. Don't add up or save totals yourself. If any item is invalid, nothing is logged.
- Confirm briefly, for example "Logged — 1,240 cal so far, 560 remaining." With no calorie goal saved, just "Logged — 1,240 cal so far."

## Corrections

- Make all of a day's fixes in one call, and all of a day's removals in one call.
- Wrong amount: `update_logged_food` with `items: [{id, servings}, …]`. It rescales every nutrient.
- Wrong food or wrong values: `delete_logged_food` with `ids`, then log them again.
- Get item ids from the `log_food` result or, for food logged earlier, from `list_logged_foods`, which shows the user nothing. Use `get_daily_nutrition_total` only when the user wants to see their day.
- A whole past day given as totals: `backfill_day_total`. It only works on a day with nothing logged. If the day already has entries, ask whether to log the foods instead or clear the day first, and don't clear it unasked.
- Only delete things when the user asks.

## Custom foods

- When the user says "save this food" or "save as a custom food", store it with `create_custom_food`. Values are per single serving, with the serving size defined.
- If the user is refining a food that already exists, update it with `update_custom_food` rather than creating a near-duplicate.
- Updating a saved food doesn't change items already logged from it. If the user wants a day to reflect the change, delete and re-log those items.

## Goals

The user's goals are saved on the server, and every tool that shows totals compares against them on its own. Don't pass `targets` to those tools.

- When the user gives a new goal or changes one, save it with `set_nutrition_targets`. `targets` adds or replaces only the goals you pass, for example `{"targets": {"added_sugar": 25}}`. To drop one, use `remove`, for example `{"remove": ["added_sugar"]}`. Never make up a goal.
- A plain number is a limit for most nutrients and a goal to reach for protein, fiber, potassium and the unsaturated fats. Use `{"value": n, "type": "min" | "max"}` to say which, or `{"type": "range", "min": a, "max": b}` for a range.
- When the user asks what their goals are, call `get_nutrition_targets`.
- For "what if" questions only (for example "how would last week look at 2,000 cal?"), pass `targets` to the read tool for that one call. It replaces all the saved goals for that call, so get them first and include any that should still be compared. Don't save these unless the user says so.

## Views

`log_food`, `update_logged_food`, `delete_logged_food`, `backfill_day_total`, `get_daily_nutrition_total`, `get_daily_nutrition_totals_by_date_range`, `get_average_daily_nutrition_totals_by_date_range`, `get_average_daily_nutrition_totals_by_day_of_week`, `get_custom_food` and `get_logged_food` show the user their result as an interactive view.

- When a result says the user can already see it, don't repeat its numbers or lists. Answer the question in a sentence or two, or say nothing if they just wanted to see it.
- Leave `verbose` off, which is the default. Set `verbose: true` only when a detailed analysis the user asked for needs more than the summary, such as comparing specific days, finding patterns across days, or breaking a day down by food or nutrient. Never set it just to show or restate the data.

## Errors

- A failed call returns an error that says what was wrong. `log_food`, `update_logged_food`, `delete_logged_food` and `set_nutrition_targets` change nothing when they fail, so fix what the error names and resend the whole call.
- An error that's only "Error executing tool …" with no reason is a server problem. Don't retry it; tell the user it didn't go through.

## Times

Tool timestamps are UTC. Convert them to the user's local time when you show them.
