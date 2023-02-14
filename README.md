# quay-api
To use the api you have to have a bearer token in place with the correct scope
```
export bearer_token=XXXXXXXXXXXXXXXX
export quay_registry=REGISTRY_FQDN_OR_IP
export orga=ORGANIZATION_NAME
export repo=REPOSITORY_NAME
export team=TEAM_NAME
export user=USER_NAME
export robot=ROBOT_SHORT_NAME
```

***Tested with quay version 3.7.7***


## Organizations (Namespaces)
### List all existing `organizations`
> Token scope: *`super:user`*
```
curl -X GET \
     -H "Authorization: Bearer ${bearer_token}" \
     https://${quay_registry}/api/v1/superuser/organizations/ | jq
```
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/superuser/organizations/ | jq
```
### Get the details for the organization `$orga`
> Token scope: *`super:user` or `org:admin` or `repo:create`*
```
curl -X GET \
     -H "Authorization: Bearer ${bearer_token}" \
     https://${quay_registry}/api/v1/organization/${orga} | jq
```
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/${orga} | jq
```
Success is HTTP `200`, no success is HTTP `404`

### List the applications for the organization `$orga`
> Token scope: *`org:admin`*
```
curl -X GET \
     -H "Authorization: Bearer ${bearer_token}" \
     https://${quay_registry}/api/v1/organization/${orga}/applications | jq
```
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/${orga}/applications | jq
```
Success is HTTP `200`

### Delete the organization `$orga`
> Token scope: *`super:user` or `org:admin`*
```
curl -X DELETE \
     -H "Authorization: Bearer ${bearer_token}" \
     https://${quay_registry}/api/v1/organization/${orga} | jq
```
```
curl -X DELETE -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/${orga} | jq
```
Success is HTTP `204`, no success is HTTP `403`


## Repositories
### List all repositories the user who created the token has access to
```
curl -X GET \
     -H "Authorization: Bearer ${bearer_token}" \
     https://${quay_registry}/api/v1/repository?public=true | jq
```
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/repository?public=true | jq
```
### List all existing repositories in organization `$orga`
```
curl -X GET \
     -H "Authorization: Bearer ${bearer_token}" \
     https://${quay_registry}/api/v1/repository?namespace=${orga} | jq
```
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/repository?namespace=${orga} | jq
```
### Create the repository `$repo` in organization `$orga`
```
curl -X POST \
     -H "Authorization: Bearer ${bearer_token}" \
     -H "Content-Type: application/json" \
     --data '{\
              "namespace": "'$orga'", \
              "repository": "'$repo'", \
              "visibility": "public", \
              "description"" "", \
              "repo_kind": "image"              
             }' \
     https://${quay_registry}/api/v1/repository | jq
```
Success is HTTP `201`, no success is HTTP `400`

Required properties are `namespace`, `repository`, `visibility` and `descriprtion`

Optional property is `repo_kind` which defaults to `image`

- Valid values for `visibility` are `public` or `private`
- Valid values for `description` are `empty string` or `markdown encoded text`
- Valid values for `repo_kind` are `image` or `application`
### Change the visibility of the repository `$repo` within the organization `$orga` to `public`
```
curl -X POST \
     -H "Authorization: Bearer ${bearer_token}" \
     -H "Content-Type: application/json" \
     --data '{\
              "visibility": "public"\
             }' \
     https://${quay_registry}/api/v1/repository/${orga}/${repo}/changevisibility | jq
```
```
curl -X POST -H "Authorization: Bearer ${bearer_token}" -H "Content-Type: application/json" --data '{"visibility": "public"}' https://${quay_registry}/api/v1/repository/${orga}/${repo}/changevisibility | jq
```
Success is HTTP `200`, no success is HTTP `400`

Valid values are `public` or `private`

### Change the repository state to `Readonly` for repository `$repo` within the organization `$orga`
```
curl -X PUT \
     -H "Authorization: Bearer ${bearer_token}" \
     -H "Content-Type: application/json" \
     --data '{\
              "state": "READ_ONLY"\
             }' \
     https://${quay_registry}/api/v1/repository/${orga}/${repo}/changestate | jq
