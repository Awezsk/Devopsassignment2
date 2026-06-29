# DevOps Assignment 2 — Complete Documentation

**Name:** Awez SK
**GitHub:** [Awezsk/Devopsassignment2](https://github.com/Awezsk/Devopsassignment2)
**Docker Hub:** awezsk
**EC2 Instance:** 18.234.150.115 (Ubuntu 22.04, t2.micro, AWS Free Tier)
**EC2 Private Hostname:** ip-172-31-28-220

---

## Architecture Overview

```
Developer (Laptop)
    │
    ├── VS Code (Edit code)
    ├── Docker Desktop (Local testing, Parts 1–4)
    └── Git Push → GitHub (Awezsk/Devopsassignment2)
                        │
                        └── Jenkins (EC2: 18.234.150.115:8080)
                                │
                                ├── docker build
                                ├── docker push → Docker Hub (awezsk/myapp)
                                └── kubectl deploy → Kind Cluster (EC2)
                                                        │
                                                        └── myapp-deployment (3 Pods)
                                                                │
                                                        myapp-service (NodePort)
                                                                │
                                                        Browser (via SSH tunnel)
```

---

## Part 0 — EC2 Instance Launch

**Execution location:** AWS Console (browser on laptop) → SSH terminal on EC2

### Steps Performed
- Launched EC2 Ubuntu 22.04 LTS t2.micro instance named `devops-assignment`
- Created key pair: `devops-assignment.pem`
- Security Group inbound rules: SSH (22), Jenkins (8080), Kubernetes NodePort (30000–32767) — all restricted to My IP
- Storage: 16 GB gp3 EBS volume
- Added 2 GB swap space for RAM relief on t2.micro

---

## Part 1 — Docker Fundamentals

**Execution location:** 💻 Laptop — Docker Desktop + PowerShell / VS Code terminal

### Files Created

**`Dockerfile`** (at repo root):
```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
```

**`index.html`** (at repo root):
```html
<!DOCTYPE html>
<html>
<head><title>DevOps Training</title></head>
<body><h1>Welcome to DevOps Training</h1></body>
</html>
```

### Commands Run (Laptop PowerShell)
```powershell
docker build -t myapp:v1 .
docker run -d -p 8081:80 --name myapp-container myapp:v1
```
> Note: Port 8081 used instead of 8080 because 8080 was already occupied on the laptop.

### Screenshots

**Docker build — image created:**

![Part 1 Docker create](https://raw.githubusercontent.com/Awezsk/Devopsassignment2/main/part1-Docker%20create.png)

**Docker run — container running locally:**

![Part 1 Docker run locally](https://raw.githubusercontent.com/Awezsk/Devopsassignment2/main/part1%205Docker%20run%20locally.png)

### Q&A

**Q1. What problem does Docker solve?**
Docker eliminates the "works on my machine" problem by packaging an application with all its dependencies into a single portable container unit. This container runs identically on any machine — laptop, test server, or cloud — regardless of the underlying OS configuration.

**Q2. Difference between a VM and a Container?**
A VM virtualizes an entire computer including its own OS kernel, making it heavy and slow to boot (minutes, gigabytes of disk). A container shares the host machine's kernel and is lightweight, starting in seconds and using only megabytes, while still maintaining process isolation.

**Q3. What is an Image?**
A Docker image is a read-only, immutable blueprint built from Dockerfile instructions, containing the application code, runtime, libraries, and configuration. It is like a recipe card — it describes exactly what to build but does not run anything by itself.

**Q4. What is a Container?**
A container is a running instance created from a Docker image — the "cooked dish" made by following the recipe. Multiple isolated containers can be launched from the same image simultaneously, each with its own writable layer.

**Q5. What happens if a container is deleted?**
Any data written inside the container's writable layer at runtime is permanently lost when the container is removed. The image itself remains intact on disk, so a fresh container can be created from it again at any time with no data from the original run.

---

## Part 2 — Docker Operations

**Execution location:** 💻 Laptop — PowerShell / VS Code terminal

### Commands Run
```powershell
docker ps                        # list running containers
docker stop myapp-container      # gracefully stop
docker start myapp-container     # restart it
docker rm myapp-container        # remove (must be stopped first)
docker rmi myapp:v1              # remove the image
docker logs myapp-container      # view container logs
```

### Screenshot

![Part 2 Docker Operations](https://raw.githubusercontent.com/Awezsk/Devopsassignment2/main/part2%20Docker%20oprations-1.png)

### Q&A

**Q1. Image vs Container?**
An image is a static, read-only blueprint stored on disk, while a container is a live, running instance of that image with its own writable layer. The relationship is like a class (image) and an object (container) in programming.

**Q2. docker stop vs docker rm?**
`docker stop` gracefully halts a running container but keeps it on disk in a stopped state so it can be restarted. `docker rm` permanently deletes the container's writable layer and metadata from disk; the container cannot be restarted after this.

**Q3. Where are logs stored?**
Docker stores container logs by default at `/var/lib/docker/containers/<container-id>/<container-id>-json.log` on the host filesystem. In practice, you access them via `docker logs <container-name>` rather than reading the file directly.

---

## Part 3 — Docker Volumes

**Execution location:** 💻 Laptop — PowerShell / VS Code terminal

### Commands Run
```powershell
docker volume create myvolume
docker run -it --name vol-test -v myvolume:/data alpine sh
# inside container:
# echo "hello" > /data/test.txt
# exit

docker rm vol-test
docker run -it --name vol-test2 -v myvolume:/data alpine sh
# cat /data/test.txt   → "hello" still present
# exit
```

### Screenshot

![Part 3 Docker Volumes](https://raw.githubusercontent.com/Awezsk/Devopsassignment2/main/part3%20Docker%20volumes.png)

### Q&A

**Q1. Why are volumes needed?**
Containers are ephemeral — their writable layer is destroyed when the container is removed. Volumes provide persistent storage that exists independently of any container lifecycle, ensuring data survives container deletion and can be shared between containers.

**Q2. What happens without volumes?**
Without volumes, any data written inside a container's filesystem lives in its writable layer and is permanently lost the moment the container is deleted. There is no way to recover that data once the container is removed.

**Q3. Volume vs Bind Mount?**
A Docker-managed volume stores data in Docker's own storage area (`/var/lib/docker/volumes/`) and is fully portable across environments. A bind mount maps a specific host filesystem path directly into the container, making it host-path-dependent and less portable across different machines.

---

## Part 4 — Docker Networking

**Execution location:** 💻 Laptop — PowerShell / VS Code terminal

### Commands Run
```powershell
docker network create mynet
docker run -d --name c1 --network mynet alpine sleep infinity
docker run -d --name c2 --network mynet alpine sleep infinity
docker exec c1 ping -c 3 c2    # resolves "c2" by name
```

### Screenshot

![Part 4 Docker Networking](https://raw.githubusercontent.com/Awezsk/Devopsassignment2/main/Part4-Docker%20networking%20task.png)

### Q&A

**Q1. Why use custom networks?**
Docker's default bridge network does not support DNS-based name resolution between containers — you must hardcode IP addresses, which change on every restart. Custom networks include a built-in embedded DNS server so containers can reach each other by name, and also provide network-level isolation from other containers.

**Q2. Bridge vs Host network?**
Bridge mode gives each container its own isolated virtual network with its own IP address, requiring explicit port mapping to expose services to the outside. Host mode shares the host machine's network stack directly — no isolation, no port mapping needed — but this removes container networking isolation and only works on Linux.

**Q3. How do containers communicate?**
On a custom bridge network, Docker creates virtual ethernet interfaces connected to a software bridge and runs an embedded DNS server. When container `c1` pings `c2` by name, Docker's DNS resolves `c2` to its current internal IP address, routing the packet through the virtual bridge — completely transparent to the application.

---

## Part 5 — Jenkins Installation

**Execution location:** ☁️ EC2 SSH terminal + 🌐 Browser (http://18.234.150.115:8080)

### Commands Run (EC2 terminal)
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y openjdk-17-jdk

# Docker
sudo apt install -y docker.io
sudo systemctl enable docker --now
sudo usermod -aG docker ubuntu

# Jenkins (using 2026 key — old key was rotated in Dec 2025)
sudo rm -f /usr/share/keyrings/jenkins-keyring.asc
sudo rm -f /etc/apt/sources.list.d/jenkins.list
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2026.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update
sudo apt install -y jenkins
sudo usermod -aG docker jenkins
sudo systemctl restart docker
sudo systemctl enable jenkins --now

# Fix /tmp size issue (Jenkins node goes offline on small tmpfs)
sudo systemctl mask tmp.mount
sudo systemctl daemon-reload
sudo reboot
```

### First Freestyle Job
- New Item → `hello-job` → Freestyle project
- Build Step: Execute shell → `echo "Hello DevOps"`
- Result: `Finished: SUCCESS`

### Q&A

**Q1. What is Jenkins?**
Jenkins is an open-source automation server that automatically builds, tests, and deploys software whenever code changes are pushed, eliminating the need for developers to manually trigger these steps. It is the central orchestration engine in a CI/CD pipeline.

**Q2. What problem does it solve?**
Jenkins removes manual, repetitive, error-prone build and release steps by automating them — every code push triggers the same consistent sequence of compile, test, package, and deploy actions without human intervention, reducing errors and accelerating delivery.

**Q3. CI vs CD?**
Continuous Integration (CI) automatically builds and tests code on every commit to catch integration bugs early. Continuous Delivery (CD) extends this by automatically packaging a validated build ready for release, stopping at a manual approval gate before production; Continuous Deployment goes fully automated all the way to production without any manual step.

---

## Part 6 — Jenkins Pipeline (3 Stages)

**Execution location:** 🌐 Browser (Jenkins UI at http://18.234.150.115:8080) — pipeline runs on ☁️ EC2

### Pipeline Job: `myapp-pipeline`

```groovy
pipeline {
    agent any
    stages {
        stage('Clone Repository') {
            steps {
                git url: 'https://github.com/Awezsk/Devopsassignment2.git', branch: 'main'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t myapp:v1 .'
            }
        }
        stage('Run Docker Container') {
            steps {
                sh 'docker rm -f myapp-test || true'
                sh 'docker run -d --name myapp-test -p 8081:80 myapp:v1'
            }
        }
    }
}
```

> Note: Build context is `.` (repo root) since Dockerfile and index.html are at root level — not `./app`.

### Q&A

**Q1. What is a Jenkins Pipeline?**
A Jenkins Pipeline is the entire build, test, and deploy process defined as code (a Jenkinsfile stored in Git), broken into named stages that execute in sequence. This makes the process version-controlled, reviewable, and automatically repeatable rather than a sequence of manual UI clicks.

**Q2. Why pipelines over manual?**
Pipelines are version-controlled in Git, automatically triggered by code pushes, and execute identically every run — eliminating the human errors, inconsistencies, and missing steps that plague manual deployment. They also provide a full audit trail of every build and deployment in the Jenkins history.

**Q3. Freestyle vs Pipeline?**
A Freestyle project is a single, GUI-configured job with no version control support and limited logic capabilities — suitable for simple, one-step tasks. A Pipeline is a multi-stage, code-defined workflow stored in Git that supports complex logic, parallel execution, conditionals, and shared libraries across teams.

---

## Part 7 — Container Registry (Docker Hub)

**Execution location:** ☁️ EC2 SSH terminal + 🌐 hub.docker.com + 🌐 Jenkins UI

### Commands Run (EC2 terminal)
```bash
docker login -u awezsk          # authenticated with Personal Access Token
docker tag myapp:v1 awezsk/myapp:v1
docker push awezsk/myapp:v1
```

### Docker Hub Repository
`https://hub.docker.com/r/awezsk/myapp` — tag `v1` visible after push

### Jenkins Credential Added
- Kind: Username with password
- Username: `awezsk`
- Password: Docker Hub Personal Access Token
- ID: `dockerhub-creds` *(exact string required by Jenkinsfile)*

### Screenshot

![Part 7 Container Registry Docker Hub](https://raw.githubusercontent.com/Awezsk/Devopsassignment2/main/Part%207%20%E2%80%94%20Container%20Registry%20(Docker%20Hub).png)

### Q&A

**Q1. Why use a registry?**
A container registry provides a central, versioned, network-accessible store for images so any server or cluster can pull the exact same image without rebuilding it. This ensures every environment — development, staging, production — runs identical artifacts.

**Q2. Local vs registry image?**
A local image exists only on the machine that built it and cannot be accessed by any other server or cluster. A registry image is centrally versioned and pullable by any authorized host with a network connection, enabling consistent deployments across environments.

**Q3. Why not build on production servers?**
Building on production servers wastes CPU resources needed for serving live traffic, risks pushing untested code directly to production, creates environment drift between build and runtime, and mixes concerns that should be separated for security and reliability.

---

## Part 8 — Kubernetes Installation (Kind)

**Execution location:** ☁️ EC2 SSH terminal

### Commands Run
```bash
# kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl && sudo mv kubectl /usr/local/bin/

# Kind (Kubernetes in Docker)
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.23.0/kind-linux-amd64
chmod +x ./kind && sudo mv ./kind /usr/local/bin/kind

kind create cluster --name devops-cluster
kubectl get nodes
```

### Screenshot

![Part 8 Kubernetes Installation Kind on EC2](https://raw.githubusercontent.com/Awezsk/Devopsassignment2/main/Part%208%20%E2%80%94%20Kubernetes%20Installation%20(Kind%2C%20on%20EC2).png)

### Q&A

**Q1. What is Kubernetes?**
Kubernetes is an open-source container orchestration platform that automates deploying, scaling, networking, health monitoring, and management of containerized applications across a cluster of machines. It treats the cluster as a single unit rather than a collection of individual servers.

**Q2. Why not run containers directly?**
At any meaningful scale, manually placing containers on servers, restarting crashed ones, scaling them up and down, and managing networking between them becomes impossible to do reliably. Kubernetes automates all of this declaratively — you describe the desired state and it continuously ensures reality matches it.

**Q3. What problems does it solve?**
Kubernetes solves self-healing (automatically restarts crashed containers), auto-scaling (adjusts replica count based on load), load balancing (distributes traffic across healthy instances), rolling updates and rollbacks (deploys new versions without downtime), and service discovery (stable DNS names despite changing Pod IPs).

---

## Part 9 — Pods

**Execution location:** ☁️ EC2 SSH terminal (kubectl commands) + 💻 Laptop (SSH tunnel for browser access)

### Commands Run (EC2 terminal)
```bash
kubectl run myapp-pod --image=awezsk/myapp:v1 --port=80
kubectl get pods
kubectl port-forward pod/myapp-pod 8090:80   # Window A — leave running
```
> Port 8090 used because 8080 is occupied by Jenkins on EC2.

### SSH Tunnel (Laptop PowerShell — Window B)
```powershell
cd C:\Users\awez7\Downloads
ssh -i devops-assignment.pem -L 8090:localhost:8090 ubuntu@18.234.150.115
```
Then visit `http://localhost:8090` in the browser.

### Screenshot

![Part 9 Pods](https://raw.githubusercontent.com/Awezsk/Devopsassignment2/main/Part%209%20%E2%80%94%20Pods.png)

### Q&A

**Q1. What is a Pod?**
A Pod is the smallest deployable unit in Kubernetes — a wrapper around one or more containers that share the same network namespace (same IP address and port space) and can share storage volumes. All containers in a Pod are always scheduled and run together on the same node.

**Q2. Why doesn't Kubernetes deploy containers directly?**
Pods provide a consistent abstraction unit for the scheduler and allow tightly-coupled containers (e.g., an app and its sidecar logging agent) to share localhost networking and mounted volumes. This co-location model cannot be expressed by scheduling individual containers independently.

**Q3. Can a Pod contain multiple containers?**
Yes — this is the "sidecar" pattern. For example, a main application container and a log-forwarding container can share a Pod, communicating over localhost and sharing a log volume, while appearing as a single schedulable unit to Kubernetes.

---

## Part 10 — Deployments

**Execution location:** ☁️ EC2 SSH terminal

### File Created: `~/k8s/deployment.yaml`
```bash
mkdir -p ~/k8s && cd ~/k8s
nano deployment.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
  labels:
    app: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp
          image: awezsk/myapp:v1
          ports:
            - containerPort: 80
          imagePullPolicy: Always
```

### Commands Run
```bash
kubectl apply -f deployment.yaml
kubectl get deployments
kubectl get pods
kubectl delete pod myapp-deployment-<random-suffix>   # delete one to test self-healing
kubectl get pods                                       # replacement appears automatically
```

### Screenshots

![Part 10 Deployments - a](https://raw.githubusercontent.com/Awezsk/Devopsassignment2/main/Part%2010%20%E2%80%94%20Deployments-a.png)

![Part 10 Deployments - b](https://raw.githubusercontent.com/Awezsk/Devopsassignment2/main/Part%2010%20%E2%80%94%20Deployments-b.png)

### Q&A

**Q1. Why did the Pod return automatically?**
The Deployment continuously compares the actual running replica count against the declared desired state (replicas: 3). When a Pod is deleted and the count drops to 2, the ReplicaSet controller immediately creates a replacement to restore the declared count — this is Kubernetes's self-healing in action.

**Q2. Pod vs Deployment?**
A standalone Pod has no self-healing or scaling capabilities — if deleted, it is simply gone. A Deployment manages a ReplicaSet which enforces a desired number of identical Pod replicas, automatically healing failures, enabling declarative scaling, and supporting rolling updates and rollbacks.

**Q3. What is desired state?**
Desired state is the configuration you declare in the YAML manifest — for example, `replicas: 3`. Kubernetes treats this as a standing, continuously-enforced instruction and runs a control loop that constantly corrects any drift between what is running and what you declared should be running.

---

## Part 11 — Services

**Execution location:** ☁️ EC2 SSH terminal + 💻 Laptop (SSH tunnel) + 🌐 Browser

### File Created: `~/k8s/service.yaml`
```bash
cd ~/k8s
nano service.yaml
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```

### Commands Run
```bash
kubectl apply -f service.yaml
kubectl get svc
kill 5736                                          # free port 8090 from old pod forward
kubectl port-forward svc/myapp-service 8090:80     # Window A
```

### Screenshots

![Part 11 Services - a](https://raw.githubusercontent.com/Awezsk/Devopsassignment2/main/Part%2011%20%E2%80%94%20Services-a.png)

![Part 11 Services - b](https://raw.githubusercontent.com/Awezsk/Devopsassignment2/main/Part%2011%20%E2%80%94%20Services-b.png)

### Q&A

**Q1. Why do Pods need Services?**
Pod IP addresses are temporary and change every time a Pod is recreated. A Service provides a single stable IP address and DNS name that automatically load-balances incoming traffic across whatever Pods currently match its label selector — clients never need to know individual Pod IPs.

**Q2. ClusterIP vs NodePort?**
ClusterIP creates a virtual IP reachable only from within the cluster — suitable for internal service-to-service communication. NodePort additionally opens a fixed port (30000–32767 range) on every cluster node, making the service reachable from outside the cluster via `<node-ip>:<nodePort>`.

**Q3. What happens when a Pod IP changes?**
Nothing breaks from the client's perspective. The Service continuously watches for Pod changes via label selector matching and automatically updates its internal endpoints list. Traffic is seamlessly redirected to healthy Pods without any client-side change needed.

---

## Part 12 — Rolling Updates

**Execution location:** 💻 VS Code (edit) → 💻 Laptop terminal (git push) → ☁️ EC2 (build, push, deploy)

### Steps

**1. Edit `index.html` on laptop (VS Code):**
```html
<h1>Welcome to DevOps Training - v2!</h1>
```

**2. Push to GitHub (Laptop PowerShell):**
```powershell
git add .
git commit -m "v2 update"
git push
```

**3. Pull on EC2, build and push v2:**
```bash
cd ~/app && git pull
docker build -t awezsk/myapp:v2 .
docker push awezsk/myapp:v2
```

**4. Trigger rolling update:**
```bash
kubectl set image deployment/myapp-deployment myapp=awezsk/myapp:v2
kubectl rollout status deployment/myapp-deployment
kubectl get pods
```

### Screenshots

![Part 12 Rolling Updates - a](https://raw.githubusercontent.com/Awezsk/Devopsassignment2/main/Part%2012%20%E2%80%94%20Rolling%20Updates-a.png)

![Part 12 Rolling Updates - b](https://raw.githubusercontent.com/Awezsk/Devopsassignment2/main/Part%2012%20%E2%80%94%20Rolling%20Updates-b.png)

### Q&A

**Q1. What is a rolling update?**
A rolling update is a deployment strategy that replaces old Pod replicas with new ones gradually — one at a time — rather than stopping all instances simultaneously. Kubernetes creates new Pods with the updated image, waits for them to become healthy, then terminates old ones, cycling through until all replicas are updated.

**Q2. Why is it safer?**
Working replicas of the old version keep serving live traffic throughout the entire update process, meaning users experience no downtime. If the new version has a bug, only a fraction of traffic hits it before the rollout is paused or reversed — unlike a big-bang deployment where all instances fail simultaneously.

**Q3. What is zero-downtime deployment?**
Zero-downtime deployment means the Service always has at least one healthy Pod behind it at every moment during an update. Since Kubernetes adds new v2 Pods and confirms they are Running and Ready before removing v1 Pods, the service never drops below the minimum healthy count, making the update invisible to end users.

---

## Part 13 — Rollback

**Execution location:** ☁️ EC2 SSH terminal

### Commands Run
```bash
# Simulate a bad deployment
kubectl set image deployment/myapp-deployment myapp=awezsk/myapp:does-not-exist
kubectl get pods    # shows ImagePullBackOff

# Rollback to last known good version
kubectl rollout undo deployment/myapp-deployment
kubectl rollout status deployment/myapp-deployment
```

### Screenshot

![Part 13 Rollback](https://raw.githubusercontent.com/Awezsk/Devopsassignment2/main/Part%2013%20%E2%80%94%20Rollback.png)

### Q&A

**Q1. Why are rollbacks important?**
Rollbacks allow teams to instantly revert a broken release to the last known-good version, minimizing the time users are exposed to a defective deployment. Without rollback capability, fixing a bad deployment requires a full new build-test-deploy cycle, which could take hours.

**Q2. How does Kubernetes maintain availability during rollback?**
`kubectl rollout undo` is itself a rolling update in reverse — Kubernetes uses the same gradual replacement strategy, bringing up healthy Pods running the previous image version while terminating the broken ones. Enough healthy replicas remain running throughout the rollback to continue serving traffic uninterrupted.

---

## Part 14 — Jenkins + Kubernetes Integration (Full CI/CD Pipeline)

**Execution location:** ☁️ EC2 SSH terminal (kubeconfig setup) + 🌐 Jenkins UI + 💻 Browser (result verification)

### Step 1: Give Jenkins Access to Kubernetes (EC2 terminal)
```bash
sudo mkdir -p /var/lib/jenkins/.kube
sudo cp /home/ubuntu/.kube/config /var/lib/jenkins/.kube/config
sudo chown -R jenkins:jenkins /var/lib/jenkins/.kube

# Verify jenkins user can reach the cluster
sudo -u jenkins kubectl get nodes
```

### Step 2: Full 5-Stage Jenkinsfile
Updated in existing `myapp-pipeline` job via Jenkins UI → Configure:

```groovy
pipeline {
    agent any

    environment {
        DOCKERHUB_USER  = 'awezsk'
        IMAGE_NAME      = "${DOCKERHUB_USER}/myapp"
        IMAGE_TAG       = "v${BUILD_NUMBER}"
        DEPLOYMENT_NAME = 'myapp-deployment'
        CONTAINER_NAME  = 'myapp'
    }

    stages {

        stage('Clone Repository') {
            steps {
                git url: 'https://github.com/Awezsk/Devopsassignment2.git', branch: 'main'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh "kubectl set image deployment/${DEPLOYMENT_NAME} ${CONTAINER_NAME}=${IMAGE_NAME}:${IMAGE_TAG}"
            }
        }

        stage('Verify Deployment') {
            steps {
                sh "kubectl rollout status deployment/${DEPLOYMENT_NAME} --timeout=120s"
                sh "kubectl get pods -o wide"
            }
        }
    }

    post {
        success { echo 'Pipeline finished successfully -- app deployed!' }
        failure { echo 'Pipeline failed -- check the stage logs above.' }
    }
}
```

### Q&A

**Q1. How does Jenkins communicate with Kubernetes?**
Jenkins communicates with Kubernetes by running `kubectl` CLI commands in its pipeline shell steps. The `kubectl` binary uses the kubeconfig file copied to `/var/lib/jenkins/.kube/config`, which contains the cluster's API server address, TLS certificates, and authentication credentials — the same file the ubuntu user uses interactively.

**Q2. Why automate deployments?**
Automated deployments remove all the manual, error-prone steps that a developer would otherwise type by hand — eliminating wrong image tags, forgotten push steps, or missed kubectl commands. Every deployment follows the exact same verified sequence, is fully auditable through Jenkins build history, and can be triggered instantly by any authorized team member.

**Q3. What risks exist in manual deployments?**
Manual deployments carry risks of human error (wrong tag, skipped step), lack of audit trail (no record of exactly what was deployed when), environment inconsistency (different developers running slightly different command sequences), and difficulty reliably reproducing a rollback — all of which this automated pipeline eliminates.

---

## Troubleshooting Log

| Issue Encountered | Root Cause | Resolution |
|---|---|---|
| `docker run` — image not found | `docker build` hadn't been run yet; wrong directory | `cd` into correct folder, run `docker build -t myapp:v1 .` |
| Port 8080 conflict on laptop | Another service already using 8080 | Used `-p 8081:80` instead |
| Jenkins GPG key error | Jenkins rotated signing key in Dec 2025 | Used updated `jenkins.io-2026.key` and `/debian` repo path |
| Jenkins service crash loop | Java version / startup issue | Installed openjdk-17-jdk, reviewed `/var/log/jenkins/jenkins.log` |
| Jenkins UI not reachable (timeout) | Security Group IP was stale after IP change | Re-selected "My IP" in Security Group inbound rules for port 8080 |
| Jenkins node offline | `/tmp` tmpfs only 454 MB; Jenkins threshold was 1 GB | Masked `tmp.mount` and rebooted so `/tmp` uses real disk space |
| `kubectl port-forward` port conflict | Old port-forward (PID 5736) still running on 8090 | `kill 5736` to free the port before starting new forward |
| SSH tunnel not binding | Wrong `.pem` filename (`devops-key.pem` vs `devops-assignment.pem`) | Used correct filename `devops-assignment.pem` from Downloads folder |
| `--record` flag error in kubectl | Flag removed in current kubectl versions | Dropped `--record` from `kubectl set image` command |

---

## Deliverables Checklist

- [x] `Dockerfile` — at repo root (GitHub: Awezsk/Devopsassignment2)
- [x] `index.html` — served by nginx container
- [x] `~/k8s/deployment.yaml` — 3-replica Deployment on EC2
- [x] `~/k8s/service.yaml` — NodePort Service on EC2
- [x] Jenkinsfile — 5-stage pipeline in Jenkins UI
- [x] Docker Hub image pushed — `awezsk/myapp:v1`, `awezsk/myapp:v2`
- [x] All Q&A answered (Parts 1–14)
- [x] Screenshots attached for all parts
- [x] Architecture diagram (see top of document)
