# Running Unit Testing

This guide provides quick instructions for running different types of tests in the Argo CD project.

## Prerequisites

Before running tests, ensure you have:
- Go installed
- A Kubernetes cluster
- Make installed

## Running Unit Tests

Unit tests validate individual components and functions without requiring external dependencies.

### Run All Unit Tests

```bash
make test-local
```

This will run all unit tests except e2e tests. Test results will be saved to the `test-results/` directory.

## Running Specific Unit Tests

You can run tests for a specific module or package using the `TEST_MODULE` variable.

### Test a Specific Module

```bash
make test-local TEST_MODULE=github.com/argoproj/argo-cd/v3/controller
```

### Test a Specific Package with go test

You can also use `go test` directly for more control:

```bash
# Test a specific package
go test ./controller/...

# Test a specific file
go test ./controller/appcontroller_test.go ./controller/appcontroller.go

# Run a specific test function
go test ./controller -run TestAppController

# Run tests with verbose output
go test -v ./controller/...

# Run tests with coverage
go test -cover ./controller/...
```

### Common go test Flags

- `-v`: Verbose output
- `-run <pattern>`: Run only tests matching the pattern
- `-count=1`: Disable test caching
- `-timeout <duration>`: Set test timeout (default: 10m)
- `-race`: Enable race detection
- `-cover`: Enable coverage reporting
- `-coverprofile=coverage.out`: Write coverage profile to file

### Examples

```bash
# Run all controller tests with verbose output
go test -v ./controller/...

# Run specific test function
go test -v ./util/lua -run TestLuaHealthScript

# Run tests without cache
go test -count=1 ./controller/...

# Run tests with coverage
go test -cover -coverprofile=coverage.out ./controller/...
go tool cover -html=coverage.out  # View coverage in browser
```

## Running Health Customization Tests

Health customization tests validate Lua scripts used for custom resource health assessments. These tests are located in the `resource_customizations/` directory.

### How Health Tests Work

Each resource customization in `resource_customizations/<group>/<kind>/` may contain:
- `health.lua`: Lua script defining health assessment logic
- `health_test.yaml`: Test cases for the health script
- `testdata/`: Directory containing test resource manifests

### Run Health Customization Tests

The health tests are part of the standard unit test suite:

```bash
# Run health tests with Docker
make test TEST_MODULE=github.com/argoproj/argo-cd/v3/util/lua

# Run health tests locally
go test ./util/lua -run TestLuaHealthScript

# Run with verbose output to see each test case
go test -v ./util/lua -run TestLuaHealthScript
```

### Test Structure Example

Here's the structure for a health customization test:

**`resource_customizations/apps/Deployment/health_test.yaml`:**
```yaml
tests:
- healthStatus:
    status: Healthy
    message: "Deployment is healthy"
  inputPath: testdata/healthy.yaml
- healthStatus:
    status: Degraded
    message: "Deployment has failed pods"
  inputPath: testdata/degraded.yaml
```

### Adding New Health Customization Tests

1. Create or modify the `health.lua` script in your resource directory
2. Create a `health_test.yaml` file with test cases
3. Add test resource manifests in the `testdata/` subdirectory
4. Run the tests to validate your health script

```bash
# Test your new health customization
go test ./util/lua -run TestLuaHealthScript -v
```

### Running Custom Action Tests

Similarly, custom actions can be tested:

```bash
go test ./util/lua -run TestLuaCustomAction -v
```