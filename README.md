# sftp
# SFTP Centos Docker Image Creation  

1. Download the files in docker_files folder
2. Navigate to downloaded folder and run:  
    docker build -t [your image name]:[versionTag] .
3. Push this image to your docker registry:  
    docker push [your image name]:[versionTag]  
*//Optional Step:4 to test your docker container//*  
4. Docker run container:
    docker run --cap-add AUDIT_WRITE -v [hostPath]:[mountPath] -p [exposedPort]:[containerPort] -d [dockerimage]:[versionTag] [SFTPuser]:[password]:[uid]:[guid]:[directory]  
    optional params: uid,guid

# SFTP service containerized
Steps to be followed:  
1. Ensure to have docker image ready as mentioned in SFTP Centos Image Creation
2. Create namespace:  
    `oc create namespace sftp `  
2a. use the sftp project:  
        `oc project sftp`     
2b. Create a secret to store your docker credentials:  
        `oc create secret docker-registry regcred --docker-server=docker.io --docker-username=<username> --docker-password=<pass> --docker-email=<email> -n sftp`  
*// Update the files according to your custom values if necessary before applying for the following steps //*  


3. Create persistent volume:  
    `oc apply -f persistent_volume.yaml` 
4. Create volume claim:  
    `oc apply -f pvolume_claim.yaml`
5. For privileged access run the command:  
    `oc adm policy add-scc-to-user privileged -z default -n sftp`  


6. Create deployment and service:  
      `oc apply -f deployment.yaml`

--------------------------------------
In case ha-proxy is available:

7. Configure loadbalancer :

    Edit /etc/haproxy/haproxy.cfg to add the entries: 

    ```
    frontend fe_ssh
        bind *:2222 #exposed port
        mode tcp
        default_backend ssh-all
    backend ssh-all
        balance source
        mode tcp
        server master0 10.17.7.206:31336 check #internal IP of nodes:port of loadbalancer 
        server master1 10.17.16.156:31336 check
        server master2 10.17.16.173:31336 check
        server worker0 10.17.22.235:31336 check
        server worker1 10.17.23.98:31336 check
        server worker2 10.17.28.68:31336 check
   ```
   Refresh the haproxy service by:  
   `sudo systemctl restart haproxy.service`

8. Create route:  
    oc expose service [service name]  
    Eg: oc expose service sftp
9. Access sftp service through:  
    sftp -P [exposed port] [user]@[sftp-route]   
    Eg: sftp -P 2222 admin@sftp-rgv-lb-sftp.apps.raghavocp46.cp.fyre.ibm.com
    
-----------------------
In case ha-proxy isn't available and you want to reach the sftp using NodePort:  
 
7. Access sftp service through:  
`sftp -P <NodePort> <user>@<master node IP>`  

    for example: here NodePort is: 30654  
    `oc get svc  
    NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE sftp NodePort 172.30.121.53 <none> 2222:30654/TCP 12h`  

    and master node IP can be any one of the following internal IP's:  
    `oc get no -o wide` 

