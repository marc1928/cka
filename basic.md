
#Docker vs conatainerD

  within k8s we have Container Runtime Interface (CRI) 
  within CRI we have vendor (fournissseur) who is contain Open Container Initiative(OCI)
  and OCI have : imagespec and runtimespec
  
  ctr :    
  nerdctl:
  crictl:
  
# ETC
  run etcd service : ./etcd
  port by default: 2379
  command line client : ./etcdctl (e.g: ./etcdctl set key1 value1)
  
  To set the right version of API set the environment variable ETCDCTL_API command
	export ETCDCTL_API=3
	
# kube-api server
  
  - AUthenticate user
  - Vaidate Request
  - Store and Retrieve data in the ETCD database
  - Update ETCD 
  - Scheduler 
  - Kubelet

# Controler Manager
The controler its the process that 
  - watch the status
  - Remadiate situation