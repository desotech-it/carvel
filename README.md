# carvel
Carvel for VMUG 2021


In this example you can understand how to use a full integration of CARVEL Tools.

We should install Carvel Tools.

```
sudo -i
```

```
curl -L https://carvel.dev/install.sh | bash
```

```
exit
```

## kapp
### Deploy an application with `kubectl`

Using a Deployment kind of Kubernetes we can deploy a GOlang application.

`whoami` permit to obtain information about container and also can perform some task for stress cpu, memory and so on.

Find Detail around this project on GitHub Project [https://github.com/desotech-it/whoami](https://github.com/desotech-it/whoami)

When passing an environment variable to the container `whoami` we can obtain different landing page.

Usually a Kubernetes Developer deploy an application using `kubectl` command:

```
kubectl apply -n vmug2021 -f 01/whoami-app.yaml
```

This command create one deployment and one service.

> ```
> deployment.apps/vmug2021 configured
> service/vmug2021 created
> ```

Also create some other object.

```
kubectl get endpoints vmug2021
```

> ```
> NAME       ENDPOINTS      AGE
> vmug2021   10.1.0.90:80   2m21s
> ```

We did't create endpoint directly. Usually when we works with custom CRD we need a software that permit us to control what should be installed.

Now delete this application.

```
kubectl delete -f 01/whoami-app.yaml
```

> ```
> deployment.apps "vmug2021" deleted
> service "vmug2021" deleted
> ```

### Deploy Application with `kapp`

We try to use a different command `kapp` for deploy the same application:
```
kapp deploy -a my-application -f 01/whoami-app.yaml
```

> ```
> Target cluster 'https://kubernetes.docker.internal:6443' (nodes: docker-desktop)
>
> Changes
>
> Namespace  Name      Kind        Conds.  Age  Op      Op st.  Wait to    Rs  Ri
> default    vmug2021  Deployment  -       -    create  -       reconcile  -   -
> ^          vmug2021  Service     -       -    create  -       reconcile  -   -
>
> Op:      2 create, 0 delete, 0 update, 0 noop
> Wait to: 2 reconcile, 0 delete, 0 noop
>
> Continue? [yN]:
> ```

When you run application you will see result of what will happen inside your kubernetes cluster.

You can accept with `y`:

```
10:05:45AM: ---- applying 2 changes [0/2 done] ----
10:05:45AM: create deployment/vmug2021 (apps/v1) namespace: default
10:05:45AM: create service/vmug2021 (v1) namespace: default
10:05:45AM: ---- waiting on 2 changes [0/2 done] ----
10:05:45AM: ok: reconcile service/vmug2021 (v1) namespace: default
10:05:45AM: ongoing: reconcile deployment/vmug2021 (apps/v1) namespace: default
10:05:45AM:  ^ Waiting for generation 2 to be observed
10:05:45AM:  L ok: waiting on replicaset/vmug2021-d7cc4dd46 (apps/v1) namespace: default
10:05:45AM:  L ongoing: waiting on pod/vmug2021-d7cc4dd46-rmg4n (v1) namespace: default
10:05:45AM:     ^ Pending: ContainerCreating
10:05:45AM: ---- waiting on 1 changes [1/2 done] ----
10:05:46AM: ongoing: reconcile deployment/vmug2021 (apps/v1) namespace: default
10:05:46AM:  ^ Waiting for 1 unavailable replicas
10:05:46AM:  L ok: waiting on replicaset/vmug2021-d7cc4dd46 (apps/v1) namespace: default
10:05:46AM:  L ongoing: waiting on pod/vmug2021-d7cc4dd46-rmg4n (v1) namespace: default
10:05:46AM:     ^ Pending: ContainerCreating
10:05:48AM: ok: reconcile deployment/vmug2021 (apps/v1) namespace: default
10:05:48AM: ---- applying complete [2/2 done] ----
10:05:48AM: ---- waiting complete [2/2 done] ----

Succeeded
```

You will obtain a `reconcile` information about all objects created. Using kapp you observe what happen and understand if the operation will be completed correctly.

```
kapp inspect -a my-application --tree
```

```
Target cluster 'https://kubernetes.docker.internal:6443' (nodes: docker-desktop)

Resources in app 'my-application'

Namespace  Name                           Kind           Owner    Conds.  Rs  Ri  Age
default    vmug2021                       Service        kapp     -       ok  -   2m
default     L vmug2021                    Endpoints      cluster  -       ok  -   2m
default     L vmug2021-x4fwh              EndpointSlice  cluster  -       ok  -   2m
default    vmug2021                       Deployment     kapp     2/2 t   ok  -   2m
default     L vmug2021-d7cc4dd46          ReplicaSet     cluster  -       ok  -   2m
default     L.. vmug2021-d7cc4dd46-rmg4n  Pod            cluster  4/4 t   ok  -   2m

Rs: Reconcile state
Ri: Reconcile information

6 resources

Succeeded
```

As you can see Cluster created some extra object `Endpoints`, `EndpointSlice`, `ReplicaSet` and `Pod`.
`kapp` track all object created.


```
kapp list
```

```
Target cluster 'https://kubernetes.docker.internal:6443' (nodes: docker-desktop)

Apps in namespace 'default'

Name            Namespaces  Lcs   Lca
my-application  default     true  4m

Lcs: Last Change Successful
Lca: Last Change Age

1 apps

Succeeded
```

You can read logs of the container with a simple command:

```
kapp logs -f -a my-application
```


> ```
> Target cluster 'https://kubernetes.docker.internal:6443' (nodes: docker-desktop)
>
> # starting tailing 'vmug2021-d7cc4dd46-rmg4n > whoami' logs
> vmug2021-d7cc4dd46-rmg4n > whoami | {"cpu_load":[3.0927835051568593,2.538071065990097,1.0256410256431034,2.010050251256611,2.0202020202025714,2.0408163265316186],"level":"info","msg":"Logging CPU load","time":"2021-11-27T10:34:12Z"}
> ```

Stop the process with `CTRL-C`


> ```
> ^C
> ```

## Working with a web application

When a developer wants to interact from his computer with the application, he use the `kubectl port-forward` command that permit to forward a kubernetes internal service port to the local computer port.

```bash
kubectl port-forward svc/vmug2021 8080:80
```

> ```
> Forwarding from 127.0.0.1:8080 -> 80
> Forwarding from [::1]:8080 -> 80
> ```

Open another terminal or browser and `curl` on `localhost` on the port `8080` to:

```
curl localhost:8080
```

```
@@@@@@@@@@@LtC@@@@@0tf8@@@@@@@@@@@@@@@@@@@@@@
@@@@@@@88@0ii;G@8@0i;;C@@@@@@@@@@@@@@@@@@@@@@
@@@@@8LLfLCtfLLLLfffftCGCLL0@@@@@@@@@@@@@@@@@
@@@@@8LLLLLfLLLLLtfLLLLLfLfC@@@@@@@@@@@@@@@@@
@@@@@@8LLCi18LLLf,C0LLLLfLL8@@@@@@@@@@@@@@@@@
@@@@@@@GLLi,tLLLf:;CLLLLLG8@@@@@@@@@@@@@@@@@@
@@@@@@0LLLCLLLLLLCLLLLLLG@@@@@@@@@@@@@@@@@@@@
@@@@@@CLLLGGGGCCCLLLLLLLC@@@@@@@@@@@@@@@@@@@@
@@@@@0fCCCGGGGCCGCLLLLLLC@@@@@@@@@@@@@@@@@@@@
@@@@@@GCGGGGGCLLGGLLLLLLC@@@@@@@@@@@@@@@@@@@@
@@@@@@@0CGCffftfGLLLLLfG@@@@@@@@@@@@@@@@@@@@@
@@@@@@@@80GLfLLCLLLLLLL8@@@@@@@@@@@@@@@@@@@@@
@@@@@@@@@@@LLLLLLLLLLCL8@@@@@@@@@@@@@@@@@@@@@
@@@@@@@@@@8CLLLLLLLLft10@@@@@@@@@@@@@@@@@@@@@
@@@@@@@@@@8f11fLLLLfii;C@@@@@@@@@@@@@@@@@@@@@
@@@@@@@@@@0iii1LLLL1i1it@@@@@@@@@@@@@@@@@@@@@
@@@@@@@@@@0iiiLLLL1iiiit@@@@@@@@@@@@@@@@@@@@@
@@@@@@@@@@C;iiLCLLLt1tfLG@@@@@@@@@@@@@@@@@@@@
@@@@@@@@@@LttfLttfLLCLLLL@@@@@@@@@@@@@@@@@@@@
@@@@@@@@@8LLLL1iiii1fLLLLG@@@@@@@@@@@@@@@@@@@
@@@@@@@@@GLLLLiiiiii1LLLLLGGG008@@@@@@@@@@@@@
@@@@@@@@@CLLLLt111ffLLLLLLLLL1i1fLG@@@@@@@@@@
@@@@@@@@@LLLffCLLLCLtffLLLLLLt11i;i1C@@@@@@@@
@@@@@@@@8LLfi1tfLLLti11fLLLLLCLLf1ii1C0@@@@@@
@@@@@@@@8LLfiiiitLLLLLLLLft1tfLLLLLLLffC@@@@@
@@@@@@@@8LLLLfffLLLf1i1LfiiiitLLLLf1i1tfC8@@@
@@@@@@@@@CLLffLCLL1iiifLtii1LLLLiiiiiiitiL@@@
@@@@@@@@@GLfii1fLL1ii1LLLLfLLLLL1ii1tt1fG@@@@
@@@@@@@@@8fLLtiifLLffLLLLLLLLfftfLfLLLLC@@@@@
@@@@@@@@@@LLLLffLLLLLLLLLLLLLf1itffL1ifG@@@@@
@@@@@@@@@@GLLLLLLLffLLLLLLLLffffLLLLLLf8@@@@@
@@@@@@@@@@@LLLLLLLLLLLLLLLLLfLffLLLLLLL@@@@@@
@@@@@@@@@@@0LLffffC8LfLLLLfC8080LLLLLC8@@@@@@
@@@@@@@@@@@@@88088@@80GCCG0@@@@@@88@@@@@@@@@@
```

Now kill the pod:

```
kubectl get pod
```

```
NAME                       READY   STATUS    RESTARTS   AGE
vmug2021-d7cc4dd46-rmg4n   1/1     Running   0          95m
```

```
kubectl delete pod vmug2021-d7cc4dd46-rmg4n
pod "vmug2021-d7cc4dd46-rmg4n" deleted
```

And run `curl` again:

```
curl localhost:8080
```

```
curl: (52) Empty reply from server
```

You need to restart again the `kubectl port-forward` command.

Kill the `kubectl port-forward` cli with `CTRL-C`

> ```
> ^C%
> ```

In this case you can use another tools of the suite Carvel.

# kwt

[`kwt`](https://github.com/vmware-tanzu/carvel-kwt) come to help developer to provide a set of command for develop application with Kubernetes.
This tools permit to interact with Network to make Kubernetes services and pods accessible to your local machine.

We have a `ClusterIP` service `vmug2021`:

```
kubectl get svc vmug2021
```

```
NAME       TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
vmug2021   ClusterIP   10.102.71.235   <none>        80/TCP    105m
```

You don't have any record in your dns resolution:

```
dig vmug2021.default.svc.cluster.local +short
```


Try to run `kwt` with a simple command:

```bash
sudo -E kwt net start
```

> ```
> 11:46:53AM: info: KubeEntryPoint: Creating networking client secret 'kwt-net-ssh-key' in namespace 'default'...
> 11:46:53AM: info: KubeEntryPoint: Creating networking host secret 'kwt-net-host-key' in namespace 'default'...
> 11:46:53AM: info: KubeEntryPoint: Creating networking pod 'kwt-net' in namespace 'default'
> 11:46:53AM: info: KubeEntryPoint: Waiting for networking pod 'kwt-net' in namespace 'default' to start...
> 11:46:56AM: info: dns.FailoverRecursorPool: Starting with '8.8.8.8:53'
> 11:46:56AM: info: dns.DomainsMux: Registering cluster.local.->kube-dns
> ```

```
dig vmug2021.default.svc.cluster.local +short
```

> ```
> 10.102.71.235
> ```

Now you have correct resolution, running `curl` you will obtain output from pod:

```
curl http://vmug2021.default.svc.cluster.local
```

You can see the output of your container:

> ```
> @@@@@@@@@@@LtC@@@@@0tf8@@@@@@@@@@@@@@@@@@@@@@
> @@@@@@@88@0ii;G@8@0i;;C@@@@@@@@@@@@@@@@@@@@@@
> @@@@@8LLfLCtfLLLLfffftCGCLL0@@@@@@@@@@@@@@@@@
> @@@@@8LLLLLfLLLLLtfLLLLLfLfC@@@@@@@@@@@@@@@@@
> @@@@@@8LLCi18LLLf,C0LLLLfLL8@@@@@@@@@@@@@@@@@
> @@@@@@@GLLi,tLLLf:;CLLLLLG8@@@@@@@@@@@@@@@@@@
> @@@@@@0LLLCLLLLLLCLLLLLLG@@@@@@@@@@@@@@@@@@@@
> @@@@@@CLLLGGGGCCCLLLLLLLC@@@@@@@@@@@@@@@@@@@@
> @@@@@0fCCCGGGGCCGCLLLLLLC@@@@@@@@@@@@@@@@@@@@
> @@@@@@GCGGGGGCLLGGLLLLLLC@@@@@@@@@@@@@@@@@@@@
> @@@@@@@0CGCffftfGLLLLLfG@@@@@@@@@@@@@@@@@@@@@
> @@@@@@@@80GLfLLCLLLLLLL8@@@@@@@@@@@@@@@@@@@@@
> @@@@@@@@@@@LLLLLLLLLLCL8@@@@@@@@@@@@@@@@@@@@@
> @@@@@@@@@@8CLLLLLLLLft10@@@@@@@@@@@@@@@@@@@@@
> @@@@@@@@@@8f11fLLLLfii;C@@@@@@@@@@@@@@@@@@@@@
> @@@@@@@@@@0iii1LLLL1i1it@@@@@@@@@@@@@@@@@@@@@
> @@@@@@@@@@0iiiLLLL1iiiit@@@@@@@@@@@@@@@@@@@@@
> @@@@@@@@@@C;iiLCLLLt1tfLG@@@@@@@@@@@@@@@@@@@@
> @@@@@@@@@@LttfLttfLLCLLLL@@@@@@@@@@@@@@@@@@@@
> @@@@@@@@@8LLLL1iiii1fLLLLG@@@@@@@@@@@@@@@@@@@
> @@@@@@@@@GLLLLiiiiii1LLLLLGGG008@@@@@@@@@@@@@
> @@@@@@@@@CLLLLt111ffLLLLLLLLL1i1fLG@@@@@@@@@@
> @@@@@@@@@LLLffCLLLCLtffLLLLLLt11i;i1C@@@@@@@@
> @@@@@@@@8LLfi1tfLLLti11fLLLLLCLLf1ii1C0@@@@@@
> @@@@@@@@8LLfiiiitLLLLLLLLft1tfLLLLLLLffC@@@@@
> @@@@@@@@8LLLLfffLLLf1i1LfiiiitLLLLf1i1tfC8@@@
> @@@@@@@@@CLLffLCLL1iiifLtii1LLLLiiiiiiitiL@@@
> @@@@@@@@@GLfii1fLL1ii1LLLLfLLLLL1ii1tt1fG@@@@
> @@@@@@@@@8fLLtiifLLffLLLLLLLLfftfLfLLLLC@@@@@
> @@@@@@@@@@LLLLffLLLLLLLLLLLLLf1itffL1ifG@@@@@
> @@@@@@@@@@GLLLLLLLffLLLLLLLLffffLLLLLLf8@@@@@
> @@@@@@@@@@@LLLLLLLLLLLLLLLLLfLffLLLLLLL@@@@@@
> @@@@@@@@@@@0LLffffC8LfLLLLfC8080LLLLLC8@@@@@@
> @@@@@@@@@@@@@88088@@80GCCG0@@@@@@88@@@@@@@@@@
> ```

Now kill the pod again:

```
kubectl get pod
```

> ```
> NAME                       READY   STATUS    RESTARTS   AGE
> kwt-net                    1/1     Running   0          14m
> vmug2021-d7cc4dd46-dws92   1/1     Running   0          20m
> ```

Take note of the name of the pod and delete it.

```
kubectl delete pod vmug2021-d7cc4dd46-dws92
```

> ```
> pod "vmug2021-d7cc4dd46-dws92" deleted
> ```