# DevOps Interview Questions & Answers

> Organized by topic. **Must Know** = highly likely to be asked in Pakistani tech company interviews (Systems Limited, Netsol, 10Pearls, Arbisoft, Folio3, i2c, Contour, TPS, Tkxel, etc.)

---

## Table of Contents
1. [Docker](#1-docker)
2. [Kubernetes](#2-kubernetes)
3. [Terraform](#3-terraform)
4. [Jenkins](#4-jenkins)
5. [ArgoCD & GitOps](#5-argocd--gitops)
6. [AWS](#6-aws)
7. [Linux & System Administration](#7-linux--system-administration)
8. [Observability & Monitoring](#8-observability--monitoring)
9. [CI/CD General](#9-cicd-general)
10. [Behavioral & Soft Skills](#10-behavioral--soft-skills)
11. [Linux Administration — Senior Level](#11-linux-administration--senior-level)
12. [Prometheus & Grafana](#12-prometheus--grafana)
13. [Ansible](#13-ansible)
14. [AWS CodePipeline & AWS CI/CD](#14-aws-codepipeline--aws-cicd)

---

## 1. Docker

### Q1. What is Docker and what are its main advantages? **Must Know**
Docker is a containerization tool that packages an application along with all its dependencies, libraries, and configurations into a container, ensuring it runs consistently across different environments.

**Advantages:**
- Packages app + dependencies together — runs the same on dev, test, and prod
- Containers share the host OS kernel — much lighter and faster than VMs
- Containers start in seconds
- Isolated environments prevent dependency conflicts
- Runs on any system with Docker installed regardless of OS

---

### Q2. What is the difference between Docker and Docker Compose? **Must Know**

| Feature | Docker | Docker Compose |
|---|---|---|
| Purpose | Manages individual containers | Orchestrates multiple containers as one app |
| Scope | Single container (build, run, stop) | Multi-container stacks (app + DB + cache) |
| Config | Dockerfile + CLI flags | `docker-compose.yml` |
| Commands | `docker run`, `docker build` | `docker-compose up`, `docker-compose down` |

**In short:** Docker is the engine. Docker Compose is the assembly manual for the full stack.

---

### Q3. Explain Docker Networking. **Must Know**

Docker networking allows containers to communicate with each other, the host, and the outside world.

- **Bridge (Default):** Containers on the same bridge network communicate via IP or container name. Best for standalone containers on the same host.
- **Host:** Removes network isolation between container and host. High performance but no isolation.
- **None:** Completely disables networking. Used for secure isolated processing.
- **Overlay:** Connects multiple Docker daemons. Used in Docker Swarm for cross-host communication.
- **Macvlan:** Assigns a physical MAC address to a container — makes it appear as a physical device on the network.

---

### Q4. What is a Dockerfile? **Must Know**
A plain text file with instructions (`FROM`, `RUN`, `COPY`, `CMD`) that Docker reads line-by-line to build a Docker image automatically.

---

### Q5. What is the difference between `COPY` and `ADD` in a Dockerfile?
Both copy files into the image. `COPY` only copies local files. `ADD` can also download files from URLs and auto-extract `.tar.gz` archives. **Best practice:** use `COPY` unless you specifically need `ADD`'s extra features.

---

### Q6. What is the difference between `CMD` and `ENTRYPOINT`? **Must Know**
- `ENTRYPOINT` sets the primary executable — cannot be easily overridden.
- `CMD` provides default arguments for `ENTRYPOINT` — can be overridden by passing arguments in `docker run`.

---

### Q7. How do you reduce Docker image size? **Must Know**
- Use **multi-stage builds** — compile in a heavy stage, copy only the binary into a minimal runtime stage
- Use lightweight base images like `alpine`
- Group `RUN` commands with `&&` to reduce layers
- Use `.dockerignore` to exclude `node_modules`, `.git`, logs, etc.

---

### Q8. What are Docker Volumes and why are they better than bind mounts? **Must Know**
Volumes persist data independently of the container lifecycle. Unlike bind mounts (which depend on the host's directory structure), volumes are fully managed by Docker, portable, and easier to back up.

---

### Q9. What happens to data inside a container when it is deleted?
If a container is **stopped**, data stays. If it is **deleted**, all data written to the writable layer is permanently lost — unless it was stored in a Docker Volume or bind mount.

---

### Q10. What does the `-d` flag do in `docker run`?
Runs the container in **detached mode** — the container runs in the background and the terminal is freed.

---

### Q11. What are Docker restart policies?
- `no` — never restart (default)
- `on-failure` — restart only if exit code is non-zero
- `always` — always restart
- `unless-stopped` — always restart unless manually stopped

---

### Q12. What is the difference between `docker stop` and `docker kill`? **Must Know**
- `docker stop` sends `SIGTERM` first, allowing graceful shutdown, then `SIGKILL` if needed.
- `docker kill` immediately sends `SIGKILL` — no graceful shutdown.

---

### Q13. What is a dangling image?
An image no longer tagged and not referenced by any container. Usually created when a new image is built with the same tag. Shows up as `<none>:<none>`. Clean up with `docker image prune`.

---

### Q14. What is Docker Swarm vs Kubernetes?
- **Docker Swarm:** Native Docker orchestration. Easier to set up, tightly integrated with Docker Compose. Good for smaller setups.
- **Kubernetes:** More powerful, complex, industry-standard. Designed for large-scale, highly available enterprise deployments.

---

### Scenario: Data disappeared after container restart **Must Know**
**Problem:** Developer ran PostgreSQL inside a container. After the container was removed and recreated, all data was gone.

**Answer:** Data was written to the container's writable layer which is destroyed on deletion. Fix: mount a **Docker Volume** to `/var/lib/postgresql/data`. New containers attach to the same volume and retain all data.

---

### Scenario: Slow CI/CD Docker builds (15 min per push) **Must Know**
**Answer:** Poor use of Docker layer caching. Fix: copy `package.json` and run `npm install` **before** copying the rest of the source code. Docker caches the heavy install layer and only rebuilds when dependencies change. Also add `.dockerignore` to exclude `node_modules`.

---

### Scenario: Container crashes immediately — how to debug? **Must Know**
1. `docker logs <container_id>` — check for startup errors
2. `docker ps -a` — check the exit code
3. `docker run -it --entrypoint /bin/sh <image>` — override entrypoint to get a shell inside

---

### Scenario: One container consuming all host RAM
**Answer:** Use `--memory="512m"` and `--cpus="1.0"` flags on `docker run`. In Docker Compose, use `deploy.resources.limits`. Docker will `OOMKill` that specific container instead of crashing the host.

---

### Scenario: "No space left on device" error on Docker host
**Answer:** Accumulated dangling images, old build caches, stopped containers, unused volumes. Run `docker system prune` to clean. Add a cron job to run this periodically.

---

### Scenario: Python container can't reach Redis at `localhost:6379`
**Answer:** `localhost` inside the Python container refers to itself, not Redis. In Docker Compose, use the **service name** as the hostname: `redis:6379`. Docker's internal DNS resolves service names to container IPs.

---

## 2. Kubernetes

### Q1. What is Kubernetes and what problem does it solve? **Must Know**
Kubernetes is an open-source container orchestration platform. While Docker manages individual containers, Kubernetes manages clusters of containers at scale — automating deployment, scaling, load balancing, and self-healing (auto-restarting failed containers).

---

### Q2. What is a Pod? **Must Know**
The smallest deployable unit in Kubernetes. Represents one running process. Can contain one or multiple containers that share the same network namespace and storage volumes.

---

### Q3. What is a Namespace?
A mechanism for isolating groups of resources within a single cluster. Think of them as "virtual clusters" — used to separate environments (dev, staging, prod) or teams.

---

### Q4. What is a Deployment? **Must Know**
A declarative way to manage a set of identical Pods. You define the desired state (e.g., "3 replicas of image v2.0") and the Deployment controller continuously ensures the actual state matches — enabling rolling updates and rollbacks.

---

### Q5. What are the Kubernetes Control Plane components? **Must Know**

| Component | Role |
|---|---|
| `kube-apiserver` | Front-end of the control plane — all communication goes through here via REST API |
| `etcd` | Key-value store — the cluster's brain, stores all state and config |
| `kube-scheduler` | Assigns newly created Pods to nodes |
| `kube-controller-manager` | Runs controllers (Node, ReplicaSet, etc.) to regulate cluster state |

---

### Q6. What are the Worker Node components? **Must Know**

| Component | Role |
|---|---|
| `kubelet` | Agent on every node — ensures containers are running in Pods |
| `kube-proxy` | Maintains network rules for Pod communication |
| Container Runtime | Actually runs containers (containerd, CRI-O) |

---

### Q7. What is etcd and why is it critical?
etcd is the single source of truth for the cluster. It stores all config, state, and metadata. If etcd is lost and not backed up, the cluster is effectively dead.

---

### Q8. Difference between ReplicaSet and DaemonSet? **Must Know**
- **ReplicaSet:** Ensures N replicas of a Pod run anywhere in the cluster.
- **DaemonSet:** Ensures exactly **one** copy of a Pod runs on **every** node (used for log agents like Fluentd, monitoring agents like Prometheus node-exporter).

---

### Q9. Explain ClusterIP vs NodePort vs LoadBalancer. **Must Know**

| Type | Access | Use Case |
|---|---|---|
| `ClusterIP` | Internal only (within cluster) | Service-to-service communication |
| `NodePort` | Via node IP + port (30000–32767) | Dev/test access from outside |
| `LoadBalancer` | Public IP via cloud provider | Production external exposure |

---

### Q10. What is an Ingress? **Must Know**
An API object that manages external HTTP/HTTPS access to cluster services. Unlike `LoadBalancer` (one LB per service), Ingress provides single-IP routing rules (path-based or host-based) to multiple services — much more cost-effective.

---

### Q11. What is the difference between PV and PVC?
- **PV (Persistent Volume):** Physical storage provisioned by an admin (AWS EBS, NFS).
- **PVC (Persistent Volume Claim):** A request for storage by a Pod. It finds a matching PV and binds to it.

---

### Q12. ConfigMaps vs Secrets? **Must Know**
- **ConfigMap:** Non-sensitive config data in plain text (env vars, config files).
- **Secret:** Sensitive data (passwords, tokens, SSH keys) — base64-encoded, can be encrypted at rest in etcd.

---

### Q13. What is a StatefulSet and when do you use it? **Must Know**
Like a Deployment, but provides stable, unique Pod identities (db-0, db-1) and dedicated storage per Pod. Used for stateful apps like databases (MySQL, MongoDB, Kafka).

---

### Q14. What are Taints and Tolerations?
- **Taint:** Applied to a node to repel Pods (e.g., "only GPU pods allowed here").
- **Toleration:** Applied to a Pod to allow it to schedule on tainted nodes.

---

### Q15. What is Node Affinity?
Attracts Pods to specific nodes based on node labels (e.g., "schedule only on nodes with `disktype=ssd`"). Opposite of taints.

---

### Q16. Liveness Probe vs Readiness Probe? **Must Know**

| Probe | Question it asks | On failure |
|---|---|---|
| **Liveness** | Is the container still running/healthy? | Restart the container |
| **Readiness** | Is the container ready for traffic? | Remove from Service endpoints (no restart) |

**Production danger:** A liveness probe that checks the DB will restart all pods if the DB slows down. A readiness probe with a tight timeout drops all traffic on a minor network blip.

---

### Q17. Pod Status Reference Table **Must Know**

| Status | Component | Meaning |
|---|---|---|
| `Pending` | Kube Scheduler | Can't place pod — insufficient resources, missing PVC, taint mismatch |
| `ContainerCreating` | Kubelet | Pulling images, mounting volumes |
| `Running` | Kubelet + Runtime | All containers running |
| `CrashLoopBackOff` | Kubelet | Container crashes repeatedly on start |
| `ImagePullBackOff` | Kubelet | Can't pull image — auth, network, wrong tag |
| `Completed` | Container Runtime | Container finished successfully (exit code 0) |
| `Failed` | Container Runtime | Container exited with non-zero code |
| `Terminating` | API Server + Kubelet | Pod received deletion request |
| `Unknown` | Node/Kubelet | Control plane can't communicate with the node |

---

### Scenario: Pod stuck in Pending state **Must Know**
The scheduler can't find a node. Causes: insufficient CPU/Memory, missing PVC, mismatched node selector or taints.

**Fix:** `kubectl describe pod <pod-name>` → check the **Events** section at the bottom.

---

### Scenario: Pod in CrashLoopBackOff **Must Know**
Container starts and crashes immediately, K8s keeps restarting it.

**Troubleshooting:**
1. `kubectl logs <pod-name>` — check app errors
2. `kubectl logs <pod-name> --previous` — logs from the previous crashed instance
3. `kubectl describe pod <pod-name>` — check for missing Secrets/ConfigMaps

---

### Scenario: What happens when a Worker Node dies?
`kube-controller-manager` detects no heartbeat (after ~5 min), marks node `NotReady`. Pods managed by Deployments/ReplicaSets are rescheduled on healthy nodes automatically.

---

### Scenario: Zero-downtime deployment in Kubernetes **Must Know**
Use a `Deployment` with `RollingUpdate` strategy. Set `maxUnavailable: 0` and `maxSurge: 1`. K8s spins up new pods, waits for Readiness Probe to pass, then terminates old pods one by one — traffic always has a healthy pod.

---

### Scenario: Users get 504 errors but pods show Running
**Answer:**
1. Check if pods are actually `1/1 Ready` (running ≠ ready)
2. `kubectl exec` into pod and `curl localhost:<port>` — if it hangs, app is deadlocked
3. Check DB connection pool exhaustion or external API timeouts

---

### Scenario: Container works on dev machine, fails in Kubernetes
Check:
- **Architecture mismatch:** Mac M1/M2 (ARM64) image failing on AMD64 cluster nodes
- **Missing env vars/Secrets** that exist in local `.env` but not in K8s manifest
- **Security Context:** `readOnlyRootFilesystem` or non-root enforcement blocking local path access

---

### Scenario: Users can't access app via Ingress
Trace outside-in:
1. **DNS** — does the domain resolve to the correct Ingress Controller IP?
2. `kubectl describe ingress <name>` — check hostname/path rules match
3. `kubectl get svc` — check service name and port match Ingress rules
4. `kubectl get endpoints <service-name>` — if empty, Service selector doesn't match Pod labels

---

## 3. Terraform

### Q1. What is Terraform? **Must Know**
An open-source Infrastructure as Code (IaC) tool by HashiCorp. You define infrastructure (VPCs, EC2, databases) in declarative HCL config files, and Terraform provisions and manages their lifecycle.

---

### Q2. Terraform vs Ansible? **Must Know**
- **Terraform:** Provisions infrastructure (creates VMs, networks, databases).
- **Ansible:** Configures software inside those machines (installs packages, edits config files).
- They complement each other — Terraform builds the server, Ansible configures it.

---

### Q3. Core Terraform workflow? **Must Know**

| Command | What it does |
|---|---|
| `terraform init` | Downloads providers, initializes working directory |
| `terraform plan` | Shows what will change — no actual changes made |
| `terraform apply` | Executes the plan and provisions infrastructure |
| `terraform destroy` | Tears down all managed resources |

---

### Q4. What is a Provider?
A plugin that lets Terraform talk to an external API (AWS, Azure, GCP, Kubernetes, GitHub). Translates HCL into API calls.

---

### Q5. What is the difference between `resource` and `data` blocks? **Must Know**
- `resource` — **creates/manages** infrastructure (e.g., creates a new EC2 instance)
- `data` — **reads** existing infrastructure Terraform didn't create (e.g., fetches an existing subnet ID)

---

### Q6. What is `terraform.tfstate`? **Must Know**
A JSON file that maps your HCL code to real-world resource IDs. Terraform uses it to track what exists, detect changes, and plan future runs. **Treat it as sacred — losing it is very painful.**

---

### Q7. What is a Remote Backend and why use it? **Must Know**
By default, state is stored locally. A remote backend (S3, GCS, Terraform Cloud) stores state centrally — enabling team collaboration, state locking, and CI/CD integration.

---

### Q8. What is State File Locking?
Prevents two people/pipelines from running `apply` simultaneously, which would corrupt the state file. On AWS, implemented with a DynamoDB table alongside an S3 backend.

---

### Q9. What happens when Terraform state drifts? **Must Know**
Someone manually changes infrastructure outside Terraform (e.g., in AWS Console). Next `terraform plan` detects the mismatch and plans to **overwrite** the manual changes to enforce your code as truth.

---

### Q10. What does `terraform import` do?
Brings existing manually-created infrastructure under Terraform management by adding it to the state file without recreating it.

---

### Q11. What are Terraform Modules?
Self-contained, reusable packages of Terraform config. Group related resources (VPC + Subnets + Route Tables) into one logical unit — promotes DRY principles.

---

### Q12. How do you handle secrets in Terraform? **Must Know**
- **Never hardcode** secrets in `.tf` files
- Declare variable with `sensitive = true` (redacts from CLI output)
- Pass via environment variables (`TF_VAR_db_password`) or `.tfvars` files excluded from Git
- Best practice: use AWS Secrets Manager or HashiCorp Vault — don't pass actual secrets through Terraform at all

---

### Q13. What are Terraform Workspaces?
Allow multiple state files from a single config directory (dev, staging, prod). **Dangerous at scale** — it's easy to accidentally apply to the wrong workspace. Separate directories (or Terragrunt) is safer for large orgs.

---

### Q14. `count` vs `for_each` — what's the difference and why does switching break things? **Must Know**
- `count` identifies resources by index (0, 1, 2). Remove middle item → indices shift → Terraform destroys and recreates downstream resources.
- `for_each` identifies resources by string key (`"subnet-a"`). Removal only affects that specific key — much safer.
- **Switching between them changes resource addresses in state** — Terraform treats old resources as deleted and new ones as creates. Use `moved` block in Terraform 1.1+ to migrate safely.

---

### Q15. What is the `lifecycle` block?
Controls how Terraform handles resource changes:
- `create_before_destroy` — zero-downtime replacements
- `prevent_destroy` — prevents accidental deletion (use on production databases)
- `ignore_changes` — ignores manual changes to specific attributes

---

### Q16. What is `depends_on` and when is it needed?
Terraform infers dependencies automatically when resources share data references (implicit). `depends_on` is used when a dependency exists but no data is referenced in code — e.g., an IAM role must fully propagate before a Lambda using it is created.

---

### Q17. How do you refactor Terraform code without destroying production resources?
Use the `moved` block (Terraform 1.1+) to declare that a resource moved from one address to another. Terraform safely relocates the identity in state without touching real infrastructure. Historically done with `terraform state mv`.

---

### Scenario: State file accidentally deleted — how to recover?
- **With remote backend + versioning (S3):** restore previous version from S3
- **With backup file:** restore `terraform.tfstate.backup`
- **No backup:** rewrite HCL code and run `terraform import` for every resource manually

---

### Scenario: `terraform apply` fails halfway through
Don't panic. Terraform writes successfully created resources to state before exiting. Fix the error (permissions, quota, typo), then run `terraform plan` again — Terraform skips already-created resources and finishes the rest.

---

### Scenario: State is locked after a crashed apply
1. Confirm no other pipelines or team members are running apply
2. Copy the Lock ID from the error message
3. Run `terraform force-unlock <LOCK_ID>`

---

### Scenario: Developer ran `terraform destroy` on a production RDS
**Prevention:** Add `lifecycle { prevent_destroy = true }` to the RDS resource block. Any destroy plan will immediately fail before execution.

---

### Scenario: Terraform plan shows no change but apply still modifies resources
Happens with:
- **Provisioners:** `local-exec`/`remote-exec` side effects aren't tracked in state
- **Data Sources:** An AMI data source fetching `most_recent` may return a new AMI between plan and apply, triggering recreation

---

## 4. Jenkins

### Q1. What is Jenkins? **Must Know**
An open-source automation server that automates building, testing, and deploying software — implementing CI/CD. Catches bugs early and automates the release process.

---

### Q2. Jenkins Master-Agent architecture? **Must Know**
- **Controller (Master):** Central server — schedules jobs, monitors agents, presents the UI. Should NOT run heavy builds.
- **Agent (Node):** A machine (VM, Docker container, K8s Pod) that connects to the Controller and runs the actual build tasks.

---

### Q3. CI vs CD vs Continuous Deployment? **Must Know**
- **CI (Continuous Integration):** Developers frequently merge code → automated build + tests triggered.
- **Continuous Delivery:** Pipeline builds and packages automatically → human approval required to deploy to prod.
- **Continuous Deployment:** Every passing change is automatically deployed to prod with no human intervention.

---

### Q4. What is a Jenkinsfile? **Must Know**
A text file that stores the entire CI/CD pipeline as code. Checked into Git alongside app code — versioned, reviewed, and auditable.

---

### Q5. Declarative vs Scripted Pipeline?
- **Declarative:** Modern, recommended. Strict structure (`pipeline`, `agent`, `stages`, `steps`). Readable and easy to write.
- **Scripted:** Older, Groovy-based. Very flexible but can become complex and hard to maintain.

---

### Q6. How do you trigger a pipeline automatically on code push? **Must Know**
Use **Webhooks** — configure GitHub/GitLab to send an HTTP POST to Jenkins on push/merge. Jenkins receives it and triggers the job instantly. (SCM polling is an older, less efficient alternative.)

---

### Q7. What is a Shared Library in Jenkins?
A collection of reusable Groovy scripts pulled into Jenkinsfiles. If 50 microservices all build Docker images the same way, write it once in a Shared Library — DRY principle, consistent pipelines.

---

### Q8. How do you securely use credentials in a Jenkinsfile? **Must Know**
Never hardcode secrets. Store them in **Jenkins Credentials Manager**. In Declarative Pipeline, use the `credentials()` helper in the `environment` block to inject them as environment variables at runtime.

---

### Q9. What is the `post` block in a Declarative Pipeline?
Defines actions at the end of a pipeline/stage based on outcome:
- `always` — runs no matter what (e.g., clean workspace)
- `success` — runs if build passes (e.g., Slack notification)
- `failure` — runs if build fails (e.g., email alert)

---

### Q10. How do you run tasks in parallel in Jenkins?
Use the `parallel` block inside a stage. Run unit tests, linting, and security scans simultaneously instead of sequentially — drastically cuts build time.

---

### Q11. What is `JENKINS_HOME` and why is it critical?
The directory where Jenkins stores all configs, logs, job histories, plugins, and secrets. Losing it = losing everything. Back it up regularly.

---

### Q12. Why should builds NOT run on the Jenkins Controller?
Heavy builds consume CPU, RAM, and disk on the Controller. If it runs out of resources, the entire Jenkins UI crashes and the CI/CD process halts. All builds must run on Agents.

---

### Scenario: Build is stuck in "Pending/Queued" forever **Must Know**
1. Are all executors on the target agent busy?
2. Does the `agent { label 'XYZ' }` in Jenkinsfile match labels on online nodes?
3. Is the target agent offline/disconnected?

---

### Scenario: Pipeline failing with "No space left on device"
**Immediate fix:** Delete old workspaces, clear dangling Docker images on the agent.

**Prevention:**
- Configure **Discard Old Builds** (keep last N builds)
- Add `cleanWs()` in `post { always { ... } }` block to auto-wipe workspace after every run

---

### Scenario: Pass a `.jar` file from Stage 1 to Stage 2 on different agents
Use `stash` (saves file to Controller temporarily) in Stage 1, then `unstash` in Stage 2 to retrieve it on the new agent's filesystem.

---

### Scenario: Jenkins Controller is destroyed — how to restore?
1. Spin up a new server, install the **same Jenkins version**
2. Stop the Jenkins service
3. Replace the new empty `JENKINS_HOME` with the backed-up one
4. Fix ownership permissions (`chown -R jenkins:jenkins JENKINS_HOME`)
5. Start the service — everything is restored

---

### Scenario: Build environment must be clean and identical every run **Must Know**
Use Docker agent in Jenkinsfile: `agent { docker { image 'node:18-alpine' } }`. Jenkins pulls the image, runs build steps inside the container, and destroys it after. Fully isolated, reproducible environment.

---

## 5. ArgoCD & GitOps

### Q1. What is ArgoCD? **Must Know**
A declarative, GitOps-based Continuous Delivery tool built for Kubernetes. ArgoCD lives inside the cluster and continuously syncs it to match the desired state defined in a Git repository.

---

### Q2. Jenkins (Push) vs ArgoCD (Pull)? **Must Know**
- **Jenkins (Push):** CI pipeline builds the image and pushes deployment into the cluster from outside using `kubectl`/Helm.
- **ArgoCD (Pull):** Installed inside the cluster. Detects changes in Git and pulls them into the cluster to match Git state.

---

### Q3. What is Sync and Drift in ArgoCD? **Must Know**
- **Drift:** Live cluster state differs from the Git-defined state (e.g., someone ran `kubectl edit` manually).
- **Sync:** ArgoCD eliminates drift by making the cluster match Git exactly.

---

### Q4. ArgoCD Architecture components?

| Component | Role |
|---|---|
| API Server | Handles requests from Web UI, CLI, CI systems |
| Repository Server | Caches Git repos, generates final K8s manifests (Helm/Kustomize) |
| Application Controller | The "brain" — compares live vs desired state, triggers syncs |
| Redis | Caches cluster state for performance |

---

### Q5. What are ArgoCD Applications and Projects? **Must Know**
- **Application:** Links a Git repo path (source) to a K8s cluster+namespace (destination).
- **AppProject:** Groups Applications with enforced boundaries — restricts which Git repos, which namespaces, and which K8s resource types can be used. Essential for multi-team environments.

---

### Q6. Manual Sync vs Auto-Sync?
- **Manual Sync:** ArgoCD detects Git changes but waits for human approval to apply.
- **Auto-Sync:** Applies Git changes automatically. Can be enhanced with:
  - **Prune:** Auto-deletes K8s resources removed from Git
  - **Self-Heal:** Reverts any manual cluster changes back to Git state

---

### Q7. How do you perform a rollback in ArgoCD? **Must Know**
Best practice: **revert the Git commit** — ArgoCD detects the revert and syncs the cluster back.
Emergency: Use `argocd app rollback <app-name>` from CLI or use the UI to roll back to a previous sync revision.

---

### Q8. How does ArgoCD handle Helm, Kustomize, and plain manifests?
ArgoCD has native support for all three. It auto-detects:
- `Chart.yaml` → renders as Helm chart
- `kustomization.yaml` → builds Kustomize overlay
- Raw `.yaml` files → applies directly

---

### Q9. How do you handle Kubernetes Secrets in GitOps without committing them in plain text? **Must Know**
Never commit plain-text secrets to Git. Use:
- **Bitnami Sealed Secrets:** Encrypt secrets locally, commit the encrypted version. A controller in the cluster decrypts them.
- **External Secrets Operator:** Fetches secrets from AWS Secrets Manager / Vault at runtime and injects them as K8s Secrets.

---

### Q10. How do you restrict teams from deploying to wrong namespaces?
Use **ArgoCD AppProjects** to enforce that Team A can only deploy to `team-a-dev` namespace from their specific Git repo, and cannot create `ClusterRoles`. Combined with SSO-backed RBAC for full isolation.

---

## 6. AWS

### Q1. What is the difference between EC2, ECS, EKS, and Lambda? **Must Know**
| Service | Type | Use Case |
|---|---|---|
| EC2 | Virtual Machine | Full control over OS and software |
| ECS | Container Orchestration (AWS-native) | Run Docker containers without managing K8s |
| EKS | Managed Kubernetes | Run K8s clusters without managing the control plane |
| Lambda | Serverless Functions | Event-driven code, no servers to manage |

---

### Q2. What is a VPC and why does it matter? **Must Know**
Virtual Private Cloud — your isolated network in AWS. Everything (EC2, RDS, EKS) runs inside a VPC. You control subnets (public/private), route tables, Internet Gateways, NAT Gateways, and Security Groups.

---

### Q3. Public Subnet vs Private Subnet?
- **Public Subnet:** Has a route to the Internet Gateway. Resources can have public IPs. Use for ALBs, bastion hosts.
- **Private Subnet:** No direct internet access. Use for EC2 app servers, RDS databases. Outbound internet via NAT Gateway.

---

### Q4. What is a Security Group vs NACL? **Must Know**
| | Security Group | NACL |
|---|---|---|
| Level | Instance level | Subnet level |
| State | Stateful (return traffic auto-allowed) | Stateless (must allow inbound + outbound explicitly) |
| Rules | Allow only | Allow and Deny |

---

### Q5. What is AWS IAM and what are best practices? **Must Know**
Identity and Access Management — controls who can do what in AWS.

**Best practices:**
- Principle of Least Privilege — grant minimum required permissions
- Never use root account for day-to-day operations
- Enable MFA on all accounts
- Use IAM Roles for EC2/ECS/Lambda (not access keys)
- Rotate access keys regularly

---

### Q6. What is S3 and what are its storage classes?
Simple Storage Service — object storage. Key storage classes:
- **Standard** — frequent access
- **Standard-IA** — infrequent access (cheaper)
- **Glacier** — archival (cheapest, retrieval takes minutes-hours)
- **Intelligent-Tiering** — auto-moves objects based on access patterns

---

### Q7. What is CloudWatch? **Must Know**
AWS monitoring service. Collects metrics (CPU, memory, network), logs, and events from AWS resources. You set Alarms to trigger SNS notifications or Auto Scaling actions when thresholds are breached.

---

### Q8. What is AWS Lambda and what is a Cold Start? **Must Know**
Lambda runs code in response to events without provisioning servers. A **Cold Start** is the latency added when Lambda initializes a new execution environment after a period of inactivity. Mitigate with **Provisioned Concurrency** (keeps instances pre-warmed).

**Key Lambda limits:**
- Max execution time: **15 minutes**
- Memory: 128 MB – 10 GB
- Deployment package: 50 MB zipped / 250 MB unzipped / 10 GB via container image

---

### Q9. What is RDS Multi-AZ vs Read Replica? **Must Know**
| | Multi-AZ | Read Replica |
|---|---|---|
| Purpose | High availability / failover | Read scaling / performance |
| Replication | Synchronous | Asynchronous |
| Failover | Automatic (CNAME flips) | Manual promotion |
| Used for | Production HA | Offloading read queries |

---

### Q10. What is Auto Scaling Group (ASG)?
Automatically adds or removes EC2 instances based on demand (CPU, request count, schedule). Works with an ALB to distribute traffic across healthy instances.

---

### Q11. What is the difference between ALB, NLB, and CLB? **Must Know**
| | ALB | NLB | CLB |
|---|---|---|---|
| Layer | Layer 7 (HTTP/HTTPS) | Layer 4 (TCP/UDP) | Layer 4/7 (legacy) |
| Routing | Path/host-based | IP/port-based | Basic |
| Use case | Web apps, microservices | High-performance, low latency | Legacy only |

---

### Q12. What is AWS Security Hub?
A CSPM service that centralizes security findings from GuardDuty, Inspector, Macie, and third-party tools into one dashboard. Runs automated compliance checks against CIS, PCI-DSS, and AWS best practices benchmarks.

---

### Scenario: Entire AZ goes down — what happens to your EKS + RDS setup? **Must Know**
If correctly architected:
- **EKS Node Group** spans multiple AZs — ALB routes traffic to surviving AZ nodes
- **RDS Multi-AZ** — automatically fails over to standby replica in the healthy AZ
- **ALB** with cross-zone load balancing enabled — reroutes instantly

If you only used one AZ: total outage. This is why multi-AZ is non-negotiable for prod.

---

### Scenario: EKS pods getting "Too many connections" errors after autoscaling
ALB + HPA scaled pods perfectly but RDS was overwhelmed — every pod opened a new direct DB connection. **Fix:** Add **Amazon RDS Proxy** between EKS and RDS. It pools and reuses connections, protecting the database from connection floods.

---

### Scenario: ASG not launching new instances during traffic spikes
Check in AWS Console → EC2 → Auto Scaling → Activity History for exact error. Common causes:
- Hit AWS vCPU service quota limit
- Subnets ran out of private IP addresses
- AMI in Launch Template was deleted

---

### Scenario: AWS bill spiked 3x overnight **Must Know**
1. Open **Cost Explorer**, group by Service and Region
2. If EC2 in unknown region → **assume compromised IAM key** → immediately rotate keys, terminate rogue instances, check CloudTrail
3. If same region → check ASG activity history for thrashing (bad health check cycling instances)
4. Check for S3/data transfer loops causing massive network charges

---

### Scenario: Deployment succeeded but traffic still goes to old version
Debug path:
1. **CDN/browser cache** — hard refresh, check CloudFront TTL
2. **ALB Target Group** — is it routing to the right targets?
3. **K8s Service selector** (`kubectl get endpoints`) — if Deployment updated Pod labels but Service selector didn't change, it still routes to old pods
4. **ArgoCD sync** — is the application truly Synced or OutOfSync?

---

## 7. Linux & System Administration

### Q1. CPU bottleneck vs I/O bottleneck — how to identify each? **Must Know**
- **CPU bottleneck:** Processor is maxed out running calculations. In `top`/`htop`: `us` (user) or `sy` (system) near 100%.
- **I/O bottleneck:** CPU is sitting idle waiting for disk/network. In `top`: `wa` (iowait) percentage is high. Use `iostat` to check disk queue lengths.

---

### Q2. Why do systems fail even when CPU and memory look fine?
- **Thread/Connection Pool Exhaustion** — app ran out of worker threads, new requests queue and time out
- **File Descriptor / Port Exhaustion** — OS ran out of open file slots or ephemeral ports (`ulimit`)
- **Deadlocks** — two DB queries waiting on each other forever
- **Garbage Collection Pauses** — JVM freezes the app to clear memory; CPU/RAM look fine but app hangs

---

### Q3. Common Linux commands every DevOps engineer must know **Must Know**

| Command | Use |
|---|---|
| `top` / `htop` | CPU, memory, process monitoring |
| `df -h` | Disk space usage |
| `du -sh *` | Directory size breakdown |
| `free -m` | RAM usage |
| `netstat -tulpn` / `ss -tulpn` | Open ports and listening services |
| `ps aux` | Running processes |
| `journalctl -u <service>` | Systemd service logs |
| `systemctl status/start/stop/restart` | Service management |
| `lsof -i :<port>` | Which process is using a port |
| `curl -v <url>` | Test HTTP connectivity |
| `iostat` | Disk I/O stats |
| `strace` | System call tracing for debugging |

---

### Q4. What is the difference between a process and a thread?
- **Process:** Independent program with its own memory space.
- **Thread:** A unit of execution within a process — shares the same memory space. Threads are lighter but a crash can affect all threads in the process.

---

### Q5. What is `ulimit` and why does it matter in production?
Controls resource limits per process/user — open file descriptors, max processes, stack size. In production, if an app opens too many connections or files and hits the `ulimit`, it starts rejecting new connections even if the server has plenty of RAM and CPU.

---

### Q6. What is `cron` and how does it work?
A time-based job scheduler in Linux. Jobs are defined in `crontab` with the format:
```
* * * * * command
│ │ │ │ │
│ │ │ │ └── Day of week (0-7)
│ │ │ └──── Month (1-12)
│ │ └────── Day of month (1-31)
│ └──────── Hour (0-23)
└────────── Minute (0-59)
```

---

### Q7. How do you check what is consuming disk space?
```bash
df -h          # See which filesystem is full
du -sh /*      # Find which top-level directory is biggest
du -sh /var/*  # Drill down further
```

---

### Q8. What is SSH and how does key-based authentication work? **Must Know**
SSH (Secure Shell) is an encrypted protocol for remote server access. Key-based auth:
1. Generate key pair: `ssh-keygen` → creates private key (keep secret) + public key
2. Copy public key to server: `ssh-copy-id user@server` → appended to `~/.ssh/authorized_keys`
3. Connect: `ssh -i private_key user@server`

No password needed — much more secure than password auth.

---

## 8. Observability & Monitoring

### Q1. What is the difference between Metrics, Logs, and Traces? **Must Know**

| | Metrics | Logs | Traces |
|---|---|---|---|
| What | Aggregated numbers over time | Discrete timestamped text events | End-to-end journey of a single request |
| Use | Detect a problem exists (CPU 90%, error rate 5%) | Find the exact root cause (exception on line 42) | Isolate which service in a chain is slow |
| Tools | Prometheus, CloudWatch, Datadog | ELK Stack, CloudWatch Logs, Loki | Jaeger, Zipkin, Datadog APM |
| Start here when | Alert fires | You know which service, need the exact error | You know there's latency, need to find where |

---

### Q2. What is Prometheus and Grafana? **Must Know**
- **Prometheus:** Open-source metrics collection and alerting. Scrapes metrics from targets (apps, nodes). Stores time-series data. Uses PromQL for queries.
- **Grafana:** Visualization layer. Connects to Prometheus (and other sources) to build dashboards and charts.

---

### Q3. What is an SLI, SLO, and SLA?
- **SLI (Service Level Indicator):** A specific measurable metric (e.g., request success rate, latency p99)
- **SLO (Service Level Objective):** A target value for the SLI (e.g., 99.9% requests succeed)
- **SLA (Service Level Agreement):** A formal contract with a customer — consequences (credits, penalties) if SLO is breached

---

### Q4. What does "Watermelon Monitoring" mean?
Monitoring that is green on the outside (infrastructure metrics look fine) but red on the inside (users are experiencing issues). Happens when SLIs measure the wrong things — infrastructure metrics (CPU, RAM) are fine but application metrics (DB lock waits, thread pools, GC pauses) are broken.

---

### Q5. What tools make up the ELK Stack?
- **Elasticsearch:** Stores and indexes log data for fast searching
- **Logstash:** Collects, parses, and transforms logs from various sources
- **Kibana:** Web UI for searching and visualizing logs

Often replaced with **EFK Stack** (Elasticsearch + Fluentd + Kibana) in Kubernetes environments.

---

### Q6. How do you automate alert remediation? **Must Know**
Use **EventBridge** (AWS) or **Alertmanager** (Prometheus) to route alerts to Lambda functions or runbooks that automatically remediate:
- S3 bucket becomes public → Lambda re-applies private ACL
- Disk hits 80% → trigger cleanup script
- Security Hub finding → SSM Automation document remediates it

---

## 9. CI/CD General

### Q1. What is the difference between Continuous Delivery and Continuous Deployment? **Must Know**
- **Continuous Delivery:** Pipeline automates build + test + package. Deployment to prod requires a human approval click.
- **Continuous Deployment:** Every change that passes tests is automatically deployed to prod — zero human intervention.

---

### Q2. What is Blue/Green Deployment? **Must Know**
Maintain two identical environments (Blue = current prod, Green = new version). Deploy to Green, test it, then shift traffic from Blue to Green via load balancer. Blue stays alive for instant rollback. Zero downtime, lowest risk.

---

### Q3. What is a Canary Deployment?
Gradually shift a small percentage of traffic (e.g., 5%) to the new version. Monitor for errors. If healthy, increase percentage until 100%. If not, roll back only the canary. Much safer than all-at-once deployment for high-traffic systems.

---

### Q4. Why do deployments succeed but users still see errors?
- **Database schema mismatch** — new code expects a column that hasn't been migrated yet
- **CDN caching** — edge is serving old `index.html` even though backend is updated
- **Shallow health checks** — container returns 200 on `/health` but is missing a critical env var needed for real transactions

---

### Q5. How do you optimize a slow CI/CD pipeline (40+ minutes)? **Must Know**
1. **Dependency caching** — cache `node_modules`/`.m2` between runs (biggest win)
2. **Docker layer optimization** — copy `package.json` + install before copying source code
3. **Parallel execution** — run unit tests, linting, and security scans concurrently
4. **Multi-stage builds** — keep final image small so push to ECR/registry is fast
5. **Incremental builds** — only build/test changed modules (monorepo tools like Nx, Turborepo)

---

### Q6. How do you decide: rollback vs hotfix during an outage? **Must Know**
- **Rollback:** When the root cause is unclear, downtime is active, and reverting is safe (no DB migration already applied).
- **Hotfix (roll forward):** When the bug is isolated and trivial to fix, OR when rolling back is impossible (DB schema already migrated and can't be downgraded safely).

---

### Q7. A secret was accidentally committed to GitHub — what do you do? **Must Know**
1. **Immediately revoke the secret** — deactivate the key/token in the provider (AWS IAM, GitHub, DB)
2. **Generate a new secret** and update production
3. **Rewrite Git history** — use `git filter-repo` or BFG Repo-Cleaner to scrub it from all commits, then force-push
4. **Audit for misuse** — check CloudTrail / access logs for unauthorized usage
5. **Prevent recurrence** — add `git-secrets` or TruffleHog pre-commit hook + CI pipeline scan

---

### Q8. What is GitOps?
A practice where Git is the single source of truth for both application code and infrastructure configuration. Any change to the system is made through a Git commit. Tools like ArgoCD or Flux watch Git and automatically apply changes to the cluster — making deployments auditable, reproducible, and reversible.

---

## 10. Behavioral & Soft Skills

### Q1. Tell me about a time you caused a production outage. **Must Know**
**What interviewers want:** Accountability (don't blame tools or teammates), a clear timeline of how you fixed it, communication with stakeholders during the incident, and a post-mortem that prevented recurrence.

**Example structure:** "I misconfigured X → it caused Y → I detected it by Z → I fixed it by doing A → I prevented recurrence by implementing B."

---

### Q2. How do you handle a situation with multiple high-priority tasks at the same time?
**What interviewers want:** A logical prioritization framework — business impact, urgency, and SLA risk. Show that you communicate proactively with your manager instead of silently burning out or guessing.

---

### Q3. Tell me about a time you strongly disagreed with a technical decision.
**What interviewers want:** Data-driven disagreement, not emotional arguing. You listened, brought evidence or a POC, and ultimately committed to the final decision even if it wasn't yours ("disagree and commit").

---

### Q4. How do you work with a difficult team member?
**What interviewers want:** Empathy first — did you pull them aside to understand if they had a blocker? Direct communication before escalating to management. Leadership and collaboration over complaint.

---

### Q5. Tell me about a new technology you had to learn quickly.
**What interviewers want:** Your learning framework — hands-on labs, reading docs, building small projects, finding mentors. Show that you are self-driven and not dependent on formal training.

---

### Q6. Describe a situation where monitoring showed everything green but users complained.
**What interviewers want:** Critical thinking. You understand the difference between infrastructure metrics and application health. You bypass the dashboard and check APM tracing, DB wait times, and real user experience data.

---

### Q7. How do you handle on-call incidents at 3am?
**What interviewers want:** A calm, methodical process. Have a runbook. Stabilize first (stop the bleeding), then investigate root cause. Don't start debugging in prod while users are down. Communicate the ETA to stakeholders. Document everything for the post-mortem.

---

*Sources: Common questions gathered from Glassdoor, LinkedIn interview reports, GitHub DevOps interview repos, and tech community discussions in Pakistani tech companies.*

---

## 11. Linux Administration — Senior Level

> Focus: how the OS actually works, not command syntax.

---

### Q1. What happens between pressing Enter on `ssh user@server` and getting a shell prompt? **Must Know**
1. DNS resolves the hostname to an IP
2. TCP 3-way handshake on port 22
3. SSH protocol negotiates encryption algorithm (key exchange)
4. Server sends its host key — client verifies it against `~/.ssh/known_hosts` (TOFU model)
5. Client authenticates (public key or password)
6. Server spawns a shell process (via `sshd` forking), attaches a PTY if interactive
7. Your shell prompt appears

Interviewers ask this to see if you understand networking, encryption, and process spawning together.

---

### Q2. What is the Linux boot process? **Must Know**
1. **BIOS/UEFI** — hardware POST, finds bootable device
2. **Bootloader (GRUB2)** — loads the kernel image into memory
3. **Kernel** — initializes hardware, mounts root filesystem (initramfs first, then real root)
4. **PID 1 (systemd)** — the first process, becomes the parent of everything. Reads unit files, starts services in dependency order
5. **Login/Shell** — systemd starts the display manager or getty for login

---

### Q3. What is a zombie process and how is it different from an orphan process? **Must Know**
- **Zombie:** A process that has finished executing but its parent hasn't called `wait()` to collect its exit code. It stays in the process table as a `<defunct>` entry consuming a PID slot. Fix: fix the parent code or kill the parent (init/systemd will reap it).
- **Orphan:** A process whose parent died before it did. It gets re-parented to PID 1 (systemd/init), which will collect its exit code properly. Orphans are not harmful.

---

### Q4. What is the OOM Killer and when does Linux invoke it? **Must Know**
When the system runs out of physical RAM and swap, the kernel's Out-Of-Memory Killer selects a process to kill based on an `oom_score` (how much memory it uses, how long it's been running, etc.) and sends it `SIGKILL`. In Kubernetes, hitting a container's memory limit triggers a container-level OOM kill (`OOMKilled` exit code 137). You can see OOM events in `dmesg` or `journalctl -k`.

---

### Q5. Explain Linux file permissions and what `chmod 755` means. **Must Know**
Permissions are three groups of three bits: Owner | Group | Others. Each group has Read (4), Write (2), Execute (1).

`755` = Owner: 7 (rwx), Group: 5 (r-x), Others: 5 (r-x)

Special bits matter too:
- **setuid (4000):** Execute file as the file owner (e.g., `passwd` runs as root even for normal users)
- **setgid (2000):** Execute as the file's group / new files in a directory inherit the directory's group
- **sticky bit (1000):** Only file owner can delete a file in a shared directory (e.g., `/tmp`)

---

### Q6. What is the difference between a hard link and a soft (symbolic) link? **Must Know**
- **Hard link:** Another directory entry pointing to the same inode (same data on disk). Deleting the original file does not remove the data — the data is removed only when all hard links to the inode are gone. Cannot span filesystems.
- **Soft link (symlink):** A pointer to a file path. If the target is deleted, the symlink breaks. Can span filesystems and point to directories.

---

### Q7. What happens when you run out of inodes even though disk space is free?
Every file/directory occupies one inode regardless of its size. If you have millions of tiny files (e.g., mail queues, session files, cache), you exhaust inodes while the disk still shows free space. `df -i` shows inode usage. Fix: delete the small files or reformat the filesystem with more inodes. This is a real production issue and a classic senior interview trap.

---

### Q8. What is the difference between a process's virtual memory, RSS, and swap usage? **Must Know**
- **VSZ (Virtual Size):** Total virtual address space the process has mapped — includes shared libraries, memory-mapped files, and unallocated pages. Usually much larger than actual usage.
- **RSS (Resident Set Size):** Physical RAM the process is actually using right now.
- **Swap:** Pages that were in RAM but got moved to disk because RAM was under pressure.

A process with high VSZ but low RSS is fine. A process with high RSS that keeps growing has a memory leak.

---

### Q9. How does `systemd` manage service dependencies and why does it matter for production? **Must Know**
`systemd` unit files declare dependencies using `After=`, `Requires=`, `Wants=`, and `Before=` directives. `Requires=` is a hard dependency (failure stops the dependent unit). `Wants=` is soft (failure is tolerated). `After=` controls ordering without enforcing dependency.

In production this matters because if a service starts before its database dependency is ready (wrong `After=`), it crashes on boot and enters a restart loop — `journalctl -u <service>` and `systemctl status <service>` are how you debug this.

---

### Q10. What is a race condition at the OS level and how do you prevent it?
A race condition occurs when two processes or threads access and modify a shared resource concurrently without proper synchronization, and the outcome depends on which runs first. At the OS level, tools to prevent it:
- **Mutex/Lock:** Only one process holds the lock at a time
- **Semaphore:** Limits concurrent access to N processes
- **File locks (`flock`):** Used by processes like package managers (apt, yum) to prevent concurrent runs — this is why you get "dpkg is locked" errors

---

### Q11. Explain the TCP 3-way handshake and what a SYN flood attack exploits. **Must Know**
1. Client sends **SYN** — "I want to connect"
2. Server responds **SYN-ACK** — "OK, I'm ready" — and allocates a half-open connection entry
3. Client sends **ACK** — connection established

**SYN flood:** Attacker sends thousands of SYN packets with spoofed source IPs. Server sends SYN-ACKs that never get acknowledged, filling the half-open connection table until the server can't accept new legitimate connections. Mitigated with **SYN cookies** (server doesn't allocate state until the final ACK arrives) and rate limiting.

---

### Q12. What is `iptables` / `nftables` and how does Linux filter network traffic? **Must Know**
`iptables` is the userspace tool to configure the Linux kernel's `netfilter` packet filtering framework. Traffic passes through **chains** (INPUT, OUTPUT, FORWARD, PREROUTING, POSTROUTING) and rules are evaluated top-to-bottom. First matching rule wins.

In Kubernetes, `kube-proxy` heavily uses iptables rules to implement Service ClusterIP and NodePort routing. If you ever delete `kube-proxy` accidentally, Services stop working because the iptables rules are gone.

---

### Q13. What is `strace` and when would you use it in production? **Must Know**
`strace` traces every system call a process makes (file opens, network calls, memory allocations). Use it when:
- An app silently fails with no useful logs
- You need to see which files/sockets it's trying to access
- Diagnosing permission errors (`EACCES`, `ENOENT`)

**Warning:** `strace` adds significant overhead — use on a non-critical instance or with very short trace windows in production.

---

### Q14. What is `mmap` and why do databases use it?
`mmap` maps a file directly into a process's virtual address space. Instead of `read()`/`write()` system calls, the process accesses file data as if it were memory — the kernel handles paging it in/out. Databases (PostgreSQL, MongoDB, RocksDB) use it because it bypasses the userspace buffer, reduces copies, and lets the OS manage caching efficiently.

---

### Q15. A production server is suddenly unreachable but you can still ping it. What do you check? **Must Know**
Ping working = network layer fine. Problem is higher up:
1. **Port blocked?** — Check `iptables -L` or Security Group rules
2. **Service down?** — `systemctl status <service>` / `ss -tlnp` to see if the port is listening
3. **File descriptor exhaustion?** — App accepted connections but hit `ulimit -n` (max open files)
4. **Connection queue full?** — `netstat -s | grep overflow` — kernel's backlog is full, new connections are silently dropped
5. **Disk full?** — App can't write logs/pidfiles, appears stuck

---

### Q16. What is the difference between `kill -9`, `kill -15`, and `kill -1`? **Must Know**
- `kill -15` (SIGTERM): Graceful shutdown request. The process can catch this, clean up, and exit.
- `kill -9` (SIGKILL): Immediate, unconditional termination. Cannot be caught or ignored by the process — kernel forcibly removes it. Use only as a last resort (data may be corrupted).
- `kill -1` (SIGHUP): Originally meant "terminal hangup." For daemons (nginx, sshd), it means "reload your config file without restarting."

---

### Q17. What is dirty memory and how does it cause production issues?
**Dirty pages** are memory pages that have been modified but not yet written back to disk. Linux batches disk writes for performance. If dirty memory grows too large, the kernel forces a flush (`pdflush`/`kworker`), which can cause sudden I/O spikes. In high-write databases or log-heavy apps, misconfigured `vm.dirty_ratio` and `vm.dirty_background_ratio` sysctl settings cause periodic latency spikes — you'll see `iowait` spike in `top` every few seconds even when the workload is constant.

---

### Q18. How does Linux decide which CPU core to run a process on, and how can you override it? **Must Know**
The kernel scheduler assigns processes to cores based on load balancing and cache affinity. You can override with:
- `taskset -c 0,1 <command>` — pin a process to specific CPU cores (CPU affinity)
- `numactl` — on multi-socket servers, bind a process to a specific NUMA node to avoid cross-socket memory latency

Used in high-performance databases and networking applications to prevent cache thrashing.

---

## 12. Prometheus & Grafana

---

### Q1. How does Prometheus collect metrics — push or pull? **Must Know**
**Pull model.** Prometheus scrapes metrics by sending HTTP GET requests to a `/metrics` endpoint on each target at a configured interval. This means your app must expose a `/metrics` endpoint in Prometheus exposition format.

Exception: **Pushgateway** — used for short-lived batch jobs that finish before Prometheus can scrape them. The job pushes metrics to the Pushgateway, and Prometheus scrapes that.

---

### Q2. What is a Prometheus metric type? Explain all four. **Must Know**

| Type | What it tracks | Example |
|---|---|---|
| **Counter** | Monotonically increasing number (never goes down) | Total HTTP requests, total errors |
| **Gauge** | Value that goes up and down freely | Current memory usage, active connections |
| **Histogram** | Samples observations into configurable buckets | Request latency distribution (p50, p95, p99) |
| **Summary** | Like histogram but calculates quantiles on client side | Request duration quantiles |

**Key rule:** Use a Counter for things that only increase. Use a Gauge for current state. Use Histogram when you need percentile latency (p99).

---

### Q3. What is PromQL and how do you write a basic query? **Must Know**
PromQL is Prometheus's query language. Key functions:

- `rate(http_requests_total[5m])` — per-second rate of increase over 5 minutes (use on Counters)
- `increase(http_errors_total[1h])` — total increase over 1 hour
- `histogram_quantile(0.99, rate(http_request_duration_seconds_bucket[5m]))` — p99 latency
- `sum by (pod) (container_memory_usage_bytes)` — memory per pod
- `up == 0` — which targets are down

---

### Q4. What is the Alertmanager and how does it work with Prometheus? **Must Know**
Prometheus evaluates alerting rules (PromQL expressions). When a condition is met, it sends a **firing** alert to **Alertmanager**. Alertmanager then:
- **Groups** related alerts to prevent alert storms (e.g., one cluster failure = 1 alert, not 500)
- **Silences** alerts during maintenance windows
- **Routes** alerts to the right receiver (Slack, PagerDuty, email, webhook)
- **Deduplicates** the same alert from multiple Prometheus instances in HA setups

---

### Q5. What is a ServiceMonitor in Kubernetes and why does it matter?
When using **kube-prometheus-stack** (Prometheus Operator), you don't edit `prometheus.yml` directly. Instead, you create a `ServiceMonitor` CRD that tells the Prometheus Operator which Kubernetes Services to scrape, on which port and path. The Operator watches for ServiceMonitors and auto-generates the Prometheus scrape config. This is how Prometheus scales — each team owns their ServiceMonitor without touching the central config.

---

### Q6. How do you prevent Prometheus from running out of disk space in production? **Must Know**
- Set `--storage.tsdb.retention.time` (e.g., `15d`) — data older than this is deleted
- Set `--storage.tsdb.retention.size` (e.g., `50GB`) — delete oldest blocks when size is exceeded
- Use **Thanos** or **Cortex** for long-term storage — they ship Prometheus data to S3/GCS, letting you set short retention locally
- Monitor the Prometheus disk itself — add an alert for `node_filesystem_avail_bytes` on the Prometheus PV

---

### Q7. What is the difference between Grafana Dashboards and Grafana Alerts?
- **Dashboards:** Visual panels (graphs, tables, heatmaps) for humans to read. Good for exploring trends and investigating incidents.
- **Grafana Alerts:** Rules that evaluate queries on a schedule. When a threshold is breached, Grafana sends notifications via contact points (Slack, PagerDuty, email). In modern setups, it's better to keep alerting in Alertmanager (closer to the data source) and use Grafana for visualization only.

---

### Q8. What is a Grafana data source and what are the common ones?
A data source is a connection Grafana uses to query data. Common ones:
- **Prometheus** — metrics (most common in K8s)
- **Loki** — logs (Grafana's own log aggregation tool)
- **Elasticsearch** — logs and search
- **CloudWatch** — AWS metrics
- **InfluxDB** — time-series data
- **Jaeger/Tempo** — distributed traces

---

### Q9. Your Prometheus is showing a target as `DOWN`. How do you debug it? **Must Know**
1. Go to Prometheus UI → **Status → Targets** — check the error message (connection refused, 404, timeout)
2. `curl http://<target-ip>:<port>/metrics` from the Prometheus pod — confirms network connectivity
3. Check **NetworkPolicy** — is there a policy blocking Prometheus from reaching the target namespace?
4. Check the target pod's **container port** vs the ServiceMonitor **port name** — they must match exactly
5. If auth is required, check if Prometheus has the right `bearerTokenFile` or `basicAuth` config

---

### Q10. What is recording rules in Prometheus and why use them? **Must Know**
Recording rules pre-compute expensive PromQL queries and store the result as a new time-series. Instead of computing `histogram_quantile(0.99, rate(http_request_duration_seconds_bucket[5m]))` on every dashboard load, you store the result as `job:request_latency_p99:rate5m`. Dashboard loads become instant. Essential when you have many Grafana users hitting the same expensive queries.

---

## 13. Ansible

---

### Q1. What is Ansible and how is it different from Terraform? **Must Know**
Ansible is an **agentless configuration management and automation tool**. It connects to servers over SSH and runs tasks defined in YAML **Playbooks**. No agent needs to be installed on target servers.

**Ansible vs Terraform:**
- Terraform **provisions** infrastructure (creates servers, networks, databases)
- Ansible **configures** what runs on those servers (installs packages, deploys code, edits config files)
- They complement each other: Terraform builds the server, Ansible configures it

---

### Q2. What is a Playbook, a Role, and an Inventory? **Must Know**
- **Inventory:** A list of target servers (IPs or hostnames), grouped by role (webservers, dbservers). Can be static (a file) or dynamic (queried from AWS, Azure, etc.)
- **Playbook:** A YAML file that maps host groups from the inventory to a set of tasks to run on them
- **Role:** A reusable, structured way to organize playbooks — has folders for `tasks/`, `handlers/`, `templates/`, `vars/`, `defaults/`. A role is like a Terraform module — write once, use everywhere

---

### Q3. What is idempotency and why is it critical in Ansible? **Must Know**
Idempotency means running a playbook 10 times produces the same result as running it once. Ansible modules are designed to be idempotent — the `apt` module only installs a package if it's not already installed. The `copy` module only copies a file if the content changed.

This is critical because Ansible is run repeatedly (in CI/CD, on-boarding new servers, drift remediation). Non-idempotent tasks cause unintended side effects on re-runs.

---

### Q4. What is the difference between a `task` and a `handler`?
- **Task:** Runs every time the playbook executes (e.g., "ensure nginx config is present")
- **Handler:** Only runs when **notified** by a task that changed something, and only runs **once** at the end of the play, even if notified multiple times. Classic use case: update nginx config → notify "restart nginx" → handler restarts nginx only if the config actually changed

---

### Q5. How does Ansible handle secrets? **Must Know**
Using **Ansible Vault** — encrypts sensitive variables, files, or entire playbooks with AES-256 encryption. The vault password is provided at runtime (`--ask-vault-pass` or `--vault-password-file`). You can commit encrypted vault files safely to Git. For production, vault passwords are stored in a secrets manager (HashiCorp Vault, AWS Secrets Manager) and retrieved by the CI/CD pipeline.

---

### Q6. What is the difference between `ansible` (ad-hoc) and `ansible-playbook`?
- **Ad-hoc (`ansible`):** One-off commands against inventory without a playbook file. Good for quick checks: `ansible webservers -m ping`, `ansible all -m shell -a "df -h"`
- **Playbook (`ansible-playbook`):** Runs a full YAML playbook — structured, version-controlled, idempotent automation. Used for production deployments

---

### Q7. What are Ansible facts and how are they used?
Facts are system information automatically gathered by Ansible at the start of a play (`gather_facts: yes`) — OS version, IP addresses, CPU count, memory, etc. You use them in conditionals: `when: ansible_os_family == "Debian"` — install `apt` packages on Ubuntu, `yum` on RHEL. Can be disabled with `gather_facts: no` for speed when not needed.

---

### Q8. What is the difference between `serial` and `forks` in Ansible?
- **`forks`:** How many hosts Ansible connects to in **parallel** (default: 5). Increase this for large inventories.
- **`serial`:** Controls rolling deployment — `serial: 1` means run the entire play on 1 host at a time before moving to the next. Essential for zero-downtime deployments: update nginx config on 1 server, verify it works, then move to the next.

---

### Q9. Scenario: You use Ansible to deploy an app to 100 servers. Halfway through, 20 servers fail. What happens and how do you retry? **Must Know**
Ansible stops processing failed hosts but continues on the rest (unless `any_errors_fatal: true`). After the run, it writes a **retry file** (`playbook.retry`) containing the failed hostnames. You re-run with `--limit @playbook.retry` to only target the failed servers. Fix the root cause first (e.g., network issue, disk space) before retrying.

---

### Q10. What is Ansible Galaxy?
A public repository of pre-built, community-contributed Ansible Roles. Like npm for Node.js or pip for Python — instead of writing a role to install and configure MySQL from scratch, you download one: `ansible-galaxy install geerlingguy.mysql`. In production, pin role versions to avoid breaking changes on re-runs.

---

## 14. AWS CodePipeline & AWS CI/CD

---

### Q1. What is AWS CodePipeline and what problem does it solve? **Must Know**
AWS CodePipeline is a fully managed CI/CD service that automates the build, test, and deploy stages of your release process. It integrates natively with AWS services (CodeBuild, CodeDeploy, ECR, ECS, Lambda, S3) and third-party tools (GitHub, Jenkins). No infrastructure to manage — you pay per pipeline per month.

---

### Q2. What are the core components of a CodePipeline? **Must Know**

| Component | Role |
|---|---|
| **Source Stage** | Monitors a source (GitHub, CodeCommit, S3, ECR) for changes and triggers the pipeline |
| **Build Stage** | Runs `CodeBuild` — compiles code, runs tests, builds Docker images, pushes to ECR |
| **Test Stage** | Optional — runs integration/E2E tests |
| **Deploy Stage** | Deploys to CodeDeploy (EC2/ECS/Lambda), Elastic Beanstalk, CloudFormation, or EKS |
| **Approval Action** | Manual approval gate before deploying to production |

---

### Q3. What is AWS CodeBuild and how does it differ from Jenkins? **Must Know**
CodeBuild is a fully managed build service — AWS provisions, runs, and terminates the build container for each run. No build servers to manage or scale. You define the build steps in a `buildspec.yml` file.

**vs Jenkins:**
- Jenkins: self-managed server, you maintain agents, plugins, updates, backups
- CodeBuild: serverless, scales automatically, pay per build minute, deeply integrated with AWS IAM/ECR/S3
- **Use Jenkins** when you need maximum flexibility, non-AWS tools, or complex pipeline logic
- **Use CodeBuild** when you're all-in on AWS and want zero maintenance

---

### Q4. What is `buildspec.yml` and what are its key phases? **Must Know**
`buildspec.yml` defines the build instructions for CodeBuild:

```yaml
version: 0.2
phases:
  install:      # Install dependencies, runtime versions
  pre_build:    # Login to ECR, run pre-checks
  build:        # Run tests, docker build
  post_build:   # Push image to ECR, generate artifacts
artifacts:      # Files to pass to the next pipeline stage
```

---

### Q5. What is AWS CodeDeploy and what deployment strategies does it support? **Must Know**
CodeDeploy automates deployments to EC2 instances, ECS, Lambda, and on-premises servers.

| Strategy | How it works |
|---|---|
| **All-at-once** | Deploy to all instances simultaneously. Fast but causes downtime. |
| **Rolling** | Deploy to a few instances at a time. Reduced capacity during deploy. |
| **Rolling with additional batch** | Adds extra capacity during deploy so full capacity is maintained. |
| **Blue/Green** | Deploys to new instances, shifts traffic via ALB, terminates old instances. Zero downtime. |
| **Canary (Lambda/ECS)** | Shift small % of traffic to new version, then 100% after validation period. |

---

### Q6. How does CodePipeline integrate with ECR and ECS for container deployments? **Must Know**
Typical ECS deployment flow:
1. **Source:** CodeCommit/GitHub push triggers pipeline
2. **Build (CodeBuild):** `docker build` → `docker push` to ECR → outputs `imagedefinitions.json` (maps container name to new ECR image tag)
3. **Deploy (CodeDeploy or ECS Deploy Action):** Reads `imagedefinitions.json` and updates the ECS Task Definition to use the new image → triggers a new ECS service deployment (rolling or blue/green)

---

### Q7. What is AWS CodeArtifact and when would you use it?
A fully managed artifact repository (like Nexus or JFrog Artifactory) for Maven, npm, pip, NuGet packages. Use it when:
- You want to cache public packages (npm, PyPI) internally for security and speed
- You need to publish private internal packages to share across teams
- You want to block developers from pulling packages with known vulnerabilities

---

### Q8. What is the difference between CodePipeline and GitHub Actions? **Must Know**
| | CodePipeline | GitHub Actions |
|---|---|---|
| Hosting | AWS-managed | GitHub-managed |
| Integration | Deep AWS native (ECR, ECS, Lambda, CloudFormation) | Strong GitHub integration, supports any cloud |
| Config format | Console/CDK/CloudFormation (JSON/YAML) | `.github/workflows/*.yml` |
| Build runners | CodeBuild (managed) or Jenkins | GitHub-hosted or self-hosted runners |
| Cost | Per pipeline/month + CodeBuild minutes | Free tier + per-minute for private repos |
| Best for | Full AWS shop, want native integrations | Multi-cloud, open source, strong community |

---

### Q9. How do you implement a manual approval gate in CodePipeline before production deploy? **Must Know**
Add an **Approval Action** stage between your staging deploy and production deploy. When the pipeline reaches it:
1. It pauses and sends an SNS notification (email/Slack)
2. A reviewer logs into the AWS Console (or uses CLI: `aws codepipeline put-approval-result`) to approve or reject
3. If approved within the timeout window (max 7 days), the pipeline continues to the production deploy stage
4. If rejected or timed out, the pipeline stops

---

### Q10. How does CodePipeline handle failed deployments and rollbacks?
- **CodeDeploy** (EC2): Automatically rolls back to the previous deployment revision if the deployment fails health checks. Configurable: rollback on deployment failure or alarm breach.
- **ECS Blue/Green via CodeDeploy:** If health checks on the Green target group fail, traffic is automatically shifted back to Blue and the Green task set is terminated.
- **Lambda canary/linear:** If a CloudWatch alarm fires during the traffic shift, CodeDeploy stops shifting and rolls back to the previous Lambda version automatically.

---

### Q11. Scenario: CodePipeline build succeeds but the ECS service is not updating to the new image. What do you check? **Must Know**
1. Check if `imagedefinitions.json` was correctly generated and passed as a build artifact — this file is what tells CodeDeploy which image to use
2. Check the ECS service deployment in the AWS Console — is it stuck waiting for tasks to become healthy?
3. Check the ECS task's CloudWatch logs — did the new container crash on startup?
4. Check if the ECS task execution role has permission to pull from ECR (`ecr:GetAuthorizationToken`, `ecr:BatchGetImage`)
5. Verify the task definition CPU/memory limits aren't too low for the new image
