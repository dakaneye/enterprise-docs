---
title: "Image Runtime Inventory"
linkTitle: "Image Inventory"
weight: 1
---

Anchore can connect to your runtime environment and discern which images exist in each context (i.e. Kubernetes Namespace).

Currently, supported runtime environments include:
* Kubernetes

# Kubernetes Runtime Inventory 
## Configuration
Anchore uses a go binary called [kai](https://github.com/anchore/kai) that leverages the [Kubernetes Go SDK](https://github.com/kubernetes/client-go) to reach out and list
pods in a configurable set of namespaces. This binary is installed into the Enterprise Image and can be configured using the REST API

Authentication with the Kubernetes API can be done similarly to how a kubeconfig is configured. Anchore supports either a Service Account token or a client private key/certificate combination.

To generate a Service Account token with the necessary permissions (view), follow these steps:
1. Create a Service Account: `$ kubectl create serviceaccount <name> -n <namespace>`
    * ex. `$ kubectl create serviceaccount anchore-runtime-inventory -n default`
1. Bind the Service Account to a Role: `$ kubectl create clusterrolebinding <binding-name> --clusterrole=view --serviceaccount=<namespace>:<name>`
    * ex. `$ kubectl create clusterrolebinding anchore-runtime-inventory-binding --clusterrole=view --serviceaccount=default:anchore-runtime-inventory`
1. Get the token name: ```$ TOKENNAME=`kubectl -n <namespace> get serviceaccount/<name> -o jsonpath='{.secrets[0].name}'```
    * ex. ```$ TOKENNAME=`kubectl -n default get serviceaccount/anchore-runtime-inventory -o jsonpath='{.secrets[0].name}'```
1. Get the token value, which corresponds to the "credential" field below: ```$ TOKEN=`kubectl -n <namespace> get secret $TOKENNAME -o jsonpath='{.data.token}'```
    * ex. ```$ TOKEN=`kubectl -n default get secret $TOKENNAME -o jsonpath='{.data.token}'```

The endpoints are documented under the Enterprise Swagger, but are also demonstrated below
```shell script
$ cat kai-demo-cluster.json
{
	"cluster_name": "docker-desktop-token",
	"inventory_type": "kubernetes",
	"cluster_config": {
		"cluster_certificate": "ABC", # redacted
		"cluster_server": "https://kubernetes.docker.internal:6443",
		"credential": "XYZ", #redacted
		"credential_type": "token",
		"namespaces": [
			"all"
		]
	}
}
$ curl -X POST -u admin:foobar -H "Content-Type: application/json" --data-binary "@kai-demo-cluster.json" "http://localhost:8228/v1/enterprise/inventories/clusters"
```
Some of the fields above within the `cluster_config` correspond to fields you would see in your kubeconfig:
* **cluster_certificate:** corresponds to `clusters[0].cluster.certificate-authority-data`
* **cluster_server:** corresponds to `clusters[0].cluster.server`
* **credential:** corresponds to either a service account token or the private key of a user. Note: if a private key is supplied, then a **client_certificate** must also be specified. 
    * The private key may be found at `users[0].user.client-key-data`.
    * The token, if included in a kubeconfig, would be found at `users[0].user.token`
* **client_certificate:** (required only for private key authentication), corresponds to `users[0].user.client-certificate-data`
* **credential_type:** values can either be "token" or "private_key" 
* **namespaces:** configures which namespaces anchore will pull a runtime inventory from

### Usage
Once a cluster is configured, Anchore will execute the kai binary on an interval, parse the JSON output, and then store the current snapshot of the inventory.
That inventory may be listed by running HTTP GET on `/v1/enterprise/inventories`.

To configure the interval on which the kai binary runs, adjust the value of the `ANCHORE_CATALOG_K8S_INVENTORY_REPORT_INTERVAL_SEC` environment variable on the enterprise catalog service. By default, it runs every 5 minutes.

Clusters may be retrieved in list format via HTTP GET on `/v1/enterprise/inventories/clusters` or by name, with HTTP GET on `v1/enterprise/inventories/clusters/<name>`
They may also be deleted via HTTP DELETE on `/v1/enterprise/inventories/clusters/<name>`. Once the cluster is removed, inventory will no longer be reported.

## External Configuration
If you would prefer to not provide Kubernetes API access to Anchore directly, **KAI** can be configured to run as an agent within your cluster, or as a command line tool.
When running externally, if Anchore URL & Anchore API credentials (basic auth) are provided to KAI it can sync the inventory information to Anchore.
Head over to the **KAI** [repository](https://github.com/anchore/kai) for details on how to run it externally.