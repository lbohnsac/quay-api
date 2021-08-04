# quay-api
To use the api you have to have a Bearer token in place with the correct scope

```
export bearer_token=XXXXXXXXXXXXXXXX
export quay_registry=REGISTRY_FQDN_OR_IP
```


## Organizations (Namespaces)
##### List all existing `organizations`
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/superuser/organizations/ | jq
```
##### Get the details for the specified organization `orga`.
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/orga | jq
```
Success is HTTP `200`, no success is HTTP `404`

##### List the applications for the specified organization
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/orga/applications | jq
```
Success is HTTP `200`

##### Delete the organization `orga`
```
curl -X DELETE -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/orga | jq
```
Success is HTTP `204`, no success is HTTP `403`



## Repositories
##### List all existing repositories in organization `orga`
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/repository?namespace=orga | jq
```
##### Change the visibility of the repository `repo` within the organization `orga` to `public`
```
curl -X POST -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/repository/orga/repo/changevisibility -H "Content-Type: application/json" --data '{"visibility": "public"}' | jq
```
Success is HTTP `200`, no success is HTTP `400`

Valid values are `public` or `private`

##### Change the Repository state to `Readonly` for repository `repo` within organization `orga`
```
curl -X PUT -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/repository/orga/repo/changestate -H "Content-Type: application/json" --data '{"state": "READ_ONLY"}' | jq
```
Success is HTTP `200`

Valid values are `NORMAL`, `READ_ONLY` and `MIRROR`... take care it's case sensitive!
##### Initiate a mirror sync action of the mirrored repository `mirrorrepo`
```
curl -X POST -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/repository/orga/repo/mirrorrepo/sync-now | jq
```
##### Get the existing mirror config from the repository `mirrorrepo`
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/repository/orga/mirrorrepo/mirror | jq
```



## Teams
##### List all teams within organization `orga`
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/orga | jq
```
##### Create a team `orgateam` within the organization `orga`
```
curl -X PUT -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/orga/team/orgateam -H "Content-Type: application/json" --data '{"name": "orgateam", "role": "member", "description": "Orgateam"}' | jq
```
##### Add to the repository `repo` within the organization `orga` the team `orgateam` with role `read`
```
curl -X PUT -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/repository/orga/repo/permissions/team/orgateam -H "Content-Type: application/json" --data '{"role": "read"}' | jq
```
##### Change the role of the team `orgateam` for the repository `repo` within the organization `orga` to `write`
```
curl -X PUT -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/repository/orga/repo/permissions/team/orgateam -H "Content-Type: application/json" --data '{"role": "write"}' | jq
```
##### Remove from the repository `repo` within the organization `orga` the team `orgateam`
```
curl -X DELETE -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/repository/orga/repo/permissions/team/orgateam | jq
```
##### Add the user `abc` as member to the team `orgateam` in the organization `orga`
```
curl -X PUT -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/orga/team/orgateam/members/abc | jq
```
##### Get all members of the team `orgateam` in organization `orga`
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/orga/team/orgateam/members | jq
```
##### Remove the user `abc` as member of the team `orgateam` within the organization `orga`
```
curl -X DELETE -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/orga/team/orgateam/members/abc | jq
```
##### Delete the team `orgateam` within the organization `orga`
```
curl -X DELETE -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/orga/team/orgateam | jq
```



## Users
##### Get all existing users
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/superuser/users/ | jq
```
##### Create a user `abc` with email `abc@abc.com`
```
curl -X POST -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/superuser/users/ -H "Content-Type: application/json" --data '{"username": "abc", "email": "abc@abc.com"}' | jq
```
Success is HTTP `200`, user already exists or email is already in use is HTTP `400`
##### Get user information of user `abc`
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/users/abc | jq
Success is HTTP `200`, no success is HTTP `404`
```
##### Delete user `abc` and all repositories owned by the user
```
curl -X DELETE -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/superuser/users/abc | jq
```
Success is HTTP `200`, no success is HTTP `404`
