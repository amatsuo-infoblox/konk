# konk

konk - Kubernetes On Kubernetes - is a tool for deploying an independent Kubernetes API server within Kubernetes.

konk can be used as part of a larger application to manage resources via CustomResourceDefinitions and implement a CRUD API for those resources by leveraging kube-apiserver. Or implement an [extension API server](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/apiserver-aggregation/) without worrying about [breaking the parent cluster with a non-compliant API](https://github.com/kubernetes/kubernetes/issues/96066).

konk does not start a kubelet and therefore does not support any resources that require a node such as deployments and pods.

This repo provides a konk helm chart that can be used to deploy an instance of konk with helm, and a konk-operator that watches for konk CRs and will deploy a konk instance for each of them.

### Office Hour

Office Half Hour 9:30am PST/12:30pm EST on Fridays landing on even days [https://infoblox.zoom.us/j/98130157567](https://infoblox.zoom.us/j/98130157567)

## konk chart

Found in [helm-charts/konk](helm-charts/konk).

This chart will deploy konk.

### Example usage:

    helm install my-konk ./helm-charts/konk
    helm test my-konk

## konk operator

konk-operator is generated with operator-sdk and implements a helm operator for the konk chart and the konk-service chart. Once deployed, `Konk` and `KonkService` resources can be created in the cluster and the operator will deploy a konk instance and a konk-service instance for each of them.
konk-operator applies the contents of the CR `.spec` as values in the respective helm chart.

[`Konk` spec doc](helm-charts/konk/README.md)

[Example `Konk` CR](examples/konk.yaml)

## konk-service chart

Found in [helm-charts/konk-service](helm-charts/konk-service).

The konk-service chart or `KonkService` CR will deploy the resources required to register an `APIService` in konk. It requires an existing konk to be deployed in the cluster. You need to specify the name of the konk and the name of the service that the `APIService` object being created should point to. This would be specified under `konk.name` and `service.name` in the `values.yaml` file. Also specify the `group` and `version` values to be populated in the generated APIService.

[`KonkService` spec doc](helm-charts/konk-service/README.md)

[Example `KonkService`](examples/konk-service.yaml)

### front-proxy ingress

Setting `.spec.ingress.enabled=true` in your `KonkService` will provision front-proxy ingress to the `APIService` you registered in konk. The front-proxy certs are provisioned automatically.

```yaml
kind: KonkService
...
spec:
  ...
  ingress:
    enabled: true
    hosts:
    - host: my-api.example.com
    tls:
    - hosts:
      - my-api.example.com
      secretName: my-api.example.com-tls
```

## example apiserver chart

Found in [helm-charts/example-apiserver](helm-charts/example-apiserver).

This chart will deploy an example-apiserver instance, which is a reference implementation of an extension API server and its usage with konk. This chart requires an existing konk to be deployed in the cluster. This chart also assumes that the konk-operator has been deployed to the cluster, since it involves creating a `KonkService` CR.

# Getting Started

Add the konk repo:

    helm repo add konk https://infobloxopen.github.io/konk

### Optional, stand up KIND

    make kind kind-load-konk
    # Teardown cluster
    make kind-destroy

### Install

    helm install konk-operator konk/konk-operator --devel

### Create a konk

[examples/konk.yaml](examples/konk.yaml) is a simple manifest for a konk resource.

    % kubectl apply -f https://raw.githubusercontent.com/infobloxopen/konk/master/examples/konk.yaml
    konk.konk.infoblox.com/example-konk created

### Observe your new konk

    % kubectl get pods -l app.kubernetes.io/instance=example-konk
    NAME                                READY   STATUS    RESTARTS   AGE
    example-konk-6b4996448c-skjxw       2/2     Running   0          83s
    example-konk-init-d6cff859c-2w4bm   1/1     Running   0          83s

### Deploy example apiserver with konk

    make deploy-example-apiserver
    make test-example-apiserver

# Usage

The [example-apiserver](helm-charts/example-apiserver) in this project provides a complete usage example of Konk. This section will cover some of the most important parts.

## Accessing konk's apiserver

You will need a kubeconfig to authenticate with konk's apiserver. If you deploy a `KonkService`, a kubeconfig will be generated and stored in a `Secret` that can be mounted in your pods. The secret will be named like `[KonkService.metadata.name]-konk-service-kubeconfig`. A good example of accessing konk is [test-setup.yaml](helm-charts/konk-service/templates/tests/test-setup.yaml). It relies on a template to determine the name of the secret. You may wish to reuse this template. It can be found in [helpers.tpl](helm-charts/example-apiserver/templates/_helpers.tpl#L72-L80).

## Server Certificates in your extension API server

`KonkService` will register your extension API server with konk. Trust is established with a CA bundle. The certs are generated by the `KonkService` and cert-manager and must be loaded into the server for konk to trust it. An example of that can be found in the [example-apiserver deployment](helm-charts/example-apiserver/templates/deployment.yaml#L99-L101). Here a secret named like `[KonkService.metadata.name]-konk-service-server` is mounted as a volume into the path `/apiserver.local.config/certificates` and then loaded into the API server via command line arguments.

Optionally, your service can bring its own certificates instead of relying on those generated by `KonkService` and cert-manager. To specify a custom certificate secret name, set `.spec.service.caSecretName` in `KonkService`, and it will be registered with Konk by the `KonkService`.

# Troubleshooting

konk-operator will update the `status` field on Konk and KonkService resources. The contents here can be particularly helpful for determining why konk is not functioning properly.

Here's an example of a healthy konk status:
```json
% kubectl get konks runner-konk -o jsonpath='{.status.conditions}' | jq
[
  {
    "lastTransitionTime": "2020-11-10T00:29:56Z",
    "status": "True",
    "type": "Initialized"
  },
  {
    "lastTransitionTime": "2020-11-10T00:29:59Z",
    "message": "1. Get the application URL by running these commands:\n  export POD_NAME=$(kubectl get pods --namespace default -l \"app.kubernetes.io/name=konk,app.kubernetes.io/instance=runner-konk\" -o jsonpath=\"{.items[0].metadata.name}\")\n  echo \"Visit http://127.0.0.1:8080 to use your application\"\n  kubectl --namespace default port-forward $POD_NAME 8080:80\n",
    "reason": "InstallSuccessful",
    "status": "True",
    "type": "Deployed"
  }
]
```
`"reason": "InstallSuccessful"` is a good sign!

Here's an example of an unhealthy konk:
```json
% k -n aggregate get konks tagging-aggregate-api-konk -o jsonpath='{.status.conditions}' | jq
[
  {
    "lastTransitionTime": "2020-11-10T13:50:38Z",
    "status": "True",
    "type": "Initialized"
  },
  {
    "lastTransitionTime": "2020-11-10T13:50:41Z",
    "message": "failed to install release: rendered manifests contain a resource that already exists. Unable to continue with install: Service \"tagging-aggregate-api-konk\" in namespace \"aggregate\" exists and cannot be imported into the current release: invalid ownership metadata; label validation error: missing key \"app.kubernetes.io/managed-by\": must be set to \"Helm\"; annotation validation error: missing key \"meta.helm.sh/release-name\": must be set to \"tagging-aggregate-api-konk\"; annotation validation error: missing key \"meta.helm.sh/release-namespace\": must be set to \"aggregate\"",
    "reason": "InstallError",
    "status": "True",
    "type": "ReleaseFailed"
  }
]
```
`"reason": "InstallError"` means there was a problem while trying to reconcile the konk's resources. The message explains exactly what the problem is; there's a resource name conflict: `Service \"tagging-aggregate-api-konk\" in namespace \"aggregate\" exists and cannot be imported into the current release`. One potential solution to this problem is to choose a more unique name for your Konk, but in this particular case the conflicting resource was not longer needed and the problem was resolved by manually deleting it.
