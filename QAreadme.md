# DevOps Assignment 2 — All Questions Answered

**Student:** Awez SK | **GitHub:** Awezsk/Devopsassignment2 | **Docker Hub:** awezsk

---

## Part 1 — Docker Fundamentals

---

### Q1. What problem does Docker solve?

Before Docker, software worked on one machine but broke on another because every machine had different OS versions, libraries, and configurations — the classic **"works on my machine"** problem.

Docker solves this by packaging your application together with **everything it needs** (code, runtime, libraries, config) into one portable unit called a **container** that runs identically everywhere.

```
Without Docker:
  Dev Laptop        →   Test Server      →   Production
  Python 3.9        →   Python 3.6       →   Python 3.11
  Works ✅          →   Breaks ❌        →   Breaks ❌

With Docker:
  Dev Laptop        →   Test Server      →   Production
  [Container]       →   [Container]      →   [Container]
  Same image        →   Same image       →   Same image
  Works ✅          →   Works ✅         →   Works ✅
```

Docker is named after **shipping containers** — before standardized containers, every odd-shaped cargo needed custom handling. Standardized containers meant any ship, truck, or train could carry anything without caring what's inside. Docker does the same for software.

---

### Q2. Difference between VM and Container?

```
┌─────────────────────────────────────────────────────┐
│                    VM Architecture                   │
│                                                     │
│   ┌──────────┐  ┌──────────┐  ┌──────────┐         │
│   │  App A   │  │  App B   │  │  App C   │         │
│   ├──────────┤  ├──────────┤  ├──────────┤         │
│   │ Guest OS │  │ Guest OS │  │ Guest OS │         │
│   │ (Ubuntu) │  │(Windows) │  │ (CentOS) │         │
│   └──────────┘  └──────────┘  └──────────┘         │
│   ┌─────────────────────────────────────────┐       │
│   │            Hypervisor (VMware/VirtualBox)│       │
│   └─────────────────────────────────────────┘       │
│   ┌─────────────────────────────────────────┐       │
│   │              Host OS                    │       │
│   └─────────────────────────────────────────┘       │
│   ┌─────────────────────────────────────────┐       │
│   │              Hardware                   │       │
│   └─────────────────────────────────────────┘       │
└─────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────┐
│                Container Architecture                │
│                                                     │
│   ┌──────────┐  ┌──────────┐  ┌──────────┐         │
│   │  App A   │  │  App B   │  │  App C   │         │
│   ├──────────┤  ├──────────┤  ├──────────┤         │
│   │ Libs/Deps│  │ Libs/Deps│  │ Libs/Deps│         │
│   └──────────┘  └──────────┘  └──────────┘         │
│   ┌─────────────────────────────────────────┐       │
│   │          Container Runtime (Docker)      │       │
│   └─────────────────────────────────────────┘       │
│   ┌─────────────────────────────────────────┐       │
│   │         Host OS (shared kernel)         │       │
│   └─────────────────────────────────────────┘       │
│   ┌─────────────────────────────────────────┐       │
│   │              Hardware                   │       │
│   └─────────────────────────────────────────┘       │
└─────────────────────────────────────────────────────┘
```

| Property | VM | Container |
|---|---|---|
| Boots its own OS | ✅ Yes | ❌ No — shares host kernel |
| Startup time | 1–5 minutes | 1–3 seconds |
| Size | GBs | MBs |
| Isolation | Full (separate kernel) | Process-level |
| Portability | Harder | Very easy |

**One-line summary:** A VM is like building a separate house inside your apartment. A container is like a locked room in the same building — isolated, but sharing the foundation.

---

### Q3. What is an Image?

A Docker **Image** is a **read-only blueprint** that contains everything needed to run an application. It is built once from a `Dockerfile` and never changes after that.

```
Dockerfile (instructions)
      │
      ▼
docker build
      │
      ▼
┌─────────────────────┐
│   Docker Image      │  ← Read-only, stored on disk
│                     │
│  Layer 4: App code  │  ← COPY index.html
│  Layer 3: Config    │  ← EXPOSE 80
│  Layer 2: Nginx     │  ← FROM nginx:alpine
│  Layer 1: Alpine OS │
└─────────────────────┘
```

Think of an image like a **recipe card** — it describes exactly what to make, but the recipe card itself is not food. It just sits there until you use it to make something.

---

### Q4. What is a Container?

A **Container** is a **running instance** created from an image. If an image is the recipe, the container is the actual cooked dish.

```
Image (recipe card)  →  docker run  →  Container (running dish)

Same image can create many containers:

myapp:v1  →  Container 1 (port 8080)
myapp:v1  →  Container 2 (port 8081)
myapp:v1  →  Container 3 (port 8082)

Each container is isolated from the others.
```

The container adds a thin **writable layer** on top of the read-only image — any files written at runtime go here.

---

