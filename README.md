# simple-rails-container-to-ecr

🔆🔆🔆 簡単なRailsアプリケーションをECRにデプロイしてみる！  

[![ci](https://github.com/osawa-koki/simple-rails-container-to-ecr/actions/workflows/ci.yml/badge.svg)](https://github.com/osawa-koki/simple-rails-container-to-ecr/actions/workflows/ci.yml)
[![cd](https://github.com/osawa-koki/simple-rails-container-to-ecr/actions/workflows/cd.yml/badge.svg)](https://github.com/osawa-koki/simple-rails-container-to-ecr/actions/workflows/cd.yml)
[![Dependabot Updates](https://github.com/osawa-koki/simple-rails-container-to-ecr/actions/workflows/dependabot/dependabot-updates/badge.svg)](https://github.com/osawa-koki/simple-rails-container-to-ecr/actions/workflows/dependabot/dependabot-updates)

## 実行方法

DevContainerに入ります。  
※ `~/.aws/credentials`にAWSの認証情報があることを前提とします。  

まずは、ECRリポジトリをプロビジョニングします。  

```shell
cdk synth
cdk bootstrap
cdk deploy
```

その後、DockerイメージをビルドしてECRにPushします。  

```shell
export ECR_REPOSITORY_URI=$(aws ecr describe-repositories --repository-names rails-app --query 'repositories[0].repositoryUri' --output text)
aws ecr get-login-password | docker login --username AWS --password-stdin $ECR_REPOSITORY_URI
docker build -t rails-app ./rails-app/
docker tag rails-app:latest $ECR_REPOSITORY_URI:latest
docker push $ECR_REPOSITORY_URI:latest
```

---

また、GitHub Actionsでデプロイすることも可能です。  
以下のシークレットを設定してください。  

| シークレット名 | 説明 |
| --- | --- |
| AWS_ACCESS_KEY_ID | AWSアクセスキーID |
| AWS_SECRET_ACCESS_KEY | AWSシークレットアクセスキー |
| AWS_REGION | AWSリージョン |

タグがプッシュされると、GitHub Actionsがデプロイを行います。  
手動でワークフローを開始することもできます。  

---

ECRにプッシュしたイメージをローカルでテストしたい場合は、以下のコマンドを実行します。  

```shell
export ECR_REPOSITORY_URI=$(aws ecr describe-repositories --repository-names rails-app --query 'repositories[0].repositoryUri' --output text)
aws ecr get-login-password | docker login --username AWS --password-stdin $ECR_REPOSITORY_URI
docker run --rm -p 3000:3000 --name rails_app -e SECRET_KEY_BASE=HOGE $ECR_REPOSITORY_URI:latest
```
