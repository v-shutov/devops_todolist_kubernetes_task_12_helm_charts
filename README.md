# Django ToDo list

This is a todo list web application with basic features of most web apps, i.e., accounts/login, API, and interactive UI. To do this task, you will need:

- CSS | [Skeleton](http://getskeleton.com/)
- JS  | [jQuery](https://jquery.com/)

## Explore

Try it out by installing the requirements (the following commands work only with Python 3.8 and higher, due to Django 4):

```
pip install -r requirements.txt
```

Create a database schema:

```
python manage.py migrate
```

And then start the server (default is http://localhost:8000):

```
python manage.py runserver
```

Now you can browse the [API](http://localhost:8000/api/) or start on the [landing page](http://localhost:8000/).

## Task

Create a kubernetes manifest for a pod which will containa ToDo app container:

1. Fork this repository.
1. Use `kind` to spin up a cluster from a `cluster.yml` configuration file.
1. Inspect Nodes for Labels and Taints
1. Taint nodes labeled with `app=mysql` with `app=mysql:NoSchedule`
1. Create a helm chart named `todoapp` inside a `helm-chart` directory
1. `todoapp` helm chart requirements:
    1. Namespace name should be controlled from a `values.yaml` file
    1. Use `.Chart.Name` as a prefix for all resources names
    1. Secrets should be controlled from a `values.yaml` file
    1. Secrets `data` should be popualted by a `range` function
    1. Inside the deployment use `range` to map secrets as environment variables
    1. Resources requests and limits should controlled from a `values.yaml` file
    1. RollingUpdate parameters should be controlled from a `values.yaml` file
    1. Image repository and tag should be controlled from a `values.yaml` file
    1. Deployment node affinity parameters should be controlled from a `values.yaml` file (key and values)
    1. `hpa` min and max replicas should be controlled from a `values.yaml` file
    1. `hpa` average CPU and Memory utilization should be controlled from a `values.yaml` file
    1. `pv` capacity should be controlled from a `values.yaml` file
    1. `pvc` requests storage should be controlled from a `values.yaml` file
    1. Service Account Name inside both `Deployment` and all rbac objects should be controld from a `values.yaml` file
1. Creata a sub-chart called `mysql` inside a `charts` directory of the `todoapp` helm chart
1. `mysql` helm chart requirements:
    1. Namespace name should be controlled from a `values.yaml` file
    1. Use `.Chart.Name` as a prefix for all resources names
    1. Secrets should be controlled from a `values.yaml` file
    1. Secrets `data` should be popualted by a `range` function
    1. StateFulSet's Replicas should be controlled from a `values.yaml` file
    1. Image repository and tag should be controlled from a `values.yaml` file
    1. `pvc` requests storage should be controlled from a `values.yaml` file
    1. Affinity and Toleration parameters should be controlled from a `values.yaml` file
    1. Resource requests and limits should controlled from a `values.yaml` file
1. Add explicit dependencies between `todoapp` and `mysql` charts
    1. Inside the Chart.yaml file of todoapp chart, add lines
        ```
        dependencies:
        - name: mysql
        ```

10. `bootstrap.sh` should containe all commands to deploy prerequsites and the `todoapp` helm chart
11. Deploy helm chart to your `kind` cluster
11. Run command `kubectl get all,cm,secret,ing -A` and put the output in a file called `output.log` in a root of the repository
12. `README.md` should have instructuions on how to validate the changes
13. Create PR with your changes and attach it for validation on a platform.

---

### Delpoy

Use:
```bash
./bootstrap.sh
```

Now you can start on the [landing page](http://localhost/) or browse the [API](http://localhost:/api/)

### Validate:

All required nodes are properly tainted:

```bash
kubectl get nodes -o go-template='{{range .items}}{{.metadata.name}}{{"\n  Labels:"}}{{range $key, $value := .metadata.labels}}{{"\n    "}}{{$key}}={{$value}}{{end}}{{"\n  Taints:"}}{{range .spec.taints}}{{"\n    "}}{{.key}}={{.value}}:{{.effect}}{{end}}{{"\n\n"}}{{end}}'
```
```yaml
kind-control-plane
  Labels:
    beta.kubernetes.io/arch=amd64
    beta.kubernetes.io/os=linux
    ingress-ready=true
    kubernetes.io/arch=amd64
    kubernetes.io/hostname=kind-control-plane
    kubernetes.io/os=linux
    node-role.kubernetes.io/control-plane=
    node.kubernetes.io/exclude-from-external-load-balancers=
  Taints:
    node-role.kubernetes.io/control-plane=<no value>:NoSchedule

kind-worker
  Labels:
    app=mysql
    beta.kubernetes.io/arch=amd64
    beta.kubernetes.io/os=linux
    kubernetes.io/arch=amd64
    kubernetes.io/hostname=kind-worker
    kubernetes.io/os=linux
  Taints:
    app=mysql:NoSchedule

kind-worker2
  Labels:
    app=mysql
    beta.kubernetes.io/arch=amd64
    beta.kubernetes.io/os=linux
    kubernetes.io/arch=amd64
    kubernetes.io/hostname=kind-worker2
    kubernetes.io/os=linux
  Taints:
    app=mysql:NoSchedule

kind-worker3
  Labels:
    app=todoapp
    beta.kubernetes.io/arch=amd64
    beta.kubernetes.io/os=linux
    kubernetes.io/arch=amd64
    kubernetes.io/hostname=kind-worker3
    kubernetes.io/os=linux
  Taints:

kind-worker4
  Labels:
    app=todoapp
    beta.kubernetes.io/arch=amd64
    beta.kubernetes.io/os=linux
    kubernetes.io/arch=amd64
    kubernetes.io/hostname=kind-worker4
    kubernetes.io/os=linux
  Taints:

kind-worker5
  Labels:
    app=todoapp
    beta.kubernetes.io/arch=amd64
    beta.kubernetes.io/os=linux
    kubernetes.io/arch=amd64
    kubernetes.io/hostname=kind-worker5
    kubernetes.io/os=linux
  Taints:

kind-worker6
  Labels:
    beta.kubernetes.io/arch=amd64
    beta.kubernetes.io/os=linux
    kubernetes.io/arch=amd64
    kubernetes.io/hostname=kind-worker6
    kubernetes.io/os=linux
  Taints:
```
All pods are runing on the right nodes:
```bash
kubectl get pods -o wide -A | grep -E 'NAMESPACE|mysql|todoapp'
```
```yaml
NAMESPACE            NAME                                         READY   STATUS      RESTARTS   AGE   IP           NODE                 NOMINATED NODE   READINESS GATES
mysql                mysql-0                                      1/1     Running     0          23m   10.244.4.3   kind-worker2         <none>           <none>
mysql                mysql-1                                      1/1     Running     0          21m   10.244.1.3   kind-worker          <none>           <none>
todoapp              todoapp-5f4748c574-47rpw                     1/1     Running     0          23m   10.244.6.2   kind-worker5         <none>           <none>
todoapp              todoapp-5f4748c574-lxsrw                     1/1     Running     0          23m   10.244.5.2   kind-worker4         <none>           <none>
```

Check Helm release history:
```bash
helm history todoapp
```
```yaml
REVISION        UPDATED                         STATUS          CHART           APP VERSION     DESCRIPTION
1               Tue Jul 23 17:38:19 2024        deployed        todoapp-0.1.0   1.16.0          Install complete
```