### Q5. What happens if a container is deleted?

```
┌──────────────────────────────────┐
│         myapp-container          │
│                                  │
│  ┌──────────────────────────┐    │
│  │  Writable layer (runtime)│ ←──┼── Files written here ARE LOST
│  │  - logs created          │    │    when container is deleted
│  │  - files uploaded        │    │
│  └──────────────────────────┘    │
│  ┌──────────────────────────┐    │
│  │  Image layers (read-only)│ ←──┼── Image stays safe on disk
│  │  myapp:v1                │    │    ✅ Not affected by deletion
│  └──────────────────────────┘    │
└──────────────────────────────────┘

docker rm myapp-container
      │
      ▼
Writable layer → GONE ❌
Image (myapp:v1) → Still exists ✅
New container can be created fresh from same image ✅
```

**Solution to data loss:** Use **Docker Volumes** (covered in Part 3) to store important data outside the container's writable layer.

---

## Part 2 — Docker Operations

---

### Q1. Difference between Image and Container?

```
Image                          Container
─────────────────────          ──────────────────────
Read-only blueprint            Running instance
Stored on disk                 Lives in memory + thin layer
Like a class in OOP            Like an object in OOP
docker images → lists them     docker ps → lists them
Created by: docker build       Created by: docker run
Never changes                  Has mutable state
Can exist without containers   Cannot exist without an image
```

---

### Q2. Difference between docker stop and docker rm?

```
docker stop myapp-container
      │
      ▼
Container is PAUSED
Still exists on disk (docker ps -a shows it)
Can be restarted: docker start myapp-container
State is preserved
Like → Putting a TV on standby


docker rm myapp-container
      │
      ▼
Container is PERMANENTLY DELETED
Gone from disk
Cannot be restarted
State is lost forever
Like → Throwing the TV in the bin
```

| Command | Container exists after? | Restartable? | Data preserved? |
|---|---|---|---|
| `docker stop` | ✅ Yes (stopped) | ✅ Yes | ✅ Yes |
| `docker rm` | ❌ No | ❌ No | ❌ No |

---

### Q3. Where are container logs stored?

By default, Docker stores logs on the host at:
```
/var/lib/docker/containers/<container-id>/<container-id>-json.log
```

You never browse this file directly. You access logs via:
```bash
docker logs myapp-container          # show all logs
docker logs -f myapp-container       # follow live (like tail -f)
docker logs --tail 50 myapp-container # last 50 lines
```

---

## Part 3 — Docker Volumes

---

### Q1. Why are volumes needed?

Containers are **ephemeral** — when deleted, everything inside is gone. But real applications need to **persist data** (databases, uploads, logs) across container restarts and deletions.

```
Without Volumes:
  Container deleted → All data GONE ❌

With Volumes:
  Container deleted → Volume stays on disk ✅
  New container mounts same volume → Data still there ✅

Example:
  MySQL container + volume → database survives container restart
  Without volume → database wiped every restart ❌
```

---

### Q2. What happens without volumes?

```
Container starts
      │
      ▼
App writes data to /data/file.txt  (inside writable layer)
      │
      ▼
docker rm container
      │
      ▼
Writable layer destroyed
/data/file.txt → GONE FOREVER ❌

Real world example:
  A database container without a volume loses ALL its data
  every time it is restarted or updated. Unusable for production.
```

---

### Q3. Difference between Volume and Bind Mount?

```
┌─────────────────────────────────────────────────────────┐
│                      VOLUME                             │
│                                                         │
│  Docker manages storage location automatically          │
│  Stored at: /var/lib/docker/volumes/myvolume/           │
│                                                         │
│  Host                    Container                      │
│  /var/lib/docker/   →    /data/                         │
│  volumes/myvolume/                                      │
│                                                         │
│  ✅ Portable — works on any machine                     │
│  ✅ Docker manages it (backup, inspect)                 │
│  ✅ Best for databases and app data                     │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│                    BIND MOUNT                           │
│                                                         │
│  You specify the exact host folder path                 │
│                                                         │
│  Host                    Container                      │
│  /home/awez/myapp/  →    /usr/share/nginx/html/         │
│                                                         │
│  ✅ Good for development (live code reload)             │
│  ❌ Host-path dependent — breaks on different machines  │
│  ❌ Not recommended for production data                 │
└─────────────────────────────────────────────────────────┘
```

---

## Part 4 — Docker Networking

---

### Q1. Why use custom networks?

The **default bridge network** does not support container name resolution — containers can only find each other by IP address, which changes on every restart.

```
Default bridge (problem):
  container-1 wants to reach container-2
  Must use: ping 172.17.0.3  ← IP changes every restart ❌

Custom network (solution):
  docker network create mynet
  container-1 can now use: ping container-2  ← by name ✅
  Docker's built-in DNS resolves names automatically
```

