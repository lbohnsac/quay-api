# quay-api
To use the api you have to have a bearer token in place with the correct scope
```
export bearer_token=XXXXXXXXXXXXXXXX
export quay_registry=REGISTRY_FQDN_OR_IP
export orga=ORGANIZATION_NAME
export repo=REPOSITORY_NAME
export team=TEAM_NAME
export user=USER_NAME
export robot
```

***Tested with quay version 3.7.7***

## Organizations (Namespaces)
##### List all existing `organizations`
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/superuser/organizations/ | jq
```
##### Get the details for the organization `$orga`
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/${orga} | jq
```
Success is HTTP `200`, no success is HTTP `404`

##### List the applications for the organization `$orga`
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/${orga}/applications | jq
```
Success is HTTP `200`

##### Delete the organization `$orga`
```
curl -X DELETE -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/${orga} | jq
```
Success is HTTP `204`, no success is HTTP `403`


## Repositories
##### List all repositories the token has access to
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/repository?public=true | jq
```
##### List all existing repositories in organization `$orga`
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/repository?namespace=${orga} | jq
```
##### Change the visibility of the repository `$repo` within the organization `$orga` to `public`
```
curl -X POST -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/repository/${orga}/${repo}/changevisibility -H "Content-Type: application/json" --data '{"visibility": "public"}' | jq
```
Success is HTTP `200`, no success is HTTP `400`

Valid values are `public` or `private`

##### Change the repository state to `Readonly` for repository `$repo` within the organization `$orga`
```
curl -X PUT -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/repository/${orga}/${repo}/changestate -H "Content-Type: application/json" --data '{"state": "READ_ONLY"}' | jq
```
Success is HTTP `200`

Valid values are `NORMAL`, `READ_ONLY` and `MIRROR`... take care it's case sensitive!


## Mirrored repositories (needs to be reviewed!)
##### Get the existing mirror config from the repository `$repo` within the organization `$orga`
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/repository/${orga}/${repo}/mirror | jq
```
##### Update the full mirror config of the mirrored repository `$repo` within the organization `$orga` (quay.io/minio/mc:latest)
The used robot `orga+robot` must already exist!

Required values to provide are
- external_reference
- sync_interval
- sync_start_date
- root_rule
```
curl -v -X PUT -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/repository/${orga}/${repo}/mirror -H "Content-Type: application/json" --data '{"is_enabled": true, "external_reference": "quay.io/minio/mc", "external_registry_username": "", "external_registry_config": {"verify_tls": false,"proxy": {"http_proxy": "", "https_proxy": "", "no_proxy": ""}},"sync_interval": 600, "sync_start_date": "2021-08-06T11:11:39Z", "sync_expiration_date": "2021-08-07T11:18:29Z", "root_rule": {"rule_kind": "tag_glob_csv", "rule_value": [ "latest" ]}, "robot_username": "orga+robot"}' | jq
```
##### Update a part of the mirror config of the mirrored repository `$repo` within the organization `$orga` (quay.io/minio/mc:latest)
Here only the sync interval will be updated
```
curl -v -X PUT -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/repository/${orga}/${repo}/mirror -H "Content-Type: application/json" --data '{"sync_interval": 1260}' | jq
```
Success is HTTP `201`
##### Initiate a mirror sync action of the mirrored repository `$repo` within the organization `$orga`
```
curl -X POST -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/repository/${orga}/repo/${repo}/sync-now | jq
```

## Teams
##### List all teams within organization `$orga` (and additional info)
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/${orga} | jq
```
##### Create a team `$team` within the organization `$orga` with role `member`
Required value to provide is
- `role` valid values are `member`, `creator` or `admin` 
```
curl -X PUT -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/${orga}/team/${team} -H "Content-Type: application/json" --data '{"role": "member", "description": "Short description of the team"}' | jq
```
success is HTTP `200`
##### Change the role to `admin` of the team `$team` within the organization `$orga`
valid values for `role` are `member`, `creator` or `admin` 
```
curl -X PUT -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/repository/orga/repo/permissions/team/orgateam -H "Content-Type: application/json" --data '{"role": "admin"}' | jq
```
Success is HTTP`200`
##### Set or change the description of the team `$team` within the organization `$orga` using standard text
Required value to provide is
- `role` valid values are `member`, `creator` or `admin`
- `description`
```
curl -X PUT -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/${orga}/team/${team} -H "Content-Type: application/json" --data '{"role": "member", "description": "Short description of the team"}' | jq
```
success is HTTP `200`
##### Set or change the description of the team `$team` within the organization `$orga` using markdown text
Required value to provide is
- `role` valid values are `member`, `creator` or `admin`
- `description`
```
curl -X PUT -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/${orga}/team/${team} -H "Content-Type: application/json" --data '{"role": "member", "description": "# Caption\nShort description of the *team*"}' | jq
```
success is HTTP `200`
##### Add the user `$user` as member to the team `$team` in the organization `$orga`
```
curl -X PUT -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/${orga}/team/${team}/members/${user} | jq
```
Success is HTTP `200`
##### Get all members of the team `$team` in organization `$orga`
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/${orga}/team/${team}/members | jq
```
##### Remove the user `$user` as member of the team `$team` within the organization `$orga`
```
curl -X DELETE -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/${orga}/team/${team}/members/${user} | jq
```
Success is HTTP `204`
##### Delete the team `$team` within the organization `$orga`
```
curl -X DELETE -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/${orga}/team/${team} | jq
```
success is HTTP `204`


