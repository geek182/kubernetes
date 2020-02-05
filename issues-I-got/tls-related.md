On the master node I get this message:  

``` Feb 02 17:24:41 k8s-master-01 kube-apiserver[24745]: I0202 17:24:41.729232   24745 log.go:172] http: TLS handshake error from 192.168.5.15:49448: remote error: tls: bad certificate ```

Ok, TLS handshake error plus the source, good to start but which components is behind this message ? first guess KUBELET

So manually I tried connected to the master:

``` curl --key k8s-worker-01.key -k https://192.168.5.10:6443/api/v1/nodes --cert k8s-worker-01.crt --cacert ca.crt ```
```
{
  "kind": "NodeList",
  "apiVersion": "v1",
  "metadata": {
    "selfLink": "/api/v1/nodes",
    "resourceVersion": "40351"
  },
  "items": []
```  
#### above the command is showing no node because I delete the node previously register.
So using the same certificates I'm able to communicate to the API so kubelet is okay.

I stopped the kubelet on this worker just to make sure it wasn't the responsible for the messages and for starting to rule out few options:
```
systemctl stop kubelet
```
But the messages keeping coming on the master.

I stopped the kube-proxy and again was expecting the messages would stop but the messages keep coming.
```
systemctl stop kube-proxy
```
so after sometime on this worker, thinking how was possible the TLS error coming to the master if the components were down.

quick look on the docker give me the answers:
```
k8s-user@k8s-worker-01 sudo docker ps
CONTAINER ID        IMAGE                  COMMAND                CREATED             STATUS              PORTS               NAMES
905717f5bac0        weaveworks/weave-npc   "/usr/bin/launch.sh"   About an hour ago   Up About an hour                        k8s_weave-npc_weave-net-7xdcv_kube-system_3f8fecb4-45d7-11ea-b14a-00155d149712_0
f1026902ba93        k8s.gcr.io/pause:3.1   "/pause"               About an hour ago   Up About an hour                        k8s_POD_weave-net-7xdcv_kube-system_3f8fecb4-45d7-11ea-b14a-00155d149712_0
```
```
docker logs 905717f5bac0 -f
```
```
ERROR: logging before flag.Parse: E0202 17:27:45.110390   35450 reflector.go:205] github.com/weaveworks/weave/prog/weave-npc/main.go:321: Failed to list *v1.Pod: Get https://10.96.0.1:443/api/v1/pods?limit=500&resourceVersion=0: x509: certificate is valid for 172.20.82.219, 192.168.5.10, 127.0.0.1, not 10.96.0.1
```
the container is trying to communicate to the master using the IP "Get https://10.96.0.1:443" but the answer is pretty clear:
 ``` x509: certificate is valid for 172.20.82.219, 192.168.5.10, 127.0.0.1, not 10.96.0.1 ```

Looking into the kube-apiserver configuration I can see the CIDR is set to 10.96.0.10.
```
/usr/local/bin/kube-apiserver \
  --service-cluster-ip-range=10.96.0.0/24 
```
### Problem and Solution:

the main issue here is the server certificate issue is not complete, the solution is re-issue the certificate with the complete list of known ips for kube-api.
