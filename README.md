# Running ShinyProxy on EKS with https termination 
This is mostly a copy of the examples provided by Openanlytics on their [Github page](https://github.com/openanalytics/shinyproxy-config-examples/tree/master/03-containerized-kubernetes).


## How to run

1. Download the `Dockerfile` from the folder `kube-proxy-sidecar`.
2. Open a terminal, go to the directory containing the Dockerfile, and run the following command to build it:

`sudo docker build . -t kube-proxy-sidecar`

3. Upload `kube-proxy-sidecar` image to ECR.
4. Download the `Dockerfile` from the folder `shinyproxy-example`.
5. Open a terminal, go to the directory containing the Dockerfile, and run the following command to build it:

`sudo docker build . -t shinyproxy-example`

6. Upload `shinyproxy-example` image to ECR.
7. Open a terminal on a master node (where the `kubectl` command is available).

8. Download the 3 `yaml` files from the folder where this README is located. 
9. Run the following command to deploy a pod containing `shinyproxy-example` and `kube-proxy-sidecar`:

`kubectl create -f sp-deployment.yaml`

10. Run the following command to grant full privileges to the `default` service account which runs the above pod:

`kubectl create -f sp-authorization.yaml`

11. 

12. Run the following command to deploy a service exposing ShinyProxy outside the cluster:

`kubectl create -f sp-service.yaml`

13. Add an alias entry in Route53 for the classic load balancer that is created automatically when the service is launched.

## Notes on the configuration

* The `kube-proxy-sidecar` container is used to make the apiserver accessible on `http://localhost:8001` to the `shinyproxy-example` container.

* The service will expose ShinyProxy over a classic load balanecer on AWS. Iin the current setup, the DNS name has to be setup manually in Route53.

* If you do not deploy the service, you can still access ShinyProxy from within the cluster on port `8080`.

* To keep the example consise, the `cluster-admin` role is granted to the `default` service account.
  Best-practice would be to add a dedicated service account and reference it via `serviceAccountName` in the deployment spec.
  The following role is the minimal set of permissions:

  ```yaml
  kind: Role
  apiVersion: rbac.authorization.k8s.io/v1
  metadata:
    namespace: example
    name: example
  rules:
  - apiGroups: [""]
    resources: ["pods", "pods/log"]
    verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
  ```
