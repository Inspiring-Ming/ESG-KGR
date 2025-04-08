# ESG-KGR System

**ESG-KGR** is a knowledge graph reasoning framework for automated sustainability reporting that addresses complex semantic challenges in ESG reporting. The system enables in-depth reasoning across ESG entities and relationships through specialized microservices.

---

## Architecture

The system follows a microservices architecture with the following components:

- **Client Interface**: Frontend React application  
- **API Gateway**: Entry point for all requests  
- **Orchestrator Service**: Coordinates workflow across services  
- **Query Service**: Translates client requests to knowledge graph queries  
- **Reasoning Services**: Four specialized reasoning services (ontological, rule-based, path-based, constraint-based)  
- **Knowledge Graph Service**: Manages semantic data model  
- **Data Service**: Handles data access and calculation  
- **Result Repository**: Stores calculation results and workflow context  
- **Report Service**: Generates reports based on templates  

---

## Getting Started

### Prerequisites

- Node.js (v14 or higher)  
- npm (v6 or higher)  

### Installation

Clone the repository:

```bash
git clone https://github.com/Inspiring-Ming/esg-kgr-system.git
cd esg-kgr-system
```

Install dependencies for all services:

```bash
./start-services.sh
```

This will start all the services at once.

Alternatively, you can start each service individually:

```bash
# Knowledge Graph Service
cd services/knowledge-graph-service && npm install && PORT=3003 node server.js

# Query Service
cd services/query-service && npm install && PORT=3004 node server.js

# Reasoning Service
cd services/reasoning-service && npm install && PORT=3005 node server.js

# Data Service
cd services/data-service && npm install && PORT=3006 node server.js

# Result Repository Service
cd services/result-repository-service && npm install && PORT=3007 node server.js

# Report Service
cd services/report-service && npm install && PORT=3008 node server.js

# Orchestrator Service
cd services/orchestrator-service && npm install && PORT=3002 node server.js

# API Gateway
cd services/api-gateway && npm install && PORT=3001 node server.js

# Client Interface
cd services/client-interface && npm install && npm start
```

---

## Usage

1. Open your browser and navigate to `http://localhost:3000`
2. Follow the workflow steps to create an ESG report:
   - Select industry (Semiconductor for this demonstration)
   - Select reporting framework (IFRS S2)
   - Select material category (GHG Emissions, Energy Management, etc.)
   - Select metric (GHGEmissionMetric, etc.)
   - Select calculation model (GHGEmissionModel, etc.)
   - Select inputs (Scope1Emission, Scope2Emission, etc.)
   - Execute calculation
   - Generate report

---

## API Documentation

### API Gateway Endpoints

- `/api/workflows/*` - Orchestrator service endpoints
- `/api/query/*` - Query service endpoints
- `/api/reasoning/*` - Reasoning service endpoints
- `/api/data/*` - Data service endpoints
- `/api/calculation/*` - Calculation endpoints
- `/api/results/*` - Result repository endpoints
- `/api/context/*` - Workflow context endpoints
- `/api/reports/*` - Report service endpoints

### Reasoning Service Endpoints

- `/api/reasoning/ontological/frameworks?industry={id}` - Get frameworks for an industry
- `/api/reasoning/ontological/equivalence?source={id}&target={id}` - Get equivalent concepts
- `/api/reasoning/rule-based/categories?industry={id}&framework={id}` - Get material categories
- `/api/reasoning/rule-based/required?industry={id}&framework={id}` - Get required metrics
- `/api/reasoning/path-based/metrics?category={id}` - Get metrics for a category
- `/api/reasoning/path-based/lineage?metric={id}` - Get data lineage for a metric
- `/api/reasoning/path-based/models?metric={id}` - Get models for a metric
- `/api/reasoning/constraint-based/inputs?model={id}` - Get required inputs for a model
- `/api/reasoning/constraint-based/validate-inputs` - Validate inputs for a model

---

## Conclusion

This implementation guide provides a complete implementation of the ESG-KGR system as described in the paper. The system uses a microservices architecture to enable in-depth reasoning across ESG entities and relationships, allowing for a complete ESG reporting workflow from framework selection to metric calculation and report generation.

Key features of the implementation include:

- Four specialized reasoning services (ontological, rule-based, path-based, constraint-based) that enable different aspects of ESG reporting
- A knowledge graph service that provides semantic relationships between ESG entities
- A data service that handles calculation execution and data access
- A result repository service that maintains workflow context and calculation results
- A report service that generates standardized reports based on templates
- An orchestrator service that coordinates the workflow across services
- A client interface that guides users through the reporting process

The implementation follows the service-oriented architecture described in the paper, with well-defined APIs and service boundaries. Each service is independently deployable and scalable, with clear responsibilities and interfaces.

By following this implementation guide, you can set up a complete ESG reporting system that addresses the semantic challenges in ESG reporting while providing a flexible and extensible foundation for future enhancements.

