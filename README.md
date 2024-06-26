# How to use simple-web helm chart

use [simple-web](https://artifacthub.io/packages/helm/simple-web/simple-web) helm chart to install all web base application

## Prepare

1. helm repo add simple-web https://dachichang.github.io/helm-simple-web/
2. helm repo update
3. helm search repo simple-web

```bash=
NAME                    CHART VERSION   APP VERSION     DESCRIPTION
simple-web/simple-web   1.1.0           latest          A Helm chart for Kubernetes
```

Your are ready to use simple-web helm chart to install your applications.

## Write your values.yaml

You can use $ENV prefix to separate the values of environments. for example

- dev-values.yaml for dev
- tst-values.yaml for tst

#### example values.yaml

```yaml=
# !!!! overwrite metadata name
nameOverride: "demo-aks"

image:
  repository: nginx
  tag: "latest"
  pullPolicy: Always
  # entrypointOverride: # support endpoint override
  #   command: ["/bin/bash"]
  #   args: ["-c", "sleep 2592000"]

service:
  port: 80 # container port = service port = 80

# strategy: # support rollout strategy
#   type: RollingUpdate
#   rollingUpdate:
#     maxSurge: 25%
#     maxUnavailable: 0

livenessProbe: # if don't provide livenessProbe will DISABLE livenessProbe not use the default value
  httpGet:
    path: /
    port: 80

readinessProbe: # if don't provide readinessProbe will DISABLE livenessProbe not use the default value
  httpGet:
    path: /
    port: 80

ingress:
  enabled: true
  className: psg # use specific ingress class
  hosts:
    - host: demo-aks.shellab.io
      paths:
        - path: /
          pathType: Prefix

env: # env will inject to container environment in plain text
  - name: key1
    value: value1
  - name: key2
    value: value2

# use env as file to inject to pod running file_system
# name will be the file name
# mountTo will be the where the file should be put.
envFiles:
  - name: env1.json # will put in pod /app/env1.json
    data: |
      {
        "a": "t1",
        "b": "t2",
        "c": "t3"
      }
    mountTo: /app/
  - name: env2.txt # will put in pod /app/env2.txt
    data: |
      Lorem ipsum dolor sit amet, officia excepteur ex fugiat reprehenderit enim
      labore culpa sint ad nisi Lorem pariatur mollit ex esse exercitation amet.
      Nisi anim cupidatat excepteur officia. Reprehenderit nostrud nostrud ipsum Lorem est aliquip amet voluptate voluptate dolor minim nulla est proident.
      Nostrud officia pariatur ut officia.
    mountTo: /app/

secret: # secret will inject to container environment by b64 decoded value.
  - name: secret1
    value: ZGFuaWVs
  - name: secret2
    value: YWxhbg==

# use secret as file to inject to pod running file_system
# data needs the result of b64
secretFiles:
  - name: hello.json
    data: eWF5YXlheWF5YXlh
    mountTo: /etc/
  - name: world.json
    data: aGVsbG93b3JsZAo=
    mountTo: /tmp/
```

## Install your web app by simple-web

> install application which named **demo-aks**

- **helm upgrade --install demo-aks simple-web/simple-web --version 1.1.1 -f demo-aks.yaml**

#### Important

Difference release name and metadata name will have different combinations.

- resource name: **demo-aks** (A == B && A)
  - release name: demo-aks
  - chart name: simple-web
  - metadata name: demo-aks (nameOverride in values.yaml)
- resource name: **test-demo-aks** (A + - + B)
  - release name: test
  - chart name: simple-web
  - metadata name: demo-aks (nameOverride in values.yaml)

Best practice: release name same as nameOverride. ex: aks-demo-v2

## Project helm

if you have multi-service need to package as one service, **helm dependenices** is good choice.

#### Create parent helm

1. helm create project-helm
2. cd project-helm && rm -rf templates # we don't need any templates just needs the dependenices of helm.
3. vim Chart.yaml

```yaml=
apiVersion: v2
name: project-helm
description: A Helm chart for Kubernetes

# A chart can be either an 'application' or a 'library' chart.
#
# Application charts are a collection of templates that can be packaged into versioned archives
# to be deployed.
#
# Library charts provide useful utilities or functions for the chart developer. They're included as
# a dependency of application charts to inject those utilities and functions into the rendering
# pipeline. Library charts do not define any templates and therefore cannot be deployed.
type: application

# This is the chart version. This version number should be incremented each time you make changes
# to the chart and its templates, including the app version.
# Versions are expected to follow Semantic Versioning (https://semver.org/)
version: 0.1.0

# This is the version number of the application being deployed. This version number should be
# incremented each time you make changes to the application. Versions are not expected to
# follow Semantic Versioning. They should reflect the version the application is using.
# It is recommended to use it with quotes.
appVersion: "1.16.0"

dependencies:
  - name: simple-web
    version: 1.1.1
    repository: https://idtech-helm.s3.us-west-2.amazonaws.com
    alias: demo-web
    condition: demo-web.enabled
  - name: simple-web
    version: 1.1.1
    repository: https://idtech-helm.s3.us-west-2.amazonaws.com
    alias: demo-api
    condition: demo-api.enabled
```

4. vim values.yaml

```yaml=
# Default values for project-helm.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

demo-web: # !!!IMPORTANT it comes from alias name
  enabled: false # default disable it.

demo-api: # !!!IMPORTANT it comes from alias name
  enabled: false
```

#### Project helm package

1. helm dep update
2. helm package . --version 1.0.0 --app-version latest
3. ls project-helm-1.0.0.tgz

#### Prepare project helm values ex: project-helm.yaml

```yaml=
# Default values for simple-web.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

# if you want to use this chart to deploy different instance, just nameOverride.
demo-web:
  enabled: true
  nameOverride: "demo-web"
  image:
    repository: nginx
    tag: "latest"
    pullPolicy: Always
  service:
    port: 80
  livenessProbe:
    httpGet:
      path: /
      port: 80
  readinessProbe:
    httpGet:
      path: /
      port: 80
  ingress:
    enabled: true
    className: psg
    hosts:
      - host: demo-web.shellab.io
        paths:
          - path: /
            pathType: Prefix
  env:
    - name: key1
      value: value1
    - name: key2
      value: value2
  envFiles:
    - name: env1.json
      data: |
        {
          "a": "t1",
          "b": "t2",
          "c": "t3"
        }
      mountTo: /app/
    - name: env2.txt
      data: |
        Lorem ipsum dolor sit amet, officia excepteur ex fugiat reprehenderit enim
        labore culpa sint ad nisi Lorem pariatur mollit ex esse exercitation amet.
        Nisi anim cupidatat excepteur officia. Reprehenderit nostrud nostrud ipsum Lorem est aliquip amet voluptate voluptate dolor minim nulla est proident.
        Nostrud officia pariatur ut officia.
      mountTo: /app/

  secret:
    - name: secret1
      value: ZGFuaWVs
    - name: secret2
      value: YWxhbg==

  secretFiles:
    - name: hello.json
      data: eWF5YXlheWF5YXlh
      mountTo: /etc/
    - name: world.json
      data: aGVsbG93b3JsZAo=
      mountTo: /tmp/

demo-api:
  enabled: true
  nameOverride: "demo-api"

  image:
    repository: shellab.azurecr.io/demo-api
    tag: "010a669"
    pullPolicy: Always
    entrypointOverride:
      command: ["/bin/bash"]
      args: ["-c", "cp /etc/demo-api/secrets.json /app/ && dotnet API.dll"]

  service:
    port: 80

  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 0

  livenessProbe:
    httpGet:
      path: /api/livez
      port: 80
    timeoutSeconds: 3

  readinessProbe:
    httpGet:
      path: /api/livez
      port: 80
    timeoutSeconds: 3

  autoscaling:
    enabled: true
    minReplicas: 1
    maxReplicas: 3
    targetCPUUtilizationPercentage: 50

  ingress:
    enabled: true
    className: psg
    hosts:
      - host: demo-api.shellab.io
        paths:
          - path: /
            pathType: Prefix

  nodeSelector:
    purpose: general

  initContainers:
    - name: sysctl-tuning
      image: docker.io/busybox
      imagePullPolicy: IfNotPresent
      command: ["/bin/sh"]
      args:
        - -c
        - sysctl -w fs.file-max=2097152 kernel.sysrq=0 kernel.core_uses_pid=1 kernel.msgmnb=65536 kernel.msgmax=65536 kernel.shmmax=68719476736 kernel.shmall=4294967296
        - sysctl -w vm.swappiness=10 vm.dirty_ratio=40 vm.dirty_background_ratio=10 net.core.somaxconn=65535
        - sysctl -w net.ipv4.conf.default.rp_filter=1 net.ipv4.conf.default.accept_source_route=0 net.ipv4.ip_forward=0 net.ipv4.ip_local_port_range="2000 65535"
        - sysctl -w net.ipv4.tcp_tw_reuse=1
        - ulimit -l unlimited
        - sleep 5
      securityContext:
        privileged: true

  env:
    - name: ASPNETCORE_ENVIRONMENT
      value: Development

  envFiles:
    - name: secrets.json
      data: |
        {
          "Email:DefaultProvider": "",
          "Email:Smtp:Password": "",
          "Sms:DefaultProvider": "",
          "Sms:TelnyxV1:Secret": "",
          "CardConnectBolt:AccessToken": "",
          "ConnectionStrings:Core": "",
          "ConnectionStrings:Payment": "",
          "ConnectionStrings:Logging": "",
          "ApplicationInsights:ConnectionString": "",
          "AppLoaderAPI:AccessKeyId": "",
          "AppLoaderAPI:SecretAccessKey": "",
          "Elavon:API:EncryptedText": "",
          "AwsSQS:AccessKey": "",
          "AwsSQS:SecretKey": "",
          "AuthService:RootAccount": ""
        }
      mountTo: /etc/demo-api/
    - name: env2.txt
      data: |
        Lorem ipsum dolor sit amet, officia excepteur ex fugiat reprehenderit enim
        labore culpa sint ad nisi Lorem pariatur mollit ex esse exercitation amet.
        Nisi anim cupidatat excepteur officia. Reprehenderit nostrud nostrud ipsum Lorem est aliquip amet voluptate voluptate dolor minim nulla est proident.
        Nostrud officia pariatur ut officia.
      mountTo: /tmp/

  secretFiles:
   - name: secrets.json
     data: eWF5YXlheWF5YXlh
     mountTo: /tmp/
```

#### Project helm install

> install applications which named **demo-project**

- **helm upgrade --install demo-project project-helm-1.0.0.tgz -f project-helm.yaml**

## Pacakge simple-web

> pacakge helm will generate a package file which named **simple-web-1.1.1.tgz**

- **helm package simple-web --version 1.1.1 --app-version latest**

## Host in Artifacthub.io

1. create new repo
   - **https://github.com/DachiChang/helm-simple-web**
2. create gh-pages (github pages to host helm package files)
   - git checkout --orphan gh-pages && git rm -rf .
3. push helm package to gh-pages branch
   - mv simple-web-1.1.1.tgz .
4. create helm repo index
   - helm repo index .
5. commit and && push gh-pages
   ```
   -rw-r--r--@ 1 dachichang  staff   160 Jun 26 11:14 artifacthub-repo.yml
   -rw-r--r--@ 1 dachichang  staff  1016 Jun 26 11:14 index.yaml
   -rw-r--r--@ 1 dachichang  staff  5226 Jun 26 11:14 simple-web-1.0.4.tgz
   -rw-r--r--@ 1 dachichang  staff  5544 Jun 26 11:14 simple-web-1.1.0.tgz
   -rw-r--r--@ 1 dachichang  staff  5557 Jun 26 11:14 simple-web-1.1.1.tgz
   ```
6. artifacthub.io create a helm repo url point to https://dachichang.github.io/helm-simple-web/
