---
title: ContractAPI
---
{% include-markdown "./_includes/contractapi_description.md" %}

### Example

```json
{
    "id": "0f12317b-85a0-4a77-a722-857ea2b0a5fa",
    "namespace": "ns1",
    "interface": {
        "id": "c35d3449-4f24-4676-8e64-91c9e46f06c4"
    },
    "location": {
        "address": "0x95a6c4895c7806499ba35f75069198f45e88fc69"
    },
    "name": "my_contract_api",
    "message": "b09d9f77-7b16-4760-a8d7-0e3c319b2a16",
    "urls": {
        "api": "http://127.0.0.1:5000/api/v1/namespaces/default/apis/my_contract_api",
        "openapi": "http://127.0.0.1:5000/api/v1/namespaces/default/apis/my_contract_api/api/swagger.json",
        "ui": "http://127.0.0.1:5000/api/v1/namespaces/default/apis/my_contract_api/api"
    },
    "published": false
}
```

### Field Descriptions

| Field Name | Description | Type |
|------------|-------------|------|
| `id` | The UUID of the contract API | [`UUID`](simpletypes.md#uuid) |
| `namespace` | The namespace of the contract API | `string` |
| `interface` | Reference to the FireFly Interface definition associated with the contract API | [`FFIReference`](#ffireference) |
| `location` | If this API is tied to an individual instance of a smart contract, this field can include a blockchain specific contract identifier. For example an Ethereum contract address, or a Fabric chaincode name and channel | [`JSONAny`](simpletypes.md#jsonany) |
| `name` | The name that is used in the URL to access the API | `string` |
| `networkName` | The published name of the API within the multiparty network | `string` |
| `message` | The UUID of the broadcast message that was used to publish this API to the network | [`UUID`](simpletypes.md#uuid) |
| `urls` | The URLs to use to access the API | [`ContractURLs`](#contracturls) |
| `published` | Indicates if the API is published to other members of the multiparty network | `bool` |

## FFIReference

| Field Name | Description | Type |
|------------|-------------|------|
| `id` | The UUID of the FireFly interface | [`UUID`](simpletypes.md#uuid) |
| `name` | The name of the FireFly interface | `string` |
| `version` | The version of the FireFly interface | `string` |


## ContractURLs

| Field Name | Description | Type |
|------------|-------------|------|
| `api` | The URL to use to invoke the API | `string` |
| `openapi` | The URL to download the OpenAPI v3 (Swagger) description for the API generated in JSON or YAML format | `string` |
| `ui` | The URL to use in a web browser to access the SwaggerUI explorer/exerciser for the API | `string` |