```
```
curl -X PUT -H "Authorization: Bearer ${bearer_token}" -H "Content-Type: application/json" --data '{"state": "READ_ONLY"}' https://${quay_registry}/api/v1/repository/${orga}/${repo}/changestate | jq
```
Success is HTTP `200`

Valid values are `NORMAL`, `READ_ONLY` and `MIRROR`... take care it's case sensitive!


## Mirrored repositories (needs to be reviewed!)
### Get the existing mirror config from the repository `$repo` within the organization `$orga`
```
curl -X GET \
     -H "Authorization: Bearer ${bearer_token}" \
     https://${quay_registry}/api/v1/repository/${orga}/${repo}/mirror | jq
```
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/repository/${orga}/${repo}/mirror | jq
```
Success is HTTP `200`, no success is HTTP `404` if there's no mirror config
### Create a a mirror config for the mirrored repository `$repo` within the organization `$orga` (quay.io/minio/mc:latest)
The used robot `orga+robot` must already exist!

Required values to provide are
- `external_reference`
- `sync_interval`
- `sync_start_date`
- `root_rule`
  - `rule_kind` is always `tag_glob_csv`
  - `rule_value` is a list of tags to sync... `*` is only valid if there's the tag `latest` at the source. If this is not the case define an existing tag and *

Optional values are
- `is_enabled` if set to `true` the repository state will be switched to `MIRROR`
- `external_registry_username`
- `external_registry_password`
- `external_registry_config`
  - `unsigned_images`
  - `verify_tls`
  - `proxy`
    - `http_proxy`
    - `https_proxy`
    - `no_proxy`
- `sync_expiration_date`

Minimal config:
```
curl -X POST \
     -H "Authorization: Bearer ${bearer_token}" \
     -H "Content-Type: application/json" \
     --data '{\
              "external_reference": "quay.io/minio/mc", \
              "external_registry_username": "", \
              "sync_interval": 600, \
              "sync_start_date": "2021-08-06T11:11:39Z", \
              "root_rule": {\
                            "rule_kind": "tag_glob_csv", \
                            "rule_value": [ "latest" ]\
                           }, \
              "robot_username": "orga+robot"\
             }' \
     https://${quay_registry}/api/v1/repository/${orga}/${repo}/mirror | jq
```
```
curl -X POST -H "Authorization: Bearer ${bearer_token}" -H "Content-Type: application/json" --data '{"external_reference": "quay.io/minio/mc", "external_registry_username": "", "sync_interval": 600, "sync_start_date": "2021-08-06T11:11:39Z", "root_rule": {"rule_kind": "tag_glob_csv", "rule_value": [ "latest" ]}, "robot_username": "orga+robot"}' https://${quay_registry}/api/v1/repository/${orga}/${repo}/mirror | jq
```
Extended config
```
curl -X POST -H "Authorization: Bearer ${bearer_token}" -H "Content-Type: application/json" --data '{"is_enabled": true, "external_reference": "quay.io/minio/mc", "external_registry_username": "username", "external_registry_password": "password", "external_registry_config": {"unsigned_images":true, "verify_tls": false, "proxy": {"http_proxy": "http://proxy.tld", "https_proxy": "https://proxy.tld", "no_proxy": "domain"}}, "sync_interval": 600, "sync_start_date": "2021-08-06T11:11:39Z", "root_rule": {"rule_kind": "tag_glob_csv", "rule_value": [ "*" ]}, "robot_username": "orga+robot"}' https://${quay_registry}/api/v1/repository/${orga}/${repo}/mirror | jq
```
```
curl -X POST \
     -H "Authorization: Bearer ${bearer_token}" \
     -H "Content-Type: application/json" \
     --data '{\
              "is_enabled": true, \
              "external_reference": "quay.io/minio/mc", \
              "external_registry_username": "username", \
              "external_registry_password": "password", \
              "external_registry_config": {\
                                           "unsigned_images":true, \
                                           "verify_tls": false, \
                                           "proxy": {\
                                                     "http_proxy": "http://proxy.tld", \
                                                     "https_proxy": "https://proxy.tld", \
                                                     "no_proxy": "domain"\
                                                     }\
                                          }, \
                                          "sync_interval": 600, \
                                          "sync_start_date": "2021-08-06T11:11:39Z", \
                                          "root_rule": {\
                                                        "rule_kind": "tag_glob_csv", \
                                                        "rule_value": [ "*" ]\
                                                       }, \
                                          "robot_username": "orga+robot"\
             }' \
     https://${quay_registry}/api/v1/repository/${orga}/${repo}/mirror | jq
```
Success is HTTP `201`, no success is HTTP `409` if a mirror config already exists
### Update a part of the mirror config of the mirrored repository `$repo` within the organization `$orga` (quay.io/minio/mc:latest)
Here only the sync interval will be updated
```
curl -X PUT \
     -H "Authorization: Bearer ${bearer_token}" \
     -H "Content-Type: application/json" \
     --data '{\
              "sync_interval": 1260\
             }' \
     https://${quay_registry}/api/v1/repository/${orga}/${repo}/mirror | jq
```
```
curl -X PUT -H "Authorization: Bearer ${bearer_token}" -H "Content-Type: application/json" --data '{"sync_interval": 1260}' https://${quay_registry}/api/v1/repository/${orga}/${repo}/mirror | jq
```
Success is HTTP `201`, no sucess is HTTP `404` if ther is no mirror config
### Initiate a mirror sync action of the mirrored repository `$repo` within the organization `$orga`
```
curl -X POST \
     -H "Authorization: Bearer ${bearer_token}" \
     https://${quay_registry}/api/v1/repository/${orga}/${repo}/mirror/sync-now | jq
```
```
curl -X POST -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/repository/${orga}/${repo}/mirror/sync-now | jq
```


