# 📊 Power BI E-commerce Sales Analysis for Olist

## 📝 Project Overview

This project transforms raw transactional data from Olist, a major Brazilian e-commerce marketplace, into an actionable business intelligence dashboard. The objective is to move beyond simple data reporting and deliver a high-impact analytical tool that tells a clear story about sales trends, customer behavior, and operational efficiency. By leveraging the full capabilities of Power BI, this analysis provides strategic, data-driven recommendations designed to guide key business decisions and uncover new growth opportunities.

---

## 🛠️ The BI Workflow & Tools

* **Tool:** Microsoft Power BI
* **Workflow:**
    1.  **Data Loading and Transformation:** Connected to the raw CSV data and performed extensive cleaning and shaping in Power Query.
    2.  **Data Modeling:** Designed a robust Star Schema data model to optimize performance and clarity for analysis.
    3.  **DAX Calculations:** Created powerful, reusable measures using Data Analysis Expressions (DAX) to calculate key business metrics.
    4.  **Report Building & Visualization:** Built a two-page interactive dashboard to present the findings in a clear, compelling, and easy-to-understand manner.

---

## 🧹 Data Preparation and Transformation (Power Query)

The initial dataset was a single flat CSV file. To prepare it for analysis, the following steps were taken in the Power Query Editor:

* **Data Type Correction:** Ensured every column was set to the correct data type (e.g., `Price` to Fixed Decimal, `Order Purchase Timestamp` to Date/Time, and ID columns like `Order ID` to Text to prevent unwanted aggregation).
* **Data Cleaning:** Filtered out irrelevant orders (e.g., "canceled" or "unavailable") to focus the analysis on successful sales.
* **Handling Nulls:** Replaced null values in columns like `Product Category Name` with "N/A" to ensure all sales were accounted for in visuals.

---

## ⭐ Data Modeling: The Star Schema

To create an efficient and scalable model, the single flat file was transformed into a **Star Schema**. This separates the descriptive dimension tables from the numerical fact table, which is a best practice in data warehousing and BI.

The final model consists of the following tables:

* **Fact Table:** `Order` (containing all transactional data like prices and IDs)
* **Dimension Tables:**
    * `Product` (Unique list of all products)
    * `Customer` (Unique list of all customers)
    * `Seller` (Unique list of all sellers)
    * `Calendar` (A custom date table for time intelligence)
    * `Date Time` (A custom date/time table for time-based analysis)

This model allows for efficient filtering and slicing of the data across different dimensions.

---

## 💻 Custom Calendars & DAX Measures

### Calendar and Date Time Tables (M Query)

To support robust time-series and time-based analysis, two custom tables were created using M Query in the Power Query Editor.

**`Calendar` Table (for Date-based Analysis):**
```powerquery
let
    // Replace this with the name of your date column
    SourceDates = #"Order"[Order Purchase Date],

    // Detect min and max dates
    StartDate = List.Min(SourceDates),
    EndDate = List.Max(SourceDates),
    
    // Generate list of dates
    DateList = List.Dates(StartDate, Duration.Days(EndDate - StartDate) + 1, #duration(1,0,0,0)),
    CalendarTable = Table.FromList(DateList, Splitter.SplitByNothing(), {"Date"}),

    // Convert to date type
    ChangedType = Table.TransformColumnTypes(CalendarTable, {{"Date", type date}}),

    // Add standard columns
    AddYear = Table.AddColumn(ChangedType, "Year", each Date.Year([Date]), Int64.Type),
    AddMonth = Table.AddColumn(AddYear, "Month", each Date.Month([Date]), Int64.Type),
    AddDay = Table.AddColumn(AddMonth, "Day", each Date.Day([Date]), Int64.Type),
    AddMonthName = Table.AddColumn(AddDay, "Month Name", each Date.ToText([Date], "MMMM"), type text),
    AddMonthNameShort = Table.AddColumn(AddMonthName, "Month Name Short", each Date.ToText([Date], "MMM"), type text),
    AddMonthYear = Table.AddColumn(AddMonthNameShort, "Month Year", each Date.ToText([Date], "MMM yyyy"), type text),
    AddQuarter = Table.AddColumn(AddMonthYear, "Quarter", each "Q" & Text.From(Date.QuarterOfYear([Date])), type text),
    AddWeekday = Table.AddColumn(AddQuarter, "Weekday", each Date.DayOfWeek([Date]), Int64.Type),
    AddWeekdayName = Table.AddColumn(AddWeekday, "Weekday Name", each Date.ToText([Date], "dddd"), type text),
    AddIsWeekend = Table.AddColumn(AddWeekdayName, "Is Weekend", each if Date.DayOfWeek([Date]) >= 5 then true else false, type logical),
    AddMonthYearNumber = Table.AddColumn(AddIsWeekend, "MonthYearNumber", each [Year]*100 + [Month], Int64.Type)
in
    AddMonthYearNumber
```

