# SingleStore DB Helm Chart for K8s environment

This Helm chart is to be used for deploying SingleStore DB in K8s environment.

## Documentation

Please visit [SingleStore DB Helm Chart for K8s](https://docs.singlestore.com).

## A. Deployment with existing storage class
We use "memsql" as namespace (You can change it)

1. Deploy the CRD and namespace creation
    1. Deployment
```
helm install singlestore singlestore -nmemsql --create-namespace
```
    2. Verify
```
helm ls -nmemsql
```

2. Deploy the operator & cluster
```
helm upgrade singlestore singlestore --set memsqlCluster.install=true --reuse-values -nmemsql
```
## B. Deployment with new local-storage class for non production environment only
1. Deploy the CRD, storage class and namespace creation
a. deployment
```
helm install singlestore singlestore -nmemsql --set hostPV.install=true --create-namespace
```
b. verify
```
helm ls -nmemsql
```
2. Provision local-storage manually
a. create folder in all k8s nodes
```
sudo mkdir -p /mnt/disk/vol{1,2,3,4,5,6,7,8,9}
sudo chown -R 1000:1000 /mnt/disk
```
b. create PV
```
helm upgrade singlestore singlestore --reuse-values -nmemsql
```
3. Deploy the operator & cluster
a. deploy with local storage
```
helm upgrade singlestore singlestore --set memsqlCluster.install=true --reuse-values -nmemsql
```
# To deploy MemSQL Studio
```
helm upgrade singlestore singlestore --set memsqlCluster.install=true --reuse-values -nmemsql
```
# To scaling up/down, example
```
helm upgrade singlestore singlestore --set memsqlCluster.leafSpec.count=3 --reuse-values -nmemsql
helm upgrade singlestore singlestore --set memsqlCluster.aggregatorSpec.count=3 --reuse-values -nmemsql
```
## Sample values-override.yaml file
Below is a sample values-override.yaml file that shows how to override 
configurable parameters of this Helm Chart.

```yaml
# fsGroup is the only parameter that you have to set. Set it to the starting value of 
# the namespace's
fsGroup: "5555"

state: "enabled"

# Ideally, choose a StorageClass that supports volume expension and WaitForFirstConsumer
# binding mode and that uses a block level storage.
storageClassName: gp

operatorImage: memsql/operator:1.2.2-93a97e50

# nodeImageVersion should be consistent with nodeImageTag
nodeImageVersion: 7.1.4
nodeImageTag: centos-7.1.4-516dfe4088
nodeImageRepo: memsql/node

# Make sure to use string for the following three values
coresPerUnit: "8"
memoryPerUnit" "32"

memsqlOperatorDeployment:
  serviceAccountName: memsql-operator
  name: memsql-operator
  replicas: 1
  additionalArgs:
  - "--priority-class-name"
  - "my-priority-class"

memsqlCluster:
  name: memsql-cluster

  # Must include either license or licenseSecret, but not both.
  # If both are specified, license will take precedence.
  license: BDAwMDA...Hbi/AA==
  licenseSecret:
    # The name of the secret that contains the license
    name: license-secret
    # The key name in the secret whose value is contains the license
    key: license

  # Must include either adminHashedPassword or adminHashedPasswordSecret, but not both.
  # If both are specified, adminHashedPassword will take precedence.

  # How to generate hassed password in python 2.x
  # from hashlib import sha1
  # print("*" + sha1(sha1('mypassword').digest()).hexdigest().upper())
  # *FABE5482D5AADF36D028AC443D117BE1180B9725
  # OR from mysql/memsql 
  # SELECT password("secret");
  # *14E65567ABDB5135D0CFD9A70B3032C179A49EE7
  adminHashedPassword: "*14E65567ABDB5135D0CFD9A70B3032C179A49EE7"

  adminHashedPasswordSecret:
    # The name of the secret that contains the hashed admin password
    name: admin-password
    # The key name in the secret whose value is contains the hashed admin password
    key: admin-password-key

  redundancyLevel: 2

  aggregatorSpec:
    count: 2
    height: 0.5
    storageGB: 10

  leafSpec:
    count: 2
    height: 0.5
    storageGB: 20

  # For additional configurations, such as serviceSpec, backupSpec, etc.
```
