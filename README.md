This repo contains assets to ease deploying openshift on kubernetes/openshift using kubevirt vms, all through a tekton pipeline

# Requisites

- kubernetes/openshift cluster
- tekton/openshift pipelines and kubevirt/openshift virtualization deployed
- storage in place (for the disks of the vms)

# Configuration

```
mkdir /tmp/creds
cp ~/.ssh/*pub /tmp/creds
cp openshift_pull.json /tmp/creds
kubectl create configmap credentials --from-file=/tmp/creds
rm -rf /tmp/creds
kubectl create -f pipeline.yml
```

Note that the pipeline can easily be extended  with the parameters available [here](https://github.com/karmab/kcli/blob/master/kvirt/openshift/kcli_default.yml)

# Launch a deployment

```
oc create -f pipelinerun.yml
```

## Parameters

|Parameter         |Default Value  |
|------------------|---------------|
|cluster           |testk          |
|domain            |karmalabs.com  |
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

Note that during a run, if not running in async mode, you can copy kubeconfig and /etc/hosts with the following commands:

```
CLUSTER=magic
POD=$(oc get pod --sort-by={'.metadata.creationTimestamp'} -o custom-columns=NAME:.metadata.name --no-headers | grep deploy-openshift | tail -1)
oc cp $POD:/tekton/home/.kcli/clusters/$CLUSTER/auth/kubeconfig kubeconfig.$CLUSTER
oc cp $POD:/etc/hosts hosts.$CLUSTER
```

# Screenshots

![wizard](img/01.png)


![exec](img/02.png)


![details](img/03.png)
