## Running E2E Tests

End-to-end tests validate complete workflows with real Argo CD components running.

### Prerequisites for E2E Tests

- Running Kubernetes cluster (e.g., k3d, kind, minikube)
- `kubectl` configured to access the cluster

### Step 1: Start E2E Test Server

First, start the Argo CD services in e2e mode:

```bash
make start-e2e-local
```

This will:
- Create `argocd-e2e*` namespaces
- Start all Argo CD components (API server, repo server, controller, etc.)
- Start supporting services (Redis, Dex)
- Set up test fixtures

The UI will be available at http://localhost:4000 with credentials:
- Username: `admin`
- Password: `password`

### Step 2: Run E2E Tests

In a **separate terminal**, run the e2e tests:

```bash
make test-e2e
```

Or with the local toolchain:
```bash
make test-e2e-local
```

### E2E Test Configuration

You can customize the e2e test environment with environment variables:

```bash
# Custom ports (set before running make start-e2e)
export ARGOCD_E2E_APISERVER_PORT=8080
export ARGOCD_E2E_REPOSERVER_PORT=8081
export ARGOCD_E2E_REDIS_PORT=6379
export ARGOCD_E2E_DEX_PORT=5556

# Custom test timeout
export ARGOCD_E2E_TEST_TIMEOUT=120m

# Then start e2e services
make start-e2e
```

If you changed the API server port, update the server address:
```bash
export ARGOCD_SERVER=localhost:8888
make test-e2e
```

### Running Specific E2E Tests

```bash
# Run specific e2e test
go test ./test/e2e -run TestApplicationCreation -timeout 90m

# Run with verbose output
go test -v ./test/e2e -timeout 90m
```

### E2E Test Isolation

Each test gets:
- A random 5-character ID
- A unique Git repository in `/tmp/argo-e2e/${id}`
- A dedicated namespace `argocd-e2e-ns-${id}`
- A unique app name `argocd-e2e-${id}`

### Debugging E2E Tests

**Shell into test server container:**
```bash
make debug-test-server
```

**Shell into test client container:**
```bash
make debug-test-client
```

## Test Results and Coverage

### Viewing Test Results

Test results are saved in JUnit XML format:
```bash
cat test-results/junit.xml
```

## Additional Resources

- [E2E Tests Documentation](test-e2e.md)
- [Development Environment Setup](development-environment.md)
- [Toolchain Guide](toolchain-guide.md)
- [Contributing Guidelines](code-contributions.md)