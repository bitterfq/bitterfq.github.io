---
title: "Decoupling OpenMetadata's Orchestration: External Airflow Integration in Practice"
date: 2025-11-29T10:00:00-08:00
draft: false
tags: ["openmetadata", "airflow", "data-engineering", "orchestration", "metadata"]
---

Recently, I was tasked to implement an ingestion framework for metadata management that would serve as my organization's centralized data wiki. The system needed to continuously catalog and document our entire data landscape, including data warehouses (PostgreSQL, Snowflake, Redshift, BigQuery), business intelligence dashboards (Tableau, Looker, Metabase, Superset), data pipelines and transformations (dbt models, Airflow DAGs, data quality checks), analytics tables and datasets (fact tables, dimension tables, materialized views), machine learning artifacts (ML models, feature stores, training datasets), API services and microservices, and column-level metadata with lineage tracking. The goal was to build a living, searchable knowledge base where data practitioners could discover datasets, understand dependencies, trace data lineage, and collaborate on data governance.

<!--more-->

This document describes the implementation of an external **Apache Airflow 2.10.4** service as the orchestration backend for **OpenMetadata 1.10.1**. The objective was to separate metadata management from workflow execution, allowing OpenMetadata to remain orchestrator-agnostic while enabling Airflow to operate as an independent, scalable service.

The core principle is straightforward: **OpenMetadata defines *what* to ingest, and Airflow defines *how* to run it.** The integration uses OpenMetadata's `PipelineServiceClient` abstraction, the `AirflowRESTClient` adapter, and the `openmetadata-managed-apis` package installed inside Airflow.

This implementation forms the backbone of a repeatable metadata ingestion and documentation framework that continuously catalogs databases, APIs, and analytical systems across environments.

## 1. Architecture Decision: Why External Orchestration

![OpenMetadata and Airflow Integration Overview](/images/openmetadata-airflow-integration-overview.png)

OpenMetadata ships with an embedded Airflow instance inside the same container as its Java backend. This default design simplifies evaluation but is unsuitable for production due to coupling, scaling, and upgrade limitations.

**Default setup**

```yaml
openmetadata:
  image: openmetadata/server:1.10.1
  # Contains both:
  # - OpenMetadata server (Java)
  # - Embedded Airflow (Python)
```

### Separation of Concerns

Running orchestration logic inside the same runtime as metadata storage violates a clean separation of concerns. OpenMetadata's role is to manage *metadata lifecycles* (connections, entities, lineage) while Airflow's role is *task orchestration* and scheduling. Decoupling them allows each layer to evolve independently:

| Concern                | Owner               | Responsibility                   |
| ---------------------- | ------------------- | -------------------------------- |
| Metadata persistence   | OpenMetadata (Java) | Store entities, serve APIs       |
| Workflow orchestration | Airflow (Python)    | Execute ingestion DAGs           |
| Interface contract     | REST (HTTP)         | Deployment, run, and status APIs |

This boundary isolates operational load and enables independent scaling, upgrades, and maintenance.

---

### Limitations of the Embedded Mode

1. **Resource contention**: JVM and Airflow share CPU and memory, leading to OOM conditions under concurrent ingestion.
2. **Upgrade coupling**: Airflow and OM versions are bound; patching one requires patching the other.
3. **Scaling limitation**: Scaling ingestion requires scaling OM's Java container as well.

**Externalized configuration**

```yaml
services:
  openmetadata:
    image: openmetadata/server:1.10.1
    # Metadata management only

  airflow2:
    build: ./deployments/airflow
    # Orchestration only
    # Integrated via openmetadata-managed-apis
```

The architecture maintains OpenMetadata's orchestration-agnostic design: it issues workflow requests through a defined API, and Airflow acts purely as an executor.

---

## 2. Communication Contract Between OpenMetadata and Airflow

OpenMetadata represents each workflow as an *ingestion pipeline*. Pipelines are deployed and managed via the `PipelineServiceClient` interface; Airflow's concrete implementation is `AirflowRESTClient`.

**Interface**

```java
public interface PipelineServiceClient {
    PipelineServiceClientResponse deployPipeline(IngestionPipeline pipeline);
    PipelineServiceClientResponse runPipeline(String pipelineId, PipelineServiceClientRequest request);
    PipelineServiceClientResponse getStatus(String pipelineId);
    PipelineServiceClientResponse deletePipeline(String pipelineId);
}
```

**AirflowRESTClient implementation**

```java
public class AirflowRESTClient implements PipelineServiceClient {
    private final String airflowEndpoint;  // http://airflow2:8080

    @Override
    public PipelineServiceClientResponse deployPipeline(IngestionPipeline pipeline) {
        POST http://airflow2:8080/api/v1/openmetadata/deploy
    }

    @Override
    public PipelineServiceClientResponse runPipeline(String pipelineId, ...) {
        POST http://airflow2:8080/api/v2/dags/{pipelineId}/dagRuns
    }

    @Override
    public PipelineServiceClientResponse getStatus(String pipelineId) {
        GET http://airflow2:8080/api/v2/dags/{pipelineId}/dagRuns/{runId}
    }
}
```

**Configuration**

