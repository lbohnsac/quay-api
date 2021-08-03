# quay-api
To use the api you have to have a Bearer token in place with the correct scope

```
export bearer_token=XXXXXXXXXXXXXXXX
export quay_registry=REGISTRY_FQDN_OR_IP
```

## Organizations (Namespaces)
### List all existing `organizations`
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/superuser/organizations/ | jq
```
### Get the details for the specified organization `orga`.
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/orga | jq
```
Success is HTTP `200`, no success is HTTP `404`

### List the applications for the specified organization
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/orga/applications | jq
```
Success is HTTP`200`

### Delete the organization `orga`
```
curl -X DELETE -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/deleteme | jq
```
Success is HTTP `204`, no success is HTTP `403`

## Repositories
List all existing repositories in organization `orga`
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/repository?namespace=orga | jq
```
