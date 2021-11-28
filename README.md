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

Understand the yaml file:

```
cat 01/whoami-app.yaml
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


## kapp updates

You can also update application using kapp.

```
cd ~/carvel/
diff 01/whoami-app.yaml 02/whoami-app.yaml
```

```
24c24
<           value: phippy
---
>           value: captainkube
```

In this file we only changed the value of Environment Variable `NAME_APPLICATION`


```
cat 02/whoami-app.yaml | grep NAME -a1
```

```yaml
        env:
        - name: NAME_APPLICATION
          value: captainkube
```

Check the changes with `kapp`

```
kapp deploy -a vmug-application -f 02/whoami-app.yaml --diff-changes
```

> ```
> 129,121           - name: NAME_APPLICATION
> 130     -           value: phippy
>     122 +           value: captainkube
> . . .
> Changes
>
> Namespace  Name      Kind        Conds.  Age  Op      Op st.  Wait to    Rs  Ri
> vmug2021   vmug2021  Deployment  -       47m  update  -       reconcile  ok  -
>
> Op:      0 create, 0 delete, 1 update, 0 noop
> Wait to: 1 reconcile, 0 delete, 0 noop
>
> Continue? [yN]:
> ```

Write and confirm with `y` to apply the updated application:

> ```
> 1:24:27PM: ---- applying 1 changes [0/1 done] ----
> 1:24:27PM: update deployment/vmug2021 (apps/v1) namespace: vmug2021
> 1:24:27PM: ---- waiting on 1 changes [0/1 done] ----
> 1:24:27PM: ongoing: reconcile deployment/vmug2021 (apps/v1) namespace: vmug2021
> 1:24:27PM:  ^ Waiting for generation 4 to be observed
> 1:24:27PM:  L ok: waiting on replicaset/vmug2021-845db75bb (apps/v1) namespace: vmug2021
> 1:24:27PM:  L ok: waiting on replicaset/vmug2021-7f5d8dd7dc (apps/v1) namespace: vmug2021
> 1:24:27PM:  L ok: waiting on pod/vmug2021-845db75bb-2gzx8 (v1) namespace: vmug2021
> 1:24:27PM:  L ongoing: waiting on pod/vmug2021-7f5d8dd7dc-czf7j (v1) namespace: vmug2021
> 1:24:27PM:     ^ Pending
> 1:24:28PM: ongoing: reconcile deployment/vmug2021 (apps/v1) namespace: vmug2021
> 1:24:28PM:  ^ Waiting for 1 unavailable replicas
> 1:24:28PM:  L ok: waiting on replicaset/vmug2021-845db75bb (apps/v1) namespace: vmug2021
> 1:24:28PM:  L ok: waiting on replicaset/vmug2021-7f5d8dd7dc (apps/v1) namespace: vmug2021
> 1:24:28PM:  L ok: waiting on pod/vmug2021-845db75bb-2gzx8 (v1) namespace: vmug2021
> 1:24:28PM:  L ongoing: waiting on pod/vmug2021-7f5d8dd7dc-czf7j (v1) namespace: vmug2021
> 1:24:28PM:     ^ Pending: ContainerCreating
> 1:24:29PM: ok: reconcile deployment/vmug2021 (apps/v1) namespace: vmug2021
> 1:24:29PM: ---- applying complete [1/1 done] ----
> 1:24:29PM: ---- waiting complete [1/1 done] ----
>
> Succeeded
> ```


## ytt

### ytt - Data Values

Generally the file yaml used for deploy application are almost the same. Working with template can help us to write less and reuse the same files for multiple deployments.

In ytt exist the concept of Data values

Data values provide a way to inject input data into a template. If you think about a `ytt` template as a function, then `data values` are the varying parameters.

```
cd ~/carvel/
```

We start from a file with some


```
cat 03/whoami-app.yaml
```

```yaml
#@ load("@ytt:data", "data")

#@ def vmuglabels():
app: "vmug-application"
#@ end

---
apiVersion: v1
kind: Service
metadata:
  namespace: vmug2021
  name: vmug-application
