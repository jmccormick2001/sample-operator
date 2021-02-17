# sample-operator

An example operator to test upgrade features with...


# Versions 

Here are versions this example has been developed and tested with:

 * k8s -  1.20
 * operator-sdk - 1.4.0
 * kubebuilder -  2.3.1
 
# Environment Setup

 * install k8s
 * install operator-sdk
 * install olm on your cluster

# Steps to Build

```bash=
make docker-build docker-push IMG=docker.io/jmccormick2001/sample-operator:v1.0.3
make bundle
```

Edit the annotations.yaml, and Makefile to specify a channel and default
channel to be **stable** instead of **alpha**

Edit the CSV to use your docker.io/jmccormick2001/sample-operator:v1.0.3 image!


```bash=
make bundle-build BUNDLE_IMG=docker.io/jmccormick2001/sample-operator-bundle:v1.0.3
make docker-push IMG=docker.io/jmccormick2001/sample-operator-bundle:v1.0.3
operator-sdk bundle validate docker.io/jmccormick2001/sample-operator-bundle:v1.0.3
opm index add --bundles docker.io/jmccormick2001/sample-operator-bundle:v1.0.3 --tag docker.io/jmccormick2001/sample-operator-index:v1.0.3
or
opm index add --bundles docker.io/jmccormick2001/sample-operator-bundle:v1.0.0,\
docker.io/jmccormick2001/sample-operator-bundle:v1.0.3 --tag docker.io/jmccormick2001/sample-operator-index:v1.0.3

podman push docker.io/jmccormick2001/sample-operator-index:v1.0.3
```

Create the catalogsource that points to this new catalog index image:
```bash=
kubectl -n operators create -f catalogsource.yaml
```

Verify it is running and serving content:
```bash=
kubectl -n operators get pod
kubectl -n operators logs --selector=olm.catalogSource=sample-operator
```

patch to perform an upgrade from 1.0.0:

```bash=
kubectl patch subscription sample-operator-subscription -n operators --type json -p '[{"op": "add", "path": "/spec/source","value": "sample-operator-hotfix"}]'
```

or 
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
sample-operator.v1.0.3   sampleoperator   1.0.3                Succeeded
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
kubectl -n operators delete csv sample-operator.v1.0.3
```
