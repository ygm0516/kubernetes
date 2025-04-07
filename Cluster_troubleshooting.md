# * QnA Cluster Trouble Shooting
- qna-cluster-2와 qna-cluster-3 에 이슈가 발생하였습니다.
- 원인을 파악하고 조치해보세요.

# 목차
- [\* QnA Cluster Trouble Shooting](#-qna-cluster-trouble-shooting)
- [목차](#목차)
- [현재 상태](#현재-상태)
- [qna-cluster-2에서 kubelet inactive 상태](#qna-cluster-2에서-kubelet-inactive-상태)
- [kubelet 및 podman 활성화](#kubelet-및-podman-활성화)
- [SchedulingDisabled 상태 해결](#schedulingdisabled-상태-해결)


# 현재 상태
```bash
ubuntu@qna-cluster-1:~$ kubectl get node
NAME            STATUS                        ROLES           AGE   VERSION
qna-cluster-1   Ready                         control-plane   44d   v1.30.4
qna-cluster-2   NotReady,SchedulingDisabled   control-plane   44d   v1.30.4
qna-cluster-3   Ready,SchedulingDisabled      control-plane   44d   v1.30.4
qna-cluster-4   Ready                         <none>          44d   v1.30.4
```

# qna-cluster-2에서 kubelet inactive 상태
```bash
ubuntu@qna-cluster-1:~$ kubectl describe node qna-cluster-2
Conditions:
  Type                 Status    LastHeartbeatTime                 LastTransitionTime                Reason              Message
  ----                 ------    -----------------                 ------------------                ------              -------
  NetworkUnavailable   False     Tue, 01 Apr 2025 07:03:58 +0000   Tue, 01 Apr 2025 07:03:58 +0000   CalicoIsUp          Calico is running on this node
  MemoryPressure       Unknown   Fri, 04 Apr 2025 00:37:19 +0000   Fri, 04 Apr 2025 00:38:08 +0000   NodeStatusUnknown   Kubelet stopped posting node status.
  DiskPressure         Unknown   Fri, 04 Apr 2025 00:37:19 +0000   Fri, 04 Apr 2025 00:38:08 +0000   NodeStatusUnknown   Kubelet stopped posting node status.
  PIDPressure          Unknown   Fri, 04 Apr 2025 00:37:19 +0000   Fri, 04 Apr 2025 00:38:08 +0000   NodeStatusUnknown   Kubelet stopped posting node status.
  Ready                Unknown   Fri, 04 Apr 2025 00:37:19 +0000   Fri, 04 Apr 2025 00:38:08 +0000   NodeStatusUnknown   Kubelet stopped posting node status.
Addresses:

---

ubuntu@qna-cluster-2:~$ sudo systemctl status kubelet
○ kubelet.service - Kubernetes Kubelet Server
     Loaded: loaded (/etc/systemd/system/kubelet.service; disabled; vendor preset: enabled)
     Active: inactive (dead) since Fri 2025-04-04 00:37:20 UTC; 6min ago
       Docs: https://github.com/GoogleCloudPlatform/kubernetes
   Main PID: 1221 (code=exited, status=0/SUCCESS)
        CPU: 2h 21min 56.422s

Apr 04 00:37:20 qna-cluster-2 kubelet[1221]:         Version            1.14.0
Apr 04 00:37:20 qna-cluster-2 kubelet[1221]:         Build Date         2023-06-19T11:40:23Z
Apr 04 00:37:20 qna-cluster-2 kubelet[1221]:         Storage Type       file
Apr 04 00:37:20 qna-cluster-2 kubelet[1221]:         HA Enabled         false
Apr 04 00:37:20 qna-cluster-2 kubelet[1221]:  >
Apr 04 00:37:20 qna-cluster-2 kubelet[1221]: I0404 00:37:20.745424    1221 dynamic_cafile_content.go:171] "Shutting down controller" name="client-ca-bundle::/etc/kubernetes/ssl/ca.crt"
Apr 04 00:37:20 qna-cluster-2 systemd[1]: Stopping Kubernetes Kubelet Server...
Apr 04 00:37:20 qna-cluster-2 systemd[1]: kubelet.service: Deactivated successfully.
Apr 04 00:37:20 qna-cluster-2 systemd[1]: Stopped Kubernetes Kubelet Server.
Apr 04 00:37:20 qna-cluster-2 systemd[1]: kubelet.service: Consumed 2h 21min 56.422s CPU time.

```


# kubelet 및 podman 활성화
```bash
ubuntu@qna-cluster-2:~$ sudo systemctl enable --now kubelet
ubuntu@qna-cluster-2:~$ sudo systemctl enable --now podman

ubuntu@qna-cluster-1:~$ kubectl get node
NAME            STATUS                     ROLES           AGE   VERSION
qna-cluster-1   Ready                      control-plane   44d   v1.30.4
qna-cluster-2   Ready,SchedulingDisabled   control-plane   44d   v1.30.4
qna-cluster-3   Ready,SchedulingDisabled   control-plane   44d   v1.30.4
qna-cluster-4   Ready                      <none>          44d   v1.30.4

```

# SchedulingDisabled 상태 해결
```bash
ubuntu@qna-cluster-1:~$ kubectl uncordon qna-cluster-2
ubuntu@qna-cluster-1:~$ kubectl uncordon qna-cluster-3

ubuntu@qna-cluster-1:~$ kubectl get node
NAME            STATUS   ROLES           AGE   VERSION
qna-cluster-1   Ready    control-plane   44d   v1.30.4
qna-cluster-2   Ready    control-plane   44d   v1.30.4
qna-cluster-3   Ready    control-plane   44d   v1.30.4
qna-cluster-4   Ready    <none>          44d   v1.30.4
```