## Teams
### List all teams within organization `$orga` (and additional info)
```
curl -X GET \
     -H "Authorization: Bearer ${bearer_token}" \
     https://${quay_registry}/api/v1/organization/${orga} | jq
```
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/${orga} | jq
```
### Create a team `$team` within the organization `$orga` with role `member`
Required value to provide is
- `role` valid values are `member`, `creator` or `admin` 
```
curl -X PUT \
     -H "Authorization: Bearer ${bearer_token}" \
     -H "Content-Type: application/json" \
     --data '{\
              "role": "member", \
              "description": "Short description of the team"\
             }' \
     https://${quay_registry}/api/v1/organization/${orga}/team/${team} | jq
```
```
curl -X PUT -H "Authorization: Bearer ${bearer_token}" -H "Content-Type: application/json" --data '{"role": "member", "description": "Short description of the team"}' https://${quay_registry}/api/v1/organization/${orga}/team/${team} | jq
```
success is HTTP `200`
### Change the role to `admin` of the team `$team` within the organization `$orga`
valid values for `role` are `member`, `creator` or `admin` 
```
curl -X PUT \
     -H "Authorization: Bearer ${bearer_token}" \
     -H "Content-Type: application/json" \
     --data '{\
              "role": "admin"\
             }' \
     https://${quay_registry}/api/v1/repository/orga/repo/permissions/team/orgateam | jq
```
```
curl -X PUT -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/repository/orga/repo/permissions/team/orgateam -H "Content-Type: application/json" --data '{"role": "admin"}' | jq
```
Success is HTTP`200`
### Set or change the description of the team `$team` within the organization `$orga` using standard text
Required value to provide is
- `role` valid values are `member`, `creator` or `admin`
- `description`
```
curl -X PUT \
     -H "Authorization: Bearer ${bearer_token}" \
     -H "Content-Type: application/json" \
     --data '{\
              "role": "member", \
              "description": "Short description of the team"\
             }' \
     https://${quay_registry}/api/v1/organization/${orga}/team/${team} | jq
```
```
curl -X PUT -H "Authorization: Bearer ${bearer_token}" -H "Content-Type: application/json" --data '{"role": "member", "description": "Short description of the team"}' https://${quay_registry}/api/v1/organization/${orga}/team/${team} | jq
```
success is HTTP `200`
### Set or change the description of the team `$team` within the organization `$orga` using markdown text
Required value to provide is
- `role` valid values are `member`, `creator` or `admin`
- `description`
```
curl -X PUT \
     -H "Authorization: Bearer ${bearer_token}" \
     -H "Content-Type: application/json" \
     --data '{\
              "role": "member", \
              "description": "# Caption\nShort description of the *team*"\
             }' \
     https://${quay_registry}/api/v1/organization/${orga}/team/${team} | jq
