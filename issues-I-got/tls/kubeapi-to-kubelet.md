On the master:
```
Feb 04 18:37:27 k8s-master-01 kube-apiserver[26967]: E0204 18:37:27.234546   26967 status.go:64] apiserver received an error that is not an metav1.Status: &url.Error{Op:"Get", URL:"https://k8s-worker-01:10250/containerLogs/kube-system/weave-net-s7gf5/weave", Err:x509.HostnameError{Certificate:(*x509.Certificate)(0xc0030d8100), Host:"k8s-worker-01"}}
```
Logs from kubelet on the worker:
```
Feb 04 20:54:07 k8s-worker-01 kubelet[107681]: I0204 20:54:07.442834  107681 log.go:172] http: TLS handshake error from 192.168.5.10:59036: remote error: tls: bad certificate
``` 

The container within the worker (kube-system/weave-net-s7gf5):

```
Error from server: Get https://k8s-worker-01:10250/containerLogs/kube-system/weave-net-s7gf5/weave-npc: x509: certificate is valid for worker-1, not k8s-worker-01
```

looking on the certificate for worker:

```
            X509v3 Subject Alternative Name:
                DNS:worker-1, IP Address:192.168.5.15
```

and the calls are pointing out to ```  Get https://k8s-worker-01:10250 ```.


### Problem and Solution
Certificate issued is missing the correct DNS name of the worker, it is need to re-issue the server certificate for kubelet.
