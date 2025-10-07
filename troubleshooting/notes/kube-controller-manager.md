# Error in kube-controller-manager

### Symptoms of kube-controller-manager instability include resources failing to start properly—for example, when you create a Deployment and neither the ReplicaSets nor the Pods are created. Additionally, the Deployment shows no events, which suggests that the controller never attempted to reconcile the resource.

Steps to troubleshoot(We assume that the API server is also down):

```bash
1. crictl ps -a or docker ps -a
Ensure that the kube-controller-manager container is there, otherwise, we need to check:
/var/log/syslog | grep kube-controller-manager

2. crictl ps -a or docker ps -a
If the container appears now, based on that, we can check:
/var/log/containers/<component>/kube-controller-manager/<number>.log

3. If we don´t find anything useful in the container log, we move to the pod log in:
/var/log/pods/<component>/kube-controller-manager/<number>.log

4. Sometimes we encounter errors such as incorrect indentation in:
 /var/kubernetes/manifests, invalid flags like --improve-controller-manager, or container runtime failures

5. The identations and flags error can be solved in the manifest:
/var/kubernetes/manifests/kube-controller-manager.yaml.
For the container runtime errors, you must check kubelet or containerd with:
journalctl -u kubelet -f
journalctl -u containerd -f

6- Also check the status of the services:
sudo systemctl status kubelet
sudo systemctl status containerd

7. If you get flag errors, you can use:
crictl ps | grep kube-controller-manager
crictl exec -it <container-id> kube-controller-manager --help | grep <flag>
With this you can check if that flag exists.
```

If the apiserver is up, then we have a much easier  approach:

```bash
1. kubectl get pods -n kube-system

2. if the kube-controller-manager appears, that implies we can skip checking the syslog and the container
 logs and the only thing we have to check are the pod logs

3. kubectl describe pods kube-controller-manager -n kube-system
We describe the pod to see if there is useful information

4. If not we check the logs in /var/log/pods/<component>/kube-controller-manager/<number>.log

5. We can encounter errors such as indentation, incorrect flags or container runtime failures

6. We check kubelet and the container runtime(in this case crio)
journalctl -u kubelet -f
journalctl -u crio -f

7. If you get flag errors, you can use:
crictl ps | grep kube-controller-manager
crictl exec -it <container-id> kube-controller-manager --help | grep <flag>
With this you can check if that flag exists.
```
