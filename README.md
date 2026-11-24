# nexus-xml
Streaming C++ XML-to-Parquet and SQLite shredder for complex, deeply nested XML. Automatically infers relational schemas mirroring your XML structure and enforced with foreign keys, guarantees atomic file transactions, and compiles XPath queries to SQL.


Built by [Novixys](https://novixys.com) on battle-tested streaming XML shredding architecture, `nexus-xml` turns complex, deeply nested enterprise XML (SEC filings, ISO 20022, HL7/FHIR, ACORD, UBL, or any other XML) into clean, queryable relational tables in seconds.

---

## Why `nexus-xml`?

* **Zero XSD or Manual Mapping Required:** Automatically discovers nested hierarchies of any depth, creates normalized parent-child tables with foreign keys, and evolves columns on the fly as optional tags appear.
  
* **No out-of-memory crashes:** Uses a streaming architecture to process arbitrarily large XML without memory issues. No need for expensive high memory cloud instances.
* **Automatically figures out relationships:** SQL tables are created mirroring your XML structure with parent-child relationships.
* **Merge data from multiple XML files:** XML parsing ensures your data is merged according to the XML structure.
* **Atomic File-Level ACID Transactions:** Every XML file is ingested inside an explicit transaction. Valid files commit cleanly across all tables; malformed or truncated files roll back 100% automatically, making retries possible with zero duplicate records.
* **Resilient to Real-World Schema Drift:** Stores raw text safely during ingestion so real-world data anomalies (such as string values appearing in numeric XSD fields or omitted optional blocks) never crash your pipeline.
* **Millisecond XPath-to-SQL Queries (`xpathsql`):** Query your multi-gigabyte shredded datasets using intuitive XPath predicates and inspect the exact compiled multi-table SQL `JOIN` before execution.
* **Warehouse-Ready Parquet Export (`nexus-parquet`):** Export all discovered relational tables and the XPath lineage metadata catalog to ZSTD-compressed `.parquet` files ready for DuckDB, Snowflake, Databricks, or BigQuery.
* **100% Air-Gapped & Zero Telemetry:** Pure local file I/O with zero network calls, zero phone-home tracking, and zero external runtime dependencies—safe for HIPAA, PCI-DSS, financial, and air-gapped environments. Shipped as a 100% statically linked Linux x86_64 binary.

---

## Toolkit Overview

| Binary / Tool | Purpose |
| :--- | :--- |
| **`nexus-xml`** | Streaming C++ XML shredder. Converts single or multiple deeply-nested XML files into normalized SQLite tables and a schema catalog useful for XPath queries. |
| **`xpathsql`** | Fast XPath-to-SQL compiler and CLI query engine for the database created by `nexus-xml`. Prints the generated SQL query and executes it in milliseconds. |
| **`nexus-parquet`** | Exports all relational tables and schema lineage metadata into ZSTD-compressed Parquet files for cloud data warehouses. |

---

## Quick Start (Linux x86_64)

### 1. Download the Latest Release
Grab the precompiled Linux release archive from the [Releases](https://github.com) page:

```bash
tar -xzf nexus-xml-linux-x86_64.tar.gz
cd nexus-xml
chmod +x nexus-xml xpathsql nexus-parquet
```

### 2. Shred XML Files into a Relational Database
Ingest one or more deeply nested XML files into an atomic SQLite database:

```bash
./nexus-xml -o absee_2026_10.db /path/to/sec_absee_xml/*.xml
```
* Each file runs in its own transaction (`BEGIN` → `COMMIT` / `ROLLBACK`).
* If a file contains syntax errors or unclosed tags, all rows from that file are rolled back cleanly so you can fix the file and re-run without duplicates.
* An internal metadata table records every discovered table, column, and source XPath so incremental imports append seamlessly into the same schema.

### 3. Query with XPath (and Inspect the Compiled SQL)
Run XPath queries across gigabytes of shredded XML in milliseconds using `xpathsql`:

```bash
./xpathsql -l warn absee_2026_10.db '/assetData/assets[assetTypeNumber = "Ford Credit"][vehicleValueAmount < 40000]'
```
`xpathsql` prints the exact multi-table SQL translation before executing, allowing data engineers to copy and reuse the generated SQL directly in downstream pipelines, Snowflake views, or DuckDB scripts.

### 4. Export to ZSTD Parquet for Data Warehouses
Convert all discovered tables and schema metadata into compressed `.parquet` files:

```bash
./nexus-parquet absee_2026_10.db ./parquet_out/
```

Query immediately with DuckDB or load into Snowflake / Databricks / BigQuery:
```sql
SELECT * FROM read_parquet('./parquet_out/*.parquet', union_by_name=true) LIMIT 10;
```

---

## Supported XPath Subset (`xpathsql`)

`xpathsql` implements a fast, SQL-compilable relational subset of XPath designed for predictable millisecond execution on multi-gigabyte datasets:

* **Paths:** Absolute and relative child paths (`/root/record/subrecord/field`)
* **Comparison Predicates:** `=`, `!=`, `<`, `<=`, `>`, `>=` (e.g., `[vehicleValueAmount < 40000]`)
* **Compound Filters:** Chained predicates (`[a = "x"][b > 10]`) and boolean `and` / `or`
* **Escape Hatch:** For aggregations (`GROUP BY`), window functions, or complex joins, copy the SQL query printed by `xpathsql` and run it directly in SQLite, DuckDB, or your cloud warehouse.

---

## Real-World Use Cases

* **Financial & Regulatory Filings:** SEC Form ABS-EE, Form N-PORT, ISO 20022 (`pacs.008`, `camt.053`), FpML
* **Healthcare & Insurance:** HL7 CDA / FHIR XML, ACORD policy & claims feeds
* **Supply Chain & Trade:** UBL, cXML, GS1, customs manifests

---

## Evaluation & Commercial Licensing

* **Free Evaluation Tier:** Test `nexus-xml` freely on development workloads and sample files.
* **Enterprise & High-Volume Production:** Need multi-terabyte throughput, custom vertical connectors, or audit-level byte-offset provenance? Open an [Issue](https://github.com) or reach out via [novixys.com](https://novixys.com).

## Roadmap

* [ ] Direct native C++ Arrow/Parquet writer (zero intermediate SQLite step)
* [ ] Optional cell-level XML byte-offset provenance for regulatory audits
* [ ] `nexus-json`: Streaming C++ shredder for deeply nested multi-GB JSON files
