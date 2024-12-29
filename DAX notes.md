# Introduction to DAX (Data Analysis Expressions)

## What is DAX?

DAX is a formula expression language used in Power BI, Power Pivot for Excel, and SQL Server Analysis Services. It's designed specifically for working with relational data and performing calculations in business intelligence solutions.

Key characteristics of DAX:
- Similar to Excel formulas but more powerful
- Designed for working with tables and relationships
- Enables complex calculations across multiple tables
- Context-aware (row context and filter context)

## Calculated Columns and Tables

### Calculated Columns

Calculated columns are computed columns that become part of your table. They:
- Are computed when data is refreshed
- Store results for each row
- Use row context (access to values in the current row)

Examples:

```dax
// Basic calculated column
Full Name = Sales[First Name] & " " & Sales[Last Name]

// Conditional calculated column
Product Category = 
    IF(
        Products[Price] >= 100,
        "Premium",
        IF(
            Products[Price] >= 50,
            "Standard",
            "Budget"
        )
    )

// Date-based calculation
Days Since Order = 
    DATEDIFF(
        Sales[OrderDate],
        TODAY(),
        DAY
    )
```

### Calculated Tables

Calculated tables are entire tables created using DAX expressions. They:
- Are recomputed on data refresh
- Can combine data from multiple tables
- Are useful for creating reference tables or specialized views

Examples:

```dax
// Creating a date dimension table
DateDimension = 
    CALENDAR(
        DATE(2020, 1, 1),
        DATE(2024, 12, 31)
    )

// Creating a filtered copy of a table
PremiumProducts = 
    FILTER(
        Products,
        Products[Price] >= 100
    )

// Combining data from multiple tables
SalesAnalysis = 
    SUMMARIZE(
        Sales,
        Sales[ProductID],
        "Total Sales", SUM(Sales[Amount]),
        "Order Count", COUNT(Sales[OrderID])
    )
```

## Measures

Measures are dynamic calculations that:
- Are computed at query time
- Respond to user interactions and filters
- Use filter context
- Are essential for creating interactive reports

### Basic Measure Concepts

1. **Filter Context**: The current set of filters applied to your data
2. **Row Context**: The current row being processed
3. **Context Transition**: Converting row context to filter context

### Key Measure Functions and Examples

#### 1. Aggregation Functions

```dax
// Basic aggregations
Total Sales = SUM(Sales[Amount])

Average Price = AVERAGE(Products[Price])

Product Count = DISTINCTCOUNT(Sales[ProductID])

// Running total
Running Total = 
    CALCULATE(
        SUM(Sales[Amount]),
        FILTER(
            ALL(Sales),
            Sales[Date] <= MAX(Sales[Date])
        )
    )
```

#### 2. Time Intelligence

```dax
// Year-to-date sales
YTD Sales = 
    CALCULATE(
        SUM(Sales[Amount]),
        DATESYTD(DimDate[Date])
    )

// Previous year comparison
PY Sales = 
    CALCULATE(
        SUM(Sales[Amount]),
        SAMEPERIODLASTYEAR(DimDate[Date])
    )

// Month-over-month growth
MoM Growth = 
    VAR CurrentSales = [Total Sales]
    VAR PreviousSales = 
        CALCULATE(
            [Total Sales],
            DATEADD(DimDate[Date], -1, MONTH)
        )
    RETURN
        DIVIDE(
            CurrentSales - PreviousSales,
            PreviousSales,
            BLANK()
        )
```

#### 3. Conditional Logic

```dax
// IF statement in measures
Profit Status = 
    IF(
        [Total Profit] >= 0,
        "Profitable",
        "Loss"
    )

// SWITCH statement for multiple conditions
Price Band = 
    SWITCH(
        TRUE(),
        [Average Price] >= 100, "Premium",
        [Average Price] >= 50, "Standard",
        "Budget"
    )

// Complex condition with CALCULATE
High Value Customers = 
    CALCULATE(
        DISTINCTCOUNT(Sales[CustomerID]),
        FILTER(
            Sales,
            Sales[Amount] >= 1000
        )
    )
```

#### 4. CALCULATE Function Deep Dive

CALCULATE is one of the most powerful DAX functions. It:
- Modifies the filter context
- Can be used with multiple filter conditions
- Essential for complex calculations

Examples:

```dax
// Basic CALCULATE
Total Sales Premium Products = 
    CALCULATE(
        SUM(Sales[Amount]),
        Products[Category] = "Premium"
    )

// Multiple filters
Sales By Region and Category = 
    CALCULATE(
        [Total Sales],
        Region[Name] = "North",
        Products[Category] = "Premium"
    )

// Using ALL to remove filters
Total Market Share = 
    DIVIDE(
        [Total Sales],
        CALCULATE(
            [Total Sales],
            ALL(Products)
        )
    )
```

## Best Practices

1. Naming Conventions:
   - Use clear, descriptive names
   - Prefix measures with relevant categories
   - Use PascalCase for measure names

2. Performance Optimization:
   - Avoid nested CALCULATE when possible
   - Use variables for complex calculations
   - Consider materialization of frequently used calculations

3. Documentation:
   - Comment complex measures
   - Document business rules and assumptions
   - Maintain a measure dictionary

## Common Pitfalls to Avoid

1. Ignoring filter context
2. Misusing row context
3. Unnecessary complexity in calculations
4. Not using variables for repeated expressions
5. Incorrect use of BLANK() vs. 0

