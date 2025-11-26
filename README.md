# Nashville Housing Data Cleaning Project

## Project Background

This project focuses on the Nashville residential real estate market, specifically analyzing housing sales data that required significant cleaning and standardization. As a data analyst working with this dataset, the primary objective was to transform raw, inconsistent housing records into a clean, analysis-ready format that could support accurate market insights and business intelligence.

The Nashville housing market encompasses thousands of property transactions with details including sale prices, property addresses, ownership information, and transaction dates. However, like many real-world datasets, this data contained inconsistencies, missing values, and formatting issues that needed to be resolved before any meaningful analysis could be conducted.

**Key data quality improvements focused on:**
- Date format standardization
- Address data normalization and parsing
- Missing value imputation
- Duplicate record identification and removal
- Data type optimization

## Data Structure & Initial Checks

The Nashville Housing database consists of a single primary table containing approximately 56,000+ property transaction records. The table structure includes the following key fields:

**NashvilleHousing Table:**
- **UniqueID**: Unique identifier for each transaction record
- **ParcelID**: Property parcel identification number (used for property matching)
- **PropertyAddress**: Full property address (initially contained combined address information)
- **SaleDate**: Date of property sale (required format conversion)
- **SalePrice**: Transaction sale price
- **LegalReference**: Legal reference number for the transaction
- **SoldAsVacant**: Indicator whether property was sold as vacant (Y/N values)
- **OwnerAddress**: Full owner address (initially contained combined address information)
- **OwnerName**: Name of property owner

**Data Quality Issues Identified:**
- Date fields stored as datetime when only date was needed
- Null values in PropertyAddress field (~5% of records)
- Address fields combining street address, city, and state in single columns
- Inconsistent "Yes/No" encoding (mix of Y/N and Yes/No values)
- Duplicate records based on key property identifiers
- Unused or redundant columns after data transformation

## Executive Summary

### Overview of Findings

This data cleaning project successfully transformed a raw Nashville housing dataset with multiple data quality issues into a standardized, analysis-ready format. The cleaning process addressed five critical data quality dimensions: date standardization, missing value imputation through intelligent matching, address parsing into separate components, categorical value standardization, and duplicate removal. As a result, the dataset is now optimized for downstream analysis including market trend evaluation, price analysis, and geographic insights—enabling stakeholders to make data-driven decisions with confidence in the underlying data integrity.

**Key Transformations Completed:**
- Converted 56,000+ date records to standardized format
- Populated missing property addresses using ParcelID matching logic
- Parsed all address fields into separate street, city, and state components
- Standardized categorical values across the dataset
- Identified and removed duplicate transaction records
- Optimized table structure by removing redundant columns

## Data Cleaning Process Deep Dive

### Category 1: Date Format Standardization

**Issue Identified:** The SaleDate field was stored as a datetime datatype with unnecessary time components, creating storage inefficiency and potential complications for date-based analysis.

**Solution Implemented:** 
- Created a new column `SaleDataConverted` with DATE datatype
- Converted all existing datetime values to date-only format using CONVERT function
- Maintained data integrity by preserving all date information while removing time components

**Impact:** This standardization enables cleaner date-based filtering, grouping, and trend analysis while reducing storage requirements.

### Category 2: Missing Value Imputation

**Issue Identified:** Approximately 5% of records contained NULL values in the PropertyAddress field, which would compromise any location-based analysis or reporting.

**Solution Implemented:**
- Leveraged the business logic that properties with the same ParcelID share the same PropertyAddress
- Used self-join technique to match records with same ParcelID but different UniqueIDs
- Applied ISNULL function to populate missing addresses from matching records
- Successfully populated all NULL PropertyAddress values

**Impact:** Zero records now contain missing property addresses, enabling complete geographic and location-based analysis without data loss.

### Category 3: Address Field Parsing

**Issue Identified:** Both PropertyAddress and OwnerAddress fields contained combined information (street address, city, state) in single columns, limiting the ability to perform city-level or state-level aggregations.

**Solution Implemented:**

**For PropertyAddress:**
- Used SUBSTRING and CHARINDEX functions to split address at comma delimiter
- Created `PropertySplitAddress` column for street address component
- Created `PropertySplitCity` column for city component

**For OwnerAddress:**
- Used PARSENAME function with REPLACE to handle three-part address structure
- Created `OwnerSplitAddress` column for street address
- Created `OwnerSplitCity` column for city
- Created `OwnerSplitState` column for state

