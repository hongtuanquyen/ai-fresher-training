# Data-intensive Backend — Tech Stack Reference

## Đặc điểm archetype
- Xử lý lượng data lớn: ETL, aggregation, analytics, reporting
- ML model serving (inference API)
- Search platform (full-text, vector, faceted)
- Time-series data (metrics, IoT sensor, financial tick)
- Heavy read hoặc heavy write (hoặc cả hai)
- Batch processing + stream processing
- Data pipeline: ingest → transform → store → serve
- Database choice là quyết định quan trọng nhất

## Stack combinations đã curate

### Combo 1: FastAPI + PostgreSQL + Celery + Redis (Python — ML/AI serving)
- **Runtime**: Python 3.12+
- **Framework**: FastAPI (async, type-safe)
- **ML**: PyTorch / TensorFlow / scikit-learn / Hugging Face Transformers
- **Model serving**: FastAPI endpoint / TorchServe / Triton (nếu cần GPU optimize)
- **Database**: PostgreSQL (metadata, user) + S3 (model artifacts, dataset)
- **Vector DB** (nếu RAG/similarity): pgvector (PostgreSQL extension) / Qdrant / Weaviate
- **Queue**: Celery + Redis (async inference, training job)
- **Cache**: Redis (model prediction cache, rate limit)
- **Monitoring**: Prometheus + Grafana (model latency, GPU util)
- **Deploy**: Docker + GPU instance (AWS g5, GCP A100) / Modal / Replicate
- **Khi nào**: ML inference API, RAG application, AI-powered feature, team Python + ML
- **Khi nào không**: không có ML component, cần low-latency non-ML API

### Combo 2: Node.js/Python + ClickHouse + Kafka (Analytics platform)
- **Ingestion**: Kafka (event stream từ nhiều source)
- **Processing**: Kafka Streams / Flink / custom consumer (Node.js/Python)
- **OLAP database**: ClickHouse (columnar, aggregate query cực nhanh, billions row)
- **OLTP database**: PostgreSQL (metadata, user, config)
- **API**: FastAPI hoặc Fastify (serve dashboard query)
- **Cache**: Redis (pre-computed dashboard)
- **Visualization**: expose API → Grafana / Metabase / custom dashboard (FE)
- **Deploy**: Docker Compose → K8s (production scale)
- **Khi nào**: product analytics, log analytics, ad-tech, financial reporting, billions of events
- **Khi nào không**: <1M rows (PostgreSQL đủ), OLTP workload

### Combo 3: Python + Airflow/Dagster + PostgreSQL + S3 (ETL / Data Pipeline)
- **Orchestrator**: Apache Airflow (mature, community lớn) hoặc Dagster (modern, asset-based)
- **Processing**: Pandas / Polars (single machine) / PySpark / DuckDB (embedded OLAP)
- **Storage**: S3 / GCS (data lake) + PostgreSQL (metadata, result) + Parquet files
- **Transformation**: dbt (SQL transform trong warehouse)
- **Warehouse**: BigQuery / Snowflake / ClickHouse / DuckDB (nhỏ)
- **Monitoring**: Airflow UI / Dagster UI + Sentry
- **Deploy**: Docker → K8s (Airflow) / Dagster Cloud
- **Khi nào**: data pipeline định kỳ (hourly/daily), ETL từ nhiều source, reporting warehouse
- **Khi nào không**: real-time stream (dùng Kafka + Flink thay), simple cron job (overkill)

### Combo 4: Go/Rust + TimescaleDB (Time-series)
- **Runtime**: Go 1.23+ hoặc Rust
- **Framework**: Chi/Echo (Go) hoặc Actix/Axum (Rust)
- **Database**: TimescaleDB (PostgreSQL extension cho time-series) hoặc InfluxDB
- **Ingestion**: direct HTTP / MQTT (IoT) / Kafka
- **Cache**: Redis (latest value cache)
- **Query**: Continuous aggregate (TimescaleDB) / materialized view
- **Monitoring**: Grafana (native TimescaleDB datasource)
- **Deploy**: Docker → VPS / K8s
- **Khi nào**: IoT sensor data, infrastructure metrics, financial tick data, cần query time-range nhanh
- **Khi nào không**: general CRUD (PostgreSQL thường đủ)

### Combo 5: Typesense/MeiliSearch + PostgreSQL (Search platform)
- **Search engine**: Typesense (typo-tolerant, faceted, fast) hoặc MeiliSearch (developer-friendly)
- **Primary DB**: PostgreSQL (source of truth)
- **Sync**: Change Data Capture (Debezium) hoặc application-level sync
- **API**: Fastify / FastAPI (wrapper hoặc search proxy)
- **Khi nào**: product search, document search, autocomplete, filter+sort phức tạp
- **Khi nào không**: simple LIKE query (PostgreSQL đủ), <10k document

### Combo 6: Elasticsearch + Logstash + Kibana (ELK — Log/Search)
- **Search/Analytics**: Elasticsearch 8 (hoặc OpenSearch)
- **Ingestion**: Logstash / Filebeat / Fluentd
- **Visualization**: Kibana / OpenSearch Dashboards
- **Primary DB**: PostgreSQL
- **Khi nào**: log aggregation, full-text search phức tạp (synonym, stemming, nested), APM
- **Khi nào không**: simple search (Typesense nhẹ hơn), budget thấp (Elasticsearch RAM-heavy)

## Layer bổ sung thường gặp

| Nhu cầu | Khuyến nghị |
|---|---|
| Stream processing | Kafka Streams / Flink / Benthos / Redpanda |
| Batch processing | Spark / Polars / DuckDB / dbt |
| Vector search | pgvector / Qdrant / Weaviate / Pinecone |
| Graph database | Neo4j / PostgreSQL + Apache AGE |
| Object storage | S3 / Cloudflare R2 / MinIO |
| Data format | Parquet (columnar) / Avro (schema evolution) / JSON (simple) |
| Schema registry | Confluent Schema Registry / Buf (Protobuf) |
| Data quality | Great Expectations / dbt tests / Soda |
| Feature store | Feast / Tecton |
| Model registry | MLflow / Weights & Biases |
| GPU serving | vLLM / TGI / Triton / BentoML |

## Anti-patterns

- PostgreSQL cho analytics query trên >100M rows → dùng columnar DB (ClickHouse, DuckDB)
- Elasticsearch làm primary database → Elasticsearch là search index, không phải source of truth
- Pandas cho dataset >RAM → dùng Polars (streaming) hoặc DuckDB hoặc Spark
- Kafka cho <1k msg/s → Redis Pub/Sub hoặc PostgreSQL LISTEN/NOTIFY đủ
- Tự build ML serving framework → dùng FastAPI + async hoặc managed service
- Không có data validation ở ingestion layer → garbage in, garbage out
- Single large Elasticsearch index → shard strategy, index lifecycle
- MongoDB cho time-series → TimescaleDB / InfluxDB tối ưu hơn
