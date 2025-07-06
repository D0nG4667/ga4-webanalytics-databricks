# GA4 Web Analytics Data Pipeline to Azure Data Lake Storage Gen2

## 📊 Project Overview

This project implements a robust, production-ready data pipeline that extracts web analytics data from Google Analytics 4 (GA4) and ingests it into Azure Data Lake Storage Gen2 (ADLS Gen2). The solution is designed for enterprise-scale data engineering workflows and supports automated data collection for downstream analytics, reporting, and machine learning applications.

## Architecture

```text
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────────┐
│   Google GA4    │    │    Python ETL    │    │   Azure ADLS Gen2   │
│   Analytics API │───▶│    Pipeline      │───▶│   Data Lake        │
└─────────────────┘    └──────────────────┘    └─────────────────────┘
                              │
                              ▼
                       ┌──────────────────┐
                       │   Local CSV      │
                       │   (Optional)     │
                       └──────────────────┘
```

## 🔧 Technical Stack

- **Python 3.13+**
- **Google Analytics Data API v1 Beta**
- **Azure Data Lake Storage Gen2**
- **Pandas** for data transformation
- **OAuth2 Service Account Authentication**
- **Environment-based Configuration**

## 📋 Prerequisites

### 1. Google Analytics 4 Setup

- GA4 property with active data collection
- Google Cloud Project with Analytics Data API enabled
- Service Account with GA4 read permissions
- Downloaded service account JSON key file

### 2. Azure Infrastructure

- Azure Storage Account with ADLS Gen2 enabled
- Storage account access key
- Container/file system created (`ga4-data`)
- **Important**: Disable Blob Storage Events and Soft Delete features

### 3. Development Environment

