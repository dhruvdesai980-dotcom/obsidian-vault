# dbt Semantic Layer POC – Findings and Observations

Environment: dbt Cloud (dbt Studio), Neon PostgreSQL, GitHub | October 2026

# dbt Semantic Layer POC – Findings and Observations

## 1. Data Sources and Physical Data Modeling

As part of the POC, we established a connection between dbt Cloud and Neon PostgreSQL and created ten banking-related datasets covering Counterparty, Agreement, Trade, Product, Account, Strategist, Meeting, Meeting Activity, Meeting Attendee, and Meeting Outcome. These datasets were initially loaded using dbt seeds and subsequently registered as sources to simulate an enterprise-style architecture. The testing confirmed that dbt can access existing database tables through source definitions and build transformations without requiring the physical data to be recreated. However, dbt remains dependent on the underlying database for physical data storage and SQL execution.

## 2. Staging Layer and Data Standardization

We created staging models to standardize source data by applying transformations such as trimming whitespace, normalizing text values, casting data types, and deriving additional business attributes. For example, the Counterparty staging model converted risk ratings into numerical risk scores and derived an active-status indicator. The testing demonstrated that dbt supports reusable transformation logic and provides a consistent interface for downstream models. However, these transformations must be explicitly defined in SQL, and changes to underlying source structures may require corresponding updates to staging models.

## 3. Data Quality Validation and Record Rejection

We implemented data-quality checks using dbt's built-in tests, including uniqueness, non-null constraints, accepted values, and referential relationships between models. The Counterparty dataset included an intentionally invalid record, allowing us to test data validation and exception handling. We created separate intermediate models to classify valid and rejected records, resulting in ten valid Counterparty records and one rejected record. This demonstrated that dbt can identify data-quality issues and support custom record-handling logic. However, record rejection and quarantine are not automatically provided by standard dbt tests; they require additional SQL models and business rules.

## 4. Intermediate Models and Business Transformations

We developed intermediate models to implement more complex business logic, including joins between Counterparty, Agreement, Trade, and Product, along with aggregations involving meetings, activities, attendees, and outcomes. We validated record counts, relationship integrity, and aggregate reconciliation to ensure that joins did not unintentionally duplicate or exclude records. For example, the Trade Exposure model maintained twenty distinct trades, while the Agreement Trade Summary preserved fifteen agreements, including two agreements without trades. These experiments confirmed that dbt supports complex relational transformations and reusable business logic through modular SQL models.

## 5. Marts and Analytical Data Models

We created analytical models such as `fct_trade` and `dim_product` to expose structured datasets for semantic modeling and analytical consumption. These marts were built using upstream intermediate and staging models, establishing a clear separation between raw data preparation and business-facing analytical structures. The testing demonstrated that dbt supports dimensional modeling and provides reusable fact and dimension datasets. However, marts are still implemented as SQL models and materialized through the underlying database, meaning dbt's semantic capabilities operate on top of its physical or logical data-modeling layer.

## 6. Semantic Metrics and Centralized Business Definitions

We configured the dbt Semantic Layer using YAML definitions to establish business metrics such as Trade Count and Settled Trade Count. Using MetricFlow, we successfully queried individual and multiple metrics, grouped results by currency and trade status, and performed time-based aggregations using a daily time spine. The Trade Count metric consistently returned twenty records, confirming that centrally defined metrics could be queried without manually rewriting their aggregation logic. This demonstrated dbt's ability to maintain reusable metric definitions independently of individual analytical queries. However, these definitions are still associated with dbt model resources and their exposed columns.

## 7. Semantic Relationships and Automatic Join Generation

We tested semantic relationships by defining Product as a foreign entity in the Trade semantic model and a primary entity in the Product semantic model. MetricFlow successfully calculated Trade Count grouped by Product Asset Class, producing nine Fixed Income trades, four Equity trades, four Derivatives trades, and three Foreign Exchange trades. Inspection of the compiled SQL confirmed that MetricFlow automatically generated a LEFT OUTER JOIN between `fct_trade` and `dim_product`, without requiring the join to be explicitly written in the metric query. This demonstrated that dbt can resolve relationships across semantic models using declared entities. However, complex scenarios involving ambiguous join paths, many-to-many relationships, and fanout prevention remain untested.

## 8. Semantic-First Modeling and Conceptual Independence

