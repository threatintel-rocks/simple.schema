# simple.schema

Auto-updated table schemas for **Microsoft Sentinel**, **Defender XDR** Advanced Hunting, and related security data sources.

Updated daily from official Microsoft documentation. See [CHANGELOG.md](CHANGELOG.md) for a full history of schema changes.

## Structure

```
latest_schemas.json          # All tables in one file
tables/
  AlertEvidence.json         # One file per table
  DeviceEvents.json
  Syslog.json
  ...
sources/
  defender-xdr-advanced-hunting-schema/
    _all.json                # All tables for this source
    AlertEvidence.json       # Individual table
    ...
  sentinel-data-source-schema-reference/
    _all.json
    AuditLogs.json
    ...
  commonsecuritylog-cef/
    ...
CHANGELOG.md                 # Full history of schema changes
```

## Schema Format

Each table entry:

```json
{
  "AlertEvidence": {
    "source": "Defender XDR Advanced Hunting Schema",
    "columns": [
      {
        "name": "Timestamp",
        "type": "datetime",
        "description": "Date and time when the event was recorded"
      }
    ]
  }
}
```

## Usage

**Single table:**
```
curl -s https://raw.githubusercontent.com/threatintel-rocks/simple.schema/main/tables/AlertEvidence.json
```

**All tables for a source:**
```
curl -s https://raw.githubusercontent.com/threatintel-rocks/simple.schema/main/sources/defender-xdr-advanced-hunting-schema/_all.json
```

**Everything:**
```
curl -s https://raw.githubusercontent.com/threatintel-rocks/simple.schema/main/latest_schemas.json
```

**Python:**
```python
import requests
schema = requests.get("https://raw.githubusercontent.com/threatintel-rocks/simple.schema/main/tables/DeviceEvents.json").json()
table = schema["DeviceEvents"]
print(f"{len(table['columns'])} columns")
for col in table["columns"][:5]:
    print(f"  {col['name']} ({col['type']})")
```

## Sources

| Source | Tables | With Columns | Description |
|--------|--------|-------------|-------------|
| Sentinel Tables | ~507 | ~95 standard + ~413 custom (_CL) | All Sentinel connector tables; standard tables have full column schemas from Azure Monitor reference |
| Defender XDR Advanced Hunting Schema | 63 | 63 | Full column schemas from Microsoft Defender XDR documentation |

**Note:** Custom log tables (`_CL` suffix) are vendor-specific and their schemas vary per deployment. They are included as metadata (table name + source) but without column definitions. Standard Sentinel tables have full column schemas fetched from Azure Monitor reference pages.

## AI Agent Integration

This schema repository is designed to be used as **guardrails for AI agents** that generate KQL (Kusto Query Language) queries for Microsoft Sentinel and Defender XDR. By validating generated queries against these schemas, you can reduce query hallucination.

**Validate a table exists:**
```python
schema = requests.get("https://raw.githubusercontent.com/threatintel-rocks/simple.schema/main/latest_schemas.json").json()
table = "DeviceProcessEvents"
if table not in schema:
    print(f"Table '{table}' does not exist")
```

**Validate columns in a query:**
```python
table_data = schema.get("DeviceProcessEvents", {})
valid_columns = {col["name"] for col in table_data.get("columns", [])}
query_columns = ["Timestamp", "DeviceName", "FileName", "NonExistentColumn"]
for col in query_columns:
    if col not in valid_columns:
        print(f"Column '{col}' does not exist in DeviceProcessEvents")
```

## How It Works

Schemas are fetched daily from official Microsoft documentation by an automated pipeline. Changes are detected by diffing the previous schema against the new one, and all modifications are recorded in the [changelog](CHANGELOG.md).

## License

Schema data is sourced from publicly available Microsoft documentation.