- Python 3.13 or higher
- VS Code with Jupyter extension
- Git for version control
- [uv](https://github.com/uv-mamba/uv) (for fast installs)

## 🚀 Installation & Setup

### 1. Clone Repository

```bash
git clone <repository-url>
cd web-analytics-ga4
```

### 2. Install Dependencies

```bash
pip install google-analytics-data azure-storage-file-datalake pandas python-dotenv
```

Ensure you have `uv` installed for fast dependency management **(Recommended)**:

```bash
uv pip install -r pyproject.toml
```

### 3. Configure Environment Variables

Create a `.env` file in the project root:

```env
# Azure Data Lake Storage Gen2 Configuration
ADLS_ACCOUNT_NAME=your_storage_account_name
ADLS_ACCOUNT_KEY=your_storage_account_key

# Optional: Additional configurations
GA4_PROPERTY_ID=your_ga4_property_id
```

### 4. Service Account Setup

1. Place your Google service account JSON file in the project directory
2. Rename it to `credentials.json` (or update the path in the notebook)
3. Ensure proper file permissions (readable by application only)

## 📊 Data Schema

### Dimensions Collected

- **date**: Event date (YYYYMMDD format)
- **country**: User's country
- **deviceCategory**: Desktop, Mobile, Tablet
- **browser**: Browser type
- **eventName**: GA4 event name

### Metrics Collected

- **activeUsers**: Count of active users
- **engagedSessions**: Number of engaged sessions
- **engagementRate**: Session engagement rate
- **averageSessionDuration**: Average session length
- **userEngagementDuration**: Total user engagement time
- **bounceRate**: Percentage of single-page sessions
- **eventsPerSession**: Average events per session
- **screenPageViews**: Total page/screen views
- **screenPageViewsPerSession**: Average views per session
- **newUsers**: Count of new users

## 🔄 Pipeline Workflow

### 1. **Authentication**

- Loads service account credentials
- Establishes secure connection to GA4 API

### 2. **Data Extraction**

- Configures report request with dimensions and metrics
- Retrieves last 7 days of data (configurable)
- Handles API pagination automatically

### 3. **Data Transformation**

- Converts API response to structured Pandas DataFrame
- Applies consistent column naming
- Validates data types and formats

### 4. **Data Loading**

- Serializes DataFrame to CSV format in memory
- Uploads to specified ADLS Gen2 path
- Supports file overwrite for incremental loads

## 📁 File Structure

```
web-analytics-ga4/
├── API for GA4.ipynb           # Main pipeline notebook
├── credentials.json            # Google service account key
├── .env                       # Environment configuration
├── ga4_ingestion_output.csv   # Local output (optional)
├── pyproject.toml            # Python project configuration
├── uv.lock                   # Dependency lock file
├── README.md                 # This documentation
└── WEB ANALYTICS ON DATABRICKS & Cloud Storage.pptx
```

## 🚀 Usage

### Running the Pipeline

1. **Open the Jupyter Notebook**

   ```bash
   code "API for GA4.ipynb"
   ```

2. **Execute Cells Sequentially**
   - Cell 1: Verify working directory
   - Cell 2: Install required packages
   - Cell 3: Run complete data pipeline

3. **Monitor Output**
   - Check for successful authentication
   - Verify data extraction metrics
   - Confirm ADLS Gen2 upload completion

### Scheduling for Production

For automated execution, consider:

```python
# Add to your scheduler (Airflow, Azure Data Factory, etc.)
def run_ga4_pipeline():
    exec(open('API for GA4.ipynb').read())
```

## ⚙️ Configuration Options

### Date Range Customization

```python
date_ranges=[DateRange(start_date="30daysAgo", end_date="yesterday")]
```

### Custom Dimensions/Metrics

```python
# Add custom dimensions
dimensions=[
    Dimension(name="sessionSource"),
    Dimension(name="sessionMedium"),
    # ... existing dimensions
]

# Add custom metrics  
metrics=[
    Metric(name="conversions"),
    Metric(name="totalRevenue"),
    # ... existing metrics
]
```

### Output Path Configuration

```python
ADLS_FILE_PATH = "ga4/ga4_ingestion_output.csv.csv"
ADLS_FILE_PATH = "analytics/ga4/daily/ga4_data_{date}.csv" **(Recommended)**
```

## 🛠️ Troubleshooting

### Common Issues

#### 1. Authentication Errors

```shell
ClientAuthenticationError: Server failed to authenticate
```

**Solution**: Verify ADLS account key and name in `.env` file

#### 2. Endpoint Compatibility Error

```shell
EndpointUnsupportedAccountFeatures
```

**Solution**: Disable Blob Storage Events and Soft Delete in Azure Storage Account

#### 3. GA4 API Quota Exceeded

```shell
QuotaExceeded: Analytics quota exceeded
```

**Solution**: Implement exponential backoff or reduce request frequency

#### 4. Missing Container Error

```shell
ResourceNotFound: The specified container does not exist
```

**Solution**: Create the `ga4-data` container in your storage account

### Debug Mode

Enable verbose logging by adding:

```python
import logging
logging.basicConfig(level=logging.DEBUG)
```

## 🔒 Security Best Practices

1. **Environment Variables**: Never commit `.env` files to version control
2. **Service Account Keys**: Store securely, rotate regularly
3. **Access Keys**: Use Azure Key Vault for production deployments
4. **Network Security**: Implement VNet integration for Azure resources
5. **Data Encryption**: Ensure encryption at rest and in transit

## 📈 Performance Optimization

### For Large Data Volumes

- Implement batch processing for date ranges
- Use parallel processing for multiple GA4 properties
- Consider data partitioning by date/region
- Implement incremental loading strategies

### Memory Management

```python
# For large datasets, use chunked processing
for chunk in pd.read_csv(large_file, chunksize=10000):
    process_chunk(chunk)
```

## 🔄 Data Quality & Monitoring

### Recommended Validations

- Row count validation
- Data freshness checks
- Schema drift detection
- Null value monitoring

### Monitoring Dashboard Metrics

- Pipeline execution time
- Data volume trends
- Error rates
- API quota utilization

## 🎯 Next Steps & Extensions

1. **Real-time Streaming**: Implement with Azure Event Hubs
2. **Data Warehouse Integration**: Connect to Azure Synapse Analytics
3. **ML Pipeline**: Build predictive models on the collected data
4. **Alerting**: Implement Azure Monitor alerts for pipeline failures
5. **Multi-tenant**: Extend for multiple GA4 properties

## 📝 Change Log

| Version | Date       | Changes |
|---------|------------|---------|
| 1.0.0   | 2025-07-06 | Initial release with basic GA4 to ADLS pipeline |

## 👥 Contributing

1. Fork the repository
2. Create a feature branch
3. Implement changes with tests
4. Submit a pull request

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 📞 Support

For technical issues or questions:

- Create an issue in the repository
- Contact the data engineering team
- Review Azure and Google Cloud documentation

---

## 👥 Authors

🕺🏻Made with ❤️ **Gabriel Okundaye**

- GitHub: [GitHub Profile](https://github.com/D0nG4667)

- LinkedIn: [LinkedIn Profile](https://www.linkedin.com/in/dr-gabriel-okundaye)
