# konk-service

![Version: 0.1.0](https://img.shields.io/badge/Version-0.1.0-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: v1.19.0](https://img.shields.io/badge/AppVersion-v1.19.0-informational?style=flat-square)

Registers an aggregate API service in Konk

## Values or `KonkService` `.spec`

When deploying with konk-operator, these configurations can be overridden in the `Konk-Service` `.spec`.

When deploying with `helm install`, these configurations are values and can be overridden in the [normal way](https://helm.sh/docs/helm/helm_install/#helm-install).

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| annotations | object | `{}` | annotations to add to the APIService created in Konk by KonkService |
| crds | string | `nil` |  |
| fullnameOverride | string | `""` |  |
| group.kinds | list | `["*"]` | resource types provided by your API service. This list is used to setup default RBAC policies. |
| group.name | string | `"example.infoblox.com"` | https://kubernetes.io/docs/reference/using-api/#api-groups |
| group.verbs | list | `["*"]` | actions to allow on your API service. This list is used to setup default RBAC policies. |
| ingress.annotations | object | `{}` |  |
| ingress.className | string | `""` |  |
| ingress.enabled | bool | `false` |  |
| ingress.hosts[0].host | string | `"localhost"` |  |
| ingress.tls | list | `[]` |  |
| insecureSkipTLSVerify | bool | `false` |  |
| kind.image.pullPolicy | string | `"Always"` |  |
| kind.image.repository | string | `"kindest/node"` |  |
| kind.image.tag | string | `"v1.19.0"` | Overrides the image tag whose default is the chart appVersion. |
| kind.resources | object | `{}` |  |
| kind.securityContext | object | `{}` |  |
| konk.name | string | `""` | should be set to the konk-name |
| konk.namespace | string | `""` |  |
| konk.scope | string | `""` | scope of the konk, must match `.scope` of the Konk |
| nameOverride | string | `""` |  |
| service.caSecretName | string | `nil` | Optional reference to the secret the service's CA certs are stored in. When omitted, KonkService will generate a CA to be used by the APIService. |
| service.name | string | `"test"` | Required to be set to the name of the service to be registered in Konk |
| serviceAccount.annotations | object | `{}` |  |
| serviceAccount.create | bool | `true` |  |
| serviceAccount.name | string | `""` |  |
| space.enabled | bool | `false` |  |
| version | string | `"v1alpha1"` |  |
