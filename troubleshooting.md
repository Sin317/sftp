In case ha-proxy isn't available and you want to reach the sftp using NodePort:

1. If the service is not working properly ie the access to nodeport isnt working, run this command:
  `oc expose deploy sftp --type=NodePort --name=sftp` 
 
2. Use this to access the sftp by:
  `sftp -P <NodePort> <user>@<master node IP>`
  
  for example: here NodePort is: 30654
  `oc get svc
  NAME   TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
  sftp   NodePort   172.30.121.53   <none>        2222:30654/TCP   12h`
  
  and master node IP can be any one of the following internal IP's:
  ` oc get no -o wide
  NAME          STATUS   ROLES                                                    AGE     VERSION           INTERNAL-IP     EXTERNAL-IP   OS-IMAGE                                                       KERNEL-VERSION                 CONTAINER-RUNTIME
  master-1      Ready    master,worker                                            5d13h   v1.19.0+e49167a   172.29.231.43   <none>        Red Hat Enterprise Linux CoreOS 46.82.202101301821-0 (Ootpa)   4.18.0-193.41.1.el8_2.x86_64   cri-o://1.19.1-7.rhaos4.6.git6377f68.el8
  master-2      Ready    master,worker                                            5d13h   v1.19.0+e49167a   172.29.231.44   <none>        Red Hat Enterprise Linux CoreOS 46.82.202101301821-0 (Ootpa)   4.18.0-193.41.1.el8_2.x86_64   cri-o://1.19.1-7.rhaos4.6.git6377f68.el8
  master-3      Ready    master,worker                                            5d13h   v1.19.0+e49167a   172.29.231.45   <none>        Red Hat Enterprise Linux CoreOS 46.82.202101301821-0 (Ootpa)   4.18.0-193.41.1.el8_2.x86_64   cri-o://1.19.1-7.rhaos4.6.git6377f68.el8
  worker-1-cu   Ready    ran-cu-bangalore-server-profile-cu-dell-r640-00,worker   4d19h   v1.19.0+e49167a   172.29.231.52   <none>        Red Hat Enterprise Linux CoreOS 46.82.202101301821-0 (Ootpa)   4.18.0-193.41.1.el8_2.x86_64   cri-o://1.19.1-7.rhaos4.6.git6377f68.el8
  worker-2-du   Ready    ran-du-mysore-server-profile-du-supermicro00,worker      4d19h   v1.19.0+e49167a   172.29.231.28   <none>        Red Hat Enterprise Linux CoreOS 46.82.202101301821-0 (Ootpa)   4.18.0-193.41.1.el8_2.x86_64   cri-o://1.19.1-7.rhaos4.6.git6377f68.el8`
  
  
--------------------------------------------------------------------------------------------------------------------------------------------------------------------

1. If you want to add the external IP to the service, patch it using this command:
  `oc patch svc sftp -p ‘{“spec”:{“externalIPs”:[“172.29.231.122”]}}'` 
  where external IP is in the subnet of the node's IP and is not being used by anything else.
  (checking if IP is not being used: ping it and it should not be reachable)
