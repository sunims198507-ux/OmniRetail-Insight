OmniRetail-Insight: Enterprise Retail BI & Generative AI Automation PipelineAn end-to-end, serverless BI and Intelligent Process Automation (IPA) architecture designed for a multi-million dollar retail chain. This pipeline automatically transforms unstructured transactional retail data, models a structured star schema, computes key commercial growth metrics, and leverages GenAI to distribute weekly performance briefings directly to executive leadership.📂 Repository StructureOmniRetail-Insight/
├── README.md                          <-- You are here (Project Overview & Docs)
├── data-schema/
│   └── star_schema_blueprint.txt      <-- Text representation of DB Relationships
├── power-query/
│   └── etl_transformation_log.md      <-- Step-by-step ETL design patterns (M-code equivalents)
├── dax-measures/
│   ├── total_revenue.dax              <-- Calculated Revenue formula
│   ├── total_orders.dax               <-- Unique transactions count formula
│   └── mom_growth_percentage.dax      <-- Month-over-Month time-intelligence formula
└── power-automate/
    └── cloud_flow_definition.json     <-- JSON export schema for the Power Automate pipeline
📈 1. Project Background & Business ValueThe ChallengeModern retail executives need near real-time operational insights to respond to supply and demand fluctuations. However, traditional reporting structures are plagued by:Delayed Cycles: Static reports compiled manually at month-end.Context Gap: Raw charts and dashboards without qualitative interpretations or strategic recommendations.Data Quality Issues: Messy, duplicated, or unformatted input streams from disparate Point-of-Sale (POS) systems.The SolutionOmniRetail-Insight implements a zero-maintenance cloud architecture:Ingestion & ETL: Power Query cleans and parses incoming sales data.Data Modeling: Power BI hosts a high-performance Star Schema data model.Analytics Engine: Custom DAX measures process complex calculations (AOV, MoM Growth %, and Net Revenue).AI & Automation Pipeline: A scheduled Power Automate Cloud Flow triggers dataset refreshes, queries the model, feeds the parameters to a Gemini / GPT engine, and emails an executive summary to the Vice President of Retail Operations.🗄️ 2. The Data Schema (Star Schema Design)To optimize query speeds, minimize table scanning times, and simplify DAX calculation logic, the project models a standardized Star Schema.                       +------------------------+
                       |      Dim_Products      |
                       +------------------------+
                       | * Product_Key (PK)     |
                       | - Product_ID           |
                       | - Product_Name         |
                       | - Category             |
                       +-----------+------------+
                                   | 1
                                   |
                                   | *
+------------------------+ * +----+----+  * +------------------------+
|      Dim_Customers     +----+Fact_Sales+---+        Dim_Store       |
+------------------------+    +----+----+    +------------------------+
| * Customer_Key (PK)    |         | * | * Store_Key (PK)       |
| - Customer_ID          |         |         | - Store_ID             |
| - Customer_Name        |         |         | - Store_Name           |
| - Segment              |         | 1       | - City                 |
+------------------------+    +----+----+    +------------------------+
                              | Dim_Date|
                              +---------+
                              |*Date_Key|
                              | - Date  |
                              +---------+
Fact_Sales: Contains transaction IDs, foreign keys to all dimension tables, and transactional metrics (Quantity, Unit_Price, Discount_Percentage, and derived Actual_Revenue).Dim_Products / Dim_Customers / Dim_Store / Dim_Date: Descriptive context tables configured with a strict $1 \rightarrow *$ relationship with the Fact table.🧹 3. Visual ETL Process (Power Query)No complex SQL queries were used; all transformations were designed visually using Power Query Editor to maximize replicability and auditability.Null/Blank Optimization: * Targeted Discount_Percentage. Replaced null cells with 0 so that multiplication operators would not fail.Stripped outer spaces from Product_Name using the Trim transform tool to ensure perfect lookup integrity during relationships.Geographic Dimension Merging:Combined separate columns City and State in Dim_Store using a Comma delimiter to construct a unified geographical column Store_Location_Full (e.g., "Bangalore, Karnataka").Data Type Casting:Explicitly cast currency fields (Unit_Price) to Fixed Decimal Number ($1.2$) for precision safety.Standardized transactional quantities (Quantity) to Whole Numbers to protect calculations against systemic float errors.🧮 4. Analytical Formula Engine (DAX Calculations)The analytical business logic is calculated dynamically using context-aware DAX expressions.A. Total RevenueComputes net sales revenue after accounting for transactional quantities, unit prices, and promotional customer discounts.$$Total\ Revenue = \sum (Quantity \times Unit\ Price \times (1 - Discount\%))$$Total Revenue = 
SUMX(
    Fact_Sales,
    Fact_Sales[Quantity] * Fact_Sales[Unit_Price] * (1 - Fact_Sales[Discount_Percentage])
)
B. Total OrdersCalculates unique, distinct purchase transactions occurring throughout the network.$$Total\ Orders = Unique\ Count\ of\ Order\ IDs$$Total Orders = 
DISTINCTCOUNT(Fact_Sales[Order_ID])
C. Month-over-Month (MoM) Growth %Calculates the relative percentage growth in revenue compared to the prior calendar month.$$MoM\ Growth\% = \frac{Current\ Month\ Revenue - Prior\ Month\ Revenue}{Prior\ Month\ Revenue} \times 100$$Month-over-Month Growth % = 
VAR CurrentMonthRevenue = [Total Revenue]
VAR PreviousMonthRevenue = 
    CALCULATE(
        [Total Revenue],
        DATEADD(Dim_Date[Calendar_Date], -1, MONTH)
    )
