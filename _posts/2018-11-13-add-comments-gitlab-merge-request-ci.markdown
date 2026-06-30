---
layout:     post
title:      "How to add comments to Gitlab 'Merge Request' from CI script"
date:       2018-11-13 00:00:00
author:     "Daniel Vela"
og_image: "/img/post-bg-09.jpg"
locale:       en
lang-ref:   add-comments-gitlab-merge-request-ci
---

This doc was made to explain how to send the output of an script (like swiftlint or other lints or quality utils) from the CI process of Gitlab to the Merge Request comment list.

At 13 Nov 2018 Gitlab don't have a enviroment variable to know the _iid_ of the merge request. Look this [StackOverflow question](https://stackoverflow.com/questions/52746338/in-gitlab-ci-is-there-a-variable-for-a-merge-requests-target-branch) to see more info.

But this _iid_ can be obtained using the Gitlab API. The trick is to query the merge request list from a commit and use the first one.

## Gitlab API doc

1. [Create new merge request comment](https://docs.gitlab.com/ce/api/notes.html#create-new-merge-request-note)
2. [Basic usage](https://docs.gitlab.com/ce/api/README.html#road-to-graphql)
3. [List Merge Requests associated with a commit](https://docs.gitlab.com/ce/api/commits.html#list-merge-requests-associated-with-a-commit)

### API Usage Sample :

**POST /projects/:id/merge_requests/:merge_request_iid/notes**

    $ curl --request POST --header "Private-Token: 9koXpg98heJpvBs5tK" https://gitlab.example.com/api/v4/projects/5/merge_requests/11/notes --data "Comment test"

## Requirements

- Install xcpretty

        $ brew install xcpretty
- Install jq to parse json response

        $ brew install jq
- Install swiftlint

        $ brew install swiftlint
- Set your API gitlab private token     
  Create a token in your Gitlab profile:      
  *https://gitlab.your-server.com/profile/personal_access_tokens*.    
  
  Set a new environment variable in the file `config.toml` of your gitlab runners.     

```toml
concurrent = 1
check_interval = 0
[[runners]]
  name = "mac-mini.local"
  url = "https://gitlab.your-server.com/"
  token = "d1c0dd187cf73f7e8480cc7ef4a0f4"
  tls-ca-file = "/Users/pepe/gitlab.your-server.com.crt"
  executor = "ssh"
  environment = ["GITLAB_PERSONAL_API_PRIVATE_TOKEN=L8zHc7asdfffp2k18hX9Fu"]
  [runners.ssh]
    user = "pepe"
    password = "pepe"
    host = "localhost"
    port = "22"
    identity_file = "/Users/pepe/.ssh/id_rsa"
  [runners.cache]
```

## .gitlab-ci.yml

This is a sample _.gitlab-ci.yml_ file to send the swiftlint output to the Merge Request comments:

```yaml
lint:
  stage: build
  script:
    - echo "body=" > swiftlint.log
    - swiftlint >> swiftlint.log
    - 'CI_MERGE_REQUEST_IID=$(curl --request GET --header "Private-Token: $GITLAB_PERSONAL_API_PRIVATE_TOKEN" "https://gitlab.your-server.com/api/v4/projects/$CI_PROJECT_ID/repository/commits/$CI_COMMIT_SHA/merge_requests" --insecure | jq --raw-output ".[0].iid")'
    - '[ -z "$CI_MERGE_REQUEST_IID" ] && curl --request POST --header "Private-Token: $GITLAB_PERSONAL_API_PRIVATE_TOKEN" -d @swiftlint.log https://gitlab.your-server.com/api/v4/projects/$CI_PROJECT_ID/merge_requests/$CI_MERGE_REQUEST_IID/notes --insecure'
    - unlink swiftlint.log
  tags:
    - xcode
```

This scripts sets **CI\_MERGE\_REQUEST\_IID** using the API to get the Merge Request for a commit using its **$CI\_COMMIT\_SHA**. The response is a json. To obtain the _iid_ use `jq --raw-output ".[0].iid"`.

Use `[ -z "$CI_MERGE_REQUEST_IID" ] &&` to avoid send the POST if the commit doesn't have any Merge Request associated.




