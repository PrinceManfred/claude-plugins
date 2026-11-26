# Custom Food Tracker

Log what you eat, save your own foods and recipes, and track your daily nutrition against your goals, all from a conversation with Claude.

## Use it

1. Install the plugin. On claude.ai or the desktop app, add the marketplace `https://github.com/PrinceManfred/claude-plugins` from **Customize > Plugins > Add > Add marketplace** and install **Custom Food Tracker**. In Claude Code, run:

   ```
   /plugin marketplace add PrinceManfred/claude-plugins
   /plugin install custom-food-tracker@ai-plugins
   ```

   Then open its **Connectors** tab and connect **custom-food-tracker**. Sign in when asked.
2. Tell Claude what you ate, for example "I had two eggs and toast for breakfast". Claude logs it under that meal and shows a Nutrition Facts label for the day, with your foods grouped by meal and dish.
3. Ask for a day, a week, or your averages by weekday to see totals and charts against your goals. Tell Claude a goal, such as "150 g of protein a day", and it saves it.

Each view has a share icon that opens it as a page in your browser, where you can share an image or a read-only link.

## Data

The plugin connects Claude to the Custom Food Tracker server at `nutrition.castleofotranto.net`. The foods you log, your custom foods and your goals are stored there under your account. Share links carry the data in the link itself, so the server never stores what you share.
