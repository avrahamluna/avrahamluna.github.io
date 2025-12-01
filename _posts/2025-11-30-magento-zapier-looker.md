---
title: "Magento → Zapier → Google Sheets → Looker Studio"
date: 2025-11-30
categories: [portfolio, automation]
tags: [automation, magento, zapier, google sheets, looker studio]
status: "Completed"
# icon: "/assets/icons/automation.svg"
# icon: "/assets/icons/automation.svg"
pin: true
image:
  path: /assets/img/magento-pipeline.png
  alt: "Data pipeline from Magento through Zapier into Google Sheets and Looker Studio"
---

## 1. Overview (TL;DR)

**Name:** BlueSteps Daily Revenue Dashboard  

**Goal:**  
Give leadership a near real time view of revenue, memberships, renewals, and ECS performance by product, day, and month, without manual exports from Magento.

**Users:**  
CEO, finance, and marketing.

**Core services:**  
Magento, Zapier, Google Sheets, Looker Studio.

**Key outputs (examples from the dashboard):**

- Daily Revenue, Daily Memberships, Daily Renewals, Daily ECS  
- MTD and YTD Revenue, Memberships, Renewals, ECS  
- New Registrants MTD and New Memberships MTD  
- Orders by Product time series and Month To Date Total line chart  

---

## 2. Architecture (Pro view)

High level architecture:
flowchart LR
    classDef section fill:#ffffff,stroke:#999,stroke-width:1px,color:#111,font-weight:bold;

    classDef source fill:#ffe6cc,stroke:#d79b00,stroke-width:1px,color:#000;
    classDef etl fill:#e1d5e7,stroke:#9673a6,stroke-width:1px,color:#000;
    classDef storage fill:#d5e8d4,stroke:#82b366,stroke-width:1px,color:#000;
    classDef bi fill:#dae8fc,stroke:#6c8ebf,stroke-width:1px,color:#000;
    classDef users fill:#fff2cc,stroke:#d6b656,stroke-width:1px,color:#000;

    subgraph S1["Architecture Diagram (Pro View)"]
    direction LR

        subgraph srcLayer["Source Layer"]
        direction LR
        M[Magento\nOrders • Memberships • Renewals • ECS]:::source
        end

        subgraph etlLayer["ETL Layer (Zapier)"]
        direction LR
        Z1[Trigger\nNew Order in Magento]:::etl
        Z2[Transform\nParse JSON → Map Fields]:::etl
        Z3[Load\nAppend Row to Google Sheets]:::etl
        Z1 --> Z2 --> Z3
        end

        subgraph storageLayer["Staging Storage Layer"]
        direction LR
        G[(Google Sheets\nFactOrders Table)]:::storage
        end

        subgraph biLayer["BI & Visualization Layer"]
        direction LR
        L[Looker Studio\nRevenue • Membership • ECS KPIs]:::bi
        end

        subgraph userLayer["Consumer Layer"]
        direction LR
        U[Executive Users\nCEO • Marketing • Finance]:::users
        end

    end

    M -->|REST API\nJSON Payload| Z1
    Z3 -->|Append Rows| G
    G -->|Native Connector\nScheduled Refresh| L
    L -->|Dashboards & KPIs| U