spec:
  ports:
  - port: #@ data.values.svc_port
    targetPort: #@ data.values.app_port
  selector: #@ vmuglabels()
---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: vmug2021
  name: vmug-application
spec:
  selector:
    matchlabels: #@ vmuglabels()
  template:
    metadata:
      labels: #@ vmuglabels()
    spec:
      containers:
      - name: vmug-application
        image: r.deso.tech/whoami/whoami:latest
        env:
        - name: APPLICATION_NAME
          value: #@ data.values.application_name
```

In the first row of the file we are going to referencing a `data values` loading a `@ytt:data` module.

In the file you will see the referecing of variable using `#@ data.values.`.
For example for change the `application_name`, in the last row we reference to `#@ data.values.application_name`

The `#@ def vmug-labels():` is a type of value that is defined directly in YAML. This parameter is called YAMLFragment and return just the content values. For example in our case will return label: `app: "vmug-application"` in:
- `matchlabels: #@ vmuglabels()`
- `labels: #@ vmuglabels()`
- `selector: #@ vmuglabels()`

The file values.yaml contain the values of the variables specified above.

```bash
cat 03/values.yaml
```

```yaml
#@data/values
---
svc_port: 80
app_port: 80
application_name: captain_kube
```

Try to run `ytt` for understand the final result

```
ytt -f 03/
```

```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: vmug2021
  name: vmug-application
spec:
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: vmug-application
---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: vmug2021
  name: vmug-application
spec:
  selector:
    matchlabels:
      app: vmug-application
  template:
    metadata:
      labels:
        app: vmug-application
    spec:
      containers:
      - name: vmug-application
        image: r.deso.tech/whoami/whoami:latest
        env:
        - name: APPLICATION_NAME
          value: captain_kube
```

We can also overwrite information overwriting the information of the `values.yaml`:

```bash
ytt -f 03/ -v application_name=goodie
```

```yaml
. . .
        env:
        - name: APPLICATION_NAME
          value: goodie
```

### Use kapp together with ytt

This tools can be used together for change and apply configuration directly on kubernetes.

```bash
kapp deploy -a vmug-application -c -f <(ytt -f 03/)
```

I'm applying the modified configuration.

```
. . .
Changes

Namespace  Name      Kind        Conds.  Age  Op      Op st.  Wait to    Rs  Ri
vmug2021   vmug2021  Deployment  2/2 t   21h  update  -       reconcile  ok  -
^          vmug2021  Service     -       21h  update  -       reconcile  ok  -

Op:      0 create, 0 delete, 2 update, 0 noop
Wait to: 2 reconcile, 0 delete, 0 noop

Continue? [yN]:
```

Update it with `y`

> ```
> 9:46:43AM: ---- applying 2 changes [0/2 done] ----
> 9:46:43AM: update deployment/vmug2021 (apps/v1) namespace: vmug2021
> 9:46:43AM: update service/vmug2021 (v1) namespace: vmug2021
> 9:46:43AM: ---- waiting on 2 changes [0/2 done] ----
> 9:46:43AM: ok: reconcile service/vmug2021 (v1) namespace: vmug2021
> 9:46:43AM: ongoing: reconcile deployment/vmug2021 (apps/v1) namespace: vmug2021
> 9:46:43AM:  ^ Waiting for generation 6 to be observed
> 9:46:43AM:  L ok: waiting on replicaset/vmug2021-845db75bb (apps/v1) namespace: vmug2021
> 9:46:43AM:  L ok: waiting on replicaset/vmug2021-7f5d8dd7dc (apps/v1) namespace: vmug2021
> 9:46:43AM:  L ok: waiting on replicaset/vmug2021-6ff748c68d (apps/v1) namespace: vmug2021
> 9:46:43AM:  L ok: waiting on pod/vmug2021-7f5d8dd7dc-czf7j (v1) namespace: vmug2021
> 9:46:43AM:  L ongoing: waiting on pod/vmug2021-6ff748c68d-fvf68 (v1) namespace: vmug2021
> 9:46:43AM:     ^ Pending
> 9:46:43AM: ---- waiting on 1 changes [1/2 done] ----
> 9:46:43AM: ongoing: reconcile deployment/vmug2021 (apps/v1) namespace: vmug2021
> 9:46:43AM:  ^ Waiting for 1 unavailable replicas
> 9:46:43AM:  L ok: waiting on replicaset/vmug2021-845db75bb (apps/v1) namespace: vmug2021
> 9:46:43AM:  L ok: waiting on replicaset/vmug2021-7f5d8dd7dc (apps/v1) namespace: vmug2021
> 9:46:43AM:  L ok: waiting on replicaset/vmug2021-6ff748c68d (apps/v1) namespace: vmug2021
> 9:46:43AM:  L ok: waiting on pod/vmug2021-7f5d8dd7dc-czf7j (v1) namespace: vmug2021
> 9:46:43AM:  L ongoing: waiting on pod/vmug2021-6ff748c68d-fvf68 (v1) namespace: vmug2021
> 9:46:43AM:     ^ Pending: ContainerCreating
> 9:46:46AM: ok: reconcile deployment/vmug2021 (apps/v1) namespace: vmug2021
> 9:46:46AM: ---- applying complete [2/2 done] ----
> 9:46:46AM: ---- waiting complete [2/2 done] ----
>
> Succeeded
> ```



