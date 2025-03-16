## Configure storage for the Image Registry

### Configure NFS server 
- Install and configure NFS for the OpenShift Registry. It is a requirement to provide storage for the Registry, emptyDir can be specified if necessary.

```yaml
Install NFS Server:
dnf install nfs-utils -y 

Create the Share, check available disk space and its location df -h :
mkdir -p /shares/registry
chown -R nobody:nobody /shares/registry
chmod -R 777 /shares/registry

Export the Share:
echo "/shares/registry  192.168.22.0/24(rw,sync,root_squash,no_subtree_check,no_wdelay)" > /etc/exports
exportfs -rv

Set Firewall rules:
firewall-cmd --zone=internal --add-service mountd --permanent
firewall-cmd --zone=internal --add-service rpc-bind --permanent
firewall-cmd --zone=internal --add-service nfs --permanent
firewall-cmd --reload

Enable and start the NFS related services:
systemctl enable nfs-server rpcbind
systemctl start nfs-server rpcbind nfs-mountd
```

### Configure storage for the Image Registry

A Bare Metal cluster does not by default provide storage so the Image Registry Operator bootstraps itself as 'Removed' so the installer can complete. As the installation has now completed storage can be added for the Registry and the operator updated to a 'Managed' state.

Create the 'image-registry-storage' PVC by updating the Image Registry operator config by updating the management state to 'Managed' and adding 'pvc' and 'claim' keys in the storage key:
```yaml
oc edit configs.imageregistry.operator.openshift.io

managementState: Managed

storage:
  pvc:
    claim: # leave the claim blank
```

```yaml
Confirm the 'image-registry-storage' pvc has been created and is currently in a 'Pending' state:
oc get pvc -n openshift-image-registry

Create the persistent volume for the 'image-registry-storage' pvc to bind to:
oc create -f ~/ocp4-metal-install/manifest/registry-pv.yaml

After a short wait the 'image-registry-storage' pvc should now be bound:
oc get pvc -n openshift-image-registry
```
