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

Sync Repo for this workshop:

```
cd ~

git clone https://github.com/desotech-it/carvel.git
```

```
cd carvel
```

Usually a Kubernetes Developer deploy an application using `kubectl` command:

```
kubectl apply -n vmug2021 -f 01/whoami-app.yaml
```

This command could produce an error, because in the yaml file the creation of `namespace` is located on the bottom of the file.

> ```
> namespace/vmug2021 created
> Error from server (NotFound): error when creating "01/whoami-app.yaml": namespaces "vmug2021" not found
> Error from server (NotFound): error when creating "01/whoami-app.yaml": namespaces "vmug2021" not found
> ```

Running again the same command:

```
kubectl apply -n vmug2021 -f 01/whoami-app.yaml
```

Now also created the deployment and service:

> ```
> deployment.apps/vmug2021 created
> service/vmug2021 created
> namespace/vmug2021 unchanged
> ```

Also create some other object.

```
kubectl get endpoints -n vmug2021 vmug2021
```

> ```
> NAME       ENDPOINTS      AGE
> vmug2021   10.1.0.90:80   2m21s
> ```

We did't create endpoint directly. Usually, when we works with custom CRD, we need a software that permit us to control what should be installed.

Now delete this application.

```
kubectl delete -f 01/whoami-app.yaml
```

> ```
> deployment.apps "vmug2021" deleted
> service "vmug2021" deleted
> namespace "vmug2021" deleted
> ```

### Deploy Application with `kapp`

We try to use a different command `kapp` for deploy the same application:

```
kapp deploy -a vmug-application -f 01/whoami-app.yaml
```

> ```
> Target cluster 'https://10.10.180.21:6443' (nodes: master01, 4+)
>
> Changes
>
> Namespace  Name      Kind        Conds.  Age  Op      Op st.  Wait to    Rs  Ri
> (cluster)  vmug2021  Namespace   -       -    create  -       reconcile  -   -
> vmug2021   vmug2021  Deployment  -       -    create  -       reconcile  -   -
> ^          vmug2021  Service     -       -    create  -       reconcile  -   -
>
> Op:      3 create, 0 delete, 0 update, 0 noop
> Wait to: 3 reconcile, 0 delete, 0 noop
>
> Continue? [yN]:
> ```

When you run application you will see result of what will happen inside your kubernetes cluster. You will not receive any error and all sorting and dependencies have been calculated by `kapp`

You can accept with `y`:

```
12:33:45PM: ---- applying 1 changes [0/3 done] ----
12:33:45PM: create namespace/vmug2021 (v1) cluster
12:33:45PM: ---- waiting on 1 changes [0/3 done] ----
12:33:45PM: ok: reconcile namespace/vmug2021 (v1) cluster
12:33:45PM: ---- applying 2 changes [1/3 done] ----
12:33:45PM: create deployment/vmug2021 (apps/v1) namespace: vmug2021
12:33:45PM: create service/vmug2021 (v1) namespace: vmug2021
12:33:45PM: ---- waiting on 2 changes [1/3 done] ----
12:33:45PM: ok: reconcile service/vmug2021 (v1) namespace: vmug2021
12:33:45PM: ongoing: reconcile deployment/vmug2021 (apps/v1) namespace: vmug2021
12:33:45PM:  ^ Waiting for generation 2 to be observed
12:33:45PM:  L ok: waiting on replicaset/vmug2021-845db75bb (apps/v1) namespace: vmug2021
12:33:45PM:  L ongoing: waiting on pod/vmug2021-845db75bb-cqzmj (v1) namespace: vmug2021
12:33:45PM:     ^ Pending
12:33:45PM: ---- waiting on 1 changes [2/3 done] ----
12:33:45PM: ongoing: reconcile deployment/vmug2021 (apps/v1) namespace: vmug2021
12:33:45PM:  ^ Waiting for 1 unavailable replicas
12:33:45PM:  L ok: waiting on replicaset/vmug2021-845db75bb (apps/v1) namespace: vmug2021
12:33:45PM:  L ongoing: waiting on pod/vmug2021-845db75bb-cqzmj (v1) namespace: vmug2021
12:33:45PM:     ^ Pending: ContainerCreating
12:33:51PM: ok: reconcile deployment/vmug2021 (apps/v1) namespace: vmug2021
12:33:51PM: ---- applying complete [3/3 done] ----
12:33:51PM: ---- waiting complete [3/3 done] ----

Succeeded
```

You will obtain a `reconcile` information about all objects created. Using kapp you observe what happen and understand if the operation will be completed correctly.

```
kapp inspect -a vmug-application --tree
```

