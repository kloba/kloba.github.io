---
layout: post
title: "From Text to DAX: Power BI's AI-Powered Quick Measure Suggestions"

---

****


In today\'s data-driven world, business intelligence (BI) tools have
become essential for organizations to make informed decisions. Among
these tools, Microsoft\'s Power BI stands out as a leader, consistently
recognized by industry analysts, such as Forrester and Gartner, for its
robust capabilities and ease of
use [\[1\]](Schlegel,%20K.,%20Sun,%20J.,%20&%208%20more.%20(2023,%20April%205).%20Magic%20Quadrant%20for%20Analytics%20and%20Business%20Intelligence%20Platforms.%20Gartner.%20Retrieved%20%5bDate%5d,%20from%20https:/www.gartner.com/doc/reprints?id=1-2955ETOT&ct=220215&st=sb?ocid=lp_pg398450_gdc_comm_az)[\[2\]](#Forrester).

Power BI is a suite of business analytics tools that allows you to
analyze data and share insights. It provides interactive visualizations
with self-service business intelligence capabilities, making it possible
for end users to create reports and dashboards by themselves without
having to depend on information technology staff or Database
Administrators.

<p align="center">
  <img src="/imgs/text-do-dax/image1.png" />
</p>

One of the key features of Power BI is its ability to create Data
Analysis Expressions (DAX). DAX is a collection of functions, operators,
and constants that can be used in a formula or expression to calculate
and return one or more values. DAX measures play a crucial role in the
exploration of data and the creation of valuable insights.

However, creating DAX measures can be complex and time-consuming,
especially for users unfamiliar with the DAX language. This is where
Power BI\'s latest feature, Quick Measure Suggestions, comes into play.
Powered by AI, this feature assists in creating DAX measures using
natural language, making it easier and more intuitive than ever
before [\[3\]](#Microsoft).

In this blog post, we will delve into the details of this innovative
feature, explore how to enable and use it, and discuss its benefits and
considerations. Whether you\'re a seasoned Power BI user or a beginner,
Quick Measure Suggestions is a feature that can enhance your data
analysis and reporting capabilities.

**2. Enabling Quick Measure Suggestions**

This feature supports a variety of common DAX measures scenarios,
including aggregated columns, count of rows, mathematical operations,
and many more. It\'s like having a DAX expert right at your fingertips,
ready to help you jump-start your measure creation process.

To enable the **Quick Measure Suggestions** feature:

1.  Open **Power BI Desktop**.

2.  Open the **Options** menu, and then select the **Preview features**
    tab.

3.  Select the **Quick measure suggestions** checkbox to enable the
    feature.

<p align="center">
  <img src="/imgs/text-do-dax/image3.png" />
</p>    

Once you\'ve enabled Quick Measure Suggestions, you can start using it
to create DAX measures.

To create DAX measures:

1.  Launch **Quick Measure** from the **Home** or **Modeling** tab of
    the ribbon.

2.  In the **Quick Measure** window, select **Suggestions**.

<p align="center">
  <img src="/imgs/text-do-dax/image4.png" />
</p>    

1.  In the **Suggestions** window, describe the measure you want to
    create. For example, show me the sum of sales.

2.  Select the **Generate** button to get a DAX measure suggestion.

<p align="center">
  <img src="/imgs/text-do-dax/image5.png" />
</p>    

Always validate the DAX suggestions to ensure they meet your needs. If
you're satisfied with a suggested measure, you can select the **Add**
button to automatically add the measure to your model.

**3. Use Cases and Examples**

Quick Measure Suggestions can be used in various scenarios to simplify
the creation of DAX measures. Here are some detailed examples of
different scenarios where this feature can be used:

**Aggregated Columns**:

-   Show me a sum of sales

-   Get total sales

-   Count products

-   How many products are there?

-   Unique users

-   Distinct count of users, no blanks

-   Get the number of unique users and exclude blanks

-   What is the max price?

-   Median age

**Optional Filters**:

-   How many customers are in London?

-   Total sold units in 2022

-   Calculate sales where Product is Word and Region is North

-   Sales where Product is Word or Region is North

-   Sales filtered to Product is Word && Region is North

-   Sales for Product is Word \|\| Region is North

**Count of Rows**:

-   Count records of the Sales table

-   Count the Sales table

-   Sales table row count

-   Count rows of the Sales table

**Aggregate per Category**:

-   Average sales per store

-   Average score per category weighted by priority

-   Min score per product

-   Max units per store

**Mathematical Operations**:

-   Sales - Cogs

-   Sales minus Cogs

-   Sales divided by target revenue times 100

-   Sales/target revenue \* 100

-   EU Sales + JP Sales + NA Sales

-   For each row in the Sales table, calculate Price \* Units and sum up
    the result

-   For each row in the Sales table, sum up Price \* Units

-   For each row in the Sales table, calculate Price \* Discount and
    then get the average

-   For the Sales table, get the average of Price \* Discount

**Selected Value**:

-   What is the selected product?

-   Which product is selected?

-   Selected value for a product

**If Condition**:

-   If sales \> 10,000, return \"high sales\", else \"low sales\"

-   If sales are greater than 10,000, display \"high sales\" otherwise,
    display \"low sales\"

-   If a selected value for a product is blank, display \"no product
    selected\", else show the selected product

-   If selected product = Power BI, show \"PBI\", else \"other\"

**Text Operations**:

-   The selected product is \" & selected product\"

-   Display \"The selected product is \" concatenated with the selected
    product

-   Header_measure & \" - \" & Subheader_measure

-   For each row in the Geography Dim table, concatenate State & \", \"
    & City and combine the result

-   For each row in the Geography Dim table, get State & \", \" & City
    and merge

**Time Intelligence**:

-   YTD sales

-   Sales fiscal YTD

-   Get the sales year to date

-   Sales MTD

-   Quarter-to-date sales

-   YTD sales for US and Canada

-   Change of sales from the previous year

-   Sales YoY change

-   Month-over-month change for sales

-   Sales QoQ Percent change

-   Sales for the same period last year

-   Sales for the same period last month

-   28 day rolling average sales

-   28--day rolling avg sales

**Relative Time Filtered Value**:

-   Unique users in the last 4 hours

-   Unique users in the last 5 days

-   Total sales for the last 6 months

-   Total sales for the last 2 years

**Most/Least Common Value**:

-   Most common value in Product

-   Which value in the Product is most common

-   What is the most common value in Product

-   Which value in the Product is least common

-   What is the least common value in the Product

**Top N Filtered Value**:

-   Total sales for the top 3 products

-   Sum of sales filtered to the top 3 products

-   Average score for the top 5 students

-   Avg score filtered to the top 5 students

**Top N Values for a Category**:

-   Top 3 products with the most total sales

-   Top 3 products by sales

-   What are the top 3 products in sales

**Information Functions**:

-   Today\'s date

-   Now

-   Return the current user email

-   Return the current domain name and username

-   Return the current user's domain login

These examples demonstrate the versatility of Quick Measure Suggestions.
Whether you\'re a seasoned Power BI user or a beginner, this feature can
enhance your data analysis and reporting capabilities.

**4. Conclusion**

In the realm of data analysis, the introduction of Quick Measure
Suggestions in Power BI represents more than just a new feature. It
signifies a shift in our approach to data exploration, a step towards a
future where technology augments our natural abilities and empowers us
to focus on what truly matters.

Quick Measure Suggestions, with its AI-powered ability to transform
natural language into DAX measures, is a testament to this shift. It
simplifies the complex, democratizes the inaccessible, and in doing so,
allows us to spend less time wrestling with the technicalities of DAX
language and more time uncovering the stories hidden in our data.

The benefits of this feature are manifold. It not only accelerates the
process of creating DAX measures but also opens the power of DAX to a
broader audience. It\'s a tool that breaks down barriers, inviting those
who might have been deterred by the complexity of DAX to participate in
the process of data exploration.

However, as we embrace this new tool, it\'s important to remember that
technology is an enabler, not a replacement. While Quick Measure
Suggestions can provide a valuable starting point, it\'s crucial to
validate the suggested measures to ensure they align with our specific
needs.

As we navigate this evolving landscape of data exploration, I encourage
you to try out Quick Measure Suggestions. It\'s a tool that can enhance
your data analysis capabilities, helping you delve deeper into your data
and uncover insights more efficiently.

**5\.  References**
1.  Schlegel, K., Sun, J., & 8 more. (2023, April 5). Magic Quadrant for
    Analytics and Business Intelligence Platforms. Gartner. Retrieved
    9th of July, 2023, from
    <https://www.gartner.com/doc/reprints?id=1-2955ETOT&ct=220215&st=sb?ocid=lp_pg398450_gdc_comm_az>
2.  Evelson, B., Katz, A., Herrington, K., Curran,
    R., Raymond, G., Sabri, F., Born, F., & Barton, J. (2023, January
    20). The Augmented Business Intelligence Platforms Landscape,
    Q1 2023. Forrester. Retrieved 9th of July, 2023, from
    <https://www.forrester.com/report/the-augmented-business-intelligence-platforms-landscape-q1-2023/RES178735>
3.  Microsoft Learn. (2023, April 13). Quick
    measure suggestions - Power BI. Retrieved 9th of July, 2023, from
    <https://learn.microsoft.com/en-us/power-bi/transform-model/quick-measure-suggestions>