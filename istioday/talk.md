# Validating your Istio Setups? The Tests Are Already Written

## Table of Contents

1. [Target Audience](#target-audience)
2. [Introduction](#introduction)
3. [Key Takeaways](#key-takeaways)
4. [Lightning Talk Structure](#lightning-talk-structure)
   - [1. Reusing Instead of Rewriting: The Case for Upstream Tests](#1-reusing-instead-of-rewriting-the-case-for-upstream-tests-2-minutes)
   - [2. Real-World Example: Validating Istio on OpenShift](#2-real-world-example-validating-istio-on-openshift-3-minutes)
   - [3. Real-World Example: Verifying External Control Plane with Sail Operator](#3-real-world-example-verifying-external-control-plane-with-sail-operator-3-minutes)
   - [4. Giving Back: From User to Contributor](#4-giving-back-from-user-to-contributor-2-minutes)
5. [Test Coverage Report & Selective Testing](#test-coverage-report--selective-testing)
6. [Appendix: Quick Reference](#appendix-quick-reference)
7. [Speaker Notes](#speaker-notes)

## Target Audience

**Target Audience:**
* Site Reliability Engineers (SREs)
* DevOps engineers
* Platform engineers
* Anyone involved in deploying, managing, and testing Istio service meshes.

Note: Basic understanding of Kubernetes and Istio concepts assumed.

## Lightning Talk Structure

### Introduction

Did you know that you can test your Istio deployments by reusing Istio's integration test framework? Instead of starting from scratch, I'll show how you can run these existing tests to verify that your setup works as expected. By leveraging what's already available, you can quickly catch problems, save time, and improve confidence in your environment. 
We'll also cover how your experience running these tests can feed back into the Istio project, helping improve coverage and reliability for everyone.

For anyone working with Istio, validating that things are working right—especially in non-standard environments—is essential. This talk shows how to use what's already there instead of building new tests from scratch. It's a simple but powerful way to save time, avoid duplication, and even give back to the project. We'll provide real examples like validating on OpenShift or running on Kubernetes with external control plane installers to show what this looks like in practice.

These are some of the key things you'll take away from this talk:
- **Understand the efficiency** of reusing existing Istio integration tests to validate complex, custom environments, avoiding redundant effort
- **Learn practical methods** for adapting Istio's integration tests to fit specific cluster configurations, including examples on OpenShift and with external control planes installations. Make it fit your environment with minimal changes.
- **Discover how contributing insights** from running these tests can directly enhance the upstream Istio project, benefiting the entire community

To illustrate this, here's a simple flowchart comparing the traditional approach of writing custom tests from scratch versus reusing the existing Istio test framework.

```
---
config:
  layout: dagre
---
flowchart LR
 subgraph subGraph0["The Hard Way (From Scratch)"]
        B("Write New Tests")
        A["You"]
        C["Custom Test Suite"]
        D(("Validate"))
        E["Result: Slow, <br>Redundant"]
  end
 subgraph subGraph1["The Smart Way (This Talk)"]
        G("Adapt/Reuse Tests")
        F["You"]
        H["Istio Integration Test Framework"]
        I(("Validate"))
        J["Result: Fast, <br> Confident"]
        K("Contribute Insights")
        L["Upstream Istio Project"]
  end
    A --> B
    B --> C
    C --> D
    D --> E
    F --> G
    G --> H
    H --> I
    I --> J
    J --> K
    K --> L
    M["Your Custom Environment"] --> A & F
```

### 1. Reusing Instead of Rewriting: The Case for Upstream Tests (2 minutes)

Validating a complex system like Istio is difficult and time-consuming

**The Problem with Custom Tests:**
- Hard to maintain and often miss critical edge cases
- Inconsistent validation across environments
- Time-consuming to develop comprehensive coverage
- Difficult to keep up with Istio evolution

**The Smarter Approach:** Reuse integration tests from the Istio project itself, with this a powerful framework already built to cover a wide range of scenarios in your own environment. This means you get:
- **Speed:** Leverage comprehensive test suite on day one without writing code. Minimizing setup time
- **Confidence:** Upstream tests cover vast functionality matrix—traffic shifting, policy enforcement, telemetry, multi-cluster communication. This ensures that your deployment behaves as expected.
- **Consistency:** Align validation with project's own quality gates, if it passes upstream, it should pass in your environment too.
- **Future-proof:** Rerun same tests after upgrades to catch regressions instantly.

**Demo #1:**
Prepare to run a basic Istio validation test against a standard Kubernetes cluster. We will need a kubernetes cluster running. The tests will install Istio automatically unless we specify otherwise. This means it will install Istio, run the tests, and then clean up.
```bash
# Basic validation in one command
go test -tags=integ ./tests/integration/pilot -run TestTraffic -v --istio.test.ci --test.timeout=30m
```

What does this do?
- Installs Istio core components (Istiod, ingress gateway)
- Deploys sample applications (httpbin, sleep)
- Runs traffic routing tests to ensure Istio is functioning correctly

You can also use a variety of flags to customize the test run:
```bash
--istio.test.kube.loadbalancer=false          # Use NodePort instead of LoadBalancer for ingress
--istio.test.nocleanup                        # Preserve test setup for post-mortem analysis
--istio.test.log_output_level=debug          # Enable verbose logging for troubleshooting
--istio.test.select +customsetup,-postsubmit  # Run only specific test categories
```

The complete list of flags is available in the Istio documentation [here](https://github.com/istio/istio/tree/master/tests/integration#command-line-flags).

**Interpreting Output:**
```bash
- Processing resources for Istio core.
✔ Istio core installed ⛵️
- Processing resources for Istiod.
- Processing resources for Istiod. Waiting for Deployment/istio-system/istiod
✔ Istiod installed 🧠
- Processing resources for Egress gateways, Ingress gateways.
- Processing resources for Egress gateways, Ingress gateways. Waiting for Deployment/istio-system/istio-egressgateway
- Processing resources for Egress gateways, Ingress gateways. Waiting for Deployment/istio-system/istio-egressgateway, Deployment/istio-system/istio-ingressgateway
✔ Egress gateways installed 🛫
- Processing resources for Ingress gateways. Waiting for Deployment/istio-system/istio-ingressgateway
✔ Ingress gateways installed 🛬
- Pruning removed resources
✔ Installation complete
...
TODO: add the successful test output example

```
### 2. Real-World Example: Validating Istio on OpenShift

**OpenShift Challenge:** Stricter security posture and unique networking constructs can pose challenges for Istio

#### Customizing the Test Run

**Minimal Configuration Changes for OpenShift:**
```bash
# Set OpenShift-specific kubeconfig
export KUBECONFIG=/path/to/openshift/config

# Adapt to Security Context Constraints (SCCs)
go test -tags=integ ./tests/integration/security -run TestMTLS \
  --istio.test.kube.config=$KUBECONFIG \
  --istio.test.openshift \
  --istio.test.istio.enableCNI=true \
  --istio.test.kube.helm.values=global.platform=openshift \
  --istio.test.ci
```

Adding some flags enable us to adapt the tests to OpenShift's unique requirements, such as using CNI for networking and adjusting Helm values for platform specifics. This can help us avoid common pitfalls.

#### What This Validates

- **Security Context Constraints** compliance
- **OpenShift Routes** integration with Istio gateways
- **Network policies** in OpenShift OVN
- **mTLS enforcement** under OpenShift's security model
- **Traffic routing** through OpenShift-specific ingress mechanisms
- **Service mesh** functionality under stricter security

### 3. Real-World Example: Verifying External Control Plane with Sail Operator (3 minutes)

**The Challenge:** Custom installers of Istio like Sail Operator - how can you be sure they deliver conformant Istio?
A few months ago was merged a feature that allow us to manage external Istio control plane installations using the existing Istio integration test. This is very powerful, for example on Sail Operator we can use this to validate that the Istio installation done by Sail Operator is conformant with upstream Istio.

To use this feature we just need to set:
* --istio.test.kube.deploy=false : This will skip the Istio installation step
* --istio.test.kube.controlPlaneInstaller=<path_to_your_installer> : This will point to the installer that will be used to manage the Istio control plane installation. For example: https://github.com/openshift-service-mesh/istio/blob/17574c9312708ac101c6026f2d5c9069fd580123/prow/setup/sail-operator-setup.sh

Take into account that skipping the installation mean that you need to handle all the related resources creation like for example the namespace creation, ingress gateway setup, istio cni if needed, etc.

### Installer script requirements

- The integration test runtime will invoke your control-plane installer script twice when --istio.test.kube.controlPlaneInstaller is set: once to perform the installation phase and once to perform cleanup. Your script must support both invocations.

- Install behavior: convert the upstream in-cluster operator configuration to the Sail Operator layout and create any required resources (namespaces, CRs, Istiod, Istio CNI if enabled, and gateway deployments such as istio-ingressgateway and istio-egressgateway). The installation step should be idempotent and tolerate existing resources.

- Cleanup behavior: remove the resources the installer created (istiod, istio-cni, istio-ingressgateway, istio-egressgateway, related namespaces/CRs) so the integration test can tear down reliably. Exit with appropriate success/failure codes so the test runner can detect outcomes.

- Logging and diagnostics: the script’s stdout/stderr is captured to the test working directory at <work_dir>/sail-operator-setup.log (set via --istio.test.work_dir). Emit sufficient logs for post-mortem debugging.

- See upstream documentation and an example installer here:
  https://github.com/openshift-service-mesh/istio/tree/master/tests/integration#running-tests-on-custom-deployment

The install process in your custom installer it will take the iop.yaml generated by the test framework and convert it to the custom format, then it will create the required resources. This is very useful because you can fully customize the Istio installation but still use the same base configuration that the test framework generates.


### 4. Giving Back: From User to Contributor (2 minutes)

**Your Experience is Valuable to the Entire Community**

#### Filing High-Quality Bug Reports

**Use Detailed Test Logs for Reproducible Reports:**
```bash
# Generate comprehensive failure report
go test -tags=integ ./tests/integration/pilot \
  --istio.test.ci > openshift-test-results.log 2>&1

# Upload to GitHub issue with:
# - Exact environment specifications
# - Complete test command used
# - Full error logs with context
# - Proposed solutions or workarounds
```

**Template for High-Quality Bug Reports:**
```markdown
## Environment
- Platform: OpenShift 4.12
- Istio: 1.19.0 via Sail Operator
- Test Command: `go test -tags=integ ./tests/integration/security`

## Failure
[Paste complete test output]

## Expected vs Actual
Expected: Test passes on standard Kubernetes
Actual: Fails on OpenShift due to SCC restrictions

## Proposed Solution
Add SCC-compatible test variant or configuration flag
```

#### Making Tests More Portable

**Simple Process for Contributing Improvements:**
```bash
# Found a test that makes platform-specific assumptions?
# 1. Fork istio/istio repository
git clone https://github.com/your-username/istio.git

# 2. Create small PR to make test more robust
# Example: Add conditional logic for OpenShift detection
if isOpenShift() {
    // Use route-based ingress validation
} else {
    // Use LoadBalancer-based validation
}

# 3. Submit PR with clear description of issue and fix
```

**Real Community Impact Examples:**
- **ARM64 compatibility** improvements from edge computing users
- **Air-gapped environment** adaptations from enterprise users
- **Custom CNI** support from platform teams
- **Security policy** enhancements from regulated industries

**This Feedback Loop Strengthens Istio for Everyone**

## Test Coverage Report & Selective Testing

Based on your note about creating a testing report, here's a framework for categorizing tests by scenario:

### Test Selection Matrix

| Scenario | Core Tests | Security Tests | Network Tests | Install Tests |
|----------|------------|----------------|---------------|---------------|
| **Basic Validation** | `pilot/TestBasic` | `security/TestMTLS` | - | - |
| **OpenShift** | `pilot/TestGateway` | `security/TestSCC` | `ambient/cni` | `helm/openshift` |
| **External Control Plane** | `pilot/TestVirtualService` | `security/TestMTLS` | `pilot/multicluster` | - |
| **Air-gapped** | `pilot/TestBasic` | `security/ca_custom_root` | - | `helm/offline` |
| **Multi-cluster** | `pilot/multicluster` | `security/external_ca` | `ambient/multinetwork` | - |

### Selective Test Execution Commands

```bash
# Basic environment validation (5-10 minutes)
go test -tags=integ ./tests/integration/pilot -run "TestBasic|TestVirtualService" \
  --istio.test.ci

# Security-focused validation (10-15 minutes)
go test -tags=integ ./tests/integration/security -run "TestMTLS|TestJWT" \
  --istio.test.ci

# Platform-specific validation (15-20 minutes)
go test -tags=integ ./tests/integration/... \
  --istio.test.select +customsetup,-postsubmit \
  --istio.test.ci

# Full conformance suite (30-45 minutes)
go test -tags=integ ./tests/integration/pilot ./tests/integration/security \
  --istio.test.ci
```

### Test Report Generation

```bash
# Generate detailed test report
go test -tags=integ ./tests/integration/... \
  --istio.test.ci \
  -json > test-results.json

# Parse results for coverage report
jq '.[] | select(.Action=="pass" or .Action=="fail") | {Test: .Test, Result: .Action}' \
  test-results.json > coverage-report.json
```

## Appendix: Quick Reference

### Essential Commands Cheat Sheet

```bash
# Basic setup validation
go test -tags=integ ./tests/integration/pilot -run TestBasic -v

# Security validation
go test -tags=integ ./tests/integration/security -run TestMTLS -v

# Custom environment discovery
go test -tags=integ ./tests/integration/... --istio.test.select +customsetup

# Preserve setup for analysis
go test -tags=integ [TEST_PATH] --istio.test.nocleanup

# Enable verbose diagnostics
go test -tags=integ [TEST_PATH] --istio.test.ci --log_output_level=debug

# Skip installation, use existing Istio
export ISTIO_TEST_SKIP_INSTALL=true
```

### Platform-Specific Configurations

```bash
# OpenShift
export KUBECONFIG=/path/to/openshift/config
go test -tags=integ ./tests/integration/ambient/cni --istio.test.ci

# External Control Plane
export ISTIO_TEST_SKIP_INSTALL=true
go test -tags=integ ./tests/integration/pilot --istio.test.ci

# Air-gapped Environment
go test -tags=integ ./tests/integration/security/ca_custom_root \
  --istio.test.hub=private-registry.com/istio
```

### Common Issues and Solutions

| Issue | Symptom | Solution |
|-------|---------|----------|
| **LoadBalancer unavailable** | `no ingress` errors | Install MetalLB: `kubectl apply -f metallb.yaml` |
| **ImagePullBackOff** | Pod startup failures | Configure registry: `--istio.test.hub=your-registry` |
| **SCC violations** | Pod security errors | Use CNI tests: `./tests/integration/ambient/cni` |
| **Custom CA issues** | mTLS failures | Run: `./tests/integration/security/ca_custom_root` |

## Speaker Notes

### Demo Preparation
- [ ] OpenShift cluster ready with appropriate access
- [ ] Sail Operator installation prepared
- [ ] Test commands verified on both environments
- [ ] GitHub issues page ready for contribution examples
- [ ] Backup commands for common failure scenarios

### Timing Guidelines
- **Section 1 (2 min):** Focus on benefits, keep technical details minimal
- **Section 2 (3 min):** Show real OpenShift failures and solutions
- **Section 3 (3 min):** Demonstrate API conformance with Sail Operator
- **Section 4 (2 min):** Emphasize community impact, keep specific

### Key Messages
1. **Don't rebuild what exists** - Leverage upstream investment
2. **Tests reveal environment specifics** - Turn challenges into insights
3. **Your experience benefits everyone** - Community-driven improvement
4. **Conformance matters** - Verify custom installations work correctly

---

*This lightning talk demonstrates the practical value of reusing Istio's integration tests while contributing back to strengthen the entire ecosystem.*
