# kube-apiserver error

### The symptoms of the kube-apiserver instability are very obvious, you cannot use kubectl

```bash
I suggest you to fix the component first before fixing any other components because it will be much faster
 to troubleshoot the other components with the api server

# The kube-apiserver default port is 6443

1. crictl ps -a or docker ps -a
Check if the container is up, otherwise we need to check:
/var/log/syslog | grep kube-api-server

you can also check the kubelet for more information:
journalctl -u kubelet -f

2. When the container gets created but it doesn´t run, then we need to check the container logs in:
/var/log/containers/<component>/kube-apiserver/<number>.log

3. If you fix the container errors, but still doesn't start, you can check the pods logs in:
/var/log/pods/<component>/kube-apiserver/<number>.log

4. Some common errors are identation, container runtime failures or incorrect flags

5. For container runtime failures you can check:
journalctl -u kubelet -f
journalctl -u containerd -f
Note: don't forget to restart the services:
For kubelet:
sudo systemctl daemon-reload
sudo systemctl restart kubelet
For the container runtime:
sudo systemctl restart <container-runtime>

6. For the flags error, you can use this:
crictl ps | grep kube-apiserver
crictl exec -it <contianer-id> kube-apiserver --help | grep <flag>
```
