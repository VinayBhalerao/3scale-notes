## Installing Openshift Service Mesh and enable 3scale API Management

```
Version
Openshift : 3.11.43
3scale: 2.5
```

## Steps

Edit the /etc/origin/master/master-config.yaml on master

```
sudo -i
oc get nodes
ssh master1.CITY-GUID.internal
vi /etc/origin/master/master-config.yaml
```

Replace `admissionConfig` with below snippet (Remember to take backup of original file)

```
admissionConfig:
  pluginConfig:
    MutatingAdmissionWebhook:
      configuration:
        apiVersion: apiserver.config.k8s.io/v1alpha1
        kubeConfigFile: /dev/null
        kind: WebhookAdmission
    ValidatingAdmissionWebhook:
      configuration:
        apiVersion: apiserver.config.k8s.io/v1alpha1
        kubeConfigFile: /dev/null
        kind: WebhookAdmission
```

Restart the master

```
/usr/local/bin/master-restart api
/usr/local/bin/master-restart controllers
```

Create projects

```
oc new-project istio-operator
oc new-project istio-system

oc apply -n istio-operator -f https://raw.githubusercontent.com/Maistra/istio-operator/maistra-0.10/deploy/servicemesh-operator.yaml

oc get pods -n istio-operator -l name=istio-operator

oc create -n istio-system -f https://gist.githubusercontent.com/PhilipGough/3ae01b461344c17e9ccc3d01ff9b575d/raw/d6e62197f861b9c869268a12e61a74fa6d599f01/cp.yml
```

If `Pilot` deployment fails, change the resource memory to `10M`. Line 135. Edit the deploymentConfig yaml.

```
resources:
requests:
  cpu: 500m
  memory: 10M
```

```
oc get cm -n istio-system istio -o jsonpath='{.data.mesh}' | grep disablePolicyChecks
oc edit cm -n istio-system istio
Locate *disablePolicyChecks: true* within the ConfigMap and change the value to false.
```

Deploy 3scale adaptor

```
git clone https://github.com/3scale/3scale-istio-adapter.git
cd 3scale-istio-adapter/
vi istio/threescale-adapter-config.yaml with correct values
oc create -f deploy/
oc create -f istio/
```

Deploy Bookinfo
```
oc new-project bookinfo
oc adm policy add-scc-to-user anyuid -z default -n bookinfo
oc adm policy add-scc-to-user privileged -z default -n bookinfo
oc apply -n bookinfo -f https://raw.githubusercontent.com/Maistra/bookinfo/master/bookinfo.yaml
oc apply -n bookinfo -f https://raw.githubusercontent.com/Maistra/bookinfo/master/bookinfo-gateway.yaml
```

Change Kiali service with correct 3scale rules