## Users
##### Get all existing users
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/superuser/users/ | jq
```
##### Create a user `abc` with email `abc@abc.com`
Works only if quay is configured with `AUTHENTICATION_TYPE: database`
```
curl -X POST -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/superuser/users/ -H "Content-Type: application/json" --data '{"username": "abc", "email": "abc@abc.com"}' | jq
```
Success is HTTP `200`, user already exists or email is already in use is HTTP `400`
##### Get user information of user `$user`
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/users/${user} | jq
```
Success is HTTP `200`, no success is HTTP `404`

##### Delete user `$user` and all repositories owned by the user
Superusers can't be deleted! 
```
curl -X DELETE -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/superuser/users/$user | jq
```
Success is HTTP `200`, no success is HTTP `404`


## OrgRobots
##### Create the orgrobot `$robot` within the organization `$orga`
```
curl -X PUT -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/${orga}/robots/${robot} | jq
```
Success is HTTP `201`, no success is HTTP `400`
##### Get information about orgrobot `$robot` (including the robbot token) within the organization `$orga`
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/${orga}/robots/${robot} | jq
```
Success is HTTP `200`, no success is HTTP `400`
##### List all existing orgrobots within the organization `$orga`
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/${orga}/robots | jq
```
Success is HTTP `200`

All robots and their tokens are listed
##### Replace the current token of orgrobot `$robot` with a new token within the organization `$orga`
```
curl -X POST -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/${orga}/robots/${robot}/regenerate | jq
```
Success is HTTP `200`

This will delete the old token and all current logins of this robot will become invalid immediately
##### Get the current permissions of orgrobot `$robot` within the organization `$orga`
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/${orga}/robots/${robot}/permissions | jq
```
Success is HTTP `200`, no success is HTTP `400`


## UserRobots (needs to be reviewed!)
##### Set the permission `write` to orgrobot `$robot` for repository `$repo` within organization `$orga`
```
curl -X PUT -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/repository/orga/repo/permissions/user/orga+robby -H "Content-Type: application/json" --data '{"role": "write"}' | jq
```
Success is HTTP `200`, no success is HTTP `400`

Valid values are `read`, `write` and `admin`
##### Delete the current permission for the robot `robby` for repository `repo` within organization `orga`
```
curl -X DELETE -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/repository/orga/repo/permissions/user/orga+robby | jq
```
Success is HTTP `204`, no success is HTTP `400`


## Quotas
##### List quota of organization `$orga`
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" -H 'Content-Type: application/json' https://${quay_registry}/api/v1/organization/${orga}/quota | jq
```
Success is HTTP `200`, no success is HTTP `404`
##### Create a quota of 100MB for organization `$orga`
> 100MB = 104857600 bytes
```
curl -X POST -H "Authorization: Bearer ${bearer_token}" -H 'Content-Type: application/json' -d '{"limit_bytes": 104857600}' https://${quay_registry}/api/v1/organization/${orga}/quota | jq
```
Success is HTTP `201`, no success is HTTP `400`
##### Delete quota id 1 for organization `$orga`
```
curl -X DELETE -H "Authorization: Bearer ${bearer_token}" -H 'Content-Type: application/json' https://${quay_registry}/api/v1/organization/${orga}/quota/1 | jq
```
Success is HTTP `204`, no success is HTTP `400`


## Take ownership
##### Take the ownership of organization `$orga`
Token scope: *super:user*
```
curl -X POST -H "Authorization: Bearer ${bearer_token}" -H 'Content-Type: application/json' https://${quay_registry}/api/v1/superuser/takeownership/${orga} | jq
```
Success is HTTP `200`, no success because orgs does not exist is HTTP `404`, no success because of insufficient priviledges is HTTP `403`
