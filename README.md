#        SAN FRANCISCO RESTAURANTS HEALTH VIOLATIONS ANALYSIS 

![alt text](https://bloximages.chicago2.vip.townnews.com/times-herald.com/content/tncms/assets/v3/editorial/f/68/f68fd3ee-337f-590b-9da3-c6da47961ee6/64011b6318f9b.image.jpg)



## Introduction

In this analysis, we explore health violations in San Francisco restaurants using a dataset contained in the table named `sf_restaurant_health_violations`. The purpose of this analysis is to understand the patterns, trends, and severity of health violations within the food industry. This dataset was studied from scratch, and through this analysis, we aim to generate insights that may be beneficial for consumers, restaurant owners, regulatory bodies, and policy-makers.In the end on the basis of analysis, I also generated a function to calculate the inspection score for a given violation. Apart from this I have also done some data transformation and use the transformed data in last two analysis. The python script for the same is attached. I have tried my best to make the analysis easy to follow and quite interactive. Hope you will enjoy it. 

Before diving into the details of the analysis, let's review the structure of the dataset, and the tools and methodologies employed.

## Dataset Structure

The `sf_restaurant_health_violations` table contains the following columns:

- **business_id**: Unique identifier for the business.
- **business_name**: Name of the business.
- **business_address**: Address of the business.
- **business_city**: City where the business is located.
- **business_state**: State where the business is located.
- **business_postal_code**: Postal code of the business address.
- **business_latitude**: Latitude coordinate of the business location.
- **business_longitude**: Longitude coordinate of the business location.
- **inspection_id**: Unique identifier for a particular inspection.
- **inspection_date**: The date the inspection took place.
- **inspection_score**: The score assigned to the business based on the inspection.
- **inspection_type**: Type of inspection conducted.
- **violation_id**: Unique identifier for the violation.
- **violation_description**: Detailed description of the violation.
- **risk_category**: The risk level associated with the violation.

## Tools and Methodology

For this analysis, SQL queries were utilized to extract and manipulate data from the `sf_restaurant_health_violations` table. To visualize and present the findings in a more interpretable manner, this analysis was conducted in a Jupyter Notebook, which allows for the integration of live code, visualizations, and narrative text. Python scripts were also used for the data transformations and for building custom functions.

## Analysis Index

###  0   :  [Data Preprocessing: Changing the Granularity of Data](#Data-Preprocessing-Changing-the-Granularity-of-Data)
### 1    :  [Yearly Trended Analysis of Health Violations](#Yearly-Trend-Analysis-of-Health-Violations)
### 2.1 :   [Analysis of Inspection Scores by Risk Category](#21-Analysis-of-Inspection-Scores-by-Risk-Category)
### 2.2 :   [Analysis of Inspection Scores for Non-Violation Category](# )
### 3    :  [Detailed Analysis of Top Violations by Risk Category](# )
### 4    :  [Analysis of Inspection Score Bins and their Associated Risk Categories](# )
### 5    :  [In-Depth Analysis of Businesses With Multiple Inspections in the Year 2016](# )
### 6    :  [Calculating points to deduct for Different Violations in all Categories](# )
### 7    :  [Calculating Inspection Scores for Restaurant Health Violations](# )

## Significance of This Analysis

Food safety is a critical component of public health. For consumers, it is essential to have trust in the sanitary practices of the restaurants they frequent. Restaurant owners need to uphold high standards not only to comply with regulations but also to maintain customer trust and their business reputation. Regulatory bodies must monitor and ensure that food establishments adhere to safety standards to safeguard public health.

This analysis was performed from scratch with the aim to unravel the intricate aspects of health violations in San Francisco's restaurants. The insights derived can help in making informed decisions, encourage restaurants to enhance their safety protocols, and facilitate data-driven policy-making for regulatory bodies.

Now, let's delve into the analysis and uncover the hidden insights.

# Data Preprocessing: Changing the Granularity of Data

Before we begin our analysis, it's important to understand the level of granularity in the original dataset, `sf_restaurant_health_violations`. In this dataset, each row corresponds to a single violation, which means that an inspection with multiple violations will have multiple rows.

However, for the purpose of our analysis, we are making an assumption and changing the granularity to the inspection level. In other words, each row in our modified dataset will correspond to a single inspection. Additionally, we want to categorize each inspection as either 'High Risk', 'Moderate Risk', 'Low Risk', or 'No Violation' if there is a null value indicating no violations were recorded during the inspection.

To accomplish this transformation, we will create a view named `restaurant_health_violations`. This view is generated by employing SQL window functions and other clauses to ensure that each inspection is represented only once and is associated with a single violation category.

Here is the SQL query that creates the view `restaurant_health_violations`:

```sql
CREATE VIEW restaurant_health_violations AS (
    SELECT
        business_id, business_name, business_address, business_city, 
        business_state, business_postal_code, business_latitude, 
        business_longitude, business_location, business_phone_number,
        inspection_id, inspection_date, inspection_score, inspection_type,
        violation_id, violation_description, risk_category
    FROM (
        SELECT
            *,
            ROW_NUMBER() OVER (PARTITION BY inspection_id) AS rn
        FROM sf_restaurant_health_violations
    ) AS a
    WHERE rn = 1
);
```

Explanation:

- We are selecting several columns from the original dataset, including business details, inspection details, and violation details.

- We use a window function `ROW_NUMBER()` to assign a unique number to each row within a partition of `inspection_id`. This helps us in selecting only one row for each inspection.

- In the outer query, we filter out the rows where the row number is 1, ensuring that only one row per inspection is present in the final view.

This view, `restaurant_health_violations`, will now serve as the foundation for our analysis. It has been designed to provide a streamlined and simplified version of the data, with each inspection being represented once and associated with a single violation category.

# 1: Yearly Trended Analysis

# Yearly Trend Analysis of Health Violations

One of the essential parts of our analysis is to study the yearly trends of health violations. This enables us to understand how the nature and severity of violations have evolved over time. By analyzing the data yearly, we can identify patterns and possibly predict future trends.

## Query Explanation

To perform this analysis, the following SQL query was used:

```sql
SELECT
    YEAR(inspection_date) AS year,
    COUNT(*) AS total_violated_inspections,
    AVG(inspection_score) AS avg_inspection_score,
    
    COUNT(CASE WHEN risk_category = "Low Risk" THEN inspection_id END) AS low_risk_total_inspections,
    ROUND(COUNT(CASE WHEN risk_category = "Low Risk" THEN inspection_id END)/COUNT(*)*100,1) AS low_risk_relative_pct,
    AVG(CASE WHEN risk_category = "Low Risk" THEN inspection_score END) AS low_risk_avg_inspection_score,
    
    COUNT(CASE WHEN risk_category = "Moderate Risk" THEN inspection_id END) AS Moderate_risk_total_inspections,
    ROUND(COUNT(CASE WHEN risk_category = "Moderate Risk" THEN inspection_id END)/COUNT(*)*100,1) AS Moderate_risk_relative_pct,
    AVG(CASE WHEN risk_category = "Moderate Risk" THEN inspection_score END) AS Moderate_risk_avg_inspection_score,
    
    COUNT(CASE WHEN risk_category = "High Risk" THEN inspection_id END) AS High_risk_total_inspections,
    ROUND(COUNT(CASE WHEN risk_category = "High Risk" THEN inspection_id END)/COUNT(*)*100,1) AS high_risk_relative_pct,
    AVG(CASE WHEN risk_category = "High Risk" THEN inspection_score END) AS High_risk_avg_inspection_score
FROM
    restaurant_health_violations
WHERE
    risk_category IN ("Low Risk", "Moderate Risk", "High Risk")
GROUP BY
    YEAR(inspection_date)
ORDER BY
    1;
```

This query extracts the following information grouped by year:

- Total number of inspections that had violations.
- Average inspection score.
- For each risk category (Low, Moderate, High):
    - Total number of inspections that had violations.
    - Percentage of total inspections that fell in this risk category.
    - Average inspection score for this risk category.

## Result Interpretation

Here's how to interpret the results produced by the query:

- **year**: This represents the year in which inspections were performed.

- **total_violated_inspections**: The total number of inspections conducted in that year which recorded violations.

- **avg_inspection_score**: The average inspection score for the year.

For each risk category (Low, Moderate, High):

- **[risk]_risk_total_inspections**: The number of inspections that were categorized as Low/Moderate/High risk in that year.

- **[risk]_risk_relative_pct**: The percentage of inspections that were categorized as Low/Moderate/High risk relative to the total violated inspections of that year.

- **[risk]_risk_avg_inspection_score**: The average inspection score for inspections that fell into the respective risk category in that year.


From the output provided, let's analyze what is concerning and what seems positive.

## Concerning Points:

1. **High-Risk Violations in 2016**: In 2016, 20.5% of the total violated inspections were categorized as "High Risk". This is a significant proportion and is higher than other years in the dataset. High-risk violations are the most concerning as they can have severe consequences for public health.

2. **Decline in Average Inspection Score in 2017**: In 2017, the average inspection score declined to 77.2 from 78.9 in 2016. A decline in the average inspection score indicates that, on average, there were more or more severe violations in that year compared to the previous year.


## Positive Points:

1. **Increase in Average Inspection Score in 2018**: In 2018, there was an increase in the average inspection score to 84.3 from 77.2 in 2017. This might indicate an improvement in the general health compliance of the restaurants.

2. **Reduction in High-Risk Violations in 2018**: Only 7.0% of inspections were categorized as "High Risk" in 2018, compared to 17.9% in 2017. This reduction in high-risk violations is a positive sign and may indicate that restaurants are taking steps to address and mitigate serious health violations.

## Other:

As we move through the years from 2015 to 2018, we can observe several trends in the data:

**Average Inspection Score Fluctuations**: The average inspection score initially declines from 85.9 in 2015 to 77.2 in 2017 but then shows improvement, rising to 84.3 in 2018. The decline may signal that violations were becoming more severe or more frequent, but the rebound in 2018 suggests some improvement in health compliance or changes in inspection criteria.

**Shifting Risk Categories**: The proportion of high-risk violations increases sharply in 2016 to 20.5%, but then we see a steady decline in subsequent years (17.9% in 2017 and 7.0% in 2018). This decline is a positive trend, indicating that severe violations are becoming less common over time.

In summary, the data shows a trend towards improvement, particularly in terms of reducing high-risk violations and stabilizing average inspection scores.


```sql
%%sql

SELECT
    YEAR(inspection_date) AS year,
    COUNT(*) AS total_violated_inspections,
    AVG(inspection_score) AS avg_inspection_score,

    COUNT(CASE WHEN risk_category = "Low Risk" THEN inspection_id END) AS low_risk_total_inspections,
    ROUND(COUNT(CASE WHEN risk_category = "Low Risk" THEN inspection_id END)/COUNT(*)*100,1) AS low_risk_relative_pct,
    AVG(CASE WHEN risk_category = "Low Risk" THEN inspection_score END) AS low_risk_avg_inspection_score,

    COUNT(CASE WHEN risk_category = "Moderate Risk" THEN inspection_id END) AS Moderate_risk_total_inspections,
    ROUND(COUNT(CASE WHEN risk_category = "Moderate Risk" THEN inspection_id END)/COUNT(*)*100,1) AS Moderate_risk_relative_pct,
    AVG(CASE WHEN risk_category = "Moderate Risk" THEN inspection_score END) AS Moderate_risk_avg_inspection_score,

    COUNT(CASE WHEN risk_category = "High Risk" THEN inspection_id END) AS High_risk_total_inspections,
    ROUND(COUNT(CASE WHEN risk_category = "High Risk" THEN inspection_id END)/COUNT(*)*100,1) AS high_risk_relative_pct,
    AVG(CASE WHEN risk_category = "High Risk" THEN inspection_score END) AS High_risk_avg_inspection_score
FROM
    restaurant_health_violations
WHERE
    risk_category IN ("Low Risk", "Moderate Risk", "High Risk")
GROUP BY
    YEAR(inspection_date)
ORDER BY
    1;
```

     * mysql://root:***@127.0.0.1:3306/hundred
    4 rows affected.
    




<table>
    <thead>
        <tr>
            <th>year</th>
            <th>total_violated_inspections</th>
            <th>avg_inspection_score</th>
            <th>low_risk_total_inspections</th>
            <th>low_risk_relative_pct</th>
            <th>low_risk_avg_inspection_score</th>
            <th>Moderate_risk_total_inspections</th>
            <th>Moderate_risk_relative_pct</th>
            <th>Moderate_risk_avg_inspection_score</th>
            <th>High_risk_total_inspections</th>
            <th>high_risk_relative_pct</th>
            <th>High_risk_avg_inspection_score</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>2015</td>
            <td>8</td>
            <td>85.8750</td>
            <td>3</td>
            <td>37.5</td>
            <td>83.3333</td>
            <td>5</td>
            <td>62.5</td>
            <td>87.4000</td>
            <td>0</td>
            <td>0.0</td>
            <td>None</td>
        </tr>
        <tr>
            <td>2016</td>
            <td>78</td>
            <td>78.9487</td>
            <td>33</td>
            <td>42.3</td>
            <td>85.6364</td>
            <td>29</td>
            <td>37.2</td>
            <td>76.6552</td>
            <td>16</td>
            <td>20.5</td>
            <td>69.3125</td>
        </tr>
        <tr>
            <td>2017</td>
            <td>78</td>
            <td>77.1795</td>
            <td>46</td>
            <td>59.0</td>
            <td>78.3478</td>
            <td>18</td>
            <td>23.1</td>
            <td>77.0000</td>
            <td>14</td>
            <td>17.9</td>
            <td>73.5714</td>
        </tr>
        <tr>
            <td>2018</td>
            <td>43</td>
            <td>84.3023</td>
            <td>20</td>
            <td>46.5</td>
            <td>89.3500</td>
            <td>20</td>
            <td>46.5</td>
            <td>81.2500</td>
            <td>3</td>
            <td>7.0</td>
            <td>71.0000</td>
        </tr>
    </tbody>
</table>



# 2:Analysis of Inspection Scores by Risk Category

# 2.1: Analysis of Inspection Scores by Risk Category

## SQL Query:
```sql
SELECT
    risk_category,
    COUNT(*) AS total_inspections_fall_under_this_category,
    AVG(inspection_score) AS avg_score,
    MIN(inspection_score) AS min_score,
    MAX(inspection_score) AS max_score
FROM
    restaurant_health_violations
GROUP BY
    risk_category;
```

## Explanation:
In this analysis, we are examining the different risk categories of health violations in restaurants and the corresponding inspection scores. The risk categories are "High Risk," "Low Risk," "Moderate Risk," and "No Violation." For each category, the total number of inspections that fall under this category, the average inspection score, the minimum inspection score, and the maximum inspection score are computed using the SQL query above.

## Insights:

1. **No Violations**: There are 72 inspections with no violations. This indicates that in these inspections, the establishments met all the health standards. However, the average score in this category is not particularly meaningful since there were no violations to evaluate.

2. **High Risk Violations**: There were 33 inspections that fell under the "High Risk" category. These inspections had an average score of 71.3, with scores ranging from 0 to 93. This is concerning as high-risk violations can have severe consequences for public health. 

3. **Low Risk Violations**: The "Low Risk" category had the highest number of inspections with 102. These had an average score of 83.0, with scores ranging from 0 to 98. The high average score and the high number of inspections in this category suggest that while violations were identified, they were generally of lesser severity and may represent areas where minor improvements are needed.

4. **Moderate Risk Violations**: There were 72 inspections that fell under the "Moderate Risk" category, with an average score of 78.8. The scores in this category ranged from 0 to 96. This indicates a middle ground between high and low-risk violations in terms of severity. 

5. **Score Range**: The minimum score of 0 in all categories except "No Violation" is concerning and indicates that there are instances where the violations were severe enough to warrant the lowest possible score. 



```sql
%%sql 

SELECT
    risk_category,
    COUNT(*) AS total_inspections_fall_under_this_category,
    AVG(inspection_score) AS avg_score,
    MIN(inspection_score) AS min_score,
    MAX(inspection_score) AS max_score
FROM
    restaurant_health_violations
GROUP BY
    risk_category
order by 1 
    
# note: the blank in  risk_category is "No Violation Category"
```

     * mysql://root:***@127.0.0.1:3306/hundred
    4 rows affected.
    




<table>
    <thead>
        <tr>
            <th>risk_category</th>
            <th>total_inspections_fall_under_this_category</th>
            <th>avg_score</th>
            <th>min_score</th>
            <th>max_score</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td></td>
            <td>72</td>
            <td>15.2778</td>
            <td>0</td>
            <td>100</td>
        </tr>
        <tr>
            <td>High Risk</td>
            <td>33</td>
            <td>71.2727</td>
            <td>0</td>
            <td>93</td>
        </tr>
        <tr>
            <td>Low Risk</td>
            <td>102</td>
            <td>83.0098</td>
            <td>0</td>
            <td>98</td>
        </tr>
        <tr>
            <td>Moderate Risk</td>
            <td>72</td>
            <td>78.7639</td>
            <td>0</td>
            <td>96</td>
        </tr>
    </tbody>
</table>



# 2.2: Analysis of Inspection Scores for Non-Violation Category


## SQL Query:
```sql
SELECT
    COUNT(*) AS total_non_violations,
    COUNT(CASE WHEN inspection_score=0 THEN inspection_score END) AS inspection_score_0,
    COUNT(CASE WHEN inspection_score=100 THEN inspection_score END) AS inspection_score_100,
    COUNT(CASE WHEN inspection_score!=100 OR inspection_score!=0 THEN NULL END) AS inspection_score_other
FROM
    restaurant_health_violations
WHERE
    risk_category NOT IN ('Low Risk', 'Moderate Risk', 'High Risk');
```

## Explanation:
In this analysis, we are focusing on the inspections that were categorized as "No Violation" and examining the distribution of their inspection scores. Specifically, we are looking at how many of these non-violation inspections were given a score of 0, how many were given a perfect score of 100, and whether there were any other scores assigned to non-violation inspections. 

## Insights:

1. **Total Non-Violation Inspections**: There are 72 inspections categorized as "No Violation".

2. **Inspection Score of 0**: Out of the non-violation inspections, 61 inspections were given a score of 0. This indicates that a score of 0 was used to represent the absence of violations. 

3. **Inspection Score of 100**: Interestingly, 11 inspections were given a perfect score of 100, which could imply an exemplary level of adherence to health standards.

4. **No Other Scores**: From the output, we can observe that there are no inspections that were assigned a score other than 0 or 100 in the non-violation category. This suggests a consistent scoring approach for non-violation inspections.

This analysis provides insights into how inspection scores are assigned in cases where no violations were identified. A score of 0 seems to be commonly used to signify the absence of violations, but a perfect score of 100 is also used, possibly to reward exceptional compliance. The absence of any other scores indicates a clear distinction in the scoring method for non-violation inspections, which is beneficial for data consistency and interpretation.


```sql
%%sql

SELECT
    COUNT(*) AS total_non_violations,
    COUNT(CASE WHEN inspection_score=0 THEN inspection_score END) AS inspection_score_0,
    COUNT(CASE WHEN inspection_score=100 THEN inspection_score END) AS inspection_score_100,
    COUNT(CASE WHEN inspection_score!=100 OR inspection_score!=0 THEN NULL END) AS inspection_score_other
FROM
    restaurant_health_violations
WHERE
    risk_category NOT IN ('Low Risk', 'Moderate Risk', 'High Risk');
```

     * mysql://root:***@127.0.0.1:3306/hundred
    1 rows affected.
    




<table>
    <thead>
        <tr>
            <th>total_non_violations</th>
            <th>inspection_score_0</th>
            <th>inspection_score_100</th>
            <th>inspection_score_other</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>72</td>
            <td>61</td>
            <td>11</td>
            <td>0</td>
        </tr>
    </tbody>
</table>



# 3: Detailed Analysis of Top Violations by Risk Category

# Analysis of Top Violations by Risk Category

## SQL Queries:
```sql
-- High Risk Violations
SELECT
    Violation_description AS high_risk_violation,
    COUNT(*) AS total_inspections_violated_due_to_this_violation
FROM
    restaurant_health_violations
WHERE
    risk_category = 'High Risk'
GROUP BY
    Violation_description
ORDER BY
    2 DESC;

-- Moderate Risk Violations
SELECT
    Violation_description AS moderate_risk_violation,
    COUNT(*) AS total_inspections_violated_due_to_this_violation
FROM
    restaurant_health_violations
WHERE
    risk_category = 'Moderate Risk'
GROUP BY
    Violation_description
ORDER BY
    2 DESC;

-- Low Risk Violations
SELECT
    Violation_description AS low_risk_violation,
    COUNT(*) AS total_inspections_violated_due_to_this_violation
FROM
    restaurant_health_violations
WHERE
    risk_category = 'Low Risk'
GROUP BY
    Violation_description
ORDER BY
    2 DESC;
```

## Output (Top 5 for each category):

#### High Risk Violations:

| high_risk_violation                                   | total_inspections_violated_due_to_this_violation |
|-------------------------------------------------------|-------------------------------------------------|
| High risk food holding temperature                    | 8                                               |
| Unclean hands or improper use of gloves               | 6                                               |
| Improper cooling methods                              | 5                                               |
| Unclean or unsanitary food contact surfaces           | 4                                               |
| Improper reheating of food                            | 2                                               |

#### Moderate Risk Violations:

| moderate_risk_violation                                   | total_inspections_violated_due_to_this_violation |
|-----------------------------------------------------------|-------------------------------------------------|
| Inadequate and inaccessible handwashing facilities        | 16                                              |
| Moderate risk food holding temperature                    | 13                                              |
| Moderate risk vermin infestation                          | 12                                              |
| Inadequately cleaned or sanitized food contact surfaces   | 10                                              |
| Foods not protected from contamination                    | 9                                               |

#### Low Risk Violations:

| low_risk_violation                                      | total_inspections_violated_due_to_this_violation |
|---------------------------------------------------------|-------------------------------------------------|
| Unapproved or unmaintained equipment or utensils        | 20                                              |
| Unclean or degraded floors walls or ceilings            | 17                                              |
| Wiping cloths not clean or properly stored or inadequate sanitizer | 11                                |
| Unclean nonfood contact surfaces                        | 10                                              |
| Improper food storage                                   | 9                                               |


## Analysis:

In this part of the analysis, we are diving deeper into the different risk categories to understand the specific types of violations that contribute to each category. 

### High Risk Violations:
The top high-risk violation is "High-risk food holding temperature" with a total of 8 inspections violated due to this. This is a critical violation since improper food temperature can lead to bacterial growth and foodborne illnesses. Other concerning high-risk violations include "Unclean hands or improper use of gloves" and "Improper cooling methods." These types of violations are considered high risk as they directly impact food safety and can have severe consequences for the consumers.

### Moderate Risk Violations:
Within the moderate risk category, the most frequent violation is “Inadequate and inaccessible handwashing facilities” with 16 inspections. This is followed by “Moderate risk food holding temperature” and “Moderate risk vermin infestation.” Although these violations are not as severe as high-risk ones, they are still significant as they can compromise food safety to some extent. Ensuring proper handwashing facilities and maintaining food at safe temperatures are important to minimize the risk of contamination.

### Low Risk Violations:
In the low-risk category, the most common violations are related to the maintenance and cleanliness of the establishment, such as “Unapproved or unmaintained equipment or utensils” and “Unclean or degraded floors walls or ceilings.” These violations are less likely to cause foodborne illnesses but are still important for the overall hygiene and functionality of the establishment.

## Insights:

1. For high-risk violations, the focus should be on training staff in food safety practices, especially around food temperature control and personal hygiene.

2. In the case of moderate-risk violations, improving the availability and accessibility of handwashing facilities and ensuring proper food temperature can help in reducing the number of violations.

3. For low-risk violations, regular maintenance and cleaning schedules can help in ensuring that the establishment meets the required standards.

4. Restaurant owners and managers should be made aware of the common violations and encouraged to take proactive steps in ensuring compliance with health standards.

Overall, while high-risk violations are the most concerning, attention must also be given to moderate and low-risk violations to ensure the overall safety and hygiene of the establishments.


```sql
%%sql

-- High Risk Violations
SELECT
    Violation_description AS high_risk_violation,
    COUNT(*) AS total_inspections_violated_due_to_this_violation
FROM
    restaurant_health_violations
WHERE
    risk_category = 'High Risk'
GROUP BY
    Violation_description
ORDER BY
    2 DESC;

```

     * mysql://root:***@127.0.0.1:3306/hundred
    10 rows affected.
    




<table>
    <thead>
        <tr>
            <th>high_risk_violation</th>
            <th>total_inspections_violated_due_to_this_violation</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>High risk food holding temperature</td>
            <td>8</td>
        </tr>
        <tr>
            <td>Unclean hands or improper use of gloves</td>
            <td>6</td>
        </tr>
        <tr>
            <td>Improper cooling methods</td>
            <td>5</td>
        </tr>
        <tr>
            <td>Unclean or unsanitary food contact surfaces</td>
            <td>4</td>
        </tr>
        <tr>
            <td>Improper reheating of food</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Contaminated or adulterated food</td>
            <td>2</td>
        </tr>
        <tr>
            <td>No hot water or running water</td>
            <td>2</td>
        </tr>
        <tr>
            <td>High risk vermin infestation</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Unauthorized or unsafe use of time as a public health control measure</td>
            <td>1</td>
        </tr>
        <tr>
            <td>Other high risk violation</td>
            <td>1</td>
        </tr>
    </tbody>
</table>




```sql
%%sql

-- Moderate Risk Violations
SELECT
    Violation_description AS moderate_risk_violation,
    COUNT(*) AS total_inspections_violated_due_to_this_violation
FROM
    restaurant_health_violations
WHERE
    risk_category = 'Moderate Risk'
GROUP BY
    Violation_description
ORDER BY
    2 DESC;
```

     * mysql://root:***@127.0.0.1:3306/hundred
    11 rows affected.
    




<table>
    <thead>
        <tr>
            <th>moderate_risk_violation</th>
            <th>total_inspections_violated_due_to_this_violation</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Inadequate and inaccessible handwashing facilities</td>
            <td>16</td>
        </tr>
        <tr>
            <td>Moderate risk food holding temperature</td>
            <td>13</td>
        </tr>
        <tr>
            <td>Moderate risk vermin infestation</td>
            <td>12</td>
        </tr>
        <tr>
            <td>Inadequately cleaned or sanitized food contact surfaces</td>
            <td>10</td>
        </tr>
        <tr>
            <td>Foods not protected from contamination</td>
            <td>9</td>
        </tr>
        <tr>
            <td>Improper thawing methods</td>
            <td>3</td>
        </tr>
        <tr>
            <td>Inadequate food safety knowledge or lack of certified food safety manager</td>
            <td>3</td>
        </tr>
        <tr>
            <td>Insufficient hot water or running water</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Inadequate procedures or records for time as a public health control</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Inadequate sewage or wastewater disposal</td>
            <td>1</td>
        </tr>
        <tr>
            <td>Noncompliance with shell fish tags or display</td>
            <td>1</td>
        </tr>
    </tbody>
</table>




```sql
%%sql

-- Low Risk Violations
SELECT
    Violation_description AS low_risk_violation,
    COUNT(*) AS total_inspections_violated_due_to_this_violation
FROM
    restaurant_health_violations
WHERE
    risk_category = 'Low Risk'
GROUP BY
    Violation_description
ORDER BY
    2 DESC;
```

     * mysql://root:***@127.0.0.1:3306/hundred
    15 rows affected.
    




<table>
    <thead>
        <tr>
            <th>low_risk_violation</th>
            <th>total_inspections_violated_due_to_this_violation</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Unapproved or unmaintained equipment or utensils</td>
            <td>20</td>
        </tr>
        <tr>
            <td>Unclean or degraded floors walls or ceilings</td>
            <td>17</td>
        </tr>
        <tr>
            <td>Wiping cloths not clean or properly stored or inadequate sanitizer</td>
            <td>11</td>
        </tr>
        <tr>
            <td>Unclean nonfood contact surfaces</td>
            <td>10</td>
        </tr>
        <tr>
            <td>Improper food storage</td>
            <td>9</td>
        </tr>
        <tr>
            <td>Improper or defective plumbing</td>
            <td>8</td>
        </tr>
        <tr>
            <td>Permit license or inspection report not posted</td>
            <td>7</td>
        </tr>
        <tr>
            <td>Low risk vermin infestation</td>
            <td>4</td>
        </tr>
        <tr>
            <td>Food safety certificate or food handler card not available</td>
            <td>4</td>
        </tr>
        <tr>
            <td>Improper storage use or identification of toxic substances</td>
            <td>3</td>
        </tr>
        <tr>
            <td>Improper storage of equipment utensils or linens</td>
            <td>3</td>
        </tr>
        <tr>
            <td>Inadequate dressing rooms or improper storage of personal items</td>
            <td>3</td>
        </tr>
        <tr>
            <td>Other low risk violation</td>
            <td>1</td>
        </tr>
        <tr>
            <td>Inadequate warewashing facilities or equipment</td>
            <td>1</td>
        </tr>
        <tr>
            <td>No person in charge of food facility</td>
            <td>1</td>
        </tr>
    </tbody>
</table>



# 4: Distribution By Inspection Score bins at Risk Level

# Analysis of Inspection Score Bins and their Associated Risk Categories

## SQL Query:
```sql
WITH cte1 AS (
    SELECT
        *,
        CASE 
            WHEN inspection_score >= 0 AND inspection_score <= 10 THEN 'score_0_to_10'
            WHEN inspection_score > 10 AND inspection_score <= 20 THEN 'score_10_to_20'
            WHEN inspection_score > 20 AND inspection_score <= 30 THEN 'score_20_to_30'
            WHEN inspection_score > 30 AND inspection_score <= 40 THEN 'score_30_to_40'
            WHEN inspection_score > 40 AND inspection_score <= 50 THEN 'score_40_to_50'
            WHEN inspection_score > 50 AND inspection_score <= 60 THEN 'score_50_to_60'
            WHEN inspection_score > 60 AND inspection_score <= 70 THEN 'score_60_to_70'
            WHEN inspection_score > 70 AND inspection_score <= 80 THEN 'score_70_to_80'
            WHEN inspection_score > 80 AND inspection_score <= 90 THEN 'score_80_to_90'
            WHEN inspection_score > 90 AND inspection_score <= 100 THEN 'score_90_to_100'
        END AS inspection_score_bin
    FROM
        restaurant_health_violations
    WHERE
        risk_category IN ("Low Risk", "Moderate Risk", "High Risk")
)
SELECT
    inspection_score_bin,
    COUNT(DISTINCT inspection_id) AS total_inspections,
    COUNT(DISTINCT CASE WHEN risk_category = "Low Risk" THEN inspection_id END) AS "Low_Risk_categorized_Inspections",
    COUNT(DISTINCT CASE WHEN risk_category = "Moderate Risk" THEN inspection_id END) AS "Moderate_Risk_categorized_Inspections",
    COUNT(DISTINCT CASE WHEN risk_category = "High Risk" THEN inspection_id END) AS "High_Risk_categorized_Inspections"
FROM
    cte1
WHERE
    risk_category IN ("Low Risk", "Moderate Risk", "High Risk")
GROUP BY
    inspection_score_bin
ORDER BY
    1;
```

## Explanation:
In this analysis, the inspection scores are binned into ranges such as 0 to 10, 40 to 50, 60 to 70, and so on. For each bin, we have calculated the total number of inspections and the number of inspections that are categorized as "Low Risk," "Moderate Risk," and "High Risk."

## Insights:

**Concern in Lower Scores**: The bin 'score_0_to_10' is concerning because it represents the lowest scores. There are 12 inspections in this range, with 6 falling under "Moderate Risk" and 2 under "High Risk". This indicates severe health code violations and a critical need for improvements.

**High Risk inspections with high inspection score**: From the analysis above we can see that there are 2 inspections with inspection score more than 90 but still categorized as high risk inspections, lets extarct the details of those inspections:

```sql
select
*
  from restaurant_health_violations
where 
      risk_category ="High Risk" and inspection_score >90;
```

The output we got from above query is :

| business_id | business_name   | business_address | business_city | business_state | business_postal_code | business_latitude | business_longitude | inspection_id  | inspection_date | inspection_score | inspection_type        | violation_id           | violation_description           | risk_category |
|-------------|-----------------|------------------|---------------|----------------|----------------------|-------------------|--------------------|----------------|-----------------|------------------|------------------------|------------------------|---------------------------------|---------------|
| 70090       | Cathead's BBQ   | 1665 FOLSOM St   | San Francisco | CA             | 94103                | 37.770000         | -122.415000        | 70090_20170105 | 2017-01-05      | 93               | Routine - Unscheduled  | 70090_20170105_103105  | Improper cooling methods        | High Risk     |
| 81579       | Salem Grocery   | 920 Geary St     | San Francisco | CA             | 94109                | 0.000000          | 0.000000           | 81579_20171221 | 2017-12-21      | 91               | Routine - Unscheduled  | 81579_20171221_103103  | High risk food holding temperature | High Risk     |

From the data, we observe that Cathead's BBQ had an inspection score of 93, but was cited for "Improper cooling methods" which is a High Risk violation. Similarly, Salem Grocery received a score of 91 and was cited for "High risk food holding temperature", another High Risk violation. 

These violations are considered high risk because they can directly contribute to foodborne illnesses. Despite the overall high scores, these specific violations are severe enough to warrant a high-risk categorization. This highlights the importance of not only looking at the overall score but also paying attention to the nature of any violations that occurred.



```sql
%%sql

# genearting bins and then using them for distributional analysis at risk level.

WITH cte1 AS (
    SELECT
        *,
        CASE 
            WHEN inspection_score >= 0 AND inspection_score <= 10 THEN 'score_0_to_10'
            WHEN inspection_score > 10 AND inspection_score <= 20 THEN 'score_10_to_20'
            WHEN inspection_score > 20 AND inspection_score <= 30 THEN 'score_20_to_30'
            WHEN inspection_score > 30 AND inspection_score <= 40 THEN 'score_30_to_40'
            WHEN inspection_score > 40 AND inspection_score <= 50 THEN 'score_40_to_50'
            WHEN inspection_score > 50 AND inspection_score <= 60 THEN 'score_50_to_60'
            WHEN inspection_score > 60 AND inspection_score <= 70 THEN 'score_60_to_70'
            WHEN inspection_score > 70 AND inspection_score <= 80 THEN 'score_70_to_80'
            WHEN inspection_score > 80 AND inspection_score <= 90 THEN 'score_80_to_90'
            WHEN inspection_score > 90 AND inspection_score <= 100 THEN 'score_90_to_100'
        END AS inspection_score_bin
    FROM
        restaurant_health_violations
    WHERE
        risk_category IN ("Low Risk", "Moderate Risk", "High Risk")
)
SELECT
    inspection_score_bin,
    COUNT(DISTINCT inspection_id) AS total_inspections,
    COUNT(DISTINCT CASE WHEN risk_category = "Low Risk" THEN inspection_id END) AS "Low_Risk_categorized_Inspections",
    COUNT(DISTINCT CASE WHEN risk_category = "Moderate Risk" THEN inspection_id END) AS "Moderate_Risk_categorized_Inspections",
    COUNT(DISTINCT CASE WHEN risk_category = "High Risk" THEN inspection_id END) AS "High_Risk_categorized_Inspections"
FROM
    cte1
WHERE
    risk_category IN ("Low Risk", "Moderate Risk", "High Risk")
GROUP BY
    inspection_score_bin
ORDER BY
    1;
```

     * mysql://root:***@127.0.0.1:3306/hundred
    6 rows affected.
    




<table>
    <thead>
        <tr>
            <th>inspection_score_bin</th>
            <th>total_inspections</th>
            <th>Low_Risk_categorized_Inspections</th>
            <th>Moderate_Risk_categorized_Inspections</th>
            <th>High_Risk_categorized_Inspections</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>score_0_to_10</td>
            <td>12</td>
            <td>4</td>
            <td>6</td>
            <td>2</td>
        </tr>
        <tr>
            <td>score_40_to_50</td>
            <td>2</td>
            <td>0</td>
            <td>0</td>
            <td>2</td>
        </tr>
        <tr>
            <td>score_60_to_70</td>
            <td>11</td>
            <td>3</td>
            <td>1</td>
            <td>7</td>
        </tr>
        <tr>
            <td>score_70_to_80</td>
            <td>40</td>
            <td>21</td>
            <td>11</td>
            <td>8</td>
        </tr>
        <tr>
            <td>score_80_to_90</td>
            <td>92</td>
            <td>41</td>
            <td>39</td>
            <td>12</td>
        </tr>
        <tr>
            <td>score_90_to_100</td>
            <td>50</td>
            <td>33</td>
            <td>15</td>
            <td>2</td>
        </tr>
    </tbody>
</table>




```sql
%%sql

#details of all the High Risk inspections which have score more than 90 but still considered as High Risk Violated 

select
*
  from restaurant_health_violations
where 
      risk_category ="High Risk" and inspection_score >90;
```

     * mysql://root:***@127.0.0.1:3306/hundred
    2 rows affected.
    




<table>
    <thead>
        <tr>
            <th>business_id</th>
            <th>business_name</th>
            <th>business_address</th>
            <th>business_city</th>
            <th>business_state</th>
            <th>business_postal_code</th>
            <th>business_latitude</th>
            <th>business_longitude</th>
            <th>business_location</th>
            <th>business_phone_number</th>
            <th>inspection_id</th>
            <th>inspection_date</th>
            <th>inspection_score</th>
            <th>inspection_type</th>
            <th>violation_id</th>
            <th>violation_description</th>
            <th>risk_category</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>70090</td>
            <td>Cathead&#x27;s BBQ</td>
            <td>1665 FOLSOM St</td>
            <td>San Francisco</td>
            <td>CA</td>
            <td>94103</td>
            <td>37.770000</td>
            <td>-122.415000</td>
            <td>None</td>
            <td></td>
            <td>70090_20170105</td>
            <td>2017-01-05</td>
            <td>93</td>
            <td>Routine - Unscheduled</td>
            <td>70090_20170105_103105</td>
            <td>Improper cooling methods</td>
            <td>High Risk</td>
        </tr>
        <tr>
            <td>81579</td>
            <td>Salem Grocery</td>
            <td>920 Geary St</td>
            <td>San Francisco</td>
            <td>CA</td>
            <td>94109</td>
            <td>0.000000</td>
            <td>0.000000</td>
            <td>None</td>
            <td></td>
            <td>81579_20171221</td>
            <td>2017-12-21</td>
            <td>91</td>
            <td>Routine - Unscheduled</td>
            <td>81579_20171221_103103</td>
            <td>High risk food holding temperature</td>
            <td>High Risk</td>
        </tr>
    </tbody>
</table>



# 5:  In-Depth Analysis of Businesses With Multiple Inspections in the Year 2016 

## SQL Queries:
```sql
-- Query 1: Identifying businesses that had more than one inspection in 2016
SELECT
    business_id, business_name
FROM
    restaurant_health_violations
WHERE
    YEAR(inspection_date) = 2016
GROUP BY
    business_id, business_name
HAVING
    COUNT(DISTINCT inspection_date) > 1;

-- Query 2: Counting the number of inspections for these businesses in 2016
WITH cte1 AS (
    SELECT
        business_id, business_name, business_address, business_city, business_state, business_postal_code,
        business_latitude, business_longitude, business_location, business_phone_number,
        inspection_id, inspection_date, inspection_score, inspection_type,
        violation_id, violation_description, risk_category
    FROM (
        SELECT *,
        ROW_NUMBER() OVER (PARTITION BY inspection_id) AS rn
        FROM
            sf_restaurant_health_violations
    ) a
    WHERE
        rn = 1
)
SELECT
    business_id, business_name, COUNT(*) AS total_inspections,
    COUNT(*) - 1 AS total_more_than_one_inspections
FROM
    restaurant_health_violations
WHERE
    (business_id, business_name) IN (
        SELECT business_id, business_name
        FROM cte1
        WHERE YEAR(inspection_date) = 2016
        GROUP BY business_id, business_name
        HAVING COUNT(DISTINCT inspection_date) > 1
    )
    AND YEAR(inspection_date) = 2016
GROUP BY
    business_id, business_name;

-- Query 3: Detailed information on inspections for a specific business in 2016
SELECT
    *
FROM
    restaurant_health_violations
WHERE
    business_id = 7747 AND YEAR(inspection_date) = 2016;
```

## Explanation:
In this analysis, we focus on businesses that underwent multiple inspections within the year 2016. Our goal is to understand the frequency of inspections for these businesses, and analyze the outcomes in terms of inspection scores and violations. 

## Insights:

1. **Businesses with Multiple Inspections**: The first query reveals that the business with ID 7747, SAFEWAY STORE #964, underwent more than one inspection in the year 2016.

2. **Count of Inspections**: The second query provides us with the total number of inspections for SAFEWAY STORE #964, which is 2. This signifies that the regulatory authorities found it necessary to conduct inspections for this business more than once in a single year, which might indicate previous violations or complaints.

3. **Inspection Details**: The third query presents us with detailed information regarding the two inspections of SAFEWAY STORE #964. The first inspection occurred on June 9, 2016, with an inspection score of 81. This inspection was classified as "Routine - Unscheduled" and the business was cited for unapproved or unmaintained equipment or utensils, which was categorized as "Low Risk". The second inspection took place on June 24, 2016, and was triggered by a complaint. The score for this inspection is recorded as 0, which can be interpreted as not being scored or the result of a complaint-based inspection.

4. **Concerns and Action Points**: It is concerning that SAFEWAY STORE #964 required multiple inspections within a short period. This could indicate non-compliance with health regulations or other issues that warranted attention from regulatory authorities. It is essential for the business to address the violations and ensure adherence to health standards. Moreover, regulatory authorities


```sql
%%sql

#Query 1: Identifying businesses that had more than one inspection in 2016

SELECT
    business_id, business_name
FROM
    restaurant_health_violations
WHERE
    YEAR(inspection_date) = 2016
GROUP BY
    business_id, business_name
HAVING
    COUNT(DISTINCT inspection_date) > 1;
```

     * mysql://root:***@127.0.0.1:3306/hundred
    1 rows affected.
    




<table>
    <thead>
        <tr>
            <th>business_id</th>
            <th>business_name</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>7747</td>
            <td>SAFEWAY STORE #964</td>
        </tr>
    </tbody>
</table>




```sql
%%sql

# Query 2: Counting the number of inspections for these businesses in 2016

WITH cte1 AS (
    SELECT
        business_id, business_name, business_address, business_city, business_state, business_postal_code,
        business_latitude, business_longitude, business_location, business_phone_number,
        inspection_id, inspection_date, inspection_score, inspection_type,
        violation_id, violation_description, risk_category
    FROM (
        SELECT *,
        ROW_NUMBER() OVER (PARTITION BY inspection_id) AS rn
        FROM
            sf_restaurant_health_violations
    ) a
    WHERE
        rn = 1
)
SELECT
    business_id, business_name, COUNT(*) AS total_inspections,
    COUNT(*) - 1 AS total_more_than_one_inspections
FROM
    restaurant_health_violations
WHERE
    (business_id, business_name) IN (
        SELECT business_id, business_name
        FROM cte1
        WHERE YEAR(inspection_date) = 2016
        GROUP BY business_id, business_name
        HAVING COUNT(DISTINCT inspection_date) > 1
    )
    AND YEAR(inspection_date) = 2016
GROUP BY
    business_id, business_name;
```

     * mysql://root:***@127.0.0.1:3306/hundred
    1 rows affected.
    




<table>
    <thead>
        <tr>
            <th>business_id</th>
            <th>business_name</th>
            <th>total_inspections</th>
            <th>total_more_than_one_inspections</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>7747</td>
            <td>SAFEWAY STORE #964</td>
            <td>2</td>
            <td>1</td>
        </tr>
    </tbody>
</table>




```sql
%%sql

-- Query 3: Detailed information on inspections for a specific business in 2016
SELECT
    *
FROM
    restaurant_health_violations
WHERE
    business_id = 7747 AND YEAR(inspection_date) = 2016;
```

     * mysql://root:***@127.0.0.1:3306/hundred
    2 rows affected.
    




<table>
    <thead>
        <tr>
            <th>business_id</th>
            <th>business_name</th>
            <th>business_address</th>
            <th>business_city</th>
            <th>business_state</th>
            <th>business_postal_code</th>
            <th>business_latitude</th>
            <th>business_longitude</th>
            <th>business_location</th>
            <th>business_phone_number</th>
            <th>inspection_id</th>
            <th>inspection_date</th>
            <th>inspection_score</th>
            <th>inspection_type</th>
            <th>violation_id</th>
            <th>violation_description</th>
            <th>risk_category</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>7747</td>
            <td>SAFEWAY STORE #964</td>
            <td>4950 Mission St</td>
            <td>San Francisco</td>
            <td>CA</td>
            <td>94112</td>
            <td>37.720000</td>
            <td>-122.439000</td>
            <td>None</td>
            <td>14155587200</td>
            <td>7747_20160609</td>
            <td>2016-06-09</td>
            <td>81</td>
            <td>Routine - Unscheduled</td>
            <td>7747_20160609_103144</td>
            <td>Unapproved or unmaintained equipment or utensils</td>
            <td>Low Risk</td>
        </tr>
        <tr>
            <td>7747</td>
            <td>SAFEWAY STORE #964</td>
            <td>4950 Mission St</td>
            <td>San Francisco</td>
            <td>CA</td>
            <td>94112</td>
            <td>37.720000</td>
            <td>-122.439000</td>
            <td>None</td>
            <td>14155587200</td>
            <td>7747_20160624</td>
            <td>2016-06-24</td>
            <td>0</td>
            <td>Complaint</td>
            <td></td>
            <td></td>
            <td></td>
        </tr>
    </tbody>
</table>



# Note:
## From here, we are using a new dataset, the columns are same, but this dataset have around 10k rows, and also for earlier analysis we have stated that the quering table have a level of granularity "inspection_id", but now from here the level of granularity is "violation_id". Also we removed the rows with same inspection_id according to our need. So, here the Risk category is not assigned to an inspection, here risk category is defined for a particular violation type. 
## Also, I have attached a python file where we loaded the dataset and remove the violations with same inspection_id and created a csv file out of it and loaded it to mysql database. 
# 6: Points to deduct for Different Violations in all Categories

## SQL Queries and Outputs:

**Query 1: High Risk Violations**
```sql

SELECT
    violation_description AS high_risk_violations,
    AVG(inspection_score) AS avg_score_for_this_violation,
    100 - AVG(inspection_score) AS avg_points_deducted
FROM
    sf_restaurant_health_violations_original
WHERE
    risk_category = "High Risk"
    AND violation_description IN (SELECT DISTINCT violation_description FROM sf_restaurant_health_violations)
GROUP BY
    violation_description
ORDER BY
    3 DESC;
```
**Output**

| high_risk_violations                      | avg_score_for_this_violation | avg_points_deducted |
|-------------------------------------------|-----------------------------|---------------------|
| High risk vermin infestation              | 55.8947                     | 44.1053             |
| Unclean hands or improper use of gloves   | 72.2222                     | 27.7778             |
| Improper cooling methods                   | 75.5614                     | 24.4386             |
| No hot water or running water             | 75.6667                     | 24.3333             |
| High risk food holding temperature        | 77.1650                     | 22.8350             |
| Improper reheating of food                 | 85.1429                     | 14.8571             |
| Unclean or unsanitary food contact surfaces| 86.9885                     | 13.0115             |
| Unauthorized or unsafe use of time as a public health control measure | 88.3333 | 11.6667 |
| Contaminated or adulterated food           | 91.0000                     | 9.0000              |


**Query 2: Moderate Risk Violations**

```sql
SELECT
    violation_description AS moderate_risk_violations,
    AVG(inspection_score) AS avg_score_for_this_violation,
    100 - AVG(inspection_score) AS avg_points_deducted
FROM
    sf_restaurant_health_violations_original
WHERE
    risk_category = "Moderate Risk"
    AND violation_description IN (SELECT DISTINCT violation_description FROM sf_restaurant_health_violations)
GROUP BY
    violation_description
ORDER BY
    3 DESC;
```

**Output**

| moderate_risk_violations                      | avg_score_for_this_violation | avg_points_deducted |
|-----------------------------------------------|-----------------------------|---------------------|
| Inadequate sewage or wastewater disposal       | 56.4000                     | 43.6000             |
| Noncompliance with shell fish tags or display  | 76.1667                     | 23.8333             |
| Improper thawing methods                       | 76.4167                     | 23.5833             |
| Foods not protected from contamination         | 84.1856                     | 15.8144             |
| Inadequate food safety knowledge or lack of certified food safety manager | 84.4286 | 15.5714 |
| Moderate risk food holding temperature         | 85.3624                     | 14.6376             |
| Inadequate and inaccessible handwashing facilities | 86.5347                  | 13.4653             |
| Moderate risk vermin infestation               | 86.9901                     | 13.0099             |
| Inadequately cleaned or sanitized food contact surfaces | 88.1091              | 11.8909             |
| Insufficient hot water or running water        | 91.5694                     | 8.4306              |
| Inadequate procedures or records for time as a public health control | 93.4000   | 6.6000              |


**Query 3: Low Risk Violations**

```sql
SELECT
    violation_description AS low_risk_violations,
    AVG(inspection_score) AS avg_score_for_this_violation,
    100 - AVG(inspection_score) AS avg_points_deducted
FROM
    sf_restaurant_health_violations_original
WHERE
    risk_category = "Low Risk"
    AND

 violation_description IN (SELECT DISTINCT violation_description FROM sf_restaurant_health_violations)
GROUP BY
    violation_description
ORDER BY
    3 DESC;
```

**Output**

| low_risk_violations                               | avg_score_for_this_violation | avg_points_deducted |
|---------------------------------------------------|-----------------------------|---------------------|
| Inadequate dressing rooms or improper storage of personal items | 77.5000         | 22.5000             |
| Food safety certificate or food handler card not available | 80.6197                | 19.3803             |
| Other low risk violation                           | 81.3684                     | 18.6316             |
| Unclean or degraded floors walls or ceilings       | 82.2308                     | 17.7692             |
| Improper or defective plumbing                     | 82.4444                     | 17.5556             |
| Improper food storage                              | 86.2581                     | 13.7419            



### WHY ? : So that we can find the inspection score for any restaurant on the basis of the type of violation. Lets see this below:


# 7:  Calculating Inspection Scores for Restaurant Health Violations


## Background

The dataset `sf_restaurant_health_violations_original` contains various columns such as `violation_description`, `inspection_score`, and `risk_category`. The `risk_category` can be 'High Risk', 'Moderate Risk', or 'Low Risk'. 

## Objective

Our goal is to develop a system that can calculate the deduction points and the final inspection score for any given violation.As we stated earlier that we are assuming only one violation per inspection.

## Approach

#### Step 1: Create a View of Violation Points

First, we create a view named `cte_violation_points` that consolidates all types of violations along with the average points to deduct for each violation category.

```sql
CREATE VIEW cte_violation_points AS (
    -- High Risk Violations
    SELECT
        violation_description AS risk_violations,
        100 - AVG(inspection_score) AS points_to_deduct,
        "High Risk" AS RiskCategory
    FROM
        sf_restaurant_health_violations_original
    WHERE
        risk_category = "High Risk"
    GROUP BY
        violation_description
    
    UNION
    
    -- Moderate Risk Violations
    SELECT
        violation_description AS risk_violations,
        100 - AVG(inspection_score) AS points_to_deduct,
        "Moderate Risk" AS RiskCategory
    FROM
        sf_restaurant_health_violations_original
    WHERE
        risk_category = "Moderate Risk"
    GROUP BY
        violation_description
    
    UNION
    
    -- Low Risk Violations
    SELECT
        violation_description AS risk_violations,
        100 - AVG(inspection_score) AS points_to_deduct,
        "Low Risk" AS RiskCategory
    FROM
        sf_restaurant_health_violations_original
    WHERE
        risk_category = "Low Risk"
    GROUP BY
        violation_description
);
```

#### Step 2: Create a Stored Procedure

We then create a stored procedure named `GetDeductionScore` that takes a violation description as input and returns both the corresponding points to deduct and the final inspection score.

```sql
DELIMITER //
CREATE PROCEDURE GetDeductionScore(IN risk_violation_param VARCHAR(255), OUT deduction_score DECIMAL(10,2), OUT inspection_score DECIMAL (10,2))
BEGIN
    SELECT points_to_deduct, 100 - points_to_deduct INTO deduction_score, inspection_score
    FROM cte_violation_points
    WHERE risk_violations = risk_violation_param;
END //
DELIMITER ;
```

#### Step 3: Calculate Deduction and Inspection Score

Finally, we call the stored procedure by providing a specific violation. The stored procedure will then return the deduction points and the final inspection score for the given violation.

```sql
CALL GetDeductionScore('Any Violation', @deduction_score, @inspection_score);
SELECT @deduction_score AS deduction_score, @inspection_score AS inspection_score;
```

### Conclusion

By using a view and stored procedure, we successfully automated the calculation of inspection scores based on restaurant health violations. This system can be utilized by health inspectors to easily calculate the scores for different restaurants based on the violations reported. This not only increases the efficiency but also ensures that the scores are standardized and transparent.

We defined the stored procedure created above in python function, which gives us the inspection score and points deducted for a violation.The input to the function is a violation description.


```python
def get_deduction_score(violation_name):
    result = %sql CALL GetDeductionScore(:violation_name, @deduction_score, @inspection_score); \
                  SELECT @deduction_score as points_deducted, @inspection_score as inspection_score;

    return result

# Ask the user for input and call the function
violation_name = input("Enter the violation name: ")
df_result = get_deduction_score(violation_name)

# Display the result
print(df_result)

```

    Enter the violation name: Inadequate sewage or wastewater disposal
     * mysql://root:***@127.0.0.1:3306/hundred
    1 rows affected.
    1 rows affected.
    +-----------------+------------------+
    | points_deducted | inspection_score |
    +-----------------+------------------+
    |      43.60      |      56.40       |
    +-----------------+------------------+
    


```python

```
