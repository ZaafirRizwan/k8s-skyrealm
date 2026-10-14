# Kubernetes Skyrealm

**An interactive 3D world that shows how Kubernetes actually works, one packet at a time.**

### ▶ Live demo: **https://k8s-skyrealm.pages.dev**

Kubernetes Skyrealm turns a running Kubernetes cluster into floating sky islands (art style inspired by Genshin Impact). The control plane sits in the sky, worker nodes float below, and every message between components is a glowing packet you can click and read: real HTTP verbs, API paths, etcd keys, Raft messages, CRI calls, DNS answers and VXLAN headers.

It's a learning tool for anyone who has typed `kubectl apply` and wondered what happens next.

---

## What you can do

- **Fly around a live cluster.** Orbit, zoom, and click any component, Pod or packet to see what it is and what it does, with a plain-English summary first.
- **Follow guided quests.** Each one tells one story end to end with narration, a progress strip, and the camera moving to the action.
- **Read every packet.** Each message carries its protocol, sender, receiver and a realistic payload. Cross-node traffic shows its encapsulation: inner HTTP → outer VXLAN UDP 8472 → inner HTTP.
- **See the data streams.** Routes are drawn before a packet travels, dashes flow toward the destination, the sender and receiver ripple, and an "In flight now" list names every live message.

## Quests

| # | Quest | What it teaches |
|---|-------|-----------------|
| ★ | **Meet the Realm** | A short plain-English tour of every component |
| I | Birth of a Deployment | `kubectl apply` → authn/RBAC/admission → etcd Raft commit → Deployment & ReplicaSet controllers → scheduler filter/score/bind → kubelet → CRI sandbox → CNI IP → image pull → running Pod → EndpointSlice → kube-proxy |
| II | A Traveler's Request | Browser → cloud load balancer → NodePort → iptables DNAT → ingress-nginx → VXLAN overlay → Pod, and the reply via conntrack |
| III | Asking the Oracle | Pod DNS via CoreDNS (`resolv.conf`, search domains), then a ClusterIP that no process listens on |
| IV | Heartbeat of the Realm | Node Leases, node failure, `unreachable:NoExecute` taint, eviction after 300 s, rescheduling, orphan cleanup |
| V | Scaling the Tide | metrics-server scraping kubelets, the API aggregation layer, HorizontalPodAutoscaler math |
| VI | Fall and Rise | Liveness probe failures, container restart in the same sandbox, readiness and endpoint removal |
| VII | Argo CD Sync | git push → webhook → repo-server render → diff → server-side apply → rollout → Synced/Healthy |
| VIII | Flux Reconcile | GitRepository artifact → kustomize-controller build → apply → health check → Kustomization Ready |
| IX | The Policy Gate | Mutating and validating admission webhooks; Kyverno rejects a `:latest` image before etcd is touched |
| X | Roots of Storage | StatefulSet → PVC → WaitForFirstConsumer → CSI CreateVolume → VolumeAttachment → NodeStage/NodePublish |
| XI | The Watchers | Prometheus Kubernetes service discovery and pull-based scraping |

## Components in the world

**Control plane:** kube-apiserver (with authn / authz / admission gate rings), etcd ×3 (Raft leader + followers), kube-scheduler, kube-controller-manager (Deployment, ReplicaSet, EndpointSlice, node-lifecycle, HPA controllers)

**Every node:** kubelet, containerd (CRI), kube-proxy / iptables, flannel VXLAN tunnel endpoint, Pods (pause sandbox + app containers)

**Workloads & add-ons:** ingress-nginx, CoreDNS, metrics-server, Argo CD (server, repo-server, application-controller), Flux (source-controller, kustomize-controller), Prometheus, Kyverno, CSI controller

**Outside the cluster:** you (kubectl / browser / git), cloud load balancer, OCI image registry, Git repository, cloud block storage

## Packet types (colour key)

| Protocol | What it carries |
|----------|-----------------|
| HTTPS REST | Requests to kube-apiserver (`:6443`) |
| Watch event | ADDED / MODIFIED / DELETED pushed down long-lived watch streams |
| etcd gRPC | apiserver ↔ etcd KV transactions (`:2379`) |
| Raft | etcd AppendEntries, acks, heartbeats (`:2380`) |
| CRI gRPC | kubelet → containerd (RunPodSandbox, PullImage, CreateContainer…) |
| OCI pull | Image manifests and layer blobs from the registry |
| App HTTP | User traffic to workloads |
| DNS | UDP `:53` queries to CoreDNS |
| VXLAN | Pod packets wrapped node-to-node on UDP `:8472` |
| Probe | kubelet liveness / readiness checks |
| Metrics | metrics-server and Prometheus scrapes |
| Git | push, fetch and webhooks |
| Internal gRPC | Between a tool's own parts (e.g. Argo CD) |
| Admission webhook | apiserver → webhook `AdmissionReview` |
| CSI / cloud API | Volume create, attach and mount |

## Controls

| Input | Action |
|-------|--------|
| Drag / right-drag / scroll | Orbit / pan / zoom |
| Click | Inspect a component, Pod or packet |
| Double-click | Fly to it |
| `Space` | Pause / resume |
| `→` or `Enter` | Continue to the next step |
| `F` | Follow the current packet |
| `L` | Toggle labels |
| `Esc` | Stop the quest and return home |

## Run it locally

It's a single static HTML file with no build step:

```bash
git clone https://github.com/ZaafirRizwan/k8s-skyrealm.git
cd k8s-skyrealm
npx serve .
```

Or open `index.html` straight in a browser.

## Deploy

Hosted on Cloudflare Pages:

```bash
npx wrangler pages deploy . --project-name k8s-skyrealm --branch main
```

Any static host works (GitHub Pages, Netlify, S3).

## Built with

- [three.js](https://threejs.org/) r128 (WebGL), with toon shading, outline hulls and custom GLSL for the sky, waterfalls and flowing data streams
- Plain JavaScript: no framework, no bundler, one file
- Fonts: Cinzel, Nunito, JetBrains Mono

## Accuracy notes

The cluster is **simulated in the browser**. Names, IPs and payloads are illustrative, but the behaviour follows Kubernetes v1.31: only the apiserver talks to etcd, the scheduler only writes Bindings, kube-proxy programs the kernel rather than sitting in the data path, ingress-nginx routes straight to Pod endpoints, a 40 s node-monitor grace period, 300 s default eviction tolerations, and so on. If you spot something wrong, please open an issue.

## License

[MIT](LICENSE) © Zaafir Rizwan

---

<sub>Keywords: kubernetes, k8s, kubernetes visualization, kubernetes architecture, interactive kubernetes tutorial, learn kubernetes, kubernetes for beginners, kube-apiserver, etcd raft, kube-scheduler, kubelet, kube-proxy, iptables, containerd, CRI, CNI, flannel, VXLAN, CoreDNS, ingress-nginx, HPA, metrics-server, GitOps, Argo CD, Flux CD, Kyverno, admission webhook, CSI, persistent volumes, Prometheus, cloud native, DevOps, three.js, WebGL, 3D visualization, Genshin Impact style</sub>