```
```
curl -X PUT -H "Authorization: Bearer ${bearer_token}" -H "Content-Type: application/json" --data '{"role": "member", "description": "# Caption\nShort description of the *team*"}' https://${quay_registry}/api/v1/organization/${orga}/team/${team} | jq
```
success is HTTP `200`
### Add the user `$user` as member to the team `$team` in the organization `$orga`
```
curl -X PUT \
     -H "Authorization: Bearer ${bearer_token}" \
     https://${quay_registry}/api/v1/organization/${orga}/team/${team}/members/${user} | jq
```
```
curl -X PUT -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/${orga}/team/${team}/members/${user} | jq
```
Success is HTTP `200`
### Get all members of the team `$team` in organization `$orga`
```
curl -X GET \
     -H "Authorization: Bearer ${bearer_token}" \
     https://${quay_registry}/api/v1/organization/${orga}/team/${team}/members | jq
```
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/${orga}/team/${team}/members | jq
```
### Remove the user `$user` as member of the team `$team` within the organization `$orga`
```
curl -X DELETE \
     -H "Authorization: Bearer ${bearer_token}" \
     https://${quay_registry}/api/v1/organization/${orga}/team/${team}/members/${user} | jq
```
```
curl -X DELETE -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/${orga}/team/${team}/members/${user} | jq
```
Success is HTTP `204`
### Delete the team `$team` within the organization `$orga`
```
curl -X DELETE \
     -H "Authorization: Bearer ${bearer_token}" \
     https://${quay_registry}/api/v1/organization/${orga}/team/${team} | jq
```
```
curl -X DELETE -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/${orga}/team/${team} | jq
```
success is HTTP `204`


## Users
### Get all existing users
Works only if quay is configured with `AUTHENTICATION_TYPE: database`
```
curl -X GET \
     -H "Authorization: Bearer ${bearer_token}" \
     https://${quay_registry}/api/v1/superuser/users/ | jq
```
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/superuser/users/ | jq
```
### Create a user `abc` with email `abc@abc.com`
Works only if quay is configured with `AUTHENTICATION_TYPE: database`
```
curl -X POST \
     -H "Authorization: Bearer ${bearer_token}" \
     -H "Content-Type: application/json" \
     --data '{\
              "username": "abc", \
              "email": "abc@abc.com"\
             }' \
     https://${quay_registry}/api/v1/superuser/users/ | jq
```
```
curl -X POST -H "Authorization: Bearer ${bearer_token}" -H "Content-Type: application/json" --data '{"username": "abc", "email": "abc@abc.com"}' https://${quay_registry}/api/v1/superuser/users/ | jq
```
Success is HTTP `200`, user already exists or email is already in use is HTTP `400`
### Get user information of user `$user`
```
curl -X GET \
     -H "Authorization: Bearer ${bearer_token}" \
     https://${quay_registry}/api/v1/users/${user} | jq
```
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/users/${user} | jq
```
Success is HTTP `200`, no success is HTTP `404`

### Delete user `$user` and all repositories owned by the user
Superusers can't be deleted! 
```
curl -X DELETE \
     -H "Authorization: Bearer ${bearer_token}" \
     https://${quay_registry}/api/v1/superuser/users/$user | jq
```
```
curl -X DELETE -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/superuser/users/$user | jq
```
Success is HTTP `200`, no success is HTTP `404`


## OrgRobots
### Create the orgrobot `$robot` within the organization `$orga`
```
curl -X PUT \
     -H "Authorization: Bearer ${bearer_token}" \
     https://${quay_registry}/api/v1/organization/${orga}/robots/${robot} | jq
```
```
curl -X PUT -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/${orga}/robots/${robot} | jq
```
Success is HTTP `201`, no success is HTTP `400`
### Get information about orgrobot `$robot` (including the robbot token) within the organization `$orga`
```
curl -X GET \
     -H "Authorization: Bearer ${bearer_token}" \
     https://${quay_registry}/api/v1/organization/${orga}/robots/${robot} | jq
```
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/${orga}/robots/${robot} | jq
```
Success is HTTP `200`, no success is HTTP `400`
### List all existing orgrobots within the organization `$orga`
```
curl -X GET \
     -H "Authorization: Bearer ${bearer_token}" \
     https://${quay_registry}/api/v1/organization/${orga}/robots | jq
```
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/${orga}/robots | jq
```
Success is HTTP `200`

