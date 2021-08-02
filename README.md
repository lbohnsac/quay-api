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
### Get the details for the specified organization.
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/orga | jq
```

## Repositories
List all existing repositories in organization `orga`
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/repository?namespace=orga | jq
```
