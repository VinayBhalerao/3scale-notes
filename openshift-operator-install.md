## Install 3scale 2.6 using Operator 

```
wget https://github.com/3scale/3scale-operator/archive/3scale-2.6.0-CR2.tar.gz

tar -xvf 3scale-2.6.0-CR2.tar.gz

#add the correct image name to pull from registry
vi deploy/operator.yaml
spec::containers::image: `registry.redhat.io/3scale-amp26/3scale-operator`

oc new-project 3scale

for i in `ls deploy/crds/*_crd.yaml`; do oc create -f $i ; done

oc create -f deploy/service_account.yaml
oc create -f deploy/role.yaml
oc create -f deploy/role_binding.yaml
oc create -f deploy/operator.yaml

#Check if operator is correctly installed
oc get deployment 3scale-operator

# Deploy Custom Resource
#Remember to edit `spec::wildcardomain` value
oc create -f operator-install.yaml
```