### ytt - Overlay

Sometimes it makes more sense to patch some YAML rather than template it.

For example, when:

- the file should not be edited directly (e.g. from a third party);
- the edit will apply to most or all documents;
- the specific variable is less commonly configured.

In our example we will update the `replicas` of our Deployment.

```
ytt -f 03/
```

```yaml
. . .
          value: captain_kube
  replicas: 2
---
apiVersion: v1
. . .
```

You will see the value of `replicas` set to `2`.


```bash
kapp deploy -a vmug-application -c -f <(ytt -f 04/)
```

> ```
> Target cluster 'https://10.10.180.21:6443' (nodes: master01, 4+)
>
> @@ update deployment/vmug2021 (apps/v1) namespace: vmug2021 @@
>   ...
> 104,104   spec:
>     105 +   replicas: 2
> 105,106     selector:
> 106,107       matchLabels:
>
> Changes
>
> Namespace  Name      Kind        Conds.  Age  Op      Op st.  Wait to    Rs  Ri
> vmug2021   vmug2021  Deployment  2/2 t   21h  update  -       reconcile  ok  -
>
> Op:      0 create, 0 delete, 1 update, 0 noop
> Wait to: 1 reconcile, 0 delete, 0 noop
>
> Continue? [yN]:
> ```

Apply it.

> ```
> 9:58:00AM: ---- applying 1 changes [0/1 done] ----
> 9:58:01AM: update deployment/vmug2021 (apps/v1) namespace: vmug2021
> 9:58:01AM: ---- waiting on 1 changes [0/1 done] ----
> 9:58:01AM: ongoing: reconcile deployment/vmug2021 (apps/v1) namespace: vmug2021
> 9:58:01AM:  ^ Waiting for generation 8 to be observed
> 9:58:01AM:  L ok: waiting on replicaset/vmug2021-845db75bb (apps/v1) namespace: vmug2021
> 9:58:01AM:  L ok: waiting on replicaset/vmug2021-7f5d8dd7dc (apps/v1) namespace: vmug2021
> 9:58:01AM:  L ok: waiting on replicaset/vmug2021-6ff748c68d (apps/v1) namespace: vmug2021
> 9:58:01AM:  L ongoing: waiting on pod/vmug2021-6ff748c68d-p9twc (v1) namespace: vmug2021
> 9:58:01AM:     ^ Pending: ContainerCreating
> 9:58:01AM:  L ok: waiting on pod/vmug2021-6ff748c68d-fvf68 (v1) namespace: vmug2021
> 9:58:02AM: ok: reconcile deployment/vmug2021 (apps/v1) namespace: vmug2021
> 9:58:02AM: ---- applying complete [1/1 done] ----
> 9:58:02AM: ---- waiting complete [1/1 done] ----
>
> Succeeded
> ```

Extremely powerful and simple to be used.

## kbld

