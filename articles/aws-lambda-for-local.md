---
title: "AWS Lambdaのローカル開発環境を構築する"
emoji: ""
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["aws lambda", "vscode", "python"]
published: true
---

# AWS Lambdaのローカル開発環境を構築する

AWS Lambdaではコンテナによるデプロイのために[Dockerコンテナイメージ](https://gallery.ecr.aws/lambda)が用意されており、それを利用してDev Container環境を構築します。 \
ここではPythonをランタイム言語として利用します。

参考資料
- [Lambdaコンテナイメージの作成](https://docs.aws.amazon.com/ja_jp/lambda/latest/dg/images-create.html#images-create-from-alt)
- [Lambdaコンテナイメージをローカルでテストする](https://docs.aws.amazon.com/ja_jp/lambda/latest/dg/images-test.html)


## コンテナイメージを作成する

`代替ベースイメージからのイメージの作成` にあるDockerfileを参考に進めます。
これに次の追加の変更を行います。
1. ローカルでのテストを実現するために `Runtime Interface Emulator` を追加する
2. Dev Containerで利用しやすいように `mcr.microsoft.com/devcontainers/` にあるイメージをベースにする


### Dockerfileの作成

```dockerfile
# Define function directory
ARG FUNCTION_DIR="/function"

FROM python:3.9-slim-buster as build-image

# Install aws-lambda-cpp build dependencies
RUN apt-get update && \
  apt-get install -y \
  g++ \
  make \
  cmake \
  unzip \
  libcurl4-openssl-dev

# Include global arg in this stage of the build
ARG FUNCTION_DIR
# Create function directory
RUN mkdir -p ${FUNCTION_DIR}

# Copy function code
COPY app/* ${FUNCTION_DIR}

# Install the runtime interface client
RUN pip install \
        --target ${FUNCTION_DIR} \
        awslambdaric

# Multi-stage build: grab a fresh copy of the base image
FROM python:3.9-slim-buster

# Include global arg in this stage of the build
ARG FUNCTION_DIR
# Set working directory to function root directory
WORKDIR ${FUNCTION_DIR}

# Copy in the build image dependencies
COPY --from=build-image ${FUNCTION_DIR} ${FUNCTION_DIR}

COPY ./entry_script.sh /entry_script.sh
ADD https://github.com/aws/aws-lambda-runtime-interface-emulator/releases/latest/download/aws-lambda-rie /usr/local/bin/aws-lambda-rie
ENTRYPOINT [ "/entry_script.sh" ]
CMD [ "app.handler" ]
```

2つの参考資料で提示されているサンプルを足して数カ所変更したものです。

提示されているサンプルとの相違点
1. ベースイメージをslimベースイメージにしている
2. `Runtime Interface Emulator`のイメージ内への設置をURLから直接行なっている


### entry_script.shの作成

[ベースイメージにRIEを組み込む](https://docs.aws.amazon.com/ja_jp/lambda/latest/dg/images-test.html#images-test-alternative)にあるスクリプトをそのまま利用します。
```bash
#!/bin/sh
if [ -z "${AWS_LAMBDA_RUNTIME_API}" ]; then
  exec /usr/local/bin/aws-lambda-rie /usr/local/bin/python -m awslambdaric $@
else
  exec /usr/local/bin/python -m awslambdaric $@
fi     
```


## 実行

コンテナイメージを実行したらcurlを利用して動作を確認します。
```bash
docker build -t <image>:latest .
docker run -p 8080:8080 <image>:latest
curl -XPOST "http://localhost:8080/2015-03-31/functions/function/invocations" -d '{}'
```

もしAPI Gatewayを利用した動作を確認するのであれば、[SAM](https://docs.aws.amazon.com/ja_jp/serverless-application-model/latest/developerguide/what-is-sam.html)の利用が必要になります。

その場合はPackage TypeをImageにしてこのDockerイメージを利用できます。
