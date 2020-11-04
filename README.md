# Mount a PV in a running pod
Apply the `running-pod.yaml`
```bash
$ kubectl apply -f running-pod.yaml
pod/running-pod created
$ kubectl get po
NAME          READY   STATUS    RESTARTS   AGE
running-pod   1/1     Running   0          37s
```
After you can create a pvc that remains pending until you create the mount pod.
Launch the mount job to perform the bind mount:
```bash
$ kubectl apply -f pvc.yaml 
persistentvolumeclaim/mount-pv-claim created
$ kubectl get pvc
NAME             STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
mount-pv-claim   Pending 
$ kubectl get po
NAME          READY   STATUS      RESTARTS   AGE
mount-prjz9   0/1     Completed   0          27s
running-pod   1/1     Running     0          2m6s
$ kubectl get job
NAME    COMPLETIONS   DURATION   AGE
mount   1/1           14s        31s
$ kubectl get pvc
NAME             STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
mount-pv-claim   Bound    pvc-7e2291f4-5aaf-4e87-9397-e8a2a4aea4a0   300Mi      RWO            standard       76s
```
You can now delete the job mount and the pv remains:
```bash
kubectl delete -f job-pvc.yaml 
job.batch "mount" deleted
$ kubectl get po
NAME          READY   STATUS    RESTARTS   AGE
running-pod   1/1     Running   0          3m10s
$ kubectl get pvc
NAME             STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
mount-pv-claim   Bound    pvc-7e2291f4-5aaf-4e87-9397-e8a2a4aea4a0   300Mi      RWO            standard       2m13s
```

You can create an additional container to create a test file and verify that the mount corrctly works:
```bash
$ $ kubectl apply -f pod.yaml 
pod/test1 created
$ kubectl get po
NAME          READY   STATUS    RESTARTS   AGE
running-pod   1/1     Running   0          5m18s
test1         1/1     Running   0          61s
$ kubectl exec -ti test1 -- touch /test/test-file
$ kubectl delete pod test1
pod "test1" deleted
$ kubectl get pod
NAME          READY   STATUS    RESTARTS   AGE
running-pod   1/1     Running   0          7m5s
$ kubectl get pvc
NAME             STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
mount-pv-claim   Bound    pvc-7e2291f4-5aaf-4e87-9397-e8a2a4aea4a0   300Mi      RWO            standard       6m21s
$ kubectl exec -ti running-pod -- ls /test/test-volume
test-file
```