> ```
> Target cluster 'https://10.10.180.21:6443' (nodes: master01, 4+)
>
> Resources in app 'vmug-application'
>
> Namespace  Name                           Kind           Owner    Conds.  Rs  Ri  Age
> vmug2021   vmug2021                       Deployment     kapp     2/2 t   ok  -   25s
> vmug2021    L vmug2021-845db75bb          ReplicaSet     cluster  -       ok  -   25s
> vmug2021    L.. vmug2021-845db75bb-cqzmj  Pod            cluster  4/4 t   ok  -   25s
> (cluster)  vmug2021                       Namespace      kapp     -       ok  -   25s
> vmug2021   vmug2021                       Service        kapp     -       ok  -   25s
> vmug2021    L vmug2021                    Endpoints      cluster  -       ok  -   25s
> vmug2021    L vmug2021-pfzq7              EndpointSlice  cluster  -       ok  -   25s
>
> Rs: Reconcile state
> Ri: Reconcile information
>
> 7 resources
>
> Succeeded
> ```

As you can see Cluster created some extra kind: `Endpoints`, `EndpointSlice`, `ReplicaSet` and `Pod`. `kapp` track all object created.


```
kapp ls
```

> ```
> Target cluster 'https://10.10.180.21:6443' (nodes: master01, 4+)
>
> Apps in namespace 'default'
>
> Name              Namespaces          Lcs   Lca
> vmug-application  (cluster),vmug2021  true  1m
>
> Lcs: Last Change Successful
> Lca: Last Change Age
>
> 1 apps
>
> Succeeded
> ```

You can read logs of the container with a simple command:

```
kapp logs -f -a vmug-application
```


> ```
> Target cluster 'https://10.10.180.21:6443' (nodes: master01, 4+)
>
> # starting tailing 'vmug2021-845db75bb-cqzmj > whoami' logs
> vmug2021-845db75bb-cqzmj > whoami | {"level":"info","msg":"Logging memory usage","percentage_used":20.05267775268053,"time":"2021-11-27T11:35:30Z","total_memory":4127277056,"used_memory":827629568}
> vmug2021-845db75bb-cqzmj > whoami | {"cpu_load":[3.5897435906161306,6.030150752857535],"level":"info","msg":"Logging CPU load","time":"2021-11-27T11:35:30Z"}
> ```

Stop the process with `CTRL-C`


> ```
> ^C
> ```

## Working with a web application

When a developer wants to interact from his computer with the application, he use the `kubectl port-forward` command that permit to forward a kubernetes internal service port to the local computer port.

```bash
kubectl port-forward -n vmug2021 svc/vmug2021 8080:80
```

> ```
> Forwarding from 127.0.0.1:8080 -> 80
> ```

Open another terminal or browser and `curl` on `localhost` on the port `8080` to:

```
curl localhost:8080
```

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

Now kill the pod:

```
kubectl delete pod -n vmug2021 -l app=vmug2021 --grace-period 0
```

> ```
> pod "vmug2021-845db75bb-cqzmj" deleted
> ```

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

## kwt

[`kwt`](https://github.com/vmware-tanzu/carvel-kwt) come to help developer to provide a set of command for develop application with Kubernetes.
This tools permit to interact with Network to make Kubernetes services and pods accessible to your local machine.

We have a `ClusterIP` service `vmug2021`:

```
kubectl get svc vmug2021 -n vmug2021
```

> ```
> NAME       TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
> vmug2021   ClusterIP   10.98.216.211   <none>        80/TCP    7m52s
> ```

You don't have any record in your dns resolution:

```
dig vmug2021.vmug2021.svc.cluster.local +short
```

You will not receive any output because this record dns do not exist in our environment.


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
dig vmug2021.vmug2021.svc.cluster.local +short
```

You will obtain the same IP of ClusterIP `vmug2021`

> ```
> 10.98.216.211
> ```

Now you have correct resolution, running `curl` you will obtain output from pod:

```
curl http://vmug2021.vmug2021.svc.cluster.local
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

Kill the pod again:

```
kubectl delete pod -n vmug2021 -l app=vmug2021 --grace-period 0
```

> ```
> pod "vmug2021-845db75bb-44r7q" deleted
> ```

Take note of the name of the pod and delete it.

```
kubectl delete pod vmug2021-d7cc4dd46-dws92
```

> ```
> pod "vmug2021-d7cc4dd46-dws92" deleted
> ```

Doing `curl` again you will obtain result without restart the command `kwt`.

```
curl http://vmug2021.vmug2021.svc.cluster.local
```

You can kill the process `kwt`

```
sudo killall kwt
```

## kapp updates

You can also update application using kapp.

```
cd ~/carvel/
```

diff ~/carvel/01/whoami-app.yaml ~/carvel/02/whoami-app.yaml