Custom networks also **isolate** groups of containers from each other for security.

---

### Q2. Difference between Bridge and Host network?

```
┌──────────────────────────────────────────────────────┐
│                  BRIDGE (default)                    │
│                                                      │
│  Container gets its OWN IP (e.g. 172.17.0.2)        │
│  Isolated from host network                          │
│  Must map ports to access from outside:              │
│    docker run -p 8080:80 → host:8080 → container:80  │
│                                                      │
│  Host Network                                        │
│  ┌──────────────┐    port mapping    ┌────────────┐  │
│  │   Browser    │ ───────────────→   │ Container  │  │
│  │ localhost:8080│                   │ port 80    │  │
│  └──────────────┘                   └────────────┘  │
└──────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────┐
│                    HOST network                      │
│                                                      │
│  Container SHARES the host's network stack           │
│  No port mapping needed                              │
│  Container port 80 IS host port 80 directly          │
│  ❌ No isolation  ❌ Linux only                      │
│                                                      │
│  docker run --network host → app on port 80          │
│  Access via: localhost:80 directly                   │
└──────────────────────────────────────────────────────┘
```

---

### Q3. How do containers communicate?

```
Custom Network: mynet

┌────────────┐         ┌─────────────────┐        ┌────────────┐
│    c1      │         │  Docker DNS      │        │    c2      │
│            │ "ping   │  (built into     │        │            │
│            │──c2"──→ │  custom network) │──IP──→ │            │
│ 172.18.0.2 │         │  c2 = 172.18.0.3│        │ 172.18.0.3 │
└────────────┘         └─────────────────┘        └────────────┘

1. c1 sends packet addressed to "c2"
2. Docker's embedded DNS resolves "c2" → 172.18.0.3
3. Packet travels through the virtual bridge
4. c2 receives it
```

---

## Part 5 — Jenkins Installation

---

### Q1. What is Jenkins?

Jenkins is an open-source **automation server** — a tool that automatically performs repetitive tasks like building, testing, and deploying software whenever code changes.

```
Without Jenkins (manual):
  Developer pushes code
        │
        ▼ (developer does this by hand, every time)
  SSH into server
  git pull
  docker build
  docker push
  kubectl set image
  Pray nothing goes wrong 🙏

With Jenkins (automated):
  Developer pushes code
        │
        ▼ (Jenkins does all of this automatically)
  ┌─────────────────────────┐
  │   Jenkins Pipeline      │
  │  1. Clone repo          │
  │  2. Build image         │
  │  3. Push to registry    │
  │  4. Deploy to K8s       │
  │  5. Verify deployment   │
  └─────────────────────────┘
```

---

### Q2. What problem does Jenkins solve?

Jenkins solves the problem of **manual, repetitive, error-prone deployments**. Without it, every release requires a developer to manually run a sequence of commands — and one missed step or typo can break production.

Jenkins ensures every deployment is:
- **Identical** — same steps, same order, every time
- **Auditable** — full history of every build and deployment
- **Automatic** — triggered by a git push, no human needed
- **Safe** — failed tests stop the pipeline before bad code reaches production

---

### Q3. Difference between CI and CD?

```
┌─────────────────────────────────────────────────────────────┐
│                    CI/CD Pipeline                           │
│                                                             │
│  Code Push                                                  │
│      │                                                      │
│      ▼                                                      │
│  ┌──────────────────────────────┐                          │
│  │   CI (Continuous Integration)│                          │
│  │                              │                          │
│  │  • Auto clone repo           │                          │
│  │  • Auto build                │                          │
│  │  • Auto test                 │                          │
│  │  • Catch bugs early          │ ← Runs on EVERY commit   │
│  └──────────────┬───────────────┘                          │
│                 │ (if tests pass)                           │
│                 ▼                                           │
│  ┌──────────────────────────────┐                          │
│  │   CD (Continuous Delivery)   │                          │
│  │                              │                          │
│  │  • Auto package              │                          │
│  │  • Ready to release          │                          │
│  │  • Needs manual approval     │ ← Human clicks "deploy"  │
│  │    before production         │                          │
│  └──────────────┬───────────────┘                          │
│                 │                                           │
│                 ▼                                           │
│  ┌──────────────────────────────┐                          │
│  │  CD (Continuous Deployment)  │                          │
│  │                              │                          │
│  │  • Fully automatic           │ ← Goes to production     │
│  │  • No manual approval        │   with ZERO human touch  │
│  │  • Straight to production    │                          │
│  └──────────────────────────────┘                          │
└─────────────────────────────────────────────────────────────┘
```

---

## Part 6 — Jenkins Pipeline

---

### Q1. What is a Jenkins Pipeline?

A Jenkins Pipeline is your **entire build and deploy process written as code** in a file called `Jenkinsfile`, stored in Git alongside your application code.