**Impact:** Enables granular geographic analysis at street, city, and state levels for both property locations and owner locations, supporting more sophisticated market segmentation.

### Category 4: Categorical Value Standardization

**Issue Identified:** The SoldAsVacant field contained inconsistent encoding with both single-character values ('Y', 'N') and full-word values ('Yes', 'No'), creating challenges for filtering and aggregation.

**Solution Implemented:**
- Analyzed value distribution using COUNT and GROUP BY to identify inconsistencies
- Applied CASE statement to convert all 'Y' values to 'Yes' and 'N' values to 'No'
- Updated all records to maintain consistent categorical encoding

**Impact:** Standardized categorical values improve query readability, prevent grouping errors, and ensure consistent reporting across all analyses.

### Category 5: Duplicate Record Removal

**Issue Identified:** Multiple duplicate records existed in the dataset based on key business identifiers (ParcelID, PropertyAddress, SalePrice, LegalReference), potentially inflating transaction counts and skewing analysis results.

**Solution Implemented:**
- Used ROW_NUMBER() window function with PARTITION BY on key identifying fields
- Created Common Table Expression (CTE) to identify duplicate records
- Deleted records where row_num > 1, retaining only the first occurrence of each unique transaction

**Impact:** Removed duplicate records ensuring accurate transaction counts and preventing double-counting in aggregate calculations and trend analysis.

### Category 6: Table Structure Optimization

**Issue Identified:** After creating parsed address columns, the original combined address columns (PropertyAddress, OwnerAddress, SaleDate) became redundant, unnecessarily increasing table size and query complexity.

**Solution Implemented:**
- Dropped original OwnerAddress and PropertyAddress columns after successful parsing
- Dropped original SaleDate column after successful conversion
- Maintained only cleaned, standardized columns in final table structure

**Impact:** Reduced table size, improved query performance, and simplified future data access by maintaining only necessary, properly formatted columns.

## Recommendations

Based on the data cleaning process and observations, I would recommend the **data engineering and analytics teams** to consider the following:

**Implement upstream data validation:** The presence of missing PropertyAddress values and inconsistent categorical encodings suggests opportunities for data quality checks at the point of data entry. Recommendation: Implement validation rules in source systems to prevent null addresses and enforce consistent categorical value entry.

**Establish data quality monitoring:** Multiple quality issues were identified requiring manual cleaning. Recommendation: Create automated data quality checks that run on new data loads to flag missing values, duplicates, and format inconsistencies before they enter the main database.

**Standardize address data collection:** Address parsing was necessary due to combined field structure. Recommendation: Modify source systems or ETL processes to collect street address, city, and state in separate fields from the start, reducing downstream processing requirements.

**Create data dictionary and lineage documentation:** The cleaning process created multiple new columns and transformations. Recommendation: Maintain comprehensive documentation of all derived fields, transformation logic, and business rules to ensure future analysts understand the data structure and cleaning decisions.

**Schedule regular deduplication processes:** Duplicates were found in the transaction data. Recommendation: Implement regular deduplication jobs as part of the data maintenance cycle to prevent duplicate accumulation over time.

## Assumptions and Caveats

Throughout the analysis, multiple assumptions were made to manage challenges with the data. These assumptions and caveats are noted below:

**Assumption 1:** Properties with the same ParcelID have the same PropertyAddress. This business rule was used to populate missing address values by matching records with identical ParcelIDs. This assumes the ParcelID system is accurate and consistently applied.

**Assumption 2:** When duplicate records were identified, the first occurrence (based on UniqueID ordering) was retained as the accurate record. This assumes that UniqueID ordering reflects the proper chronological or logical sequence of data entry.

**Assumption 3:** Address parsing assumes consistent delimiter usage (commas) in the original address fields. Any addresses with non-standard formatting may have been parsed incorrectly, though visual inspection suggested consistent formatting throughout the dataset.

**Assumption 4:** The cleaning process assumed that 'Y' and 'Yes' values in SoldAsVacant represent the same business meaning (and likewise for 'N' and 'No'). This standardization was applied uniformly across all records.

---

##  Contact & Collaboration
Hi! My name is Mohamed Ahmed.
- 💼 LinkedIn: linkedin.com/in/moe-ahmed-hersi
- Open to feedback, collaboration, and data analytics opportunities!

---

##  Acknowledgments

- Special thanks to **Alex**, also known as **Alex The Analyst** for providing clear, practical guidance on SQL data exploration techniques and demonstrating how to structure professional data analytics projects. His teaching approach helped shape the methodology and best practices applied throughout this analysis.
