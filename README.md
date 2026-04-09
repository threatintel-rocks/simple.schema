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

### Microsoft Native

| Source | Description |
|--------|-------------|
| Sentinel Data Source Schema Reference | Core Sentinel tables (AuditLogs, AzureActivity, Syslog, etc.) |
| Defender XDR Advanced Hunting Schema | 63 tables for endpoint, email, identity, and cloud hunting |
| CommonSecurityLog (CEF) | Common Event Format table used by security appliances |
| Syslog | Linux agent and network appliance syslog events |

### 3rd-Party (via Sentinel connectors)

| Source | Sentinel Table |
|--------|---------------|
| Fortinet FortiGate | CommonSecurityLog |
| Palo Alto Networks PAN-OS | CommonSecurityLog |
| Check Point | CommonSecurityLog |
| Cisco ASA/FTD | CommonSecurityLog |
| CrowdStrike Falcon | CrowdStrike_CL |
| Sophos Endpoint Protection | SophosXGFirewall_CL |
| SentinelOne | SentinelOne_CL |
| Zscaler Internet Access | CommonSecurityLog |
| F5 BIG-IP | F5Telemetry_CL |
| Trend Micro Vision One | TrendMicro_XDR_CL |
| Okta Single Sign-On | Okta_CL |
| Proofpoint TAP | ProofpointTAP_CL |
| AWS CloudTrail | AWSCloudTrail |
| Google Workspace | GWorkspace_ReportsAPI_admin_CL |

## How It Works

Schemas are fetched daily from official Microsoft documentation by an automated pipeline. Changes are detected by diffing the previous schema against the new one, and all modifications are recorded in the [changelog](CHANGELOG.md).

## License

Schema data is sourced from publicly available Microsoft documentation.
