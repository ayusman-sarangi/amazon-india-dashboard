# Amazon India Dashboard

A one-page management dashboard built for Manoj to share with his manager, Ravi. It shows how the Amazon India business is performing using simple charts and plain-English summaries, so no technical knowledge is needed to read it.

Data source: `Amazon_Sales_Data_India.xlsx` (10,000 orders, January 2024 to August 2026).

## Live Demo

**Dashboard:** [https://ayusman8260.github.io/amazon-india-dashboard/](https://ayusman8260.github.io/amazon-india-dashboard/)

## What the dashboard shows

| Section | Business question | What you see |
|---|---|---|
| Summary for Ravi | What should I know in 30 seconds? | Three short answers: are we growing, where are we losing business, where does business come from |
| 1. Overall sales and profit | How is the business doing? | Total Sales, Total Profit, Total Orders, Total Units Sold, Average Order Value, Profit Margin %, monthly sales and profit trend |
| 2. Categories and products | What drives the business? | Sales, profit and units by category, top 10 products by sales |
| 3. Order status and revenue loss | How many orders are completed and how many are lost? | Delivered, shipped, returned and cancelled orders, return rate, cancellation rate, rates by category, sales value lost |
| 4. Payment, fulfilment and states | Which channels and markets bring business? | Sales and profit by payment method and fulfilment method, state-wise sales, profit and orders |

A period selector at the top lets you switch between all periods, 2024, 2025 and 2026 (January to August). Every number and chart updates.

## Files

There are two ways to use this project.

**Option A: single file (simplest)**
- `amazon_india_dashboard.html`: the dashboard with all the data built in. Double-click it to open it in a browser.

**Option B: two files (easier to read and edit)**
- `index.html`: the dashboard code.
- `data.json`: the order data the dashboard reads.

Keep both files in the same folder. This version must be opened from a web address, such as GitHub Pages or a local server. It shows blank charts if you open `index.html` directly from your computer.

## How to run it

**Single file:** double-click `amazon_india_dashboard.html`.

**Two files, on your computer:**
```
python3 -m http.server
```
Then open `http://localhost:8000` in your browser.

**On GitHub Pages:**
1. Upload `index.html` and `data.json` to a repository (Add file, then Upload files).
2. Go to Settings, then Pages.
3. Under Source, choose Deploy from a branch, select `main` and `/ (root)`, and save.
4. After a minute or two, the dashboard is live at `https://<your-username>.github.io/<repository-name>/`.

The page loads Chart.js and the Manrope font from the internet, so it needs a connection to display properly.

## How the numbers are calculated

- **Total Sales:** the full value of every order, including returned and cancelled orders (the `Total_Sales_INR` column).
- **Total Profit:** the sum of `Profit_INR`. Returned and cancelled orders have zero profit in the data.
- **Total Orders:** the number of orders (one row per order).
- **Total Units Sold:** the sum of `Quantity`.
- **Average Order Value:** Total Sales divided by Total Orders.
- **Profit Margin %:** Total Profit divided by Total Sales.
- **Return Rate and Cancellation Rate:** returned (or cancelled) orders divided by all orders. The category chart uses each category's own orders.
- **Sales lost to returns and cancellations:** the sales value of all returned and cancelled orders.
- **Growth statement:** compares average monthly sales in the later half of the selected period with the earlier half.
- **Amounts:** shown in rupees. 1 Lakh = 1,00,000 and 1 Cr = 1,00,00,000.

If you prefer net sales (delivered and shipped orders only), the calculation in the `run()` function of the code needs a small change.

## Data columns used

`Order_ID`, `Order_Date`, `Category`, `Product`, `Quantity`, `Unit_Price_INR`, `Discount_Pct`, `Total_Sales_INR`, `Profit_INR`, `Payment_Method`, `Fulfillment`, `Order_Status`, `Ship_State`

## Updating the data

To refresh the dashboard with a new Excel file that has the same columns, run this script. It rebuilds `data.json`.

```python
import pandas as pd, json

d = pd.read_excel("Amazon_Sales_Data_India.xlsx")
d["ym"] = pd.to_datetime(d.Order_Date).dt.strftime("%Y-%m")
months = sorted(d.ym.unique())

def codes(col):
    values = sorted(d[col].unique())
    return values, {v: i for i, v in enumerate(values)}

cats, cm = codes("Category")
prods, pm = codes("Product")
pays, pym = codes("Payment_Method")
fuls, fm = codes("Fulfillment")
stats, sm = codes("Order_Status")
states, stm = codes("Ship_State")
mi = {m: i for i, m in enumerate(months)}

rows = [[mi[r.ym], cm[r.Category], pm[r.Product], int(r.Quantity),
         round(r.Total_Sales_INR, 2), round(r.Profit_INR, 2),
         pym[r.Payment_Method], fm[r.Fulfillment], sm[r.Order_Status],
         stm[r.Ship_State]] for r in d.itertuples()]

data = dict(months=months, cats=cats, prods=prods, pays=pays,
            fuls=fuls, stats=stats, states=states, rows=rows)
json.dump(data, open("data.json", "w"), separators=(",", ":"))
```

Each row in `data.json` is: month number, category, product, quantity, sales, profit, payment method, fulfilment method, order status, state. The category, product and other text values are stored as positions in the lists at the top of the file.

The single-file version has the data inside the HTML, so updating it means replacing the `const D=...` data line near the top of the script. The two-file version is easier to update.

## Notes

- The data is shown to anyone who can open the page. Use a private repository if the sales figures are confidential.
- Fulfilment terms: Amazon (FBA) means Amazon stores, packs and ships. Merchant (FBM) means the seller ships. Seller Flex means the seller ships from their own warehouse using Amazon's process.

Built with HTML, JavaScript and [Chart.js](https://www.chartjs.org/).