```yaml
openmetadata:
  environment:
    PIPELINE_SERVICE_CLIENT_ENABLED: "true"
    PIPELINE_SERVICE_CLIENT_CLASS_NAME: "org.openmetadata.service.clients.pipeline.airflow.AirflowRESTClient"
    PIPELINE_SERVICE_CLIENT_ENDPOINT: "http://airflow2:8080"
    AIRFLOW_USERNAME: "airflow"
    AIRFLOW_PASSWORD: "airflow"
    SERVER_HOST_API_URL: "http://openmetadata:8585/api"
```

OpenMetadata deploys and triggers workflows through Airflow's REST API, while Airflow writes ingestion results back to OpenMetadata via the same network boundary.

---

## 3. The Bridge: openmetadata-managed-apis

Airflow's native REST API can trigger and inspect DAG runs but cannot generate DAGs dynamically. The `openmetadata-managed-apis` plugin extends Airflow's API surface with the ability to receive workflow definitions directly from OpenMetadata.

**Capabilities**

| Endpoint                               | Function                                       |
| -------------------------------------- | ---------------------------------------------- |
| `/api/v1/openmetadata/deploy`          | Accept workflow JSON and generate DAGs         |
| `/api/v1/openmetadata/status/<dag_id>` | Report run status                              |
| DAG templating                         | Generate DAG Python from Jinja2 templates      |
| Workflow factory                       | Support for metadata, profiler, lineage, usage |

**Installation**

```dockerfile
RUN pip install --no-cache-dir \
    'openmetadata-ingestion[postgres]==1.10.1' \
    'openmetadata-managed-apis==1.10.1'
```

**Runtime behavior**

1. OM to Airflow: `POST /api/v1/openmetadata/deploy`
2. Airflow writes `<dag_id>.json` to `/opt/airflow/dag_generated_configs`
3. Airflow renders a Python DAG from a Jinja2 template
4. Airflow's DAG scanner auto registers it

Each generated DAG imports a workflow factory that executes the ingestion logic:

```python
workflow = workflow_factory.WorkflowFactory.create(
    "/opt/airflow/dag_generated_configs/{{ dag_id }}.json"
)
workflow.generate_dag(globals())
dag = workflow.get_dag()
```

---

## 4. File and Volume Architecture

A shared Docker volume handles artifact exchange between containers. OpenMetadata writes configuration files; Airflow reads and executes them.

```yaml
volumes:
  airflow2_dag_generated_configs:
    name: om-airflow2-dag-configs

services:
  openmetadata:
    volumes:
      - airflow2_dag_generated_configs:/opt/airflow/dag_generated_configs

  airflow2:
    volumes:
      - airflow2_dag_generated_configs:/opt/airflow/dag_generated_configs
      - ./dags:/opt/airflow/dags
```

**Permissions**

```dockerfile
RUN mkdir -p /opt/airflow/dag_generated_configs && \
    chown -R airflow:root /opt/airflow/dag_generated_configs && \
    chmod -R 775 /opt/airflow/dag_generated_configs
```

This allows both containers to read/write without resorting to world-writable permissions.

**Execution lifecycle**

1. User defines pipeline in the OM UI.
2. OM generates workflow JSON.
3. OM deploys via `POST /api/v1/openmetadata/deploy`.
4. Airflow generates and loads DAG.
5. OM triggers DAG run via `/api/v2/dags/{uuid}/dagRuns`.
6. Airflow extracts metadata and writes results back to OM.

---

## 5. Implementation Notes

* Communication is fully HTTP based; no shared SDK or dependency coupling.
* Only one extra dependency (`openmetadata-managed-apis`) is required on the Airflow side.
* Shared volumes are sufficient for lightweight deployments; external object storage could replace them later.
* UID/GID alignment across containers avoids permission conflicts.

---

## 6. End-to-End Flow

![OpenMetadata External Airflow Integration Flow](/images/openmetadata-airflow-integration-flow.png)

---

## 7. Outcome

The externalized orchestration model provides a modular and fault-isolated ingestion system. OpenMetadata now delegates all scheduling and task execution to Airflow while maintaining full visibility through REST APIs.

**Key benefits**

| Aspect            | Improvement                                                                           |
| ----------------- | ------------------------------------------------------------------------------------- |
| **Scalability**   | Airflow can scale independently of OM                                                 |
| **Upgradability** | Airflow upgrades no longer blocked by OM releases                                     |
| **Resilience**    | Failures in one service do not affect the other                                       |
| **Extensibility** | Other orchestrators (Argo, Prefect, Temporal) can plug in via `PipelineServiceClient` |

**Data assets overview after ingestion**

![OpenMetadata Data Assets Dashboard](/images/openmetadata-data-assets-dashboard.png)

This design establishes a repeatable, orchestrator-agnostic ingestion layer capable of supporting continuous metadata synchronization across diverse systems.

---

## 8. Summary

**Implemented:**

* External Airflow 2.10.4 backend
* `PipelineServiceClient` integration via `AirflowRESTClient`
* `openmetadata-managed-apis` plugin for DAG deployment
* Shared Docker volume for workflow configs
* HTTP only communication boundary

The result is a cleanly decoupled metadata ingestion framework: stable, scalable, and portable across orchestration systems. It adheres to the separation of concerns principle and enables OpenMetadata to operate as a true metadata service rather than an orchestrator runtime hybrid.
