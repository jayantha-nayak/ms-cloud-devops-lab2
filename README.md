# Lab 2 — hello-app on Kubernetes

Your copy of this repository is your submission. The lab sheet is on the course site; this file is where you record what you did.

What's here:

| File | What it is |
|---|---|
| `src/`, `index.html` | a small React page that shows which Pod answered |
| `nginx.conf` | serves the page, and answers `/whoami` with the Pod name |
| `Dockerfile` | builds the page, then serves it with nginx on port 80 |

You don't change any of these. You add `hello-app.yaml` and fill in the sections below.

## Before you push, check

- [*] `hello-app.yaml` is in the root of this repository
- [*] `screenshot.png` is in the root of this repository
- [*] every section below is filled in

---

## 1. My image

Docker Hub image I deployed (for example `ravi123/hello-app:v1`):

```
jayantha2026/hello-app:v1
```

## 2. Pods across nodes

Output of `kubectl get pods -o wide` after you applied `hello-app.yaml`:

```
NAME                         READY   STATUS    RESTARTS   AGE   IP            NODE                            NOMINATED NODE   READINESS GATES
hello-app-5fbbd8565f-jtbq6   1/1     Running   0          15s   10.244.0.23   ml-k8s-insecure-control-plane   <none>           <none>
hello-app-5fbbd8565f-v9k4q   1/1     Running   0          15s   10.244.0.24   ml-k8s-insecure-control-plane   <none>           <none>
hello-app-5fbbd8565f-xgqkf   1/1     Running   0          15s   10.244.0.25   ml-k8s-insecure-control-plane   <none>           <none>

After Deletion
NAME                         READY   STATUS    RESTARTS   AGE   IP            NODE                            NOMINATED NODE   READINESS GATES
hello-app-5fbbd8565f-6vg4t   1/1     Running   0          1s    10.244.0.26   ml-k8s-insecure-control-plane   <none>           <none>
hello-app-5fbbd8565f-jtbq6   1/1     Running   0          23m   10.244.0.23   ml-k8s-insecure-control-plane   <none>           <none>
hello-app-5fbbd8565f-v9k4q   1/1     Running   0          23m   10.244.0.24   ml-k8s-insecure-control-plane   <none>           <none>

```

## 3. The Service and its endpoints

Output of `kubectl get service hello-app` and `kubectl get endpointslices -l kubernetes.io/service-name=hello-app`:

```
NAME        TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)          AGE
hello-app   LoadBalancer   10.96.77.46   <pending>     8080:32037/TCP   4m7s

NAME              ADDRESSTYPE   PORTS   ENDPOINTS                             AGE
hello-app-njr98   IPv4          80      10.244.0.30,10.244.0.31,10.244.0.32   4m17s
```

## 4. Browser

Add `screenshot.png`: the page at `http://localhost:8080` with at least two Pods in the "Answers so far" table.

## 5. The four traps

For each trap, write what you saw and why it happened, in two or three lines of your own words.

**Trap 1 — the local image.**
## Conclusion

I learned how to run and expose apps on a Kubernetes cluster using both CLI commands and YAML files. 
The troubleshooting traps taught me that Kubernetes relies on precise connections:
worker nodes need images from Docker Hub, Services need exact matching labels to find pods, and target ports must match where the app is listening.
Overall, this lab made it much easier to understand how Kubernetes keeps everything running and connected behind the scenes.

**Trap 2 — the selector typo.**
The Service selector was changed to app: helo-app. 
Because the Pods created by the Deployment have the label app: hello-app, no Pods matched the Service's selector. 
As a result, the EndpointSlice had no endpoints, meaning the Service had nowhere to route incoming traffic, causing the browser to show an error.

**Trap 3 — the wrong targetPort.**
The Service successfully matched the pods using the correct label (app: hello-app), so it populated the EndpointSlice. 
However, because targetPort was set to 8080, the Service routed incoming traffic to port 8080 on the pods. 
Since Nginx inside the container only listens on port 80, nothing was listening on port 8080, causing the connection to be refused.

**Trap 4 — the EXTERNAL-IP address.**
Pasting the EXTERNAL-IP into your browser results in a timeout and fails to load.
Reason: That IP address lives inside Docker Desktop's private internal network, which is isolated from your system.
Solution: Access the application using http://localhost:8080 instead, which maps the load balancer port directly to your local machine.

## 6. Your YAML vs the generated YAML

Name two fields in the `--dry-run=client -o yaml` output that were not in your file. Why does your file still work without them?
loadBalancer: {}
resources: {}