```groovy
// Jenkinsfile — this IS the pipeline
pipeline {
    agent any
    stages {
        stage('Clone Repository') {   // Stage 1
            steps {
                git url: 'https://github.com/Awezsk/Devopsassignment2.git'
            }
        }
        stage('Build Docker Image') { // Stage 2
            steps {
                sh 'docker build -t myapp:v1 .'
            }
        }
        stage('Run Container') {      // Stage 3
            steps {
                sh 'docker run -d -p 8081:80 myapp:v1'
            }
        }
    }
}
```

Jenkins reads this file and executes each stage in order, showing a visual dashboard:

```
[Clone Repository] → [Build Docker Image] → [Run Container]
      ✅ 12s               ✅ 45s                ✅ 3s
                                          Finished: SUCCESS
```

---

### Q2. Why use pipelines instead of manual deployments?

| Problem with manual | How pipeline fixes it |
|---|---|
| Developer forgets a step | Pipeline always runs every step |
| Typo in a command | Jenkinsfile is reviewed and version-controlled |
| "Who deployed what when?" | Jenkins keeps full build history |
| Only one person knows the deploy process | Anyone can read the Jenkinsfile |
| Deployment takes 30 mins of human time | Pipeline runs in background, zero human time |

---

### Q3. Difference between Freestyle and Pipeline jobs?

```
FREESTYLE JOB                    PIPELINE JOB
─────────────────────────────    ────────────────────────────────
Configured by clicking buttons   Configured by writing a Jenkinsfile
Stored in Jenkins (not Git)      Stored in Git with your code
Single step / single action      Multi-stage, complex workflows
Hard to review or audit          Code-reviewed like any other code
Can't handle complex logic       Supports if/else, loops, parallel
Good for: simple quick tests     Good for: full CI/CD workflows
```

---

## Part 7 — Container Registry

---

### Q1. Why use a registry?

A registry is a **central store** for Docker images — like GitHub but for images instead of code.

```
Without registry:
  EC2 (Jenkins) builds image
        │
        │  ← How does Kubernetes get it? ❌ It can't.
        ▼
  Kind cluster (Kubernetes) — has no access to EC2's local images

With registry (Docker Hub):
  EC2 (Jenkins) builds image
        │
        ▼ docker push awezsk/myapp:v1
  ┌──────────────────┐
  │   Docker Hub     │  ← Central store
  │  awezsk/myapp:v1 │
  └──────────────────┘
        │
        ▼ docker pull (automatic)
  Kind cluster pulls image and runs it ✅
```

---

### Q2. Difference between local image and registry image?

```
LOCAL IMAGE                         REGISTRY IMAGE
────────────────────────────        ───────────────────────────────
Lives only on one machine           Accessible from anywhere
docker images → shows it            hub.docker.com → shows it
Invisible to other servers          Any server can pull it
Lost if machine is deleted          Persists independently
Good for: development/testing       Good for: production deployments
```

---

### Q3. Why not build images directly on production servers?

```
Building on production (BAD):

Production Server
├── Serving 1000 users
├── Running docker build  ← Uses CPU needed for users
├── Build fails halfway   ← App is now broken ❌
└── Untested code pushed  ← Goes live immediately ❌

Correct approach:

Jenkins (build server)    Docker Hub      Production
  docker build       →   docker push  →  docker pull
  docker test            (safe store)    (tested image only)
  ✅ Build fails here = production unaffected
  ✅ Only tested images reach production
```

---

## Part 8 — Kubernetes Installation

---

### Q1. What is Kubernetes?

Kubernetes (K8s) is an **orchestration platform** that manages containerized applications across a cluster of machines — automatically handling deployment, scaling, networking, and self-healing.

```
Without Kubernetes (you manage everything):
  "Container crashed on server 3"   → You SSH in and restart it manually
  "Traffic spike — need more copies" → You SSH in and start more manually
  "Server 2 died"                   → You redistribute manually
  (At 3am) 😴💀

With Kubernetes (it manages everything):
  "Container crashed"               → K8s restarts it automatically ✅
  "Traffic spike"                   → K8s scales up automatically ✅
  "Server 2 died"                   → K8s moves containers to server 3 ✅
  (At 3am) 😴✅
```

---

### Q2. Why not run containers directly?

```
docker run (no Kubernetes):
  • Crashes → stays crashed until you fix it
  • Want 3 copies → run 3 commands manually
  • Server dies → app goes down
  • Update app → manual stop/start on every server
  • No load balancing

kubectl (with Kubernetes):
  • Crashes → auto-restarted ✅
  • Want 3 copies → replicas: 3 ✅
  • Server dies → rescheduled on another node ✅
  • Update app → rolling update, zero downtime ✅
  • Built-in load balancing via Services ✅
```

---

### Q3. What problems does Kubernetes solve?

