# Prerequisites


## Installing Dependencies

### Quick Start with Homebrew

If you're on macOS or Linux with [Homebrew](https://brew.sh/) installed, you can install all dependencies in a single command:

```bash
brew install go node yarn typescript kubectl tilt kustomize
```
This installs:
- **go** - Go programming language
- **node** - Node.js runtime
- **yarn** - JavaScript package manager
- **typescript** - TypeScript compiler
- **kubectl** - Kubernetes CLI
- **tilt** - Tilt development tool
- **kustomize** - Kubernetes manifest customization
- **kind** - Kubernetes in Docker (local cluster)

### Code generation and tests
```bash
make install-test-tools-local
```
```bash
make install-codegen-tools-local
```

After installation, create a kind cluster if you don't already have a local cluster ready:

```bash
kind create cluster --name argocd-dev
```

Then verify everything is installed:

```bash
go version && node --version && yarn --version && tsc --version && \
kubectl version --client && tilt version && kustomize version && kind version
```

If all commands succeed, skip to [Getting Started](#getting-started)! Otherwise, follow the individual installation instructions below.

---
## Alternative ways to Install Dependencies

### 1. Go

Argo CD is written in Go. You'll need Go 1.21 or later.

**Installation:** See the [Go installation guide](https://go.dev/doc/install) for your platform.

**Verify installation:**
```bash
go version
```

### 2. Node.js and Yarn

Required for building and running the Argo CD UI.

**Installation:** 
- Node.js: See the [Node.js download page](https://nodejs.org/en/download/)
- Yarn: See the [Yarn installation guide](https://classic.yarnpkg.com/en/docs/install)

**Verify installation:**
```bash
node --version
yarn --version
```

### 3. TypeScript

TypeScript is used for the UI development.

**Installation:** See the [TypeScript installation guide](https://www.typescriptlang.org/download) or install via npm:
```bash
npm install -g typescript
```

**Verify installation:**
```bash
tsc --version
```

### 4. kubectl

Kubernetes command-line tool for interacting with your cluster.

**Installation:** See the [kubectl installation guide](https://kubernetes.io/docs/tasks/tools/#kubectl) for your platform.

**Verify installation:**
```bash
kubectl version --client
```

### 5. Tilt

Tilt orchestrates your development environment.

**Installation:** See the [Tilt installation guide](https://docs.tilt.dev/install.html) for your platform.

**Verify installation:**
```bash
tilt version
```

### 6. ctlptl
ctlptl (pronounced "cattle patrol") is a CLI for declaratively setting up local Kubernetes clusters.

**Installation:** See the [Github Readme](https://github.com/tilt-dev/ctlptl?tab=readme-ov-file#how-do-i-install-it) for your platform.

**Verify installation:**
```bash
ctlptl version
```

### 7. Kustomize

Used for customizing Kubernetes manifests.

**Installation:** See the [Kustomize installation guide](https://kubectl.docs.kubernetes.io/installation/kustomize/) for your platform.

**Verify installation:**
```bash
kustomize version
```

### 8. Kubernetes Cluster

You need a local Kubernetes cluster. Choose one of the following:

#### Option A: kind (Kubernetes IN Docker) - Recommended

**Installation:** See the [kind installation guide](https://kind.sigs.k8s.io/docs/user/quick-start/#installation).

Create a cluster:
```bash
ctlptl create cluster kind --registry=ctlptl-registry
```

#### Option B: k3d (Lightweight Kubernetes)

**Installation:** See the [k3d installation guide](https://k3d.io/stable/#installation).

Create a cluster:
```bash
ctlptl create cluster k3d --registry=ctlptl-registry
```

#### Option C: Minikube

**Installation:** See the [Minikube installation guide](https://minikube.sigs.k8s.io/docs/start/) for your platform.

Start Minikube:
```bash
ctlptl create cluster minikube --registry=ctlptl-registry
```

#### Option D: Docker Desktop

If you're using Docker Desktop, simply enable Kubernetes in the preferences/settings.

**Verify cluster access:**
```bash
kubectl cluster-info
kubectl get nodes
```