Build an `OCI` container can be done using `docker`. Carvel introduce the possibility to building images from source code using `kbld`





we have the same file created above, we just added a file `build.yaml`

```bash
cat 05/build.yaml
```

> ```yaml
> apiVersion: kbld.k14s.io/v1alpha1
> kind: Config
> sources:
> - image: r.deso.tech/vmug2021/vmug-application
> path: .
> ```

```bash
kapp deploy -a vmug-application -c -f <(ytt -f 05/ | kbld -f-)
```

> ```
> Target cluster 'https://10.10.180.21:6443' (nodes: master01, 4+)
> resolve | final: r.deso.tech/whoami/whoami:latest -> r.deso.tech/whoami/whoami@sha256:bc210554e9eae95d75f08cc498a8618dc083f89a6a81744bfc3380f8023403c4
>
> @@ update deployment/vmug2021 (apps/v1) namespace: vmug2021 @@
>   ...
>   4,  4       deployment.kubernetes.io/revision: "3"
>       5 +     kbld.k14s.io/images: |
>       6 +       - origins:
>       7 +         - resolved:
>       8 +             tag: latest
>       9 +             url: r.deso.tech/whoami/whoami:latest
>      10 +         url: r.deso.tech/whoami/whoami@sha256:bc210554e9eae95d75f08cc498a8618dc083f89a6a81744bfc3380f8023403c4
>   5, 11     creationTimestamp: "2021-11-27T11:33:45Z"
>   6, 12     generation: 8
>   ...
> 120,126             value: captain_kube
> 121     -         image: r.deso.tech/whoami/whoami:latest
>     127 +         image: r.deso.tech/whoami/whoami@sha256:bc210554e9eae95d75f08cc498a8618dc083f89a6a81744bfc3380f8023403c4
> 122,128           name: vmug2021
> 123,129   status:
>
> Changes
>
> Namespace  Name      Kind        Conds.  Age  Op      Op st.  Wait to    Rs  Ri
> vmug2021   vmug2021  Deployment  2/2 t   21h  update  -       reconcile  ok  -
>
> Op:      0 create, 0 delete, 1 update, 0 noop
> Wait to: 1 reconcile, 0 delete, 0 noop
>
> Continue? [yN]:
> ```

Something special will happen, the image will be updated with the updated one.

The Image has built and the Image updated:


> ```
> . . .
> r.deso.tech/vmug2021/vmug-application | Successfully built ebb0270b705e
> r.deso.tech/vmug2021/vmug-application | Successfully tagged kbld:rand-1638092430386377525-3817113615958-r-deso-tech-vmug2021-vmug-application
> r.deso.tech/vmug2021/vmug-application | Untagged: kbld:rand-1638092430386377525-3817113615958-r-deso-tech-vmug2021-vmug-application
> r.deso.tech/vmug2021/vmug-application | finished build (using Docker)
> . . .
> 128     -         image: r.deso.tech/whoami/whoami@sha256:bc210554e9eae95d75f08cc498a8618dc083f89a6a81744bfc3380f8023403c4
>     131 +         image: kbld:r-deso-tech-vmug2021-vmug-application-sha256-ebb0270b705e1ace2283cc3ee784b180a730a55d2f8bde08bb56456478705c40
> . . .
> Changes
>
> Namespace  Name      Kind        Conds.  Age  Op      Op st.  Wait to    Rs  Ri
> vmug2021   vmug2021  Deployment  2/2 t   22h  update  -       reconcile  ok  -
>
> Op:      0 create, 0 delete, 1 update, 0 noop
> Wait to: 1 reconcile, 0 delete, 0 noop
>
> Continue? [yN]: N
> ```

Since we are working on a multinode kubernetes cluster, if you apply, you will obtain error because the image is locally, so please press `Enter` or `N`

> ```
> kapp: Error: Stopped
> ```

We will build and upload image on remote registry:

We should login on registry for push the image:

```
docker login -u vmug2021 r.deso.tech
```

