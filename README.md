# jenkins

# Jenkins server in your new Kubernetes cluster

## Goal

* Going one step back, let's put to good use that NFS server that you have created.
* We will use the NFS share as Persistent Volume and we will create a StorageClass in order bring automation to the process.

## Prerequisites

* From official [guid](https://www.jenkins.io/doc/book/installing/kubernetes/) follow the steps 1 and 2. This will allow you to setup ClusterRole, ServiceAccount and ClusterRoleBinding for your Jenkins master server.
* Skip step 3 and do the following.
* You need to setup nfs-subdir-external-provisioner [more info here](https://kubernetes.io/docs/concepts/storage/storage-classes/#nfs)

* Using helm setup the repo.

```
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
```

Let's take full advantage of Helm capabilities to set the namespace and other parameters for our StorageClass. Note that the defaultClass=true setting will make the new StorageClass the default for the devops-tools namespace. This means that every Persistent Volume claim made in the namespace will use this StorageClass as the default, unless specified otherwise. Adjust the nfs.server value according to the hostname of your NFS server.

* Nb! pi3-00.local is the hostname of my nfs server and you need to adjust accordingly.

```
helm install nfs-devops-tools nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
--namespace=devops-tools \
--set nfs.server=pi3-00.local \
--set nfs.path=/mnt/nfs/devops-tools \
--set storageClass.name=devops-tools \
--set storageClass.defaultClass=true \
--set storageClass.reclaimPolicy=Delete \
--set storageClass.provisionerName=devops-tools/nfs-subdir-external-provisioner \
--set storageClass.allowVolumeExpansion=true
```

* Check the result with

```kubectl get pods --namespace=devops-tools
```

## Deploy Jenkins

* Use the provided manifests to set up the Persistent Volume (`pvc-jenkins.yaml`) and the Service (`svc-jenkins.yaml`).
* Query the results with:

```
kubectl get pv,pvc --namespace=devops-tools
kubectl get svc jenkins-service --namespace=devops-tools
```

* Now, deploy the Jenkins server using the `deployment-jenkins.yaml manifest`.
* Note: The readiness and liveness probes are set to 40 minutes. You may adjust the manifests as needed.

## Finalise

* Check if jenkins is persisted on the nfs!
* ssh to the nfs server and run:

```
ls -l /mnt/nfs/devops-tools/devops-tools-jenkins-pv-claim-pvc-6efb861c-.../war/
```

* Jenkins server should be running on `Worker-node IP:32000`.
* You will be asked for the token. To get the token use:

```
kubectl logs <jenkins pod name>
```

* Please make sure to replace `<nfs-server-hostname>`, `<jenkins-server-url>`, `<jenkins-pod-name>`, and the ellipsis (`...`) in the command with the appropriate values based on your setup.

* Feel free to customize the formatting and styling as per your preference.