We evaluated whether dbt supports a truly semantic-first approach by creating a conceptual Trade definition in YAML without a corresponding SQL model or physical table. Although `dbt parse` and the existing semantic validation initially succeeded, the conceptual model was not registered as a semantic resource. After creating a corresponding SQL model and completing the required semantic configuration, dbt successfully registered the conceptual model, even before the associated PostgreSQL view was materialized. However, warehouse-level semantic validation failed until the physical relation became available. These findings indicate that dbt supports defining and registering semantic metadata before database materialization, but our tests did not demonstrate the ability to maintain a fully independent conceptual semantic model without an associated dbt model definition.

## 9. Physical Rebinding and Implementation Independence

We tested physical rebinding by initially defining `conceptual_trade` using a placeholder SQL model and later replacing its implementation with a reference to the existing `fct_trade` model. Throughout this experiment, the semantic YAML, entity definitions, dimensions, and metric definitions remained unchanged. After rebuilding the SQL model, MetricFlow successfully queried the conceptual Trade Count metric, and the compiled SQL referenced the stable `conceptual_trade` interface. This demonstrated that dbt can preserve business semantic definitions while changing the underlying SQL implementation, provided the output schema remains consistent. However, the approach relies on a stable dbt SQL model interface rather than an independently managed semantic-to-physical binding mechanism.

## 10. Physical Schema Changes and Manual Remapping

We evaluated the impact of underlying physical schema changes by creating an isolated Trade model in which the `currency` column was renamed to `currency_code`. When the dependent `conceptual_trade` model continued referencing the original column name, dbt failed during the build because PostgreSQL could no longer resolve the column. After manually updating the SQL mapping to expose `currency_code AS trade_currency`, the build and semantic validation succeeded without modifying the semantic YAML. MetricFlow continued returning the expected currency-level results of thirteen USD, four CAD, and three EUR trades. This confirmed that a SQL mapping layer can shield semantic definitions from certain physical schema changes. However, dbt did not automatically remap the renamed column, and the incompatibility was identified during model execution rather than proactively during parsing.

## 11. Model Contracts and Structural Governance

We implemented an enforced model contract on `conceptual_trade` to define the expected column names and data types. The initial contract-compliant model built successfully, confirming that dbt could enforce a declared output schema. We then intentionally renamed the output column `trade_currency` to `trade_currency_new`, which caused the build to fail with a contract violation identifying the missing and unexpected columns. A separate test changed the `trade_currency` data type from TEXT to INTEGER, and dbt correctly identified the incompatible data type. These experiments demonstrated that dbt contracts provide structural governance by preventing incompatible SQL model outputs from being deployed successfully. However, model contracts do not automatically detect every change in business logic or determine the full downstream impact of a structural change.

## 12. Semantic Layer Resilience and Recovery

Following the intentional model contract violation, we executed the existing MetricFlow query to determine whether the failed build affected the previously deployed semantic interface. The query continued returning the expected twenty trades, distributed across USD, CAD, and EUR, confirming that the previous database view remained usable in our experiment. After restoring the contract-compliant SQL definition, both the dbt model build and Semantic Layer validation succeeded again. This demonstrated that enforced contracts can protect an existing semantic-facing interface from certain incompatible model replacements and allow normal operation to resume after corrections. However, this does not establish comprehensive deployment rollback capabilities or protection against all possible runtime and upstream failures.

## 13. Preliminary Architectural Assessment

Based on the completed experiments, dbt demonstrates strong capabilities for reusable business metrics, semantic relationship resolution, automatic SQL generation, modular transformations, data-quality validation, and structural governance through model contracts. The POC also confirmed that semantic definitions can remain stable while the underlying SQL implementation changes, provided the expected logical interface is preserved. However, the tested approach remains closely associated with dbt SQL model definitions and requires manually maintained mappings between physical and semantic structures. Therefore, the current assessment is that dbt provides a model-bound semantic layer with meaningful physical implementation abstraction, rather than a fully independent, conceptual-model-first modeling environment.

## 14. Areas Remaining for Evaluation

The next phase of the POC will focus on lineage and downstream impact analysis, particularly whether dbt can identify dependent models and metrics affected by breaking changes before deployment. Additional evaluations will cover model versioning, semantic definition portability, import/export capabilities, dbt Canvas, and AI-assisted development through dbt Copilot and Wizard. Complex semantic relationships and fanout prevention also remain open for testing. Power BI and Strategy integration testing have been intentionally deferred and will be evaluated separately when required.

These notes distinguish between capabilities demonstrated through our tests, limitations actually observed, and areas that still require validation. They can serve as the baseline findings section of our final POC assessment and eventual comparison with Strategy Mosaic.