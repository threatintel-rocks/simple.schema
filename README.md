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
    "verified": true,
    "columns": [
      {
        "name": "Timestamp",
        "type": "datetime",
        "description": "Date and time when the event was recorded",
        "availability": ["Defender XDR"],
        "verified": true
      }
    ]
  }
}
```

The `availability` field indicates where the column is available:
- `"Defender XDR"` — available in Defender XDR Advanced Hunting
- `"Sentinel"` — available in Microsoft Sentinel
- `"Defender for Identity"` / `"Defender for Endpoint"` / `"Defender for Cloud Apps"` — available only with specific Defender licensing
- Columns can have multiple values (e.g., `["Defender XDR", "Sentinel"]`) when available in both platforms

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
| Sentinel Tables | ~500+ | ~95 standard | All Sentinel connector tables; standard tables have full column schemas from Azure Monitor reference |
| Defender XDR Advanced Hunting Schema | 63 | 63 | Full column schemas from Microsoft Defender XDR documentation |
| 3rd-Party Connector Schemas | ~143 | ~143 | Column schemas for 3rd-party vendor tables from Azure-Sentinel CCP connector definitions |

### 3rd-Party Vendor Solutions with Column Schemas

The following 3rd-party vendor solutions have full column schemas sourced from the [Azure-Sentinel](https://github.com/Azure/Azure-Sentinel) GitHub repository:

1Password, Alibaba Cloud ActionTrail, Anvilogic, Atlassian Confluence Audit, Atlassian Jira Audit, Auth0, AWS CloudFront, AWS EKS, Azure DevOps Auditing, BigID DSPM, Bitwarden, Box, Check Point CloudGuard CNAPP, Check Point Cyberint, Cisco Secure Endpoint, Citrix Analytics, Cloudflare, Cortex XDR (Palo Alto), CyberArk Audit, Cyble Vision, Cyera DSPM, CYFIRMA, Cyren Threat Intelligence, D3 SmartSOAR, Databahn, Dragos, Druva, Ermes Browser Security, Feedly, Flare, Gigamon, GitHub Audit Logs, Google Cloud Platform, Halcyon, Illumio, Imperva Cloud WAF, IONIX, Island, Jamf Protect, Keeper Security, Lookout, MailRisk, meshStack, Miro, Morphisec, Netskope, NordPass, Obsidian, Okta SSO, OneLogin IAM, OneTrust, OpenAI, Oracle Cloud Infrastructure, Palo Alto Cortex Xpanse, Palo Alto Prisma Cloud, PingOne, Proofpoint POD, Qualys VM, Quokka, RSA ID Plus, Rubrik Security Cloud, Salesforce Service Cloud, SAP BTP, SAP ETD Cloud, SAP LogServ, SentinelOne, Slack Audit, Snowflake, SOC Prime, Sophos Endpoint Protection, Styx Intelligence, TacitRed, TheHive, Trellix, Tropico, Varonis Purview, VersasecCMS, VMware Carbon Black Cloud, ZeroFox, ZeroNetworks

**Note:** Custom log tables (`_CL` suffix) not listed above are included as metadata (table name + connector source) but without column definitions.

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

**Check column availability:**
```python
table_data = schema.get("IdentityInfo", {})
for col in table_data.get("columns", []):
    avail = col.get("availability", [])
    if "Sentinel" in avail and "Defender XDR" not in avail:
        print(f"  {col['name']} is Sentinel-only")
```

**Check if a column is verified against a live environment:**
```python
table_data = schema.get("DeviceEvents", {})
for col in table_data.get("columns", []):
    status = col.get("verified")
    if status is True:
        pass  # Column confirmed in live environment
    elif status is False:
        print(f"  {col['name']} is documented but NOT found in live environment")
    elif status == "live_only":
        print(f"  {col['name']} exists live but is NOT in documentation")
```

## Schema Verification

Schemas are periodically **verified** against a live Microsoft Defender XDR and Sentinel environment. When a table has been verified, each column gets a `verified` field:

| Value | Meaning |
|-------|---------|
| `true` | Column exists in both documentation and the live environment |
| `false` | Column is documented but was **not** found in the live environment |
| `"live_only"` | Column was found in the live environment but is **not** in documentation |
| *(absent)* | Table has not been verified yet |

Tables that have been verified also get a top-level `"verified": true` property.

## How It Works

Schemas are fetched daily from official Microsoft documentation by an automated pipeline. Changes are detected by diffing the previous schema against the new one, and all modifications are recorded in the [changelog](CHANGELOG.md).

Verification against live environments runs weekly and the results are published here automatically.

## License

Schema data is sourced from publicly available Microsoft documentation.
