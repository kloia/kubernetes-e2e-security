# Table Of Contents

* [Motivation](#motivation)
* [Container Level Security](#container-level-security)
  * [Non-Root User Inside Container](#non-root-user-inside-container)
  * [Only use official images or custom build images](#only-use-official-images-or-custom-build-images)
  * [Never Put Any Credentials Inside Your Image](#never-put-any-credentials-inside-your-image)
  * [Security Scan For Docker Images](#security-scan-for-docker-images)
* [Pod Level Security](#pod-level-security)
  * [Use Pod Security Policies](#use-pod-security-policies)
  * [Enable RBAC](#enable-rbac)
  * [Disable Auto-mounting ServiceAccount Token](#disable-auto-mounting-serviceaccount-token)
  * [Isolate Pods Using Taints](#isolate-pods-using-taints)
  * [Don't Run Containers as Privileged](#dont-run-containers-as-privileged)
  * [Drop Linux Capabilities](#drop-linux-capabilities)
* [Kubelet/Kubernetes Security Configurations](#kubernetes-security-configurations)
  * [Disable Anonymous-Auth](#disable-anonymous-auth)
  * [Disable Access to Metrics Endpoint](#disable-access-to-metrics-endpoint)
  * [Kubernetes API IP Restriction](#kubernetes-api-ip-restriction)
* [Kubernetes Secret Management](#kubernetes-secret-management)
  * [Encrypt Secrets at Rest](#encrypt-secrets-at-rest)
* [Network Driver Security Configurations](#network-driver-security-configurations)
  * [Isolate Namespaces](#isolate-namespaces)
  * [Block AWS Access](#block-aws-access)
* [Cloud Provider Security Best Practices](#cloud-provider-security-best-practices)
  * [AWS](#aws)


## Motivation

If you running your applications inside Kubernetes cluster, you should think about security in every step. Security is not something that you can do in a single command or single framework. There are many different layers/aspects of the security. For example you should think as application layer, container layer, orchestration tool layer and so on. In this repository, we are collecting all kinds of security advices/best practices and examples that are related to kubernetes world.

If you want to contribute, please feel free to send a merge request or open an issue.

## Container Level Security

#### Non-Root User Inside Container

You should always your docker containers with non-root user.
If you run as `root` user then container might modify some files that shouldn't be modified.
If attacker gains access then they will be root. For example they can install some applications inside container to listen network traffic. [example Dockerfile](https://github.com/kloia/kubernetes-e2e-security/blob/master/container/Dockerfile)

```
RUN adduser -D myuser
USER myuser
```

#### Only use official images or custom build images

If you are using non-official image then you don't know whether this image has been replaced or not in any time. Especially in kubernetes, if you configured your image pull policy `Always` then kubernetes will download your image each time.
So never use custom images that is built by 3rd person. Always build your images and push your private/public repository.

```
spec:
  containers:
    - name: example
      image: alpine
      imagePullPolicy: Always
```


#### Never put any credentials inside your image.

Always mount your secrets with using [Kubernetes Secret Manager](https://kubernetes.io/docs/concepts/configuration/secret/) and use [Configmap](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/) for your configuration files or environment variables.

#### Security Scan for Docker images

There are open source tools that checks Dockerfiles for potential security vulnerabilities. You can either integrate those tools to your CI cycle or use them manually.

https://github.com/coreos/clair


## Pod Level Security

#### Use Pod Security Policies
// TODO

#### Enable RBAC
Always enable [RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) in Api Server.
For the kops:
```
authorization:
  rbac: {}
```

If you are using Kubernetes API inside pod. You should create
a ServiceAccount for that deployment and only associate minimum policies.


#### Disable Auto-mounting ServiceAccount Token
By default, kubernetes mounts default service account token to inside your pods. You can disable it if you don't use any kubernetes api.

```
spec:
  automountServiceAccountToken: false
```

#### Isolate Pods Using Taints
Taints are added to `nodes` and Tolerations are added to `deployment` objects. (or pods)
If toleration and taint have same keys are and same effects then they matches.

```
kubectl taint nodes node1 key=value:NoSchedule
```

```
tolerations:
- key: "key"
  operator: "Equal"
  value: "value"
  effect: "NoSchedule"
```


#### Don't Run Containers as Privileged
If you run your containers in privileged mode then those containers can directly
access to system.

```
containers:
  securityContext:
    privileged: false
```

#### Drop Linux Capabilities
If you want to drop capabilities from container that you should use `RequiredDropCapabilities` field name.

```
spec:
  RequiredDropCapabilities:
    - CAP_NAME
```
Note: Capabilities listed in `RequiredDropCapabilities` must not be included in `AllowedCapabilities` or `DefaultAddCapabilities`.

## Kubelet Security Configurations

#### Disable Anonymous-Auth

If you don't disable anonymous-auth in kubelet configuration then
all the pods will have full access to kubernetes api.
So you should start kubelet with:

`--anonymous-auth=false` flag

for the kops:

```
spec:
  kubelet:
    anonymousAuth: false
```

#### Disable Access to Metrics Endpoint
By default CAdvisor exposes port 4194.
Anyone inside container can access local machine ip:4194/metrics endpoint and
can fetch metrics.

If you are using kops/aws, you can disable it in security group.
If you have a custom setup, you can disable it by network driver.

Note: Kubernetes is planning to deprecate metrics endpoint in the future
https://github.com/kubernetes/kubernetes/issues/68522

#### Kubernetes API IP Restriction
// TODO


## Kubernetes Secret Management

#### Encrypt Secrets at Rest
// TODO

## Network Driver Security Configurations


#### Isolate Namespaces
// TODO

#### Block AWS Access
// TODO


## Cloud Provider Security Best Practices
// TODO

### AWS
