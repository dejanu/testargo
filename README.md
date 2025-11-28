# testargo


* Create app: `argocd app create demo --repo https://github.com/dejanu/testargo.git --path demo --dest-server https://kubernetes.default.svc --dest-namespace default`

* Default secret: `kubectl -n argocd get secret argocd-initial-admin-secret -ojsonpath="{.data.password}" | base64 -d`

* Get address of your Argo CD API server (`argocd-server` svc): `kubectl get svc argocd-server -n argocd -ojsonpath='{.status.loadBalancer.ingress[0].ip}'`

```bash
# login to argo cd server: local vs external IP
argocd login localhost:8080 --username admin --password <pass> --insecure
argocd login 34.120.15.10 --username admin --password <pass> --insecure

# list apps
argocd app list
```

### Diff Strategies describe the algorithms/approaches Argo CD ca use to compute the diff.


* Argo CD uses a **three-way** diff that compares:
    - Desired (Git): 1 port
    - Live (cluster): 2 ports
    - Last-applied: 1 port (from annotation)

* Issues for list of maps: argoproj/argo-cd#23171  argoproj/argo-cd#18038

```bash
# svc play: add another port to live object
 - name: http
    port: 80
    protocol: TCP
    targetPort: 80
- name: https
    port: 443
    protocol: TCP
    targetPort: 9377
```

### k8s vanilla

* `kubectl edit` or `kubectl patch` to add the port, DO NOT update `last-applied-configuration` annotation, it sends your changes back to the server via a PATCH or PUT.


* `kubectl apply -f` sets the `kubectl.kubernetes.io/last-applied-configuration: '{...}`' annotation on each object. The annotation contains the contents of the object configuration file that was used to create the object.

* The annotation stores a snapshot of the manifest last used with kubectl apply. `kubectl apply` uses that snapshot + the live state, + your new manifest to do a three-way merge, so it knows what you changed and what was changed by the cluster.