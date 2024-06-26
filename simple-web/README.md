# simple-web helm template

prepare your your-project-values.yaml. here is an example.
you can checkout the default values.yaml in the helm package.

```yaml=
nameOverride: "your-project"

replicaCount: 1

image:
  repository: xxxxxxx.dkr.ecr.ap-northeast-1.amazonaws.com/repo
  tag: "latest"
  pullPolicy: Always

service:
  port: 80

livenessProbe:
  httpGet:
    path: /
    port: 80
  periodSeconds: 30

readinessProbe:
  httpGet:
    path: /
    port: 80

resources:
  limits:
    cpu: 100m
    memory: 64Mi
  requests:
    cpu: 100m
    memory: 64Mi

ingress:
  enabled: true
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/listen-ports: '[{ "HTTP" : 80 }, { "HTTPS" : 443 }]'
    alb.ingress.kubernetes.io/actions.ssl-redirect: '{ "Type" : "redirect", "RedirectConfig" : { "Protocol" : "HTTPS", "Port" : "443", "StatusCode" : "HTTP_301" } }'
    alb.ingress.kubernetes.io/group.name: xxxxx
    alb.ingress.kubernetes.io/certificate-arn: xxxxx
  hosts:
    - host: test.example.com
      paths:
        - path: /
          pathType: Prefix

healthEndpoint:
  path: /livez

env:
  - name: Key1
    value: value1
  - name: Key2
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
    mountTo: /tmp/

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
```

## install helm

- helm upgrade --install your-project simple-web-1.0.4.tgz -f your-project-values.yaml -n NAMESPACE
