---
name: food-tracking
description: Log foods and track nutrition with the Custom Food Tracker connector. Use when the user says what they ate or drank, describes a meal or snack, asks about calories, protein or other nutrients, sets or changes a nutrition goal, saves a custom food or recipe, corrects something logged, or wants to see a day, a week, or averages of what they've eaten.
---

The Custom Food Tracker keeps the user's custom foods, what they eat each day, and their daily nutrition goals. The server keeps each day's running totals, so never add them up yourself.

## Dates and times

- Log for today in the user's time zone unless they say otherwise.
- Every `date`, `start_date` and `end_date` is a calendar date in the user's time zone, not a UTC date.
- Timestamps in results are UTC. Convert them to the user's local time when you show them.

## Before logging

If the user doesn't say how much they had and it can't be easily inferred, ask before logging. With several foods, ask about all the missing quantities at once. Don't call `log_food` until the quantities are confirmed.

## Where nutrition values come from

1. Values the user gives, typed or from a label photo: use them as given.
2. The user's custom foods: check them with `list_custom_foods` before looking a food up anywhere else. Names can be informal ("my protein shake"), so match loosely.
3. For branded or restaurant foods, the published nutrition data. For generic whole foods, prefer USDA data.

Fill in every nutrient you have a value for. If a value is uncertain or estimated, flag it to the user with an asterisk and a brief note, and put that note in the item's `notes`.

## Logging

- Log everything from one meal or message in a single `log_food` call, one entry in `items` per food. Never make one call per food.
- When logging a custom food id never set the description unless explicitly asked by the user.
- Confirm briefly, for example "Logged — 1,240 cal so far, 560 remaining." With no calorie goal saved, just "Logged — 1,240 cal so far."

## Fixing and removing

- Make all of a day's corrections in one `update_logged_food` call, and remove a day's items in one `delete_logged_food` call.
- Take item ids from the `log_food` result. For items logged earlier, for example in another conversation, use `list_logged_foods`: it shows the user nothing. Use `get_daily_nutrition_total` only when the user wants to see the day.
- Only delete things when the user asks.
- If the user gives a whole past day as totals but that day already has entries, ask whether to log the foods instead or clear the day first. Don't clear it unasked.

## Custom foods

- When the user says "save this food", save it per single serving with the serving size defined.
- If they're refining a food that's already saved, update it rather than creating a near-duplicate.
- Items already logged from a saved food keep its old values. After a change, ask whether to re-log them.

## Goals

- The user's goals are saved on the server, and every tool that shows totals compares against them on its own, so don't pass `targets` to them.
- Save a goal with `set_nutrition_targets` only when the user states one, and never invent one.
- For "what if" questions, such as a goal the user is considering, pass `targets` to the read tool for that one call. Don't save it unless they say so.

## Views

`log_food`, `update_logged_food`, `delete_logged_food`, `backfill_day_total`, `get_daily_nutrition_total`, `get_daily_nutrition_totals_by_date_range`, `get_average_daily_nutrition_totals_by_date_range` and `get_average_daily_nutrition_totals_by_day_of_week` show their result to the user as an interactive view, with each total against the saved goals. With `verbose` off, which is the default, you get a summary:

- for a single day (and `backfill_day_total`): the calories, the comparison with the goals, and each logged food with its id, enough to fix or remove one;
- for `log_food`, `update_logged_food` and `delete_logged_food`: each food logged, changed or removed with its id, what they add up to, the day so far and its comparison with the goals;
- for a date range: every average, the comparison with the goals, and the highest and lowest days.

Set `verbose: true` only when a detailed analysis the user asked for needs more than that, such as comparing specific days, finding patterns across days, or breaking a day down by food or nutrient. Never set it just to show or restate the data.

`get_custom_food` and `get_logged_food` show the user a Nutrition Facts label of the one food. They have no summary or `verbose`.

The user already sees every view, so don't repeat its numbers or lists. Answer what they asked in a sentence or two, or say nothing if they just wanted to see it.

## When a call fails

- An error that names what's wrong: fix it and resend the whole call. Those tools change nothing when they fail.
- A server error: don't retry. Tell the user it failed, so they can check whether anything changed.
