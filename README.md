# ESG Data Management Implementation Plan

## Overview

This document outlines the data management approach for the ESG Reporting System. The plan ensures data integrity while providing efficient access patterns and minimal file structure complexity.

## 1. Data Organization Strategy

Our streamlined approach focuses on:

1. **Master Dataset with Index Files** - One centralized CSV with supporting index files
2. **Year-Based Access** - Separate files by reporting year for efficient retrieval 
3. **JSON Indexes** - Fast lookup without excessive file fragmentation

## 2. File Structure

```
app/processed/
├── company_industry_mapping.csv       # Company to industry mapping
├── esg_metrics_master.csv             # Complete dataset (all metrics)
├── company_years_index.json           # Available years by company
├── company_metrics_index.json         # Available metrics by company and year
├── industry_metrics_index.json        # Available metrics by industry
└── by_year/                           # Year-based data files
    ├── 2020.csv                       # All metrics for 2020
    ├── 2021.csv                       # All metrics for 2021
    └── 2022.csv                       # All metrics for 2022
```

## 3. Index File Structure

### company_years_index.json
```json
{
  "TSMC": [2022, 2021, 2020],
  "Intel": [2022, 2021, 2020],
  "JPMorgan Chase": [2022, 2021]
}
```

### company_metrics_index.json
```json
{
  "TSMC": {
    "2022": ["CO2DIRECTSCOPE1", "ENERGYUSETOTAL", "WATERWITHDRAWALTOTAL"],
    "2021": ["CO2DIRECTSCOPE1", "ENERGYUSETOTAL"]
  },
  "Intel": {
    "2022": ["CO2DIRECTSCOPE1", "ENERGYUSETOTAL"]
  }
}
```

### industry_metrics_index.json
```json
{
  "Semiconductor": [
    "ANALYTICWASTERECYCLINGRATIO",
    "CO2DIRECTSCOPE1",
    "CO2INDIRECTSCOPE2",
    "CO2INDIRECTSCOPE3",
    "ENERGYUSETOTAL"
  ],
  "Commercial Bank": [
    "CO2DIRECTSCOPE1",
    "ESGCreditAnalysis",
    "GRIEVANCE_REPORTING_PROCESS"
  ]
}
```

## 4. Data Integrity Measures

1. **Standardized Company Names**
   - All company names are standardized based on the company mapping file
   - Ensures consistent references across all data files

2. **Consistent Industry Classification**
   - Industries are normalized to standard categories (Semiconductor, Commercial Bank)
   - Facilitates accurate industry-specific reporting

3. **Data Validation**
   - Type conversion and validation during processing
   - Boolean values standardized to 1/0
   - Numerical values properly formatted

4. **Default Values**
   - Missing fields receive appropriate default values
   - Year defaults to 2022 if not specified
   - Units standardized when not provided

5. **Metric Name Standardization**
   - Consistent metric naming across all files
   - Facilitates reliable lookup and integration with knowledge graph

## 5. Retrieval Efficiency

### A. Company Context After Login

When a user logs in as a company representative:

1. The system identifies the company from the authentication service
2. Loads company industry information from `company_industry_mapping.csv`
3. Retrieves available reporting years from `company_years_index.json`
4. Populates the year selector dropdown with available years

### B. Year Selection

When a user selects a reporting year:

1. The system loads the appropriate year file from `by_year/{year}.csv`
2. Filters for the specific company
3. Retrieves available metrics for that company-year combination from `company_metrics_index.json`
4. Populates the metric selector with available metrics

### C. Framework and Category Context

1. The system uses the company's industry to identify applicable frameworks from the knowledge graph
2. Framework-specific categories and metrics are filtered based on `industry_metrics_index.json`
3. Only metrics that exist in both the knowledge graph and the data files are presented to the user

### D. Metric Calculation

1. When calculating metrics, the system first checks if the metric is directly available in the year file
2. If calculation is needed, the system efficiently retrieves required datapoints from the year file
3. Calculated metrics are cached to improve subsequent access

## 6. Implementation Steps

1. **Data Processing**
   - Process the Eurofidai company mapping file
   - Combine all ESG data files into a master dataset
   - Create year-based files and indexes
   - Generate sample data when real data is insufficient

2. **Data Access Services**
   - Implement efficient data connector service using indexes
   - Cache frequently accessed data
   - Implement pagination for large datasets

3. **User Interface**
   - Dynamic year selection based on available years for company
   - Dynamic metric selection based on available metrics for company-year
   - Clear indication of metric availability

4. **Reporting**
   - Utilize the optimized data structure for fast report generation
   - Include data lineage information in reports

## 7. Performance Considerations

1. **Minimal File Count**
   - Reduces system overhead and maintenance complexity
   - Simplifies backup and restoration processes

2. **Optimized Lookups**
   - JSON indexes provide O(1) lookup for common operations
   - Year-based files ensure efficient filtering

3. **Memory Efficiency**
   - Only load data for the selected year
   - Avoid loading entire master dataset when unnecessary

4. **Calculation Efficiency**
   - Reuse calculated metrics within a user session
   - Store calculation results for reporting

## 8. Extensibility

The data management approach is designed to be extensible:

1. **New Companies**
   - Can be added to the company mapping file
   - No changes needed to the data structure

2. **New Metrics**
   - Can be added to the master dataset
   - Index files automatically updated during processing

3. **New Years**
   - Additional year files can be added without modifying existing files
   - Index files updated to include new years

4. **New Industries**
   - Can be added to the company mapping file
   - Industry metrics index will automatically include new industries

This implementation plan ensures data integrity while providing efficient access patterns, supporting the ESG reporting system's requirements while maintaining a clean and manageable file structure.
