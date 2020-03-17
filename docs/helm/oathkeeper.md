# ORY Oathkeeper Helm Chart

The ORY Oathkeeper Helm Chart helps you deploy ORY Oathkeeper on Kubernetes using Helm.

## Installation

Installing ORY Oathkeeper using Helm with

```bash
$ helm install ory/oathkeeper
```

which sets up a very basic configuration with no access rules and no enabled authenticators, authorizers, or
credentials issuers.

This Helm Chart supports a demo mode which deploys access rules for urls

- `http://<oathkeeper-reverse-proxy-host>/authenticator/noop/authorizer/allow/mutator/noop`
- `http://<oathkeeper-reverse-proxy-host>/authenticator/anonymous/authorizer/allow/mutator/header`

that point to [httpbin.org](https://httpbin.org). To install ORY Oathkeeper in demo-mode, run:

```bash
$ helm install --set 'demo=true' ory/oathkeeper
```

Be aware that this mode uses JSON Web Keys and other secrets that are publicly accessible via GitHub.
These secrets are publicly known and should never be used anywhere. **Do not use demo-mode for anything
other than experimenting**.

## Configuration

You can pass your [ORY Oathkeeper configuration file](https://github.com/ory/oathkeeper/blob/master/docs/config.yaml)
by creating a yaml file with key `oathkeeper.config`

```yaml
# oathkeeper-config.yaml

oathkeeper:
  config:
    # e.g.:
    authenticators:
      noop:
        enabled: true
   # ...
```

and passing that as a value override to helm:

```bash
$ helm install -f ./path/to/oathkeeper-config.yaml ory/oathkeeper
```

Values such as the proxy / api port will be automatically propagated to the service and ingress definitions.
The following table lists the configurable parameters of the ORY Oathkeeper chart and their default values.

For a detailed list of configuration items

### JSON Web Key Set for Mutator `id_token`

The `id_token` mutator requires a secret JSON Web Key Set. This helm chart supports loading the JSON Web Key Set
from disk and deploying it as a Kubernetes Secret:

```bash
$ helm install \
    --set-file 'oathkeeper.mutatorIdTokenJWKs=./path/to/jwks.json' \
    ory/oathkeeper
```

Please note that any configuration values set for `oathkeeper.config.mutator.id_token.jwks_url` using e.g.
a configuration file will be overwritten by this setting.

### Access Rules

Instead of fetching access rules from remote locations, you can set your access rules directly with `--set-file`:

```bash
$ helm install \
    --set-file 'oathkeeper.accessRules=./path/to/access-rules.json' \
    ory/oathkeeper
```

Please note that any configuration values set for `oathkeeper.config.access_rules.repositories` using e.g.
a configuration file will be overwritten by this setting.

### Oathkeeper-maester
This chart includes a helper chart in the form of [Oathkeeper-maester](https://github.com/ory/k8s/blob/master/docs/helm/oathkeeper-maester.md), a k8s controller, which translates access rules object into a kubernetes native [CustomResource](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/). This component is enabled by default, and installed together with Oathkeeper, however it can be disabled by setting the proper flag:

```bash
$ helm install \
    --set 'maester.enabled=false' \
    ory/oathkeeper
```

#### Operation modes
The Oathkeeper Maester works in either of these two modes: 

**Controller mode**
In this mode, the controller is a dedicated deployment and scales independently from the Oathkeeper application. All communication with Oathkeeper is based on a configMap object, which stores the Oathkeeper configuration created based on the Rule custom resource. 
This mode requires giving elevated privileges to the Oathkeeper Maestercontroller to allow operations on the configMaps. 

**Sidecar mode**
In this mode, the Hydra Maester controller runs as an additional container in the Oathkeeper application Pod. All communication is done on the local filesystem, which can be a shared `tempfs`, mounted directory or a persistent volume, and the controller is scaled together with the Oathkeeper application. 
