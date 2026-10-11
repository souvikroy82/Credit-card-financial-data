# Credit Card Customer & Transaction Analysis

An end-to-end business analytics project: two raw CSV files were loaded into **MySQL**, modelled and analysed in **Power BI**, and turned into two dashboards that show where a credit card portfolio earns its revenue and which customer segments drive it.

## Business Questions

- How much revenue does the portfolio earn, and from which sources (interest vs. fees)?
- Which card categories, customer segments (age, job, education, gender) and spending types contribute the most?
- How do revenue and transaction counts move across quarters and months?

## Dataset

| File | Rows | Columns | Description |
|---|---|---|---|
| `data/credit_card.csv` | 10,108 | 18 | Card and transaction data per client: card category, annual fees, credit limit, revolving balance, transaction amount and volume, utilisation ratio, channel (chip / swipe / online), expense type, interest earned, delinquency flag, week start date |
| `data/customer.csv` | 10,108 | 15 | Customer profile per client: age, gender, dependents, education, marital status, state, job, income, satisfaction score |

- Both files have exactly one row per client and are joined on `Client_Num`. Every client in the card file has a match in the customer file.
- The data covers **52 weekly periods in 2023**.

## Tools

- **MySQL 8.0** (MySQL Workbench) to store the CSV data and feed it into Power BI
- **Power BI Desktop** for most of the work: data modelling, DAX calculations and dashboards

## Approach

1. Created the database `ccdb` with two tables, `cc_details` and `cust_detail`, and imported the CSV files.
2. Cleaned the data: converted day-first text dates into a proper `DATE` column and kept `Week_Num` as text (values like `Week-1`).
3. Connected Power BI to MySQL and created a relationship between the two tables on `Client_Num`.
4. Added calculated columns and measures (below).
5. Built two dashboard pages and exported them as PDFs.

## Calculations

**Revenue** is defined as what the card issuer earns: interest plus annual fees. `Total_Trans_Amt` is customer spending, which is paid to merchants, so it is shown as a separate metric (Transaction Volume) and is not counted as revenue.

```dax
Revenue = credit_card[Interest_Earned] + credit_card[Annual_Fees]

Total Revenue = SUM(credit_card[Revenue])

Age_Group =
IF([Customer_Age] <= 33, "Young",
IF([Customer_Age] <= 46, "Mid",
IF([Customer_Age] <= 59, "Late", "Senior")))
```

| Age group | Age range |
|---|---|
| Young | 21 to 33 |
| Mid | 34 to 46 |
| Late | 47 to 59 |
| Senior | 60 to 73 |

An `Income_Group` column splits customers into four income bands, and a month column derived from `Week_Start_Date` supports the monthly trend charts.

## Dashboards

Both reports are in the `reports/` folder.

- **Credit Card Transaction Report** (`credit_card_transaction_report.pdf`): revenue, interest earned, number of transactions, total transaction amount and fees as KPIs; revenue by quarter against transaction count; revenue by age group, card category, expense type, customer job and payment channel; slicers for quarter, month and card category.
- **Credit Card Customer Report** (`credit_card_report_customer.pdf`): revenue, interest, average satisfaction score and total customer income as KPIs; monthly revenue by gender; revenue by age group, job and education; revenue by expense type.

## Key Findings

All figures are for 2023.

| Metric | Value |
|---|---|
| Total revenue | 10.79M |
| Interest earned | 7.84M (73% of revenue) |
| Annual fees | 2.95M (27% of revenue) |
| Total transactions | 656K |
| Total transaction amount | 44.5M |
| Average customer satisfaction score | 3.19 |

- **Blue cards carry the portfolio.** Blue generates about 9.2M of revenue (85%) and accounts for 91% of customers. Platinum has only 67 customers and about 0.18M of revenue.
- **Mid-career and late-career customers dominate.** The Late (47 to 59) and Mid (34 to 46) age groups bring in roughly 4.95M and 4.80M, about 90% of revenue. Young and Senior customers each contribute under 0.6M.
- **Job and education matter.** Businessmen are the top segment at 3.1M (29%), and graduates lead education levels at 4.4M.
- **Bills are the biggest spending category** at 2.8M, followed by entertainment and fuel. Travel is the lowest at 1.0M.
- **Swipe is the main channel** at about 7.0M (65% of revenue); online is only about 0.66M (6%).
- **Revenue is steady through the year.** Quarterly revenue stays between 2.6M and 2.8M, with Q3 highest and Q4 lowest. Male and female customers contribute almost equally (5.46M vs 5.34M).

## Repository Structure

```
Credit-card-financial-data/
├── README.md
├── data/
│   ├── credit_card.csv
│   └── customer.csv
└── reports/
    ├── credit_card_transaction_report.pdf
    └── credit_card_report_customer.pdf
```

## How to Reproduce

1. Create a MySQL database and two tables matching the column lists above, then import the CSV files from `data/`. Convert `Week_Start_Date` to a `DATE` type during the import.
2. Open Power BI Desktop and connect to MySQL. This requires MySQL Connector/NET to be installed.
3. Load both tables and relate them on `Client_Num`.
4. Recreate the columns and measures from the Calculations section.

## Next Steps

- Delinquency and risk analysis using `Delinquent_Acc`, `Avg_Utilization_Ratio` and `Credit_Limit`.
- Customer acquisition cost against revenue per card category.

## Author

GitHub: [@souvikroy82](https://github.com/souvikroy82)
