# sample-operator

An example operator to test upgrade features with...

Here we will demonstrate how to create a hotfix (1.0.1), then
upgrade an installed operator at version 1.0.0 to that hotfix.
Lastly, we show how to upgrade the operator to something beyond
the hotfix like 1.1.0.

1.0.0 -> 1.0.1 (skipRange <1.0.1) -> 1.1.0 (skipRange <1.1.0)

We will use skipRange settings to facilitate the upgrade process.

# Versions 

Here are versions this example has been developed and tested with:

 * k8s -  1.20
 * operator-sdk - 1.4.0
 * kubebuilder -  2.3.1
 * opm -  1.16.1
 
# Environment Setup

 * install k8s
 * install operator-sdk
 * install olm on your cluster

# Steps to Build

```bash=
make docker-build docker-push IMG=docker.io/jmccormick2001/sample-operator:v0.0.1
make bundle
```

Edit the annotations.yaml, and Makefile to specify a channel and default
channel to be **stable** instead of **alpha**

Edit the CSV to use your docker.io/jmccormick2001/sample-operator:v0.0.1 image!


```bash=
make bundle-build BUNDLE_IMG=docker.io/jmccormick2001/sample-operator-bundle:v0.0.1
make docker-push IMG=docker.io/jmccormick2001/sample-operator-bundle:v0.0.1
operator-sdk bundle validate docker.io/jmccormick2001/sample-operator-bundle:v0.0.1
opm index add --bundles docker.io/jmccormick2001/sample-operator-bundle:v0.0.1 --tag docker.io/jmccormick2001/sample-operator-index:v0.0.1
podman push docker.io/jmccormick2001/sample-operator-index:v0.0.1
```

Create the catalogsource that points to this new catalog index image:
```bash=
kubectl -n operators create -f catalogsource.yaml
```

Verify it is running and serving content:
```bash=
kubectl -n operators get pod
kubectl -n operators logs sample-operator-xxxxx
```

Create the subscription:
```bash=
kubectl create -f subscription.yaml
kubectl -n operators get subscription
kubectl -n operators get csv
```

Verify you get a Succeeded CSV like this:
```bash=
kubectl -n operators get csv
NAME                     DISPLAY          VERSION   REPLACES   PHASE
sample-operator.v0.0.1   sampleoperator   0.0.1                Succeeded
```

If not, verify that the pod is running:
```bash=
 kubectl -n operators get pod
NAME                                                              READY   STATUS      RESTARTS   AGE
88e3b1c30c03749d2898fbef4352f970608b058153a4bd9a25a911f6eaqgvhv   0/1     Completed   0          20s
sample-operator-2dnpd                                             1/1     Running     0          45s
sample-operator-controller-manager-7b44c4885-qjwpw                2/2     Running     0          13s
```

Create a sampleoperator Custom Resource to test the deployed operator:
```bash=
kubectl -n operators create -f config/samples/cache_v1alpha1_sampleoperand.yaml
kubectl -n operators get sampleoperands
kubectl -n operators logs pod/sample-operator-controller-manager-7b44c4885-qjwpw -c manager
```

If everything is working you should see in the operator log a message showing  you had just created a custom resource.


## Cleanup

You can completely clean up your operator deployment, catalog, and subscription as follows:

```bash=
kubectl -n operators delete subscription/sample-operator-subscription
kubectl -n operators delete catalogsource/sample-operator
kubectl -n operators delete deploy/sample-operator-controller-manager
kubectl -n operators delete service/sample-operator-controller-manager-metrics-service deployment.apps/sample-operator-controller-manager
kubectl -n operators delete csv sample-operator.v0.0.1
```