```


- **Magento (source)**  
  - Exposes orders, memberships, renewals and ECS data through REST API.

- **Zapier (integration and staging)**  
  - Trigger: new order or updated subscription in Magento.  
  - Actions: formatter or code step to parse JSON.  
  - Action: Google Sheets to append rows.

- **Google Sheets (staging table)**  
  - Stores normalized, column based transaction data.  
  - Acts as a lightweight warehouse for Looker Studio.

- **Looker Studio (semantic and visual layer)**  
  - Connects to Google Sheets through the native connector.  
  - Uses calculated fields for MTD, YTD, product splits and prior period comparison.

Text diagram:

`Magento API → Zapier → Google Sheets → Looker Studio → Executives dashboard`

## 8. Variants and extensions

- Add CRM data by joining another sheet with campaign or lead source.
- Move the warehouse to BigQuery and point Looker Studio to BigQuery for scalable history.
- Add cohort and retention views for active members by start month and churn.
- Add alerting with Zapier or Apps Script when daily revenue drops below a threshold.

---

## 9. Design journal

**Problem**  
Leadership wanted daily visibility into revenue and membership performance instead of waiting for month-end reports.

**Constraints**  
No dedicated data warehouse or engineering team, limited budget, non-technical stakeholders.

**Choice**  
Build a lightweight pipeline using Magento, Zapier, Google Sheets, and Looker Studio.

**Results**

- Automated ingestion removed manual exports from Magento.
- Executives can see Daily, MTD, and YTD metrics in a single dashboard.

---

## 10. tl;dr

- Magento records the sale.  
- Zapier listens for new sales and copies them into a Google Sheet.  
- The Google Sheet behaves like a simple database.  
- Looker Studio reads the sheet and turns the numbers into charts and KPIs.  
- Executives open a link and see the latest revenue and membership numbers.


---

## 3. Data flow (step by step)

**Event in Magento**

- A new order, membership, or renewal is created or updated.

**Zapier trigger**

- “Magento New Order” trigger pulls the order as JSON.  
- Authentication uses a Magento API token.

**Transformation in Zapier**

- Formatter or Code by Zapier parses the JSON payload.  
- Extracted fields (examples): `order_id`, `product_name`, `product_type`, `amount`, `currency`, `order_status`, `created_datetime`, `customer_id`.  
- Optional mapping to classify rows as membership, ECS, or other products.

**Load into Google Sheets**

- Zapier appends a new row to a `Daily_Orders` sheet.  
- Each row represents one order line.  
- Columns include:
  - `date`, `order_id`, `product`, `product_group`, `amount`, `channel`, `is_membership`, `is_renewal`, `is_ecs`.

**Looker Studio connection**

- Native Google Sheets connector points to the staging sheet.  
- Refresh frequency set to every 15 minutes or hourly.

**Looker Studio metrics and visuals**

Example calculated fields:

- `Daily Revenue = SUM(amount)`  
- `MTD Revenue = SUM(amount) for dates in current month`  
- `YTD Revenue = SUM(amount) for dates in current year`  
- `Is Membership` flag based on product group.

Example visuals:

- Scorecards for Daily Revenue, Memberships, Renewals, ECS.  
- Scorecards for MTD and YTD metrics.  
- Gauges for New Registrants MTD and New Memberships MTD.  
- Stacked area chart “Orders by Product” over time.  
- Line chart “Month To Date Total” revenue.

---

## 4. Data model highlights

**Fact table: Orders (in Google Sheets)**

- `order_id`  
- `order_date`  
- `product_name`  
- `product_group` (Membership, ECS, Customized Services, etc.)  
- `revenue_amount`  
- `membership_flag`, `renewal_flag`, `ecs_flag`  
- `customer_id`  
- `source_channel`

**Dimensions (implicit):**

- Product dimension from `product_name` and `product_group`.  
- Date dimension from `order_date`, with derived fields for day, month, year.

---

## 5. Design decisions and trade offs

**Compute and storage**

- Chosen: Google Sheets as staging.  
  - Pros: very low cost, simple, native with Looker Studio.  
  - Cons: row limits, not ideal for very high volume.

- Alternative: BigQuery.  
  - Pros: handles larger history, SQL, partitioning.  
  - Cons: more setup and cost.

**Integration**

- Chosen: Zapier.  
  - Pros: no code, quick to build, built in connectors for Magento and Sheets.  
  - Cons: pricing per task, less control than custom ETL.

- Alternative: Apps Script or Python job on a schedule.

**BI layer**

- Chosen: Looker Studio.  
  - Pros: native to Sheets, easy sharing, fast to build dashboards.  
  - Cons: fewer advanced semantic modeling features compared to full BI tools.

---

## 6. Automations and scheduling

- Zapier listens to Magento events and appends rows in real time.  
- Looker Studio is scheduled to refresh so stakeholders always see fresh data with no manual exports.

---

## 7. Monitoring and data quality

Simple checks:

- Row count trend in Sheets to spot if Zapier stopped.  
- “Last order imported” date card on the dashboard: `MAX(order_date)` for non zero amounts.  
- Conditional formatting in Sheets to flag negative or zero revenue.

Example Sheets formula:

```text
=MAX(FILTER(order_date, revenue_amount > 0))