| Problem | Kubernetes Solution |
|---|---|
| Container crashes at night | Self-healing — auto restarts |
| Need more capacity under load | Horizontal auto-scaling |
| Server hardware fails | Reschedules to healthy nodes |
| Deploying new version safely | Rolling updates |
| Bad version deployed | One-command rollback |
| Finding which IP to connect to | Services — stable DNS name |
| Running many apps on one cluster | Namespace isolation |

---

## Part 9 — Pods

---

### Q1. What is a Pod?

A Pod is the **smallest deployable unit** in Kubernetes. It wraps one or more containers that share the same network and storage.

```
┌──────────────────────────────────────┐
│               POD                    │
│  Name: myapp-pod                     │
│  IP: 10.244.0.5  (one shared IP)     │
│                                      │
│  ┌─────────────────┐                 │
│  │  Container: app │  port 80        │
│  │  nginx:alpine   │                 │
│  └─────────────────┘                 │
│                                      │
│  (could also have a sidecar:)        │
│  ┌─────────────────┐                 │
│  │  Container: log │  reads same     │
│  │  log-forwarder  │  volume as app  │
│  └─────────────────┘                 │
└──────────────────────────────────────┘
        │
        ▼ scheduled on
┌──────────────────┐
│   Node (Server)  │
└──────────────────┘
```

---

### Q2. Why doesn't Kubernetes deploy containers directly?

Pods exist because some applications are made of **tightly-coupled containers** that must run together on the same machine, share the same network (communicate over localhost), and share the same storage volumes.

```
Example: Web app + log collector

They must:
  • Run on the SAME machine ← Pod guarantees this
  • Share log files ← Pod's shared volume enables this
  • Log collector reads localhost:app ← Pod's shared network enables this

If Kubernetes deployed containers independently,
these guarantees would be impossible to make.
```

---

### Q3. Can a Pod contain multiple containers?

Yes — this is the **sidecar pattern**, one of the most common Kubernetes patterns.

```
┌─────────────────────────────────────────┐
│                  POD                    │
│                                         │
│  ┌─────────────────┐  ┌──────────────┐  │
│  │  Main Container │  │   Sidecar    │  │
│  │                 │  │              │  │
│  │  Your web app   │  │ Log forwarder│  │
│  │  port 80        │  │ reads /logs/ │  │
│  └────────┬────────┘  └──────┬───────┘  │
│           │                  │          │
│           └──── shared ──────┘          │
│                 /logs/ volume           │
└─────────────────────────────────────────┘

Both containers:
  ✅ Share the same IP address
  ✅ Communicate via localhost
  ✅ Share mounted volumes
  ✅ Start and stop together
```

---

## Part 10 — Deployments

---

### Q1. Why did the Pod return automatically?

Because a **Deployment** runs a continuous control loop that enforces your declared `replicas` count.

```
You declared: replicas: 3

Control Loop (runs constantly):
  Actual pods running = 3  →  desired = 3  →  do nothing
  
You delete one pod:
  Actual pods running = 2  →  desired = 3  →  CREATE 1 NEW POD ✅

This is Kubernetes's "self-healing" — it never stops watching.
It corrects drift between actual and desired state automatically.
```

---

### Q2. Difference between Pod and Deployment?

```
POD (alone)                        DEPLOYMENT
────────────────────────────       ────────────────────────────────
One instance                       Manages multiple instances
No self-healing                    Self-healing (replaces crashed pods)
Deleted = gone forever             Deleted pod = instantly replaced
No rolling updates                 Built-in rolling update support
No scaling                         Scale with one command
Like: hiring one contractor        Like: hiring a staffing agency that
      and hoping they show up            always keeps your team staffed
```

---

### Q3. What is desired state?

Desired state is the configuration you declare in your YAML — Kubernetes treats it as a **standing order** it must continuously enforce.

```yaml
spec:
  replicas: 3        ← "I always want exactly 3 pods running"
  image: myapp:v1    ← "Each pod must run this exact image"
```

```
You declare desired state once.
Kubernetes enforces it forever.

Reality drifts →  Pod crashes    → K8s creates replacement
Reality drifts →  Node dies      → K8s reschedules pods
Reality drifts →  Image changes  → K8s does rolling update

You don't run commands to fix these.
Kubernetes fixes them automatically.
```

---

## Part 11 — Services

---

### Q1. Why do Pods need Services?

Pod IP addresses are **temporary** — every time a Pod is deleted and recreated (which happens constantly), it gets a brand new IP. No client can reliably connect to a constantly-changing IP.

```
Without Service:
  Client hardcodes: http://10.244.0.5
  Pod crashes → recreated at: http://10.244.0.8
  Client is now broken ❌

With Service:
  Client connects to: http://myapp-service  (never changes)
  Service tracks current pod IPs automatically
  Pod crashes → recreated → Service updates → client unaffected ✅

┌────────┐      ┌─────────────────┐      ┌────────────────┐
│ Client │ ───→ │  myapp-service  │ ───→ │ Pod 10.244.0.8 │
│        │      │  (stable IP)    │      │ Pod 10.244.0.9 │
└────────┘      └─────────────────┘      │ Pod 10.244.0.10│
                                         └────────────────┘
```