RETURN
    DIVIDE(
        CurrentMonthRevenue - PreviousMonthRevenue,
        PreviousMonthRevenue,
        0
    )
⚙️ 5. Automated AI Insights Pipeline (Power Automate)This cloud-native flow connects your enterprise data model to executive-level output.+--------------------------------------------------------+
| 1. Recurrence Trigger (Weekly Monday @ 8:00 AM IST)     |
+--------------------------------------------------------+
                           |
                           v
+--------------------------------------------------------+
| 2. Action: Trigger Power BI Dataset Refresh            |
+--------------------------------------------------------+
                           |
                           v
+--------------------------------------------------------+
| 3. Action: Query Power BI Dataset using DAX           |
+--------------------------------------------------------+
                           |
                           v
+--------------------------------------------------------+
| 4. Action: HTTP POST to Gemini API (JSON Payload)      |
+--------------------------------------------------------+
                           |
                           v
+--------------------------------------------------------+
| 5. Action: Send Outlook Email V2 to Vice President      |
+--------------------------------------------------------+
API Prompt Payload BlueprintThe following payload is structured and transmitted dynamically within the Power Automate HTTP action card:{
  "contents": [{
    "parts": [{
      "text": "Act as an elite Retail Operations Advisor. Read the following metric payload: Month: May 2026, Total Revenue: INR 42.5 Million, Total Orders: 184,200, MoM Revenue Growth: +14.2%. Generate a concise, high-impact 3-bullet point email briefing for the Vice President of Retail Operations. The summary must state the key wins, areas of concern, and immediate strategic recommendations. Adopt an ultra-professional, action-oriented, corporate tone."
    }]
  }]
}
📬 6. Simulated Executive DeliverableThe exact automated email delivered by the pipeline to leadership:Subject: BRIEFING: May 2026 Revenue Performance & Executive Action Items

Dear VP of Retail Operations,

Please find the automated monthly sales analysis for May 2026 below.

* **Strong Top-Line Growth Momentum:** Monthly revenue surged to INR 42.5 Million, reflecting a robust Month-over-Month (MoM) expansion of +14.2%. This performance was driven by an active volume of 184,200 total orders, signaling strong transactional traction and highly effective seasonal promotional programs.
* **Average Order Value (AOV) Deflection:** Despite impressive revenue growth, analysis reveals a slight -1.8% contraction in Average Order Value (AOV) compared to last quarter. This indicates that while customer conversion and overall footfall are high, customers are purchasing smaller baskets, highlighting a critical need to re-engage bundle pricing strategies.
* **Immediate Strategic Action Recommendations:** To sustain current momentum, it is recommended to launch multi-unit discount bundles in regional Hubs (specifically Bangalore and Hyderabad) starting next week. Additionally, we must audit high-discount transactions to ensure gross margin integrity is not being overly diluted for volume gains.

Sincerely,
OmniRetail-Insight Automation Engine
[Automated via Power Platform & AI Integrations]
🚀 How to Replicate this Project (No-Cost Developer Guide)You can build, test, and demonstrate this entire enterprise architecture using free, sandbox accounts:Step 1: Set Up Your Sandbox EnvironmentPower BI Desktop (Free): Download Power BI Desktop from the Microsoft Store. It is 100% free for building data models and designing visual dashboards locally.Power Platform Developer Plan (Free): Sign up for the Microsoft Power Apps Developer Plan. This gives you a free, fully functional development environment with premium features—including Power Automate cloud flows and custom connector capabilities.Step 2: Build the Data Model & DAXOpen Power BI Desktop, import your retail sales CSVs, and navigate to Power Query to implement the steps detailed in Section 3.Set up the schema relationships as represented in Section 2.Create a new measure table _Measures and paste the DAX codes from Section 4.Step 3: Configure the Automation FlowLog into your Power Platform environment via make.powerapps.com.Navigate to Solutions -> New Solution -> Cloud Flow -> Instant/Scheduled Cloud Flow.Replicate the sequential steps described in Section 5. You can use a free Gemini API Key or an OpenAI Developer Account to handle the GenAI generation step.🛠️ Skills DemonstratedDatabase & Data Modeling: Star Schema Design, Relationship Cardinality ($1 \rightarrow *$), Referential Integrity.Modern ETL Pipelines: Data Cleansing, Schema Mapping, Structural Normalization.Analytical Computations (DAX): Time Intelligence calculations (DATEADD), Contextual Filtering (CALCULATE), Row-by-Row Iteration (SUMX).Intelligent Process Automation (IPA): API Integrations, JSON payload parsing, Multi-system automated orchestration.Generative AI Engineering: Advanced System Prompt Engineering, API integrations, and Contextual Business Reporting.
