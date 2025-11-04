# Developing Argo CD with Tilt

This guide will help you set up and use [Tilt](https://tilt.dev) for Argo CD development, providing a modern, fast inner development loop with live updates and an intuitive web UI.

## Table of Contents

- [Why Tilt?](#why-tilt)
- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
- [Using Tilt](#using-tilt)
  - [Tilt Web UI](#tilt-web-ui)
  - [Live Updates](#live-updates)
  - [Debugging](#debugging)
  - [Quick Actions](#quick-actions)
- [Development Workflow Comparison](#development-workflow-comparison)
- [Troubleshooting](#troubleshooting)

## Why Tilt?

Tilt is a modern development tool that orchestrates your development environment to provide fast, reliable feedback loops. Unlike traditional approaches where you manually build, push, and deploy changes, Tilt automates this entire workflow and intelligently updates only what changed.

For Argo CD development, Tilt offers:

- **Real-time feedback**: See your code changes reflected in seconds, not minutes
- **Visual development environment**: A web UI that shows the status of all services at a glance
- **Smart rebuilds**: Only rebuilds and updates what actually changed
- **Integrated debugging**: Built-in support for remote debugging with Delve
- **Complete isolation**: Runs Argo CD in a real Kubernetes cluster, not just processes on your machine
- **One command setup**: `tilt up` starts everything you need

## Prerequisites

Before you can use Tilt for Argo CD development, you'll need several tools installed on your system.

### Installing Dependencies

#### Quick Start with Homebrew

If you're on macOS or Linux with [Homebrew](https://brew.sh/) installed, you can install all dependencies in a single command:

```bash
brew install go node yarn typescript kubectl tilt kustomize kind
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

#### 1. Go

Argo CD is written in Go. You'll need Go 1.21 or later.

**Installation:** See the [Go installation guide](https://go.dev/doc/install) for your platform.

**Verify installation:**
```bash
go version
```

#### 2. Node.js and Yarn

Required for building and running the Argo CD UI.

**Installation:** 
- Node.js: See the [Node.js download page](https://nodejs.org/en/download/)
- Yarn: See the [Yarn installation guide](https://classic.yarnpkg.com/en/docs/install)

**Verify installation:**
```bash
node --version
yarn --version
```

#### 3. TypeScript

TypeScript is used for the UI development.

**Installation:** See the [TypeScript installation guide](https://www.typescriptlang.org/download) or install via npm:
```bash
npm install -g typescript
```

**Verify installation:**
```bash
tsc --version
```

#### 4. kubectl

Kubernetes command-line tool for interacting with your cluster.

**Installation:** See the [kubectl installation guide](https://kubernetes.io/docs/tasks/tools/#kubectl) for your platform.

**Verify installation:**
```bash
kubectl version --client
```

#### 5. Tilt

The star of the show - Tilt orchestrates your development environment.

**Installation:** See the [Tilt installation guide](https://docs.tilt.dev/install.html) for your platform.

**Verify installation:**
```bash
tilt version
```

#### 6. Kustomize

Used for customizing Kubernetes manifests.

**Installation:** See the [Kustomize installation guide](https://kubectl.docs.kubernetes.io/installation/kustomize/) for your platform.

**Verify installation:**
```bash
kustomize version
```

#### 7. Kubernetes Cluster

You need a local Kubernetes cluster. Choose one of the following:

##### Option A: kind (Kubernetes IN Docker) - Recommended

**Installation:** See the [kind installation guide](https://kind.sigs.k8s.io/docs/user/quick-start/#installation).

Create a cluster:
```bash
kind create cluster --name argocd-dev
```

##### Option B: k3d (Lightweight Kubernetes)

**Installation:** See the [k3d installation guide](https://k3d.io/stable/#installation).

Create a cluster:
```bash
k3d cluster create argocd-dev
```

##### Option C: Minikube

**Installation:** See the [Minikube installation guide](https://minikube.sigs.k8s.io/docs/start/) for your platform.

Start Minikube:
```bash
minikube start --profile argocd-dev
```

##### Option D: Docker Desktop

If you're using Docker Desktop, simply enable Kubernetes in the preferences/settings.

**Verify cluster access:**
```bash
kubectl cluster-info
kubectl get nodes
```

## Getting Started

Once you have all prerequisites installed, starting Argo CD with Tilt is straightforward:

### 1. Navigate to the Argo CD repository

```bash
cd /path/to/argo-cd
```

### 2. Start Tilt

```bash
tilt up
```

On first run, Tilt will:
- Detect your cluster architecture (amd64/arm64)
- Build the Argo CD binary for your cluster architecture
- Install dependencies (yarn packages for UI)
- Build Docker images for all services
- Deploy Kubernetes manifests to your cluster
- Set up port forwarding
- Start the Tilt web UI

### 3. Access the Tilt UI

Tilt will automatically open your browser when you press "space" in the terminal to `http://localhost:10350`. If it doesn't, manually navigate to that URL.

### 4. Wait for resources to become ready

In the Tilt UI, you'll see all resources building and deploying. Wait until most resources show green (healthy) status. The first build can take several minutes.

### 5. Access Argo CD

Once ready, you can access:

- **Argo CD UI**: http://localhost:4000
- **Argo CD API**: http://localhost:8080
- **Redis**: localhost:6379

To log in to Argo CD:

```bash
# Get the initial admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo

# Log in via CLI
argocd login localhost:8080 --insecure --username admin

# Or use environment variables
export ARGOCD_SERVER=127.0.0.1:8080
export ARGOCD_OPTS="--plaintext --insecure"
```

## Using Tilt

### Tilt Web UI

The Tilt web UI (`http://localhost:10350`) is your command center:

**Resource List**: Shows all services, builds, and Kubernetes resources
- Green: Healthy and up-to-date
- Yellow: Building or updating
- Red: Errors or failures
- Gray: Disabled or not started

**Logs**: Click any resource to see its logs in real-time

**Endpoints**: Quick links to forwarded ports (API, UI, Redis, etc.)

**Buttons**: Custom action buttons in the top navigation:
- `make codegen-local`: Regenerate code from protobuf/OpenAPI specs
- `make test-local`: Run unit tests
- `make cli-local`: Build the argocd CLI binary

### Live Updates

Tilt's biggest feature is live updates. When you change code:

#### Go Code Changes (Backend)

1. Edit any `.go` file in the codebase
2. Tilt detects the change
3. The `build` resource rebuilds the binary (seconds)
4. Binary is synced to all running containers
5. Services automatically restart with new code
6. Check logs in Tilt UI to verify changes

**Tip**: The build uses debug flags (`-gcflags="all=-N -l"`) so you can attach debuggers.

#### UI Changes (Frontend)

1. Edit any `.tsx`, `.ts`, or `.scss` file in `ui/`
2. Webpack automatically rebuilds (running inside the container)
3. Browser auto-refreshes with changes (hot module reload)
4. No restart required!

#### Manifest Changes

1. Edit Kubernetes manifests in `manifests/dev-tilt/`
2. Tilt automatically applies changes to the cluster
3. Pods may restart depending on what changed

### Debugging

All Argo CD services run with [Delve](https://github.com/go-delve/delve) debugger attached. You can connect your IDE's debugger to any service:

| Service | Debug Port | Metrics Port |
|---------|------------|--------------|
| argocd-server | 9345 | 8083 |
| argocd-repo-server | 9346 | 8084 |
| argocd-applicationset-controller | 9347 | 7000 |
| argocd-application-controller | 9348 | 8086 |
| argocd-notifications-controller | 9349 | 8087 |
| argocd-commit-server | 9350 | 8088 |

#### VS Code Debug Configuration

Add to `.vscode/launch.json`:

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Connect to server",
            "type": "go",
            "request": "attach",
            "mode": "remote",
            "remotePath": "${workspaceFolder}",
            "port": 9345,
            "host": "127.0.0.1"
        },
        {
            "name": "Connect to repo-server",
            "type": "go",
            "request": "attach",
            "mode": "remote",
            "remotePath": "${workspaceFolder}",
            "port": 9346,
            "host": "127.0.0.1"
        },
        {
            "name": "Connect to applicationset-controller",
            "type": "go",
            "request": "attach",
            "mode": "remote",
            "remotePath": "${workspaceFolder}",
            "port": 9347,
            "host": "127.0.0.1"
        },
        {
            "name": "Connect to application-controller",
            "type": "go",
            "request": "attach",
            "mode": "remote",
            "remotePath": "${workspaceFolder}",
            "port": 9348,
            "host": "127.0.0.1"
        },
        {
            "name": "Connect to notifications-controller",
            "type": "go",
            "request": "attach",
            "mode": "remote",
            "remotePath": "${workspaceFolder}",
            "port": 9349,
            "host": "127.0.0.1"
        },
        {
            "name": "Connect to commit-server",
            "type": "go",
            "request": "attach",
            "mode": "remote",
            "remotePath": "${workspaceFolder}",
            "port": 9350,
            "host": "127.0.0.1"
        }
    ]
}
```

#### GoLand / IntelliJ Debug Configuration
Add a `.run/remote-debugging.run.xml` file with these configurations to support attaching to running pods corresponding to the service.
```xml
<component name="ProjectRunConfigurationManager">
    <configuration default="false" name="Connect to server" type="GoRemoteDebugConfigurationType" factoryName="Go Remote" focusToolWindowBeforeRun="true" port="9345">
        <option name="disconnectOption" value="LEAVE" />
        <disconnect value="LEAVE" />
        <method v="2" />
    </configuration>
    <configuration default="false" name="Connect to repo-server" type="GoRemoteDebugConfigurationType" factoryName="Go Remote" focusToolWindowBeforeRun="true" port="9346">
        <option name="disconnectOption" value="LEAVE" />
        <disconnect value="LEAVE" />
        <method v="2" />
    </configuration>
    <configuration default="false" name="Connect to applicationset-controller" type="GoRemoteDebugConfigurationType" factoryName="Go Remote" focusToolWindowBeforeRun="true" port="9347">
        <option name="disconnectOption" value="LEAVE" />
        <disconnect value="LEAVE" />
        <method v="2" />
    </configuration>
    <configuration default="false" name="Connect to application-controller" type="GoRemoteDebugConfigurationType" factoryName="Go Remote" focusToolWindowBeforeRun="true" port="9348">
        <option name="disconnectOption" value="LEAVE" />
        <disconnect value="LEAVE" />
        <method v="2" />
    </configuration>
    <configuration default="false" name="Connect to notifications-controller" type="GoRemoteDebugConfigurationType" factoryName="Go Remote" focusToolWindowBeforeRun="true" port="9349">
        <option name="disconnectOption" value="LEAVE" />
        <disconnect value="LEAVE" />
        <method v="2" />
    </configuration>
    <configuration default="false" name="Connect to commit-server" type="GoRemoteDebugConfigurationType" factoryName="Go Remote" focusToolWindowBeforeRun="true" port="9350">
        <option name="disconnectOption" value="LEAVE" />
        <disconnect value="LEAVE" />
        <method v="2" />
    </configuration>
</component>
```

### Quick Actions

The Tilt UI includes buttons for common development tasks:

#### make codegen-local

Regenerates code from:
- Protobuf definitions
- OpenAPI specs
- Kubernetes clientsets
- CRD manifests

Run this after modifying `.proto` files or API definitions.

#### make test-local

Runs the unit test suite. Useful for quick validation before committing.

#### make cli-local

Builds the `argocd` CLI binary locally (not in a container). The binary will be at `dist/argocd`.

## Troubleshooting

### Tilt can't detect my cluster

**Problem**: Tilt shows "cluster not found" or similar error.

**Solution**:
```bash
# Check your kubectl context
kubectl config current-context

# If wrong context, switch to it
kubectl config use-context <your-cluster-context>

# Verify you can access the cluster
kubectl get nodes
```

### Build fails with architecture mismatch

**Problem**: Binary built for wrong architecture (amd64 vs arm64).

**Solution**: The Tiltfile auto-detects architecture from your cluster. Ensure your cluster architecture matches your development machine, or use a cluster that supports multi-arch (like kind).

### Resources stuck in "Pending" or "CrashLoopBackOff"

**Problem**: Pods won't start.

**Solution**:
1. Click the resource in Tilt UI to see logs
2. Check for common issues:
   - Image pull failures (Docker not running, wrong image)
   - Resource limits (insufficient CPU/memory in cluster)
   - Missing dependencies (ConfigMaps, Secrets)
3. Check pod status:
```bash
kubectl get pods -n argocd
kubectl describe pod <pod-name> -n argocd
```

### Port conflicts

**Problem**: "Port already in use" errors.

**Solution**:
1. Check what's using the port:
```bash
lsof -i :8080  # or whichever port is conflicting
```
2. Stop the conflicting process or change the port in the Tiltfile
3. Common conflicts:
   - 8080: Another Argo CD instance or web server
   - 4000: Another webpack dev server
   - 6379: Local Redis instance

### Tilt is slow or unresponsive

**Problem**: Tilt UI is sluggish or builds are slow.

**Solution**:
1. Increase Docker resources (CPU/Memory in Docker Desktop settings)
2. Ensure your cluster has sufficient resources:
```bash
kubectl top nodes  # requires metrics-server
```
3. Disable resources you're not working on:
```bash
tilt up -- --only argocd-server
```
4. Clean up old resources:
```bash
tilt down
docker system prune -a
```

### Changes not reflecting

**Problem**: Code changes aren't showing up.

**Solution**:
1. Check the Tilt UI - is the build/update running?
2. Verify the file you changed is in the `deps` list in the Tiltfile
3. Check if the service restarted (look for restart message in logs)
4. For UI changes, hard refresh your browser (Cmd+Shift+R / Ctrl+Shift+R)
5. For Go changes, verify the binary was synced (check Tilt logs for "sync" messages)

### Clean slate restart

If things are really broken:

```bash
# Clean up all previously install resources Tilt
tilt down

# Clean up Kubernetes resources
kubectl delete namespace argocd
kubectl create namespace argocd

# Clean Docker
docker system prune -a

# Restart Tilt
tilt up
```

---

## Additional Resources

- [Tilt Documentation](https://docs.tilt.dev/)
- [Argo CD Developer Guide](https://argo-cd.readthedocs.io/en/latest/developer-guide/)
- [Running Argo CD Locally (Goreman)](https://argo-cd.readthedocs.io/en/latest/developer-guide/running-locally/)
- [Development Environment Setup](https://argo-cd.readthedocs.io/en/latest/developer-guide/development-environment/)
- [Toolchain Guide](https://argo-cd.readthedocs.io/en/latest/developer-guide/toolchain-guide/)
- [Development With Tilt](https://argo-cd.readthedocs.io/en/latest/developer-guide/tilt/)

## Contributing

If you find issues with the Tilt setup or have suggestions for improvements, please open an issue or PR in the [Argo CD repository](https://github.com/argoproj/argo-cd).

Happy developing! 🚀