---

### Q2. Difference between ClusterIP and NodePort?

```
┌────────────────────────────────────────────────────────┐
│                    ClusterIP                           │
│                                                        │
│  Virtual IP only reachable INSIDE the cluster          │
│  Default Service type                                  │
│                                                        │
│  Internet  ──✗──→  [ClusterIP]  →  Pods               │
│  Inside cluster  ──✅──→  [ClusterIP]  →  Pods         │
│                                                        │
│  Use for: internal service-to-service communication    │
└────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────┐
│                     NodePort                           │
│                                                        │
│  Opens a port (30000-32767) on EVERY node              │
│  Reachable from outside the cluster                    │
│                                                        │
│  Internet  ──✅──→  Node:30080  →  [Service]  →  Pods  │
│                                                        │
│  Use for: exposing apps externally (dev/testing)       │
│  In this assignment: nodePort: 30080                   │
└────────────────────────────────────────────────────────┘
```

---

### Q3. What happens when a Pod IP changes?

Nothing breaks — the Service handles it transparently.

```
Before Pod restart:
  Service endpoints: [10.244.0.5, 10.244.0.6, 10.244.0.7]

Pod 10.244.0.5 crashes and is recreated at 10.244.0.8:
  Kubernetes notifies Service automatically
  Service endpoints: [10.244.0.8, 10.244.0.6, 10.244.0.7]

Client connecting through Service:
  Never knew about 10.244.0.5
  Never knows about 10.244.0.8
  Always just connects to myapp-service ✅
```

---

## Part 12 — Rolling Updates

---

### Q1. What is a rolling update?

A rolling update replaces old Pod replicas with new ones **one at a time**, never taking all instances down simultaneously.

```
Before update: 3 pods running myapp:v1

Step 1: Start 1 new pod (myapp:v2), wait for it to be Ready
  v1 v1 v1 v2  ← temporary (4 pods)

Step 2: Terminate 1 old pod (myapp:v1)
  v1 v1 v2     ← back to 3 pods

Step 3: Start another new pod, wait for Ready
  v1 v1 v2 v2  ← temporary

Step 4: Terminate another old pod
  v1 v2 v2     ← 3 pods

Step 5: Repeat until done
  v2 v2 v2     ← all updated ✅

At NO point did the app have 0 running pods.
Users experienced zero downtime.
```

---

### Q2. Why is rolling update safer?

```
Big-bang deployment (unsafe):
  Stop all 3 v1 pods → deploy v2 → start 3 v2 pods
  
  Problem: if v2 has a bug, ALL users are affected immediately
  Downtime: yes, during the switch
  
Rolling update (safe):
  Replace 1 pod at a time
  
  v2 bug detected on pod 1 → stop rollout → only 1/3 pods affected
  Other 2 pods still running v1 → most users unaffected ✅
  Rollback is instant → kubectl rollout undo
```

---

### Q3. What is zero downtime deployment?

Zero downtime means **users never see an error or outage** during a deployment.

```
Service always has healthy pods behind it:

t=0:  [v1] [v1] [v1]      ← 3 pods serving traffic
t=1:  [v1] [v1] [v1] [v2] ← v2 starting (not yet in rotation)
t=2:  [v1] [v1] [v2]      ← v2 healthy, one v1 removed
t=3:  [v1] [v1] [v2] [v2] ← second v2 starting
t=4:  [v1] [v2] [v2]      ← second v2 healthy
t=5:  [v2] [v2] [v2]      ← all updated ✅

At every point in time: at least 2 pods serving traffic.
The Service's stable address never changes.
Users see no interruption.
```

---

## Part 13 — Rollback

---

### Q1. Why are rollbacks important?

No matter how thorough testing is, bugs sometimes only appear in production. Rollback is the **emergency brake** that instantly reverts to the last working version.

```
Without rollback capability:
  Bad v3 deployed
        │
        ▼
  Must build v4 (fix), test, push, deploy
  Time: 30 minutes to 2 hours
  Users see broken app the entire time ❌

With kubectl rollout undo:
  Bad v3 deployed
        │
        ▼
  kubectl rollout undo deployment/myapp-deployment
  Time: 30 seconds
  Kubernetes rolls back to v2 automatically ✅
```

---

### Q2. How does Kubernetes maintain availability during rollback?

`kubectl rollout undo` is itself a **rolling update in reverse** — it uses the exact same gradual pod replacement strategy, just going backwards.

