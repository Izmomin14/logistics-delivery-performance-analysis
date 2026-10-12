# Logistics-Delivery-Performance-Dashboard
Logistics Delivery Performance & Order Failure Analysis Dashboard

An interactive Power BI dashboard that tracks delivery performance across courier partners and the impact of cancellations, RTOs and returns on profitability. Data was cleaned and transformed using Power Query, and the dashboard covers shipper performance, order failures, returns and net margin across 2 pages (2023-2025).

<details> <summary><h2>📌 Problem Statement</h2></summary>

- An e-commerce business ships thousands of orders through multiple courier partners, but has no single view of which shippers deliver on time and which cause delays
- Late deliveries, customer ratings and shipping cost are not analysed together, so it is unclear whether higher shipping cost gives better service
- Cancellations, RTOs and returns cause revenue loss and wasted shipping cost, but their main reasons and most affected product categories are unknown
- Net margin is not tracked by category, so low-margin categories cannot be identified
- Management had no single dashboard to choose shippers, cut failures and improve margin

</details>

<details> <summary><h2>🗂️ Data & Tables</h2></summary>

The dataset consists of 5 related tables plus a Date table:

**Customers** Cust_ID, customer name, city, state

**Suppliers** Supplier_ID, supplier name

**Products** Product_ID, Supplier_ID, Product_Name, Category, Unit_Price

**Logistics** Sales_ID, Cust_ID, Product_ID, Order_Date, Expected_Delivery_Date, Delivery_Date, Shipper, Quantity, Shipping_Cost, Order_Status, Customer_Rating

**Returns** Sales_ID, Return_Reason

**Date** Calendar table linked to Order_Date

Record counts: Customers 250 | Products 100 | Shippers 8 | Orders ~5,000 | Returns ~300 | Period 36 months (2023-2025)

</details>

<details> <summary><h2>🔗 Data Model (Relationships)</h2></summary>

- Logistics[Product_ID] → Products[Product_ID]
- Logistics[Cust_ID] → Customers[Cust_ID]
- Returns[Sales_ID] → Logistics[Sales_ID]
- Products[Supplier_ID] → Suppliers[Supplier_ID]
- Date[Date] → Logistics[Order_Date]

</details>

<details> <summary><h2>🧹 Data Cleaning (Power Query)</h2></summary>

- Removed duplicate and blank records across all tables
- Standardized column names and data types (dates, currency, text)
- Fixed inconsistent shipper names and casing (e.g. Blue Dart, DTDC, Xpressbees)
- Handled blank Delivery_Date for undelivered orders so they are not counted as on time
- Built relationships between tables (Customers → Logistics → Returns, Products, Suppliers, Date)
- Created DAX measures for KPIs (On-Time Delivery %, Late Delivery %, Return Rate %, Cancelled + RTO %, Net Margin)

</details>

<details> <summary><h2>🗺️ Dashboard Planning</h2></summary>

Before building, a dashboard map was designed to decide what management needed to see, mapping each problem statement point to a KPI or chart. This ensured every visual directly answers a business question instead of being added arbitrarily.

Page 1 has 4 KPIs + 1 slicer + 3 charts. Page 2 has 3 KPIs + 1 slicer + 4 charts. Both pages share the same theme, card style and layout.

</details>

<details> <summary><h2>🚚 Page 1: Delivery & Shipper Performance</h2></summary>

**KPIs:** Total Orders 📦 · OTD % 🚚 · Late Del. % ⏱️ · Avg Rating ⭐

**Slicer:** Year (2023 / 2024 / 2025)

**Charts:**

1. On-Time Delivery % by Shipper
2. Monthly Orders & Late Delivery % Trend
3. Avg Shipping Cost by Shipper

**Key Findings:**

- 5K orders delivered with an overall on-time delivery rate of 73.4% and an average customer rating of 3.8
- Blue Dart and Gati have the highest on-time delivery (~81%), but also the highest average shipping cost (~₹235-245)
- Speed Post is the cheapest (~₹107) but has the lowest on-time delivery (~56%)
- Delhivery, DTDC and Shadowfax sit in the middle, balancing cost and punctuality
- Late delivery % moves up and down month to month, with visible spikes in some months, so it needs regular tracking