**`Date Time` Table (for Time-based Analysis):**
```powerquery
let
    // Replace with your datetime column
    SourceDateTime = #"Order"[Order Purchase Timestamp],

    StartDateTime = List.Min(SourceDateTime),
    EndDateTime = List.Max(SourceDateTime),
    TotalMinutes = Duration.TotalMinutes(EndDateTime - StartDateTime) + 1,

    DateTimeList = List.Transform({0..Number.RoundDown(TotalMinutes)}, each StartDateTime + #duration(0,0,_,0)),
    DateTimeTable = Table.FromList(DateTimeList, Splitter.SplitByNothing(), {"DateTime"}),

    ChangedType = Table.TransformColumnTypes(DateTimeTable, {{"DateTime", type datetime}}),

    AddDate = Table.AddColumn(ChangedType, "Date", each DateTime.Date([DateTime]), type date),
    AddTime = Table.AddColumn(AddDate, "Time", each DateTime.Time([DateTime]), type time),
    AddHour = Table.AddColumn(AddTime, "Hour", each Time.Hour([Time]), Int64.Type),
    AddMinute = Table.AddColumn(AddHour, "Minute", each Time.Minute([Time]), Int64.Type),
    AddSecond = Table.AddColumn(AddMinute, "Second", each Time.Second([Time]), Int64.Type),
    AddHourBucket = Table.AddColumn(AddSecond, "Hour Bucket", each Text.PadStart(Text.From(Time.Hour([Time])), 2, "0") & ":00", type text),
    AddDateKey = Table.AddColumn(AddHourBucket, "DateKey", each Date.ToText([Date], "yyyyMMdd"), type text)
in
    AddDateKey
```

### Core DAX Measures

The following core metrics were created to power the visualizations:

**Total Sales:**
```dax
Total Sales = SUM(Order[Price])
```

**Total Orders:**
```dax
Total Orders = DISTINCTCOUNT(Order[Order ID])
```

**Average Order Value:**
```dax
Average Order Value = DIVIDE([Total Sales], [Total Orders])
```

**Average Delivery Time (in Days):**
```dax
Average Delivery Time =
AVERAGEX(
    'Order',
    DATEDIFF('Order'[Order Purchase Timestamp], 'Order'[Order Delivered Customer Date], DAY)
)
```

---

## 📈 Dashboard Insights & Recommendations

The analysis resulted in a two-page interactive dashboard that tells a clear story about the business.

### Page 1: Sales Overview

*(Insert Screenshot of Sales Overview Page here by dragging and dropping the image onto the GitHub text editor)*

* **Insight 1: Strong Growth & Seasonality:** The business experienced rapid growth through 2017, with a massive and predictable sales spike every **November**, likely driven by Black Friday.
* **Insight 2: Business Maturity:** Sales in 2018 established a new, higher baseline compared to previous years, indicating the business has matured.
* **Recommendation:** The November peak is the most critical sales period. **Marketing and inventory planning should be heavily focused on maximizing this opportunity annually.**

### Page 2: Product, Customer & Operational Details

*(Insert Screenshot of Product & Customer Details Page here)*

* **Insight 1: Top Products:** **"beleza_saude" (Health & Beauty)** is a leading product category, representing a significant portion of sales.
* **Insight 2: Geographic Concentration:** Over **60%** of all sales originate from the state of **São Paulo (SP)**, making it the company's core market.
* **Insight 3: Operational Efficiency & Challenges:** While delivery to the core market of São Paulo is highly efficient (averaging ~9 days), remote northern states like Roraima (RR) face extreme delays of nearly **30 days**.
* **Recommendation:** Launch targeted marketing campaigns for top product categories in high-potential growth states like Rio de Janeiro (RJ). Simultaneously, **investigate alternative shipping carriers for northern routes** to improve delivery times and customer satisfaction.

---

## 🏁 Project Conclusion & Limitations

This project successfully transformed raw e-commerce data into an actionable BI dashboard. The key business drivers, trends, and operational challenges have been identified.

A key limitation is the **inability to calculate profit**, as the dataset does not include product cost information. Furthermore, the built-in forecasting tools in Power BI were found to be unreliable for this dataset due to the limited historical data (less than two full years), making it difficult to establish a confident long-term seasonal forecast.