```
Bad state: 3 pods running broken v3

kubectl rollout undo
      │
      ▼
Step 1: Start 1 pod with v2 (last good version)
  [v3] [v3] [v3] [v2]  ← v2 starting

Step 2: v2 pod is Ready → terminate 1 v3 pod
  [v3] [v3] [v2]

Step 3: Start another v2 pod
  [v3] [v3] [v2] [v2]

Step 4: Terminate another v3 pod
  [v3] [v2] [v2]

Step 5: Final replacement
  [v2] [v2] [v2]  ← fully rolled back ✅

Traffic was served throughout. Users barely noticed.
```

---

## Part 14 — Jenkins + Kubernetes Integration

---

### Q1. How does Jenkins communicate with Kubernetes?

Jenkins runs `kubectl` commands in pipeline shell steps. `kubectl` uses a **kubeconfig file** that contains the cluster's address and credentials.

```
Setup (done once):
  sudo cp /home/ubuntu/.kube/config /var/lib/jenkins/.kube/config
  sudo chown -R jenkins:jenkins /var/lib/jenkins/.kube

Pipeline step:
  sh "kubectl set image deployment/myapp-deployment myapp=awezsk/myapp:v2"
        │
        ▼
  Jenkins (as jenkins user)
        │  reads kubeconfig
        ▼
  Kubernetes API Server (on Kind cluster)
        │  authenticates + authorizes
        ▼
  Updates Deployment → triggers rolling update
```

The kubeconfig contains:
- Cluster API server address
- TLS certificate for authentication
- User credentials (token or certificate)

---

### Q2. Why automate deployments?

```
Manual deployment (risky):
  Developer SSH into EC2
  git pull  ← forgot this step once → deployed old code
  docker build -t awezsk/myapp:v2 .
  docker push awezsk/myapp:v2
  kubectl set image ...  ← typo → wrong image name → crash
  kubectl rollout status  ← forgot to check
  Goes home 🏠
  App is broken. Nobody knows until users complain.

Automated pipeline (safe):
  Developer pushes code to Git
  Jenkins detects push
  Runs identical steps every single time
  Fails fast if anything goes wrong
  Sends notification on failure
  Full audit log of every deployment
  Developer goes home 🏠
  Everything is logged and verified ✅
```

---

### Q3. What risks exist in manual deployments?

| Risk | Example | Pipeline Prevention |
|---|---|---|
| Human error | Wrong image tag pushed | Pipeline uses `v${BUILD_NUMBER}` automatically |
| Skipped step | Forgot `docker push` before deploy | Every step in Jenkinsfile always runs |
| No audit trail | "Who deployed v3 at 2am?" — nobody knows | Jenkins build history shows who, when, what |
| Inconsistency | Dev deploys differently than Ops | One Jenkinsfile = same steps for everyone |
| No rollback plan | Bad version deployed, no way back | Kubernetes keeps history; `rollout undo` always available |
| Untested code in production | "It worked on my laptop" | Pipeline runs tests before deploy stage |

---

## Troubleshooting Challenges

---

### Challenge 1 — Container exits immediately

**Root cause:** The main process (PID 1) inside the container finished or crashed immediately after starting. Docker containers only stay alive as long as their foreground process runs.

```
Common causes:
  1. Script exits normally (exit code 0) — no long-running process
  2. Application crashes on startup (missing config, wrong env var)
  3. Wrong CMD in Dockerfile

Diagnosis:
  docker logs myapp-container     ← see what the app printed before dying
  docker inspect myapp-container  ← check ExitCode field

Fix:
  Ensure your process runs in foreground (not background/daemon mode)
  Example: nginx -g 'daemon off;'  ← correct for nginx containers
```

---

### Challenge 2 — Works locally, not in Kubernetes

**Root cause:** Environment differences between your laptop and the cluster.

```
Common causes:
  1. Image not pushed to registry — K8s can't pull from local Docker
  2. Wrong image tag in deployment.yaml
  3. Missing environment variables or ConfigMaps
  4. Hardcoded "localhost" references (localhost = the pod itself in K8s)
  5. OOMKilled — pod ran out of memory

Diagnosis:
  kubectl describe pod <pod-name>    ← check Events section at bottom
  kubectl logs <pod-name>            ← check application errors
  kubectl get events                 ← cluster-wide events
```

---

### Challenge 3 — Service exists but application inaccessible

**Root cause:** Usually a **label selector mismatch** between the Service and the Pods.

```
Diagnosis:
  kubectl get endpoints myapp-service

  If output shows: ENDPOINTS = <none>
  → Service selector doesn't match any pod labels ❌

Check:
  Service selector:        Pod labels:
  selector:                metadata:
    app: myapp               labels:
                               app: myApp   ← capital M! mismatch ❌

Other causes:
  • Wrong targetPort (Service points to port 8080, app listens on 80)
  • Readiness probe failing (pod exists but not Ready)
  • Security Group blocking the NodePort on EC2
```

---