> ```
> Password:
> WARNING! Your password will be stored unencrypted in /home/hermedia/.docker/config.json.
> Configure a credential helper to remove this warning. See
> https://docs.docker.com/engine/reference/commandline/login/#credentials-store
>
> Login Succeeded
> ```

```bash
kapp deploy -a vmug-application -c -f <(ytt -f 06/ -v push_images_repo=r.deso.tech/vmug2021/vmug-application | kbld -f-)
```

Now the image is correctly tagged:

```
. . .
120,129             value: captain_kube
121     -         image: r.deso.tech/whoami/whoami:latest
    130 +         image: r.deso.tech/vmug2021/vmug-application@sha256:84bc6be5c4eb64733e9c960155dfd8ad6779f36a72cdfc29b1de446151ee2d75
. . .
Changes

Namespace  Name      Kind        Conds.  Age  Op      Op st.  Wait to    Rs  Ri
vmug2021   vmug2021  Deployment  2/2 t   22h  update  -       reconcile  ok  -

Op:      0 create, 0 delete, 1 update, 0 noop
Wait to: 1 reconcile, 0 delete, 0 noop

Continue? [yN]:
```

You can insert `y` and press `Enter`

```
. . .
10:53:21AM: ---- applying complete [1/1 done] ----
10:53:21AM: ---- waiting complete [1/1 done] ----

Succeeded
```

Open your application

```
curl http://vmug2021.vmug2021.svc.cluster.local
```

### Remove application

With `kapp` you can remove application and clean your kubernetes environment.

```bash
kapp delete -a vmug-application
```

> ```
> Target cluster 'https://10.10.180.21:6443' (nodes: master01, 4+)
>
> Changes
>
> Namespace  Name                       Kind           Conds.  Age  Op      Op st.  Wait to  Rs  Ri
> (cluster)  vmug2021                   Namespace      -       22h  delete  -       delete   ok  -
> vmug2021   test                       Endpoints      -       5m   -       -       delete   ok  -
> ^          test                       Service        -       5m   -       -       delete   ok  -
> ^          test-5ld7w                 EndpointSlice  -       5m   -       -       delete   ok  -
> ^          vmug2021                   Deployment     2/2 t   22h  delete  -       delete   ok  -
> ^          vmug2021                   Endpoints      -       22h  -       -       delete   ok  -
> ^          vmug2021                   Service        -       22h  delete  -       delete   ok  -
> ^          vmug2021-58f64df977        ReplicaSet     -       54m  -       -       delete   ok  -
> ^          vmug2021-644c5cf6f6        ReplicaSet     -       2m   -       -       delete   ok  -
> ^          vmug2021-644c5cf6f6-dq47h  Pod            4/4 t   2m   -       -       delete   ok  -
> ^          vmug2021-644c5cf6f6-x89jv  Pod            4/4 t   2m   -       -       delete   ok  -
> ^          vmug2021-6ff748c68d        ReplicaSet     -       1h   -       -       delete   ok  -
> ^          vmug2021-7564b87db         ReplicaSet     -       16m  -       -       delete   ok  -
> ^          vmug2021-7f5d8dd7dc        ReplicaSet     -       21h  -       -       delete   ok  -
> ^          vmug2021-845db75bb         ReplicaSet     -       22h  -       -       delete   ok  -
> ^          vmug2021-bb69c9c97         ReplicaSet     -       27m  -       -       delete   ok  -
> ^          vmug2021-pfzq7             EndpointSlice  -       22h  -       -       delete   ok  -
>
> Op:      0 create, 3 delete, 0 update, 14 noop
> Wait to: 0 reconcile, 17 delete, 0 noop
>
> Continue? [yN]:
> ```

## Remove kwt
You can kill the process `kwt` with `CTRL-C`

```
sudo pkill -SIGINT kwt
```

```
01:16:40PM: info: StartOptions: Shutting down

Succeeded
```

```
kubectl get pod
```

> ```
> NAME      READY   STATUS    RESTARTS   AGE
> kwt-net   1/1     Running   0          8s
> ```


```
sudo -E kwt net clean-up
```

> ```
> Succeeded
> ```

```
kubectl get pod
```

> ```
> No resources found in default namespace.
> ```