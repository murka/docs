---
editable: false
sourcePath: en/_api-ref/mdb/greenplum/api-ref/ResourcePreset/get.md
---


# Method get
Returns the specified resource preset.
 
To get the list of available resource presets, make a [list](/docs/managed-greenplum/api-ref/ResourcePreset/list) request.
 
## HTTP request {#https-request}
```
GET https://mdb.api.cloud.yandex.net/managed-greenplum/v1/resourcePresets/{resourcePresetId}
```
 
## Path parameters {#path_params}
 
Parameter | Description
--- | ---
resourcePresetId | Required. Required. ID of the resource preset to return. To get the resource preset ID, use a [list](/docs/managed-greenplum/api-ref/ResourcePreset/list) request.
 
## Response {#responses}
**HTTP Code: 200 - OK**

```json 
{
  "id": "string",
  "zoneIds": [
    "string"
  ],
  "cores": "string",
  "memory": "string",
  "type": "string",
  "minHostCount": "string",
  "maxHostCount": "string",
  "hostCountDivider": "string",
  "maxSegmentInHostCount": "string"
}
```
A preset of resources for hardware configuration of Greenplum hosts.
 
Field | Description
--- | ---
id | **string**<br><p>ID of the resource preset.</p> 
zoneIds[] | **string**<br><p>IDs of availability zones where the resource preset is available.</p> 
cores | **string** (int64)<br><p>Number of CPU cores for a Greenplum host created with the preset.</p> 
memory | **string** (int64)<br><p>RAM volume for a Greenplum host created with the preset, in bytes.</p> 
type | **string**<br><p>Host type</p> <ul> <li>MASTER: Greenplum master host.</li> <li>SEGMENT: Greenplum segment host.</li> </ul> 
minHostCount | **string** (int64)<br><p>Min host count</p> 
maxHostCount | **string** (int64)<br><p>Max host count</p> 
hostCountDivider | **string** (int64)<br><p>The number of hosts must be divisible by host_count_divider</p> 
maxSegmentInHostCount | **string** (int64)<br><p>Max segment count in host (actual only for segment host)</p> 