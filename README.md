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
##### List all repositories the token has accecc to
```curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/repository?public=true | jq
```
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


### Mirrored repositories
##### Get the existing mirror config from the repository `mirrorrepo`
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/repository/orga/mirrorrepo/mirror | jq
```
##### Update the full mirror config of the mirrored repository `mirrorrepo` within the organization `orga` (docker.io/minio/mc:latest)
The used robot `orga+robot` must already exist!

Required values to provide are
- external_reference
- sync_interval
- sync_start_date
- root_rule
```
curl -v -X PUT -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/repository/orga/mirrorrepo/mirror -H "Content-Type: application/json" --data '{"is_enabled": true, "external_reference": "docker.io/minio/mc", "external_registry_username": "", "external_registry_config": {"verify_tls": false,"proxy": {"http_proxy": "", "https_proxy": "", "no_proxy": ""}},"sync_interval": 600, "sync_start_date": "2021-08-06T11:11:39Z", "sync_expiration_date": "2021-08-07T11:18:29Z", "root_rule": {"rule_kind": "tag_glob_csv", "rule_value": [ "latest" ]}, "robot_username": "orga+robot"}' | jq
```
##### Update a part of the mirror config of the mirrored repository `mirrorrepo` within the organization `orga` (docker.io/minio/mc:latest)
Here only the sync interval will be updated
```
curl -v -X PUT -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/repository/orga/mirrorrepo/mirror -H "Content-Type: application/json" --data '{"sync_interval": 1260}' | jq
```
Success is HTTP `201`
##### Initiate a mirror sync action of the mirrored repository `mirrorrepo`
```
curl -X POST -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/repository/orga/repo/mirrorrepo/sync-now | jq
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

## Robots
##### Create the robot `robby` within the organization `orga`
```
curl -X PUT -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/orga/robots/robby | jq
```
Success is HTTP `201`, no success is HTTP `400`
##### List all existing robots with in the organization `orga`
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/orga/robots | jq
```
Success is HTTP `200`

All robots and their tokens are listed
##### Replace the current token of robot `robby` with a new token
```
curl -X POST -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/orga/robots/robby/regenerate | jq
```
Success is HTTP `200`

This will delete the old token and all current logins of this robot will become invalid immediately
##### Set the permission `write` to robot `robby` for repository `repo` within organization `orga`
```
curl -X PUT -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/repository/orga/repo/permissions/user/orga+robby -H "Content-Type: application/json" --data '{"role": "write"}' | jq
```
Success is HTTP `200`, no success is HTTP `400`

Valid values are `read`, `write` and `admin`
##### Get information about robot `robby` (including the robbot token)
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/orga/robots/robby | jq
```
Success is HTTP `200`, no success is HTTP `400`
##### Get the current permissions of robot `robby` within the organization `orga`
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/orga/robots/robby/permissions | jq
```
Success is HTTP `200`, no success is HTTP `400`
##### Delete the current permission for the robot `robby` for repository `repo` within organization `orga`
```
curl -X DELETE -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/repository/orga/repo/permissions/user/orga+robby | jq
```
Success is HTTP `204`, no success is HTTP `400`
## Quotas
##### List quota of organization orga
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" -H 'Content-Type: application/json' https://${quay_registry}/api/v1/organization/orga/quota | jq
```
Success is HTTP `200`, no success is HTTP `404`
##### Create a quota for organization orga
```
curl -X POST -H "Authorization: Bearer ${bearer_token}" -H 'Content-Type: application/json' -d '{"limit_bytes": 104857600}' https://${quay_registry}/api/v1/organization/orga/quota | jq
```
##### Delete quota id 1 for organization orga
```
curl -X DELETE -H "Authorization: Bearer ${bearer_token}" -H 'Content-Type: application/json' https://${quay_registry}/api/v1/organization/orga/quota/1 | jq
```