</details>

<details> <summary><h2>⚠️ Page 2: Order Failures, Returns & Profit Analysis</h2></summary>

**KPIs:** Net Margin 💰 · Return Rate 🔄 · Cancelled + RTO ❌

**Slicer:** Shipper Filter

**Charts:**

1. Orders by Status (Delivered / Cancelled / RTO)
2. Category-wise Net Margin
3. Category-wise Return Count
4. Return Reasons Breakdown

**Key Findings:**

- Net Margin of ₹9.26M, a Return Rate of 6.00% and Cancelled + RTO of 7.82%
- 92.18% of orders are delivered, 4.28% are cancelled and the rest are RTO
- Home & Kitchen and Fashion earn the highest net margin (~₹2.8-2.9M each), followed by Electronics (~₹2.0M)
- Sports & Fitness and Grocery have the lowest net margin (~₹0.2M each)
- Damaged, Size / fit issue and Not as described are the top return reasons, followed by Defective product and Wrong item received
- Beauty & Personal Care has the highest return count among categories

</details>

<details> <summary><h2>🧮 Key DAX Measures</h2></summary>

```dax
Total Orders = DISTINCTCOUNT(Logistics[Sales_ID])

On Time Delivery Count =
CALCULATE(
    [Total Orders],
    FILTER(
        Logistics,
        NOT ISBLANK(Logistics[Delivery_Date]) &&
        Logistics[Delivery_Date] <= Logistics[Expected_Delivery_Date]
    )
)

% on Time Delivery = DIVIDE([On Time Delivery Count], [Total Orders])

Late Delivery % =
DIVIDE([Total Orders] - [On Time Delivery Count], [Total Orders])

Return Rate % =
DIVIDE(DISTINCTCOUNT(Returns[Sales_ID]), DISTINCTCOUNT(Logistics[Sales_ID]))

Cancelled + RTO % =
DIVIDE(
    CALCULATE(COUNTROWS(Logistics), Logistics[Order_Status] IN {"Cancelled", "RTO"}),
    COUNTROWS(Logistics)
)

Total Net Margin =
VAR Revenue =
    SUMX(
        FILTER(Logistics, Logistics[Order_Status] = "Delivered"),
        Logistics[Quantity] * RELATED(Products[Unit_Price])
    )
VAR ShippingCost =
    CALCULATE(SUM(Logistics[Shipping_Cost]), Logistics[Order_Status] = "Delivered")
RETURN
    Revenue - ShippingCost
```

</details>

<details> <summary><h2>💡 Suggestions</h2></summary>

- Use Blue Dart and Gati (highest on-time, highest cost) only for high-value or urgent orders
- Reduce Speed Post (cheapest, lowest on-time %) or add SLA penalties for late delivery
- Shift volume to mid-cost shippers like Delhivery and DTDC for a better cost vs punctuality balance
- Cut returns (Damaged, Size / fit, Not as described) with better packaging, size charts and accurate product details
- Run quality checks and supplier reviews first in the categories with the most returns
- Reduce Cancellations and RTOs with address verification and order confirmation before dispatch, especially for COD
- Review pricing and shipping cost for Sports & Fitness and Grocery (lowest margin), and promote high-margin categories

</details>

<details> <summary><h2>🛠️ Tools Used</h2></summary>

- Power BI: data modeling, DAX measures, dashboard design
- Power Query: data cleaning and transformation
- SQL / Python: data preparation

</details>

<details> <summary><h2>✅ Outcome</h2></summary>

The dashboard gives management a single, unified view of delivery performance and order failures, replacing scattered data across multiple sources. Operations can now compare shippers on punctuality, cost and rating, the business can see where revenue is lost to cancellations, RTOs and returns, and leadership can choose shippers and focus on high-margin categories using data.

</details>

<details> <summary><h2>👤 About Me</h2></summary>

**Momin Izhar Nisar**
Data Analyst (Fresher) | Power BI · SQL · Python · Excel

- 📞 Phone: 9834011201
- 📧 Email: mominizhar45@gmail.com
- 💻 GitHub: [github.com/Izmomin14](https://github.com/Izmomin14)

</details>
