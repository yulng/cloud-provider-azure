# Kubernetes e2e tests

## Prerequisite

- An azure service principal

    Please follow this [guide](https://github.com/Azure/aks-engine/blob/master/docs/topics/service-principals.md) for creating an azure service principal
    The service principal should either have:
    - Contributor permission of a subscription
    - Contributor permission of a resource group. In this case, please create the resource group first

- Docker daemon enabled

## How to run Kubernetes e2e tests locally

1. Prepare dependency project

- [aks-engine](https://github.com/Azure/aks-engine)

  Binary downloads for the latest version of aks-engine for are available [on Github](https://github.com/Azure/aks-engine/releases/latest). Download AKS Engine for your operating system, extract the binary and copy it to your `$PATH`.

  On macOS, you can install aks-engine with [Homebrew](https://brew.sh/). Run the command `brew install Azure/aks-engine/aks-engine` to do so. You can install Homebrew following the [instructions](https://brew.sh/).

  On Windows, you can install aks-engine via [Chocolatey](https://chocolatey.org/) by executing the command `choco install aks-engine`. You can install Chocolatey following the [instructions](https://chocolatey.org/install).

  On Linux, it could also be installed by following commands:
  
  ```sh
  $ curl -o get-akse.sh https://raw.githubusercontent.com/Azure/aks-engine/master/scripts/get-akse.sh
  $ chmod 700 get-akse.sh
  $ ./get-akse.sh
  ```

- [Kubernetes](https://github.com/kubernetes/kubernetes)

    This serves as E2E tests case source, it should be located at `$GOPATH/src/k8s.io/kubernetes`.

    ```sh
    cd $GOPATH/src
    go get -d k8s.io/kubernetes
    ```

- [kubectl](https://kubectl.docs.kubernetes.io/)

  Kubectl allows you to run command against Kubernetes cluster, which is also used for deploying CSI plugins. You can follow [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-binary-with-curl) to install kubectl. e.g. on Linux
  
  ```sh
  curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
  chmod +x kubectl
  sudo mv kubectl /usr/local/bin/
  ```

2. Build docker image `azure-cloud-controller-manager` and push it to your docker image repository.

    ```sh
    git clone https://github.com/kubernetes/cloud-provider-azure $GOPATH/src/k8s.io/cloud-provider-azure
    cd $GOPATH/src/k8s.io/cloud-provider-azure
    export IMAGE_REGISTRY=<username>
    make image push
    ```

3. Deploy a Kubernetes cluster with the above `azure-cloud-controller-manager` image.

   To deploy a cluster, export all the required environmetal variables first and then invoke `make deploy`:

    ```sh
    export RESOURCE_GROUP_NAME=<resource group name>
    export K8S_AZURE_LOCATION=<location>
    export K8S_AZURE_SUBSID=<subscription ID>
    export K8S_AZURE_SPID=<client id>
    export K8S_AZURE_SPSEC=<client secret>
    export K8S_AZURE_TENANTID=<tenant id>
    export USE_CSI_DEFAULT_STORAGECLASS=<true/false>
    
    make deploy
    ```

   To connect the cluster:

    ```sh
    export KUBECONFIG=$GOPATH/src/k8s.io/cloud-provider-azure/_output/$(ls -t _output | head -n 1)/kubeconfig/kubeconfig.$LOCATION.json
    kubectl cluster-info
    ```

To check out more of the deployed cluster , replace `kubectl cluster-info` with other `kubectl` commands. To further debug and diagnose cluster problems, use `kubectl cluster-info dump`


4. Run E2E tests

Please first ensure the kubernetes project locates at `$GOPATH/src/k8s.io/kubernetes`, the e2e tests will be built from that location.

```sh
cd $GOPATH/src/k8s.io/kubernetes

make WHAT='test/e2e/e2e.test'
make WHAT=cmd/kubectl
make ginkgo

export KUBERNETES_PROVIDER=azure
export KUBERNETES_CONFORMANCE_TEST=y
export KUBERNETES_CONFORMANCE_PROVIDER=azure
export CLOUD_CONFIG=$GOPATH/src/k8s.io/cloud-provider-azure/tests/k8s-azure/manifest/azure.json

# some test cases require ssh configurations
export KUBE_SSH_KEY_PATH=path/to/ssh/privatekey
export KUBE_SSH_USER={ssh_user}

# Replace the test_args with your own.
go run hack/e2e.go -- --test --provider=local --check-version-skew=false --test_args='--ginkgo.focus=Port\sforwarding'
```
