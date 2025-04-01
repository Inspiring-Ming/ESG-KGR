# ESG-KGR
# Deployment and Testing Guide

This guide covers how to deploy the complete ESG-KGR system using Docker Compose and test its functionality.

## Prerequisites

- Docker and Docker Compose installed
- All services built and ready for deployment
- ESG ontology and sample data files prepared

## Deployment Steps

### 1. Prepare Environment

Create a deployment directory:

```bash
mkdir -p esg-kgr-deploy
cd esg-kgr-deploy
```

### 2. Copy Docker Compose File

Copy the Docker Compose file from the development environment:

```bash
cp /path/to/docker-compose.yml .
```

### 3. Create Data Directories

Create directories for persistent data:

```bash
mkdir -p fuseki-data
mkdir -p ontology
mkdir -p data
```

### 4. Copy Ontology File

Copy the ESG ontology file to the ontology directory:

```bash
cp /path/to/esg-ontology.ttl ontology/
```

### 5. Prepare Sample Eurofidai Data

Create a sample Eurofidai CSV file for testing:

```bash
cat > data/eurofidai-sample.csv << 'EOL'
COMPANY_NAME,METRIC_NAME,METRIC_VALUE,METRIC_UNIT,REPORTED_DATE,DISCLOSURE
Semiconductor Inc.,CO2DIRECTSCOPE1,7632,tCO2e,2023-01-01,Mandatory
Semiconductor Inc.,CO2INDIRECTSCOPE2,23145,tCO2e,2023-01-01,Mandatory
Semiconductor Inc.,CO2INDIRECTSCOPE3,54320,tCO2e,2023-01-01,Voluntary
Semiconductor Inc.,ENERGYUSETOTAL,210000,MWh,2023-01-01,Mandatory
Semiconductor Inc.,WATERWITHDRAWALTOTAL,750000,m3,2023-01-01,Mandatory
EOL
```

### 6. Start the System

Start all services using Docker Compose:

```bash
docker-compose up -d
```

This command will:
- Pull or build all required images
- Create and start containers for each service
- Set up the necessary network connections

### 7. Monitor Deployment

Check the status of the services:

```bash
docker-compose ps
```

Check logs for any errors:

```bash
docker-compose logs
```

To follow logs from a specific service:

```bash
docker-compose logs -f knowledge-graph-service
```

### 8. Load Ontology into Fuseki

Once the Fuseki service is running:

1. Open the Fuseki admin interface at http://localhost:3030/
2. Log in with username `admin` and password `admin123`
3. Create a new dataset named `esg`
4. Upload the ontology file using the "upload data" tab

### 9. Import Sample Data

To import the sample data through the API:

```bash
curl -X POST -F "file=@data/eurofidai-sample.csv" http://localhost:8080/api/data/eurofidai/upload
```

## Testing the System

### Testing Backend Services

#### 1. Test Knowledge Graph Service

```bash
# Get available frameworks
curl http://localhost:8080/api/frameworks

# Expected response:
# [{"framework":"http://example.org/esg/IFRS_S2","label":"IFRS S2"},{"framework":"http://example.org/esg/TCFD","label":"TCFD"}]
```

#### 2. Test Reasoning Service

```bash
# Get material categories for semiconductor industry and IFRS S2 framework
curl "http://localhost:8080/api/categories?industry=http://example.org/esg/SemiconductorIndustry&framework=http://example.org/esg/IFRS_S2"

# Expected response:
# [{"category":"http://example.org/esg/GHGEmissions","label":"GHG Emissions"},{"category":"http://example.org/esg/EnergyManagement","label":"Energy Management"},{"category":"http://example.org/esg/WaterWastewaterManagement","label":"Water & Wastewater Management"},{"category":"http://example.org/esg/WasteHazardousMaterials","label":"Waste & Hazardous Materials Management"}]
```

#### 3. Test Category-Metric Relationship

```bash
# Get metrics for GHG Emissions category
curl "http://localhost:8080/api/categories/metrics?category=http://example.org/esg/GHGEmissions"

# Expected response:
# [{"metric":"http://example.org/esg/GHGEmission","label":"GHG Emission"}]
```

#### 4. Test Metric-Model Relationship

```bash
# Get models for GHG Emission metric
curl "http://localhost:8080/api/metrics/models?metric=http://example.org/esg/GHGEmission"

# Expected response:
# [{"model":"http://example.org/esg/ModelGHGEmission","label":"GHG Emission Model"}]
```

