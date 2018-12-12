### Reconciliation Algorithm for EtcdCluster CRD

Following is the list of tasks that needs to be carried out every time reconciliation is triggered. Note, the input to the ansible playbook would be extra-vars with meta key having the metadata of the CR like the name, namespace, etc and the spec variables converted to snake case as ansible variables. The list of task to be carried out during a reconciliation cycle is as follows:

1. Query the Kube API server to find the number of services and pods in the cluster using appropriate label selector.
2. If the number of services returned is 0, assume that the cluster does not exist and set etcd_cluster_state, as “new”.
    1. Create the two services using k8s modules with appropriate labels.
    2. Create the pod names. It is important to pre-compute the pod names. As described earlier, this would help in creating all the configuration values that the pods need in order to start the etcd process.
    3. Create all the configuration values described in Background knowledge, Deploying the cluster subsection. 
    4. Iterate over the pod names and create the pods in the kubernetes cluster.
3. If the length of the services return is not 0, set etcd_cluster_state as “existing”
4. If etcd_cluster_state is “existing”, iterate over all the pods to ensure they are using the desired etcd version.
5. Query the etcd cluster member and store the results in etcd_cluster_members.
6. Filter the pods that are not part of etcd_cluster_member and add them etcd_remove_pods.
7. Filter the pods with status.Phase != “Running” and add them to etcd_remove_pods.
8. Iterate over etcd_remove_pods and delete the pods. This would delete all the pods that are not part of the etcd cluster
9. If the size of the cluster is less than the desired size:
    1. Generate a single pod name.
    2. Add the pod to the etcd cluster using the etcd module.
    3. With the current etcd member names, generate all the configuration values for the pods described in Background knowledge, Deploying the Cluster subsection.
    4. Create the pods with appropriate labels.
10. If the size of the cluster is greater than the desired size:
    1. Randomly select one pod name from the existing pods to delete.
    2. Get the pod id from the pod’s name
    3. Remove the etcd cluster member using the etcd module, by giving it the id to remove.
    4. Delete the pods from the k8s cluster.


The etcd–ansible-operator triggers a reconciliation loop every 8 seconds. During the first run of the playbook when etcd_cluster_phase is “new”, the operator will spin up all the pods of the cluster. However, after this, if the size is changed, the operator scales the cluster by creating/deleting the pod one at a time. The above algorithm will implement deployment, scaling and failover for etcd clusters. Slight modification to the above algorithm can deploy and manage a TLS enabled etcd cluster. During the generation of the configuration in step 2.c of the above algorithm, if TLS is enabled, each pod can be mounted with the TLS secrets that carry the certs. This would make sure that the certs are populated inside each pod and then the etcd process is started with TLS enabled. It is important to note that if client communication is secured by TLS then the operator has to communicate with the etcd cluster using the same certs.
