# quay-api
To use the api you have to have a bearer token in place with the correct scope
```
export bearer_token=XXXXXXXXXXXXXXXX
export quay_registry=REGISTRY_FQDN_OR_IP
export orga=ORGANIZATION_NAME
export email=email@domain.tld
export repo=REPOSITORY_NAME
export policy_uuid=POLICY_UUID
export tag=TAG
export newtag=NEWTAG
export manifest_digest=MANIFEST_DIGEST
export team=TEAM_NAME
export user=USER_NAME
export robot=ROBOT_SHORT_NAME
```

***Tested with quay version 3.11.0***

Table of contents
=================
<!--ts-->
   * [Organizations](#organizations-namespaces)
     * [List all existing `organizations`](#list-all-existing-organizations)
     * [Create the organization `$orga`](#create-the-organization-orga)
     * [Get the details for the organization `$orga`](#get-the-details-for-the-organization-orga)
     * [List the applications for the organization `$orga`](#list-the-applications-for-the-organization-orga)
     * [Delete the organization `$orga`](#delete-the-organization-orga)
   * [Prototypes (Default permissions)](#prototypes-default-permissions)
     * [Get current existing permission protoypes](#get-current-existing-permission-protoypes)
     * [Create a permission prototype for team `$team` to `admin`](#create-a-permission-prototype-for-team-team-to-admin)
     * [Create a permission prototype for user `$user` to `write`](#create-a-permission-prototype-for-user-user-to-write)
     * [Create a permission prototype for robot `$robot` to `read`](#create-a-permission-prototype-for-robot-robot-to-read)
     * [Delete a prototype permission](#delete-a-prototype-permission)
   * [Organization auto-prune policy (Namespaces))](#organization-auto-prune-policy-namespaces)
     * [List the current auto-prune policy for organization `$orga`](#list-the-current-auto-prune-policy-for-organization-orga)
     * [Get the uuid of the existing auto-prune policy for organization `$orga`](#get-the-uuid-of-the-existing-auto-prune-policy-for-organization-orga)
     * [Set the auto-prune policy for organization `$orga`](#set-the-auto-prune-policy-for-organization-orga)
     * [Modify the existing auto-prune policy for organization `$orga`](#modify-the-existing-auto-prune-policy-for-organization-orga)
     * [Delete the existing auto-prune policy for organization `$orga`](#delete-the-existing-auto-prune-policy-for-organization-orga)
   * [User auto-prune policy (Current user)](#user-auto-prune-policy-current-user)
     * [List the current auto-prune policy for the current user](#list-the-current-auto-prune-policy-for-the-current-user)
     * [Get the uuid of the existing auto-prune policy for the current user](#get-the-uuid-of-the-existing-auto-prune-policy-for-the-current-user)
     * [Set the auto-prune policy for the current user](#set-the-auto-prune-policy-for-the-current-user)
     * [Modify the existing auto-prune policy for the current user](#modify-the-existing-auto-prune-policy-for-the-current-user)
     * [Delete the existing auto-prune policy for the current user](#delete-the-existing-auto-prune-policy-for-the-current-user)
   * [Repository auto-prune policy](#repository-auto-prune-policy)
     * [List the current auto-prune policy for repository `$repo` within the organization `$orga`](#list-the-current-auto-prune-policy-for-repository-repo-within-the-organization-orga)
     * [Get the uuid of the existing auto-prune policy for repository `$repo` within the organization `$orga`](#get-the-uuid-of-the-existing-auto-prune-policy-for-repository-repo-within-the-organization-orga)
     * [Set the auto-prune policy for repository `$repo` within the organization `$orga`](#set-the-auto-prune-policy-for-repository-repo-within-the-organization-orga)
     * [Modify the existing auto-prune policy for repository `$repo` within the organization `$orga`](#modify-the-existing-auto-prune-policy-for-repository-repo-within-the-organization-orga)
     * [Delete the existing auto-prune policy for repository `$repo` within the organization `$orga`](#delete-the-existing-auto-prune-policy-for-repository-repo-within-the-organization-orga)
   * [Repositories](#repositories)
     * [List all repositories the user who created the token has access to](#list-all-repositories-the-user-who-created-the-token-has-access-to)
     * [List all existing repositories in organization `$orga`](#list-all-existing-repositories-in-organization-orga)
     * [Create the repository `$repo` in organization `$orga`](#create-the-repository-repo-in-organization-orga)
     * [Change the visibility of the repository `$repo` within the organization `$orga` to `public`](#change-the-visibility-of-the-repository-repo-within-the-organization-orga-to-public)
     * [Change the repository state to `Readonly` for repository `$repo` within the organization `$orga`](#change-the-repository-state-to-readonly-for-repository-repo-within-the-organization-orga)
   * [Mirrored repositories](#mirrored-repositories-needs-to-be-reviewed)
      * [Get the existing mirror config from the repository `$repo` within the organization `$orga`](#get-the-existing-mirror-config-from-the-repository-repo-within-the-organization-orga)
      * [Create a a mirror config for the mirrored repository `$repo` within the organization `$orga` (quay.io/minio/mc:latest)](https://github.com/lbohnsac/quay-api?tab=readme-ov-file#create-a-a-mirror-config-for-the-mirrored-repository-repo-within-the-organization-orga-quayiominiomclatest)
      * [Update a part of the mirror config of the mirrored repository `$repo` within the organization `$orga` (quay.io/minio/mc:latest)](#update-a-part-of-the-mirror-config-of-the-mirrored-repository-repo-within-the-organization-orga-quayiominiomclatest)
      * [Initiate a mirror sync action of the mirrored repository `$repo` within the organization `$orga`](https://github.com/lbohnsac/quay-api?tab=readme-ov-file#initiate-a-mirror-sync-action-of-the-mirrored-repository-repo-within-the-organization-orga)
   * [Tags](#tags)
      * [List all tags in repository `$repo` within organization `$orga`](#list-all-tags-in-repository-repo-within-organization-orga)
      * [List all ***active*** tags in repository `$repo` within organization `$orga`](#list-all-active-tags-in-repository-repo-within-organization-orga)
      * [Create a new tag `$newtag` for manifest digest `$manifest_digest` in reopsitory `$repo` within organization `$orga`](#create-a-new-tag-newtag-for-manifest-digest-manifest_digest-in-reopsitory-repo-within-organization-orga)
      * [Set or change the expiration date to `Tue May 2 16:33:45 CEST 2023` of tag `$tag` for repository `$repo` in organization `$orga]`](#set-or-change-the-expiration-date-to-tue-may-2-163345-cest-2023-of-tag-tag-for-repository-repo-in-organization-orga)
      * [Reset the expiration date to `Never` (delete the expiration date) of tag `$tag` in reopsitory `$repo` within organization `$orga`](#reset-the-expiration-date-to-never-delete-the-expiration-date-of-tag-tag-in-reopsitory-repo-within-organization-orga)
      * [Delete tag `$tag` in repository `$repo` within organization `$orga`](#delete-tag-tag-in-repository-repo-within-organization-orga)
   * [Teams](#teams)
   * [Users](#users)
   * [OrgRobots](#orgrobots)
   * [UserRobots](#userrobots-needs-to-be-reviewed)
   * [Quotas](#quotas)
       * [List quota of organization `$orga`](#list-quota-of-organization-orga)
       * [Create a quota of 100MB for organization `$orga`](#create-a-quota-of-100mb-for-organization-orga)
       * [Delete quota id 1 for organization `$orga`](#delete-quota-id-1-for-organization-orga)
   * [Take ownership](#take-ownership)
       * [Take the ownership of organization `$orga`]([#take-the-ownership-of-organization-orga)
<!--te-->

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
### Create the organization `$orga`
```
curl -X POST \
     -H "Authorization: Bearer ${bearer_token}" \
     -H "Content-Type: application/json" \
     --data '{\
              "name": "${orga}", \
              "email": "${email}" \
             }' \
     https://${quay_registry}/api/v1/organization/ | jq
```
```
curl -X POST -H "Authorization: Bearer ${bearer_token}" -H "Content-Type: application/json" --data '{"name": "${orga}"}, "email": "${email}"' https://${quay_registry}/api/v1/organization/ | jq
```
Success is HTTP `201`
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


## Prototypes (Default permissions)
> Default permissions are granted automatically to newly created repositories, in addition to the default of the repository's creator.
Default permissions are not added to already existing repositories.
### Get current existing permission protoypes
```
curl -X GET \
     -H "Authorization: Bearer ${bearer_token}" \
     https://${quay_registry}/api/v1/organization/${orga}/prototypes | jq
```
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/${orga}/prototypes | jq
```
Success is HTTP`200`
### Create a permission prototype for team `$team` to `admin`
```
curl -X POST \
     -H "Authorization: Bearer ${bearer_token}" \
     -H "Content-Type: application/json" \
     --data '{\
              "delegate": {\
                           "name": "${team}", \
                           "kind": "team" \
                           }, \
              "role": "admin"              
             }' \
     https://${quay_registry}/api/v1/organization/${orga}/prototypes | jq
```
```
curl -X POST -H "Authorization: Bearer ${bearer_token}" -H "Content-Type: application/json" --data '{"delegate":{"name": "${team}","kind": "team"}, "role": "admin"}' https://${quay_registry}/api/v1/organization/${orga}/prototypes | jq
```

Required properties are `delegate` and `role`

Optional properties is `activating_user`

Property `delegate` requires `name` and `kind`
Property `activating_user` requires `name`

- Valid values for `role` are `admin`, `write` and `read`
- Valid values for kind are `team` and `user`

Success is HTTP`200`
### Create a permission prototype for user `$user` to `write`
```
curl -X POST \
     -H "Authorization: Bearer ${bearer_token}" \
     -H "Content-Type: application/json" \
     --data '{\
              "delegate": {\
                           "name": "${user}", \
                           "kind": "user" \
                           }, \
              "role": "write"              
             }' \
     https://${quay_registry}/api/v1/organization/${orga}/prototypes | jq
```
```
curl -X POST -H "Authorization: Bearer ${bearer_token}" -H "Content-Type: application/json" --data '{"delegate":{"name": "${user}","kind": "user"}, "role": "write"}' https://${quay_registry}/api/v1/organization/${orga}/prototypes | jq
```

Required properties are `delegate` and `role`

Optional properties is `activating_user`

Property `delegate` requires `name` and `kind`
Property `activating_user` requires `name`

- Valid values for `role` are `admin, `write` and `read`
- Valid values for kind are `team` and `user`

Success is HTTP`200`
### Create a permission prototype for robot `$robot` to `read`
A robot is a user with the robot acount name (`${orga}+${robot}`)
```
curl -X POST \
     -H "Authorization: Bearer ${bearer_token}" \
     -H "Content-Type: application/json" \
     --data '{\
              "delegate": {\
                           "name": "'${orga}+${robot}'", \
                           "kind": "user" \
                           }, \
              "role": "read"              
             }' \
     https://${quay_registry}/api/v1/organization/${orga}/prototypes | jq
```
```
curl -X POST -H "Authorization: Bearer ${bearer_token}" -H "Content-Type: application/json" --data '{"delegate":{"name": "'${orga}+${robot}'","kind": "user"}, "role": "read"}' https://${quay_registry}/api/v1/organization/${orga}/prototypes | jq
```

Required properties are `delegate` and `role`

Optional properties is `activating_user`

Property `delegate` requires `name` and `kind`
Property `activating_user` requires `name`

- Valid values for `role` are `admin, `write` and `read`
- Valid values for kind are `team` and `user`

Success is HTTP`200`
### Delete a prototype permission
Get `${prototypes_id}` by querying the prototypes
```
curl -X DELETE \
     -H "Authorization: Bearer ${bearer_token}" \
     https://${quay_registry}/api/v1/organization/${orga}/prototypes/${prototypes_id} | jq
```
```
curl -X DELETE -H "Authorization: Bearer ${bearer_token} https://${quay_registry}/api/v1/organization/${orga}/prototypes/${prototypes_id} | jq
```
Success is HTTP`204`

## Organization auto-prune policy (Namespaces)
> Auto-pruning policies for organizations are available since quay `v3.10.0`
> 
> Auto-pruning policies work at the organization level and apply to **all** repositories of this organization.
> 
> There can only be one policy at a time.

### List the current auto-prune policy for organization `$orga`
```
curl -X GET \
     -H "Authorization: Bearer ${bearer_token}" \
     https://${quay_registry}/api/v1/organization/${orga}/autoprunepolicy/ | jq
```
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/${orga}/autoprunepolicy/ | jq
```
Success is HTTP `200`

If there is no auto-prune policy set the response will be an empty array `policies`. E.g.`{"policies": []}`

If there is an existing auto-prune policy the response is the array `policies` containing the current set policy. E.g. `{"policies": [{"uuid": "cb44b3b7-bca6-42e4-8eaf-4350f09b5a63", "method": "number_of_tags", "value": 10}]}`

### Get the uuid of the existing auto-prune policy for organization `$orga`
```
curl -X GET \
     -H "Authorization: Bearer ${bearer_token}" \
     https://${quay_registry}/api/v1/organization/${orga}/autoprunepolicy/ | jq -r '.policies[].uuid'
```
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/${orga}/autoprunepolicy/ | jq -r '.policies[].uuid'
```
Success is HTTP `200`

### Set the auto-prune policy for organization `$orga`
```
curl -X POST \
     -H "Authorization: Bearer ${bearer_token}" \
     -H "Content-Type: application/json" \
     --data '{ \
              "method": "creation_date", \
              "value": "30d" \
             }' \
     https://${quay_registry}/api/v1/organization/ ${orga}/autoprunepolicy/ | jq
```
```
curl -X POST -H "Authorization: Bearer ${bearer_token}" -H "Content-Type: application/json" --data '{"method": "creation_date", "value": "30d"}' https://${quay_registry}/api/v1/organization/${orga}/autoprunepolicy/ | jq
```
Success is HTTP `201`, no success is HTTP `400`

Valid `method` values are
- `creation_date`
- `number_of_tags`.

Method `number_of_tags` expects an integer.

Method `creation_date` expects a string containing the amount of time the image is not deleted since creation. Valid options are
- `s` seconds
- `m` minutes
- `h` hours
- `d` days
- `w` weeks
  
 E.g. `30d`.

### Modify the existing auto-prune policy for organization `$orga`
> First, get the current uuid (${policy_uuid}) to modify it
```
curl -X PUT \
     -H "Authorization: Bearer ${bearer_token}" \
     -H "Content-Type: application/json" \
     --data '{ \
              "method": "creation_date", \
              "value": "7d" \
             }' \
     https://${quay_registry}/api/v1/organization/${orga}/autoprunepolicy/${policy_uuid} | jq
```
```
curl -X PUT -H "Authorization: Bearer ${bearer_token}" -H "Content-Type: application/json" --data '{"method": "creation_date"}, "value": "7d"' https://${quay_registry}/api/v1/organization/${orga}/autoprunepolicy/${policy_uuid} | jq
```
Success is HTTP `204`, no success is HTTP `404`

Successful modification has no response!

### Delete the existing auto-prune policy for organization `$orga`
> First, get the current uuid (${policy_uuid}) to delete it
```
curl -X DELETE \
     -H "Authorization: Bearer ${bearer_token}" \
     https://${quay_registry}/api/v1/organization/${orga}/autoprunepolicy/${policy_uuid} | jq
```
```
curl -X DELETE -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/${orga}/autoprunepolicy/${policy_uuid} | jq
```
Success is HTTP `200`, no success is HTTP `404`

A successful deletion will return the deleted uuid. E.g. `{"uuid": "cb44b3b7-bca6-42e4-8eaf-4350f09b5a63"}`

## User auto-prune policy (Current user)
> Auto-pruning policies for the current user are available since quay `v3.10.0`
>
> **The current user is the user who created the bearer token in use.**
> 
> There can only be one policy at a time.

### List the current auto-prune policy for the current user
```
curl -X GET \
     -H "Authorization: Bearer ${bearer_token}" \
     https://${quay_registry}/api/v1/user/autoprunepolicy/ | jq
```
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/user/autoprunepolicy/ | jq
```
Success is HTTP `200`

If there is no auto-prune policy set the response will be an empty array `policies`. E.g.`{"policies": []}`

If there is an existing auto-prune policy the response is the array `policies` containing the current set policy. E.g. `{"policies": [{"uuid": "cb44b3b7-bca6-42e4-8eaf-4350f09b5a63", "method": "number_of_tags", "value": 10}]}`

### Get the uuid of the existing auto-prune policy for the current user
```
curl -X GET \
     -H "Authorization: Bearer ${bearer_token}" \
     https://${quay_registry}/api/v1/user/autoprunepolicy/ | jq -r '.policies[].uuid'
```
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/user/autoprunepolicy/ | jq -r '.policies[].uuid'
```
Success is HTTP `200`
### Set the auto-prune policy for the current user
```
curl -X POST \
     -H "Authorization: Bearer ${bearer_token}" \
     -H "Content-Type: application/json" \
     --data '{ \
              "method": "creation_date", \
              "value": "30d" \
             }' \
     https://${quay_registry}/api/v1/user/autoprunepolicy/ | jq
```
```
curl -X POST -H "Authorization: Bearer ${bearer_token}" -H "Content-Type: application/json" --data '{"method": "creation_date", "value": "30d"}' https://${quay_registry}/api/v1/user/autoprunepolicy/ | jq
```
Success is HTTP `201`, no success is HTTP `400`

Valid `method` values are
- `creation_date`
- `number_of_tags`.

Method `number_of_tags` expects an integer.

Method `creation_date` expects a string containing the amount of time the image is not deleted since creation. Valid options are
- `s` seconds
- `m` minutes
- `h` hours
- `d` days
- `w` weeks
  
 E.g. `30d`.

### Modify the existing auto-prune policy for the current user
> First, get the current uuid (${policy_uuid}) to modify it
```
curl -X PUT \
     -H "Authorization: Bearer ${bearer_token}" \
     -H "Content-Type: application/json" \
     --data '{ \
              "method": "creation_date", \
              "value": "7d" \
             }' \
     https://${quay_registry}/api/v1/user/autoprunepolicy/${policy_uuid} | jq
```
```
curl -X PUT -H "Authorization: Bearer ${bearer_token}" -H "Content-Type: application/json" --data '{"method": "creation_date", "value": "7d"}' https://${quay_registry}/api/v1/user/autoprunepolicy/${policy_uuid} | jq
```
Success is HTTP `204`, no success is HTTP `404`

Successful modification has no response!

### Delete the existing auto-prune policy for the current user
> First, get the current uuid (${policy_uuid}) to delete it
```
curl -X DELETE \
     -H "Authorization: Bearer ${bearer_token}" \
     https://${quay_registry}/api/v1/user/autoprunepolicy/${policy_uuid} | jq
```
```
curl -X DELETE -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/user/autoprunepolicy/${policy_uuid} | jq
```
Success is HTTP `200`, no success is HTTP `404`

A successful deletion will return the deleted uuid. E.g. `{"uuid": "cb44b3b7-bca6-42e4-8eaf-4350f09b5a63"}`

## Repository auto-prune policy
> Auto-pruning policies for repositories are available since quay `v3.11.0`
> 
> Auto-pruning policies work at the repository level.
> 
> There can only be one policy at a time.

### List the current auto-prune policy for repository `$repo` within the organization `$orga`
```
curl -X GET \
     -H "Authorization: Bearer ${bearer_token}" \
     https://${quay_registry}/api/v1/repository/${orga}/${repo}/autoprunepolicy/ | jq
```
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/repository/${orga}/${repo}/autoprunepolicy/ | jq
```
Success is HTTP `200`

If there is no auto-prune policy set the response will be an empty array `policies`. E.g.`{"policies": []}`

If there is an existing auto-prune policy the response is the array `policies` containing the current set policy. E.g. `{"policies": [{"uuid": "cb44b3b7-bca6-42e4-8eaf-4350f09b5a63", "method": "number_of_tags", "value": 10}]}`
### Get the uuid of the existing auto-prune policy for repository `$repo` within the organization `$orga`
```
curl -X GET \
     -H "Authorization: Bearer ${bearer_token}" \
     https://${quay_registry}/api/v1/repository/${orga}/${repo}/autoprunepolicy/ | jq -r '.policies[].uuid'
```
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/repository/${orga}/${repo}/autoprunepolicy/ | jq -r '.policies[].uuid'
```
Success is HTTP `200`

### Set the auto-prune policy for repository `$repo` within the organization `$orga`
```
curl -X POST \
     -H "Authorization: Bearer ${bearer_token}" \
     -H "Content-Type: application/json" \
     --data '{ \
              "method": "creation_date", \
              "value": "30d" \
             }' \
     https://${quay_registry}/api/v1/repository/${orga}/${repo}/autoprunepolicy/ | jq
```
```
curl -X POST -H "Authorization: Bearer ${bearer_token}" -H "Content-Type: application/json" --data '{"method": "creation_date", "value": "30d"}' https://${quay_registry}/api/v1/repository/${orga}/${repo}/autoprunepolicy/ | jq
```
Success is HTTP `201`, no success is HTTP `400`

The response is the uuid for the created policy. E.g. `{"uuid": "cb44b3b7-bca6-42e4-8eaf-4350f09b5a63"}`

Valid `method` values are
- `creation_date`
- `number_of_tags`.

Method `number_of_tags` expects an integer.

Method `creation_date` expects a string containing the amount of time the image is not deleted since creation. Valid options are
- `s` seconds
- `m` minutes
- `h` hours
- `d` days
- `w` weeks
  
 E.g. `30d`.
### Modify the existing auto-prune policy for repository `$repo` within the organization `$orga`
> First, get the current uuid (${policy_uuid}) to modify it
```
curl -X PUT \
     -H "Authorization: Bearer ${bearer_token}" \
     -H "Content-Type: application/json" \
     --data '{ \
              "method": "creation_date", \
              "value": "7d" \
             }' \
     https://${quay_registry}/api/v1/repository/${orga}/${repo}/autoprunepolicy/${policy_uuid} | jq
```
```
curl -X PUT -H "Authorization: Bearer ${bearer_token}" -H "Content-Type: application/json" --data '{"method": "creation_date", "value": "7d"}' https://${quay_registry}/api/v1/repository/${orga}/${repo}/autoprunepolicy/${policy_uuid} | jq
```
Success is HTTP `204`, no success is HTTP `404`

Successful modification has no response!

### Delete the existing auto-prune policy for repository `$repo` within the organization `$orga`
> First, get the current uuid (${policy_uuid}) to delete it
```
curl -X DELETE \
     -H "Authorization: Bearer ${bearer_token}" \
     https://${quay_registry}/api/v1/organization/${orga}/autoprunepolicy/${policy_uuid} | jq
```
```
curl -X DELETE -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/${orga}/autoprunepolicy/${policy_uuid} | jq
```
Success is HTTP `200`, no success is HTTP `404`

A successful deletion will return the deleted uuid. E.g. `{"uuid": "cb44b3b7-bca6-42e4-8eaf-4350f09b5a63"}`
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
              "description": "", \
              "repo_kind": "image"              
             }' \
     https://${quay_registry}/api/v1/repository | jq
```
```
curl -X POST -H "Authorization: Bearer ${bearer_token}" -H "Content-Type: application/json" --data '{"namespace": "'$orga'","repository": "'$repo'","visibility": "public","description": "","repo_kind": "image"}' https://${quay_registry}/api/v1/repository | jq
```
Success is HTTP `201`, no success is HTTP `400`

Required properties are `namespace`, `repository`, `visibility` and `description`

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
Success is HTTP `201`, no sucess is HTTP `404` if there is no mirror config
### Initiate a mirror sync action of the mirrored repository `$repo` within the organization `$orga`
```
curl -X POST \
     -H "Authorization: Bearer ${bearer_token}" \
     https://${quay_registry}/api/v1/repository/${orga}/${repo}/mirror/sync-now | jq
```
```
curl -X POST -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/repository/${orga}/${repo}/mirror/sync-now | jq
```


## Tags
### List all tags in repository `$repo` within organization `$orga`
```
curl -X GET \
     -H "Authorization: Bearer ${bearer_token}" \
     https://${quay_registry}/api/v1/repository/${orga}/${repo}/tag/ | jq
```
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/repository/${orga}/${repo}/tag/ | jq
```
Success is HTTP `200`, repo does not exist is HTTP `404`

### List all ***active*** tags in repository `$repo` within organization `$orga`
> Active tags are the currently available (active) tags, not deleted tags still restoreable from the time machine
```
curl -X GET \
     -H "Authorization: Bearer ${bearer_token}" \
     https://${quay_registry}/api/v1/repository/${orga}/${repo}/tag/?onlyActiveTags=true | jq
```
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/repository/${orga}/${repo}/tag/?onlyActiveTags=true | jq
```
Success is HTTP `200`, repo does not exist is HTTP `404`

### Create a new tag `$newtag` for manifest digest `$manifest_digest` in reopsitory `$repo` within organization `$orga`
```
curl -X GET \
     -H "Authorization: Bearer ${bearer_token}" \
     -H "Content-Type: application/json" \
     --data '{\
              "manifest_digest": "sha256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx" \
             }' \  
     https://${quay_registry}/api/v1/repository/${orga}/${repo}/tag/${newtag} | jq
```
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" -H "Content-Type: application/json" --data '{"manifest_digest": "sha256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"}' https://${quay_registry}/api/v1/repository/${orga}/${repo}/tag/${newtag} | jq
```
The provided manifest digest has to be the full digest in format `sha256:64-characters`

Success is HTTP`201`

### Set or change the expiration date to `Tue May 2 16:33:45 CEST 2023` of tag `$tag` for repository `$repo` in organization `$orga`

Convert given date into a UNIX time stamp
```
date -d "2023-05-02 16:33:45 CEST" +%s
1683038025
```
```
curl -X PUT \
     -H "Authorization: Bearer ${bearer_token}" \
     -H "Content-Type: application/json" \
     --data '{\
              "expiration": 1683038025 \
             }' \  
     https://${quay_registry}/api/v1/repository/${orga}/${repo}/tag/${tag} | jq
```
```
curl -X PUT -H "Authorization: Bearer ${bearer_token}" -H "Content-Type: application/json" --data '{"expiration": 1683038025}' https://${quay_registry}/api/v1/repository/${orga}/${repo}/tag/${tag} | jq
```
Success is HTTP `201`

### Reset the expiration date to `Never` (delete the expiration date) of tag `$tag` in reopsitory `$repo` within organization `$orga`
```
curl -X PUT \
     -H "Authorization: Bearer ${bearer_token}" \
     -H "Content-Type: application/json" \
     --data '{\
              "expiration": null \
             }' \  
     https://${quay_registry}/api/v1/repository/${orga}/${repo}/tag/${tag} | jq
```
```
curl -X PUT -H "Authorization: Bearer ${bearer_token}" -H "Content-Type: application/json" --data '{"expiration": null}' https://${quay_registry}/api/v1/repository/${orga}/${repo}/tag/${tag} | jq
```
Success is HTTP `201`

### Delete tag `$tag` in repository `$repo` within organization `$orga`
```
curl -X DELETE \
     -H "Authorization: Bearer ${bearer_token}" \
     https://${quay_registry}/api/v1/repository/${orga}/${repo}/tag/${tag} | jq
```
```
curl -X DELETE -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/repository/${orga}/${repo}/tag/${tag} | jq
```
Success is HTTP `204`

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
### Get all permissions for team `$team` in organization `$orga`
```
curl -X GET \
     -H "Authorization: Bearer ${bearer_token}" \
     https://${quay_registry}/api/v1/organization/${orga}/team/${team}/permissions | jq
```
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/${orga}/team/${team}/permissions | jq
```
Success is HTTP `200`
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
### Enable LDAP group team sync
```
curl -X POST \
     -H "Authorization: Bearer ${bearer_token}" \
     -H "Content-Type: application/json" \
     --data '{\
              "group_dn": "cn=ldapgroup,dc=dmain,dc=tld" \
             }' \
     https://${quay_registry}/api/v1/organization/${orga}/team/${team}/syncing | jq
```
```
curl -X POST -H "Authorization: Bearer ${bearer_token}" -H "Content-Type: application/json" --data '{"group_dn": "cn=ldapgroup,dc=dmain,dc=tld"}' https://${quay_registry}/api/v1/organization/${orga}/team/${team}/syncing | jq
```
success is HTTP `200`, could not sync to group: group does not exist or is empty is HTTP `400`
### Remove LDAP group team sync
```
curl -X DELETE \
     -H "Authorization: Bearer ${bearer_token}" \
     https://${quay_registry}/api/v1/organization/${orga}/team/${team}/syncing | jq

```
```
curl -X DELETE -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/${orga}/team/${team}/syncing | jq
```
success is HTTP `200`
## Users
### Get all existing users
Works only if quay is configured with `AUTHENTICATION_TYPE: database` or `AUTHENTICATION_TYPE: LDAP`
```
curl -X GET \
     -H "Authorization: Bearer ${bearer_token}" \
     https://${quay_registry}/api/v1/superuser/users/ | jq
```
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/superuser/users/ | jq
```
### Get info about the currently logged in user
```
curl -X GET \
     -H "Authorization: Bearer ${bearer_token}" \
     https://${quay_registry}/api/v1/user/ | jq
```
```
curl -X GET -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/user/ | jq
```
Success is HTTP `200`
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
### Get information about orgrobot `$robot` (including the robot token) within the organization `$orga`
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
### Delete the orgrobot `$robot` within the organization `$orga`
```
curl -X DELETE \
     -H "Authorization: Bearer ${bearer_token}" \
     https://${quay_registry}/api/v1/organization/${orga}/robots/${robot} | jq
```
```
curl -X DELETE -H "Authorization: Bearer ${bearer_token}" https://${quay_registry}/api/v1/organization/${orga}/robots/${robot} | jq
```
Success is HTTP `204`, no success is HTTP `400`

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