All robots and their tokens are listed
### Replace the current token of orgrobot `$robot` with a new token within the organization `$orga`
```
curl -X POST \
     -H "Authorization: Bearer ${bearer_token}" \
     https://${quay_registry}/api/v1/organization/${orga}/robots/${robot}/regenerate | jq
```
```
curl -X POST -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/${orga}/robots/${robot}/regenerate | jq
```
Success is HTTP `200`

This will delete the old token and all current logins of this robot will become invalid immediately
### Get the current permissions of orgrobot `$robot` within the organization `$orga`
```
curl -X GET \
     -H "Authorization: Bearer ${bearer_token}" \
     https://${quay_registry}/api/v1/organization/${orga}/robots/${robot}/permissions | jq
```
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/${orga}/robots/${robot}/permissions | jq
```
Success is HTTP `200`, no success is HTTP `400`


## UserRobots (needs to be reviewed!)
### Set the permission `write` to orgrobot `$robot` for repository `$repo` within organization `$orga`
```
curl -X PUT \
     -H "Authorization: Bearer ${bearer_token}" \
     -H "Content-Type: application/json" \
     --data '{\
              "role": "write"\
             }' \
     https://${quay_registry}/api/v1/repository/orga/repo/permissions/user/orga+robby | jq
```
```
curl -X PUT -H "Authorization: Bearer ${bearer_token}" -H "Content-Type: application/json" --data '{"role": "write"}' https://${quay_registry}/api/v1/repository/orga/repo/permissions/user/orga+robby | jq
```
Success is HTTP `200`, no success is HTTP `400`

Valid values are `read`, `write` and `admin`
### Delete the current permission for the robot `robby` for repository `repo` within organization `orga`
```
curl -X DELETE \
     -H "Authorization: Bearer ${bearer_token}" \
     https://${quay_registry}/api/v1/repository/orga/repo/permissions/user/orga+robby | jq
```
```
curl -X DELETE -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/repository/orga/repo/permissions/user/orga+robby | jq
```
Success is HTTP `204`, no success is HTTP `400`


## Quotas
### List quota of organization `$orga`
```
curl -X GET \
     -H "Authorization: Bearer ${bearer_token}" \
     -H 'Content-Type: application/json' \
     https://${quay_registry}/api/v1/organization/${orga}/quota | jq
```
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" -H 'Content-Type: application/json' https://${quay_registry}/api/v1/organization/${orga}/quota | jq
```

Success is HTTP `200`, no success is HTTP `404`
### Create a quota of 100MB for organization `$orga`
> *100MB = 104857600 bytes*
```
curl -X POST \
     -H "Authorization: Bearer ${bearer_token}" \
     -H 'Content-Type: application/json' \
     -data '{\
             "limit_bytes": 104857600\
            }' \
     https://${quay_registry}/api/v1/organization/${orga}/quota | jq
```
```
curl -X POST -H "Authorization: Bearer ${bearer_token}" -H 'Content-Type: application/json' -data '{"limit_bytes": 104857600}' https://${quay_registry}/api/v1/organization/${orga}/quota | jq
```
Success is HTTP `201`, no success is HTTP `400`
### Delete quota id 1 for organization `$orga`
```
curl -X DELETE \
     -H "Authorization: Bearer ${bearer_token}" \
     -H 'Content-Type: application/json' \
     https://${quay_registry}/api/v1/organization/${orga}/quota/1 | jq
```
```
curl -X DELETE -H "Authorization: Bearer ${bearer_token}" -H 'Content-Type: application/json' https://${quay_registry}/api/v1/organization/${orga}/quota/1 | jq
```
Success is HTTP `204`, no success is HTTP `400`


## Take ownership
### Take the ownership of organization `$orga`
> Token scope: *`super:user`*
```
curl -X POST \
     -H "Authorization: Bearer ${bearer_token}" \
     -H 'Content-Type: application/json' \
     https://${quay_registry}/api/v1/superuser/takeownership/${orga} | jq
```
```
curl -X POST -H "Authorization: Bearer ${bearer_token}" -H 'Content-Type: application/json' https://${quay_registry}/api/v1/superuser/takeownership/${orga} | jq
```

Success is HTTP `200`, no success because orgs does not exist is HTTP `404`, no success because of insufficient priviledges is HTTP `403`