### Challenge 4 — ImagePullBackOff

**Root cause:** Kubernetes cannot pull the container image from the registry.

```
kubectl describe pod <pod-name>  ← check Events

Common causes and fixes:

Cause 1: Wrong image name or tag
  image: awezsk/myapp:v99   ← tag doesn't exist on Docker Hub
  Fix: docker push awezsk/myapp:v99  OR correct the tag in YAML

Cause 2: Private registry, no credentials
  Fix: Create imagePullSecrets in Kubernetes pointing to registry login

Cause 3: Typo in username
  image: Awezsk/myapp:v1   ← capital A vs lowercase a
  Fix: image: awezsk/myapp:v1

Cause 4: Network issue
  Node can't reach Docker Hub
  Fix: Check node internet connectivity
```

---

### Challenge 5 — Delete a Pod manually, app stays available

**Why:** The **Deployment + Service** combination ensures availability even when individual pods are deleted.

```
You delete: myapp-deployment-abc123

Immediately:
  Deployment detects: actual=2, desired=3
  Creates replacement pod: myapp-deployment-xyz789

During this time:
  Service still has 2 healthy pods in its endpoints list
  Traffic is load-balanced to those 2 pods
  Users see no interruption

Within ~10 seconds:
  New pod is Running and Ready
  Service adds it to endpoints
  Back to 3 pods ✅

This is why Deployments + Services exist —
individual pod failures are meaningless to users.
```

---

### Challenge 6 — Pipeline fails during Docker build

**Root cause:** Error in Dockerfile, wrong file path, or Jenkins environment issue.

```
Diagnosis: Jenkins Console Output → read the exact error line

Common causes:

Cause 1: COPY file not found
  COPY app/index.html ...  ← but file is at root, not in app/
  Fix: COPY index.html ...  (match actual repo structure)

Cause 2: Jenkins not in docker group
  Got permission denied while trying to connect to Docker daemon
  Fix: sudo usermod -aG docker jenkins && sudo systemctl restart docker

Cause 3: Base image unavailable
  FROM nginx:alpine → network error pulling image
  Fix: check EC2 internet connectivity

Cause 4: Disk full
  no space left on device
  Fix: docker system prune -f  (cleans old images/containers)
```

---

### Challenge 7 — Deployment succeeds but users see old version

**Root cause:** Usually cached images or browser cache, not a real deployment failure.

```
Cause 1: imagePullPolicy not set to Always
  K8s used cached old image from node
  Fix: add imagePullPolicy: Always to deployment.yaml

Cause 2: Using :latest tag
  :latest was already cached, K8s didn't re-pull
  Fix: use specific versioned tags (v1, v2, v3...)

Cause 3: Browser cache
  Browser is showing old page from cache
  Fix: Ctrl+Shift+R (hard refresh) or open incognito window

Cause 4: Rollout not finished yet
  kubectl rollout status deployment/myapp-deployment
  Wait for: "successfully rolled out"

Cause 5: Old pods still in rotation
  kubectl get pods  ← check all pods show new image
  kubectl describe pod <name> | grep Image
```

---

## Final Architecture — Component Explanations

```
Developer
    │  writes code, pushes to Git
    ▼
Git Repository (GitHub: Awezsk/Devopsassignment2)
    │  Jenkins polls for changes / webhook triggers
    ▼
Jenkins Pipeline (EC2: 18.234.150.115:8080)
    │  Stage 1: git clone
    │  Stage 2: docker build -t awezsk/myapp:vN .
    │  Stage 3: docker push awezsk/myapp:vN
    │  Stage 4: kubectl set image deployment/...
    │  Stage 5: kubectl rollout status
    ▼
Docker Hub Registry (hub.docker.com/r/awezsk/myapp)
    │  Stores versioned images (v1, v2, v3...)
    │  Kubernetes pulls from here
    ▼
Kubernetes Deployment (Kind cluster on EC2)
    │  Manages 3 replica pods
    │  Self-heals, rolls updates, enables rollback
    ▼
Kubernetes Service (myapp-service, NodePort 30080)
    │  Stable endpoint, load balances across pods
    │  Pod IPs can change — Service address never does
    ▼
Users
    │  Access via stable Service address
    │  Never experience downtime during updates
    ▼
  "Welcome to DevOps Training" ✅
```

| Component | Purpose |
|---|---|
| **Git** | Source of truth for all code and config |
| **Jenkins** | Automation engine — builds, tests, deploys |
| **Docker** | Packages app into portable container image |
| **Docker Hub** | Central image store accessible by any server |
| **Kubernetes Deployment** | Manages replicas, self-healing, rolling updates |
| **Kubernetes Service** | Stable network endpoint, load balancing |
| **kubectl** | CLI tool Jenkins uses to talk to Kubernetes |
| **kubeconfig** | Credentials file that authenticates kubectl to the cluster |
