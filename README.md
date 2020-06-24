GitHub Actions experimental repositry
==========


## aws-actionsの実例

### Fargateへのデプロイ
[docker build \-t test\-nginx\. && docker run \-p 8080:80 test\-nginx](https://dev.classmethod.jp/articles/github-actions-fargate-deploy/)


セットアップ

1. Dockerfileを用意（nginxが動くだけのものでOK）
2. AWS consoleでECRリポジトリを作成（`githubactions-nginx`）
3. AWS consoleでこのactionを試すためのIAMユーザーを作成
    - `AmazonECS_FullAccess`, `AmazonEC2ContainerRegistryFullAccess` ポリシーを付与しておく￥
4. タスク定義のjsonを作成（task-definition.json）。内容はファイルを参照
    - 参考: [タスク定義テンプレート(Fargate)](https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/userguide/create-task-definition.html#task-definition-template)
    - AWS consoleから実施する場合は、[タスク定義の作成 \- Amazon Elastic Container Service](https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/developerguide/create-task-definition.html) や [タスク定義の作成 \- Amazon ECS](https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/userguide/create-task-definition.html) を参照
5. 公式 aws-actions から落としてきたaction yamlを修正する
    - region
    - REPOSITORY
    - container-name
    - service
    - cluster
6. githubのsettingsから2つのシークレットを登録
    - AWS_ACCESS_KEY_ID
    - AWS_SECRET_ACCESS_KEY

## Tips


### Run local
Use [nektos/act: Run your GitHub Actions locally 🚀](https://github.com/nektos/act)


### API with curl

[Workflow runs \| GitHub Developer Guide](https://developer.github.com/v3/actions/workflow-runs/)

```sh
# https://developer.github.com/v3/actions/workflow-runs/#list-workflow-runs
workflow_id=xxx
curl -H "Authorization: token ${GITHUB_TOKEN}" "https://api.github.com/repos/${GITHUB_OWNER}/${GITHUB_REPOS}/actions/workflows/${workflow_id}/runs"

# https://developer.github.com/v3/actions/workflow-runs/#get-a-workflow-run
curl -H "Authorization: token ${GITHUB_TOKEN}" "https://api.github.com/repos/${GITHUB_OWNER}/${GITHUB_REPOS}/actions/runs/${run_id}"

# https://developer.github.com/v3/actions/workflow-runs/#get-workflow-run-usage  ※public beta
curl -H "Authorization: token ${GITHUB_TOKEN}" "https://api.github.com/repos/${GITHUB_OWNER}/${GITHUB_REPOS}/actions/runs/${run_id}/timing"
```


[Artifacts \| GitHub Developer Guide](https://developer.github.com/v3/actions/artifacts/)


```sh
# https://developer.github.com/v3/actions/artifacts/#list-artifacts-for-a-repository
curl -H "Authorization: token ${GITHUB_TOKEN}" "https://api.github.com/repos/${GITHUB_OWNER}/${GITHUB_REPOS}/actions/artifacts"

# https://developer.github.com/v3/actions/artifacts/#list-workflow-run-artifacts
run_id=xxx
curl -H "Authorization: token ${GITHUB_TOKEN}" "https://api.github.com/repos/${GITHUB_OWNER}/${GITHUB_REPOS}/actions/runs/${run_id}/artifacts"

# https://developer.github.com/v3/actions/artifacts/#get-an-artifact
artifact_id=xxx
curl -H "Authorization: token ${GITHUB_TOKEN}" "https://api.github.com/repos/${GITHUB_OWNER}/${GITHUB_REPOS}/actions/artifacts/${artifact_id}"

# https://developer.github.com/v3/actions/artifacts/#download-an-artifact
# curl -v -L -u octocat:$token -o Rails.zip "https://api.github.com/repos/octo-org/octo-repo/actions/artifacts/30209828/zip"
curl -H "Authorization: token ${GITHUB_TOKEN}" "https://api.github.com/repos/${GITHUB_OWNER}/${GITHUB_REPOS}/actions/artifacts/${artifact_id}/:archive_format"

# https://developer.github.com/v3/actions/artifacts/#delete-an-artifact
curl -H "Authorization: token ${GITHUB_TOKEN}" "https://api.github.com/repos/${GITHUB_OWNER}/${GITHUB_REPOS}/actions/artifacts/${artifact_id}"

```

[Gists \| GitHub Developer Guide](https://developer.github.com/v3/gists/)

```sh
# https://developer.github.com/v3/gists/#list-gists-for-a-user
curl -H "Authorization: token ${GITHUB_TOKEN}" "https://api.github.com/users/${GITHUB_OWNER}/gists"

# https://developer.github.com/v3/gists/#get-a-gist
gist_id=xxx
curl -H "Authorization: token ${GITHUB_TOKEN}" "https://api.github.com/gists/${gist_id}"
```

#### jq

sample.json

```json
{
  "id": 123456789,
  "node_id": "XXXXXXXXXXXXXXXXXXXXXXXXXXXXMzE2",
  "head_branch": "fix-ci",
  "head_sha": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
  "run_number": 46,
  "event": "pull_request",
  "status": "completed",
  "conclusion": "success",
  "workflow_id": 987654,
  "url": "https://api.github.com/repos/goldeneggg/structil/actions/runs/23456789",
  "html_url": "https://github.com/goldeneggg/structil/actions/runs/23456789",
  "pull_requests": [
    {
      "url": "https://api.github.com/repos/goldeneggg/structil/pulls/19",
      "id": 334455667,
      "number": 19,
      "head": {
        "ref": "fix-ci",
        "sha": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
        "repo": {
          "id": 445566778,
          "url": "https://api.github.com/repos/goldeneggg/structil",
          "name": "structil"
        }
      },
      "base": {
        "ref": "master",
        "sha": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
        "repo": {
          "id": 556677889,
          "url": "https://api.github.com/repos/goldeneggg/structil",
          "name": "structil"
        }
      }
    }
  ],
  "created_at": "2020-06-12T01:03:59Z",
  "updated_at": "2020-06-12T01:11:20Z"
}
```


```sh
# get only "id"
$ cat sample.json | jq '.id'
123456789

# get only "id" and "node_id"
$ cat sample.json | jq '. | {id, node_id}'
{
  "id": 123456789,
  "node_id": "XXXXXXXXXXXXXXXXXXXXXXXXXXXXMzE2"
}

# get list "pull_request"
$ cat sample.json | jq '.pull_requests'
[
  {
    "url": "https://api.github.com/repos/goldeneggg/structil/pulls/19",
    "id": 334455667,
    "number": 19,
    "head": {
      "ref": "fix-ci",
      "sha": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
      "repo": {
        "id": 445566778,
        "url": "https://api.github.com/repos/goldeneggg/structil",
        "name": "structil"
      }
    },
    "base": {
      "ref": "master",
      "sha": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
      "repo": {
        "id": 556677889,
        "url": "https://api.github.com/repos/goldeneggg/structil",
        "name": "structil"
      }
    }
  }
]

# get element index 0 from list "pull_request"
$ cat sample.json | jq '.pull_requests[0]'
{
  "url": "https://api.github.com/repos/goldeneggg/structil/pulls/19",
  "id": 334455667,
  "number": 19,
  "head": {
    "ref": "fix-ci",
    "sha": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
    "repo": {
      "id": 445566778,
      "url": "https://api.github.com/repos/goldeneggg/structil",
      "name": "structil"
    }
  },
  "base": {
    "ref": "master",
    "sha": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
    "repo": {
      "id": 556677889,
      "url": "https://api.github.com/repos/goldeneggg/structil",
      "name": "structil"
    }
  }
}

# get nested element from element index 0 from list "pull_request"
$ cat sample.json | jq '.pull_requests[0].head.ref'
"fix-ci"
```