#### 5. Test Model Inputs

```bash
# Get required inputs for GHG Emission model
curl "http://localhost:8080/api/models/inputs?model=http://example.org/esg/ModelGHGEmission"

# Expected response:
# [{"input":"http://example.org/esg/Scope1Emission","label":"Scope 1 Emission"},{"input":"http://example.org/esg/Scope2Emission","label":"Scope 2 Emission"},{"input":"http://example.org/esg/Scope3Emission","label":"Scope 3 Emission"}]
```

#### 6. Test Calculation

```bash
# Calculate GHG Emissions
curl -X POST -H "Content-Type: application/json" -d '["http://example.org/esg/Scope1Emission","http://example.org/esg/Scope2Emission"]' "http://localhost:8080/api/metrics/calculate?model=http://example.org/esg/ModelGHGEmission&company=http://example.org/esg/eurofidai/company/Semiconductor_Inc"

# Expected response (will vary based on data):
# {"value":30777.0,"unit":"tCO2e","inputs_used":["http://example.org/esg/Scope1Emission","http://example.org/esg/Scope2Emission"],"calculation_method":"sum","model":"http://example.org/esg/ModelGHGEmission","company":"http://example.org/esg/eurofidai/company/Semiconductor_Inc"}
```

### Testing Frontend Application

#### 1. Access Frontend

Open a web browser and navigate to http://localhost:3000/

#### 2. Complete Workflow Testing

Test the complete end-to-end workflow:

1. Select reporting year: 2023
2. Select reporting framework: IFRS S2
3. Select category: GHG Emissions
4. Select metric: GHG Emission
5. Select model: GHG Emission Model
6. Select inputs: Check "Scope 1 Emission" and "Scope 2 Emission"
7. Click "Calculate" button
8. Verify the calculation result appears in the right panel
9. Click "Generate Report" button
10. Verify report preview information is correct

## Troubleshooting

### Service Connection Issues

If services cannot connect to each other:

1. Check the network configuration:
   ```bash
   docker network inspect esg-network
   ```

2. Verify service names are correctly specified in connection URLs:
   ```bash
   docker-compose exec orchestrator-service cat /app/application.properties
   ```

### Data Loading Issues

If data is not appearing correctly:

1. Check Fuseki dataset content:
   - Open http://localhost:3030/dataset.html?tab=query&ds=/esg
   - Run a simple query: `SELECT * WHERE { ?s ?p ?o } LIMIT 100`

2. Check data service logs:
   ```bash
   docker-compose logs data-service
   ```

### API Error Responses

If API calls return errors:

1. Check service logs for detailed error messages:
   ```bash
   docker-compose logs api-gateway
   docker-compose logs orchestrator-service
   ```

2. Verify service health:
   ```bash
   curl http://localhost:8081/actuator/health
   curl http://localhost:8082/actuator/health
   ```

## Monitoring and Maintenance

### Health Checks

Monitor service health:

```bash
# Check specific service health
curl http://localhost:8081/actuator/health

# Check disk space
docker system df
```

### Backup and Restore

#### Backup Fuseki Data

```bash
# Stop Fuseki service
docker-compose stop fuseki

# Backup data directory
tar -czvf fuseki-backup-$(date +%Y%m%d).tar.gz fuseki-data/

# Restart Fuseki service
docker-compose start fuseki
```

#### Restore Fuseki Data

```bash
# Stop Fuseki service
docker-compose stop fuseki

# Extract backup file
tar -xzvf fuseki-backup-20230101.tar.gz

# Restart Fuseki service
docker-compose start fuseki
```

### System Updates

To update the system:

```bash
# Pull latest code
git pull

# Rebuild services
./build-all.sh

# Restart with updated images
docker-compose down
docker-compose up -d
```

## Security Considerations

For production deployment, consider these additional security measures:

1. **Enable Authentication**:
   - Configure Spring Security for all services
   - Implement OAuth2 or JWT authentication

2. **Secure Fuseki**:
   - Change default admin password
   - Configure HTTPS
   - Restrict access to authorized IPs

3. **Data Encryption**:
   - Enable TLS for all service connections
   - Encrypt sensitive data at rest

4. **Network Segmentation**:
   - Use Docker networks to isolate services
   - Implement a proper reverse proxy (like Traefik or NGINX) with rate limiting

5. **Logging and Monitoring**:
   - Implement centralized logging (ELK stack or similar)
   - Set up alerts for unusual activity
