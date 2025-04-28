## 📌 Introduction  
This repository documents an end-to-end sales data analysis, covering:  
- **Data quality assessment** (customer records & transaction validation)  
- **Market performance diagnostics** (regional trends & product analysis)  
- **Dimensional modeling** (star schema implementation)  

---

## 🗂 Repository Structure  

| Section                                                          | Content Overview                                       | Key Deliverables                                             |
| ---------------------------------------------------------------- | ------------------------------------------------------ | ------------------------------------------------------------ |
| **[🧹 Data Quality Analysis](#data-quality-analysis)**           | Investigation of customer records and sales duplicates | Null value handling strategy <br> Deduplication approach     |
| **[📈 Sales Performance Analysis](#sales-performance-analysis)** | Regional and product-level insights                    | Market prioritization framework <br>China B2C expansion plan |
| **[🛠 Data Modeling](#data-modeling)**                           | Star schema implementation details                     | Fact/dimension relationships <br>Bridge table logic          |
# Data Quality Analysis
## Customer Errors
### **Key Findings**  
During data validation, we identified a distinct segment of customer records marked by the "CS" prefix in the `CustomerCode` field, exhibiting null values across three critical attributes:  
- **Occupation**  
- **Education**  
- **Customer Name**  

#### **Pattern Observations**  
1. **100% Null Values**: All 284 records (0.02% of total customers) with the "CS" prefix lack data in these fields.  
2. **Disproportionate Revenue Impact**: Despite representing a small fraction of customer count, these accounts contribute **75% of total revenue** (validated via fact table analysis).  
	![Revenue Distribution](https://i.imgur.com/KuPiLxs.png)  


### **Root Cause Hypothesis**  
- **Non-Standard Customer Type**: The "CS" prefix and missing demographic fields suggest these are **B2B/entity accounts** (e.g., corporate resellers) rather than individual consumers.  
- **Alternative Registration Process**: Likely exempt from providing personal details (occupation/education) during onboarding.  

### **Resolution Strategy**  

#### **1. Customer Name Imputation**  
- **Approach**: Populate nulls with a standardized naming convention:  
  ```python
  df['CustomerName'].fillna('Customer_' + df['CustomerKey'].astype(str), inplace=True)
  ```  
- **Rationale**: Maintains traceability while distinguishing entity accounts.  

#### **2. Occupation & Education Handling**  
- **For Entity Accounts**:  
  - Explicitly label as "B2B" or "Corporate" to preserve data integrity:  
    ```python
    df.loc[df['CustomerCode'].str.startswith('CS'), 'Occupation'] = 'B2B Account'
    df['Education'] = 'Not Applicable'  # Or retain as null if preferred
    ```  
- **For Individual Consumers (if misclassified)**:  
  - Apply predictive imputation based on spending patterns. This involves analyzing the spending patterns associated with each occupation/education and filling the null values with the most relevant or closest occupation/education based on that analysis.  
	![](https://i.imgur.com/foTFj2w.png)
	![](https://i.imgur.com/3OjIauK.png)


#### **3. New Customer Type Field**  
Added to explicitly segment accounts:  

| Field          | Values              | Purpose                                  |     |
| -------------- | ------------------- | ---------------------------------------- | --- |
| `CustomerType` | Individual/Business | Enables targeted analytics and reporting |     |

**Implementation**:  
```python
df['CustomerType'] = np.where(df['CustomerCode'].str.startswith('CS'), 'Business', 'Individual')
```

### **Validation & Next Steps**  
1. **ETL Team Alignment**: Confirm whether "CS" accounts are intentionally exempt from demographic collection.  
2. **Impact Monitoring**: Track revenue attribution post-cleansing to ensure no unintended filtering.  
## Duplicate Records in Sales Table 
During our data quality analysis, we identified **218,008 duplicate rows** in the Sales fact table. Representing approximately 74% of total transactions. This duplication poses significant risks to analytical accuracy and business reporting.
### Root Cause Investigation
Potential sources of duplication:
- **ETL Process Issues**: Double-loading of source data
- **System Integration Errors**: Multiple feeds from POS/ERP systems
- **Lack of Unique Constraint**: Missing primary key enforcement
# Sales Performance Analysis   

## Key Findings  

### Top Customers & Market Trends  
- **Germany**: Our top-performing customers are primarily business clients, indicating strong B2B relationships in this market.  
- **USA**: The majority of our sales revenue originates from the United States, making it our most critical market.  
- **China**: All customers are business clients, with no current presence in the consumer (B2C) market.

### Product Performance  
- **Top-Selling Categories**: Home appliances, computers, and cameras drive the highest revenue, with **home appliances** being the dominant category.  
- **Declining Sales in 2009**: A noticeable decrease in sales occurred in 2009, potentially due to:  
  - **Market Expansion Oversight**: While entering the **China** region in 2009, we may have underprioritized **Germany and the USA**, leading to reduced focus on these key markets.  
  - **Supply Chain Disruptions**: The absence of **top 10 products from 2008** in 2009’s sales suggests possible supply chain issues that require further investigation.  

### China Market Analysis   
- **Actual vs. Predicted Sales**: Performance in China has fallen below projections, indicating an **overly optimistic initial forecast**. Market conditions, competitive landscape, or execution challenges may have contributed to this gap.  
- **Exclusively B2B**: All our customers in China are business clients, indicating we have not yet penetrated the consumer (B2C) market.  
- **Untapped Potential**: The Chinese consumer market represents a significant growth opportunity, particularly for our high-performing categories (home appliances, computers, cameras). 

## Recommended Actions  
1. **Market Strategy Review**:  
   - Reassess resource allocation between **China, Germany, and the USA** to ensure balanced growth.  
   - Strengthen engagement with **German business customers** to maximize high-value sales.  
   - Develop a **dual approach**: Maintain strong B2B relationships while expanding into B2C.  
   - Focus on **urban middle-class consumers**, who show high demand for electronics and smart home products.  

2. **Supply Chain Audit**:  
   - Investigate why previously top-performing products disappeared from 2009 sales.  
   - Assess potential disruptions in procurement, manufacturing, or distribution.  

3. **China Market**:  
   - Conduct a deep dive into **customer demand, pricing, and competition** in China to realign expectations.  
   - Adjust sales and marketing strategies to improve penetration.  
   - Leverage platforms like Alibaba to reach individual buyers.  
   - Utilize WeChat and Douyin (TikTok China) for influencer-driven campaigns.  

4. **Historical Trend Monitoring**:  
   - Track whether the **2009 sales decline** is an isolated event or part of a longer-term trend.  
   - Compare with macroeconomic factors (e.g., recession impacts) to contextualize performance.  
### Next Steps  
- **Data Deep Dive**: Further analyze product-level sales, inventory levels, and supplier performance in 2008–2009.  
- **Stakeholder Alignment**: Present findings to sales, supply chain, and regional teams to develop corrective actions.  

Would you like additional breakdowns by product line or customer segment?

---
# Data modeling 

During data modeling phase we choose star schema here's a detailed breakdown of how each part of the model is structured:
![](https://i.imgur.com/Ft3NdZb.png)

### 1. Fact Table:

- **FactSales**: This is the central fact table, containing the **transactions** or **sales data**. It includes:
    
    - **CustomerKey**: A foreign key linking to the **DimCustomers** table.
        
    - **ProductKey**: A foreign key linking to the **DimProducts** table.
        
    - **Net Price**, **Quantity**, **Sales 2008**, **Sales 2009**: Metrics for the sales in different years.
        
    - **Total Sales**: A computed field representing the total sales amount.
        
    - **OrderDate**: The date when the sale occurred, which will likely connect to a **DimDate** table.
        

This table represents the transactional data at the **granular level**, such as the specific products sold to specific customers in a given year.
### 2. Bridge Tables:

- **BridgeBrand**: This table bridges the relationship between the **Forecast** and **DimProducts** using the **Brand** attribute. It's used to handle different granularities between the tables and ensure that the **brand** data can be used in comparsion between Sales and Forecast.
    
- **BridgeYear**: This table bridges the relationship between **Forecast** and **DimDate** (for time-based analysis) by using the **Year** attribute to manage time-based granularity. 
- **BridgeCountry**: This table bridges the relationship between **Forecast** and **DimGeo** by using the **Country** attribute. To manage Geographic-based granularity between Sales and Forecast.
### 3. Dimension Tables:

- **DimProducts**: This table provides descriptive information about the products, including:
    
    - **ProductKey**: The primary key linking to the **FactSales** table.
        
    - **Brand**, **Category**, **Color**, **Product Name**, **Subcategory**: Attributes of the product.
        
- **DimCustomers**: This table holds customer information, with a key relationship to the **FactSales** table via **CustomerKey**:
    
    - **CustomerKey**: A unique identifier for customers.
        
    - **GeoKey**: Linked to the **DimGeo** table for geographic information.
        
    - **OccEduKey**: Linked to the **DimOcEdu** table for occupation and education details.
        
    - **Name**: Customer name.
        
- **DimDate**: A typical date dimension, holding information about time periods:
    
    - **Date**: The specific date.
        
    - **Year**: Linked to the **BridgeYear** table to allow filtering of sales by year.
        
- **DimGeo**: This dimension table contains geographic details:
    
    - **GeoKey**: A unique key linking to the **DimCustomers** table.
        
    - **City**, **Continent**, **CountryRegion**, **State**: Geographical data for the customer.
- **DimOcEdu**: Contains details about the customer’s occupation and education:
    
    - **Education**, **Occupation**: Categorized details about customer backgrounds.
        
- **DimType**: This table stores the types associated with customers:
    
    - **TypeKey**: A unique key linking to the **DimCustomers** table.
        
    - **Type**: The type attribute (could be customer type, e.g., Individual, business).
### 4. Relationships:

- **FactSales** is connected to the **DimCustomers**, **DimProducts** and **DimDate** tables.
	
- The **BridgeBrand**, **BridgeYear** and **BridgeCountry** tables act as intermediary tables to bridge the relationship between **FactSales** and the **Forecast** table (particularly useful for handling different granularities between brand, years, and country** data).
	
- **DimCustomers** is linked to **DimGeo** (geographical data), **DimOcEdu** (occupation/education data) and **DimType** .
### Conclusion:
This model is designed for performing detailed analysis while optimizing storage efficiency. The dimensional approach allows you to track sales data (through the FactSales table) and slice it across multiple dimensions like product details, customer demographics, geographical data, and time, while minimizing **data duplication** through several key design features:

1. **Normalized Dimension Tables**: Dimensions like DimProduct and DimCustomer follow a star schema approach that reduces redundancy while maintaining analytical flexibility.
    
2. **Bridge Tables**: These play a crucial role in managing many-to-many relationships while avoiding data duplication, ensuring that data at different granularities (like brand and year) can be properly related to the detailed transactional data.
    
3. **Surrogate Keys**: The use of surrogate keys in dimension tables optimizes storage by replacing verbose natural keys with compact integer identifiers.
