This repo contains assets to ease deploying openshift on openshift using kubevirt vms through a tekton pipeline

# Requisites

- openshift cluster (with admin creds)
- openshift pipelines and virtualization deployed
- storage in place

# Configuration

## On kubernetes

```
PROJECT=myproject
kubectl config set-context --current --namespace=$PROJECT
kubectl create -f tekton.yml
```

## On openshift

```
PROJECT=myproject
oc new-project $PROJECT
oc adm policy add-cluster-role-to-user cluster-admin -z pipeline -n $PROJECT
oc adm policy add-scc-to-user anyuid -z pipeline
```

## On both

```
mkdir /tmp/creds
cp ~/.ssh/*pub /tmp/creds
cp openshift_pull.json /tmp/creds
kubectl create configmap credentials --from-file=/tmp/creds
rm -rf /tmp/creds
kubectl create -f pipeline.yml
```

# Launch a deployment

```
oc create -f pipelinerun.yml
```

## Parameters

|Parameter         |Default Value  |
|------------------|---------------|
|cluster           |testk          |
|version           |stable         |
|tag               |4.9            |
|masters           |3              |
|workers           |0              |
|numcpus           |8              |
|memory            |8192           |
|disk_size         |30             |
|network_type      |OVNKubernetes  |
|async             |false          |

# Retrieve credentials

Credentials will be available shortly after the pipeline runs, before completion.

## Gathering task output

The following command will print all the information needed to access the cluster:

- /etc/hosts entry
- kubeadmin password
- full kubeconfig

```
POD=$(oc get pod --sort-by={'.metadata.creationTimestamp'} -o custom-columns=NAME:.metadata.name --no-headers | grep deploy-openshift | tail -1)
oc logs $POD
```

## Gathering task results

You can also gather kubeadmin password and the /etc/hosts entry as results of the pipeline

```
PIPELINERUN=$(oc get pipelinerun --sort-by={'.metadata.creationTimestamp'} -o name | tail -1)
KUBEADMIN_PASSWORD=$(oc get $PIPELINERUN -o jsonpath='{.status.pipelineResults[0].value}')
echo $KUBEADMIN_PASSWORD
```

```
PIPELINERUN=$(oc get pipelinerun --sort-by={'.metadata.creationTimestamp'} -o name | tail -1
oc get $PIPELINERUN -o jsonpath='{.status.pipelineResults[1].value}'
```

Kubeconfig can't be gathered this way (because of the size of the data preventing to store this way)

Note that during a run, if not running in async mode, you can copy kubeconfig with the following command:

```
CLUSTER=magic
POD=$(oc get pod --sort-by={'.metadata.creationTimestamp'} -o custom-columns=NAME:.metadata.name --no-headers | grep deploy-openshift | tail -1)
oc cp $POD:/root/.kcli/clusters/$CLUSTER/auth/kubeconfig kubeconfig.$CLUSTER
```

# Screenshots

![wizard](01.png)


![exec](02.png)


![details](03.png)
