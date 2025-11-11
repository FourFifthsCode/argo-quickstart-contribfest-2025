# Argo: Configure Your Local Setup & Contribute!
Kubecon North America 2025 ContribFest

Join the Argo maintainers for an interactive session designed to jumpstart your contribution journey! This workshop is perfect for aspiring contributors and experienced users who are ready to dive into the Argo CD codebase and resolve issues, but need a helping hand with their local development environment.

## Prerequisite Tools

- **go** - Go programming language (version specified in `go.mod`)
- **node** - Node.js runtime
- **yarn** - JavaScript package manager
- **typescript** - TypeScript compiler
- **kubectl** - Kubernetes CLI
- **tilt** - Tilt development tool
- **kustomize** - Kubernetes manifest customization
- **kubernetes cluster** - Managed by kind, k3d, minikube, docker-desktop, etc.
- **ctlptl** - Helps create popular k8s clusters with preconfigured local image registries
- **protobuf** - Protoc binary for generating code from protobufs definitions

If you're on macOS or Linux with [Homebrew](https://brew.sh/) installed, you can install most dependencies in a single command:

```bash
brew install go node yarn typescript kubectl tilt ctlptl kustomize
```

### Create k8s cluster with local registry 
kind:
```bash
ctlptl create cluster kind --registry=ctlptl-registry
```
k3d:
```bash
ctlptl create cluster k3d --registry=ctlptl-registry
```

And then run the following make commands to install remaining dependencies

```bash
make install-test-tools-local
```
```bash
make install-codegen-tools-local
```

## Developing Locally

### Cloning the Repository

```bash
git clone https://github.com/argoproj/argo-cd.git
cd argo-cd
```

### Build, Deploy, and Test with Tilt
```bash
tilt up
```

To log in to Argo CD:
UI - http://localhost:4000

```bash
# Get the initial admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo

# Log in via CLI (after running 'make cli-local')
./dist/argocd login localhost:8080 --insecure --username admin

# Or use environment variables
export ARGOCD_SERVER=127.0.0.1:8080
export ARGOCD_OPTS="--plaintext --insecure"
```

### Make Commands (optional)

#### Building the Code

```bash
# Build all components with Docker
make build

# Build locally without Docker
make build-local

# Build CLI
make cli

# Build CLI locally
make cli-local
```

#### Running Code Generation

```bash
# Run all code generation with Docker
make codegen

# Run code generation locally
make codegen-local

# Fast codegen (without vendor)
make codegen-local-fast
```

#### Running Linting Tools

```bash
# Run linter with Docker
make lint

# Run linter locally
make lint-local

# Run UI linter
make lint-ui-local
```

## Tests

### 1. Unit Tests

Run all unit tests (includes health customization tests):

```bash
# With Docker
make test

# Without Docker
make test-local
```

Run tests for a specific module:

```bash
# With Docker
make test TEST_MODULE=github.com/argoproj/argo-cd/v3/controller

# Without Docker
make test-local TEST_MODULE=github.com/argoproj/argo-cd/v3/controller
```

Run specific tests with go test:

```bash
# Test a specific package
go test ./controller/...

# Run a specific test function
go test ./controller -run TestAppController

# Run with verbose output
go test -v ./controller/...

# Run with coverage
go test -cover ./controller/...
```

### 2. Health Customization Tests

Health customization tests are part of the unit test suite and run automatically with `make test`.

Run health tests specifically:

```bash
# All health tests
go test ./util/lua -run TestLuaHealthScript

# With verbose output
go test -v ./util/lua -run TestLuaHealthScript
```

### 3. E2E Tests

**Step 1:** Start e2e environment (in terminal 1):

```bash
# With Docker
make start-e2e

# Without Docker
make start-e2e-local
```

UI available at http://localhost:4000 (admin/password)

**Step 2:** Run e2e tests (in terminal 2):

```bash
# With Docker
make test-e2e

# Without Docker
make test-e2e-local
```

Run specific e2e tests:

```bash
go test ./test/e2e -run TestApplicationCreation -timeout 90m
```

## Additional Links
* [ArgoCD Developer Guide](https://argo-cd.readthedocs.io/en/latest/developer-guide/)
* [Slides](https://docs.google.com/presentation/d/1UPNAZyd_D1RQCgiWcNHfV2a8rpvGji9jEuuQSiaiSds)