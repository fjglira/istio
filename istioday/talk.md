# Validating your Istio Setups? The Tests Are Already Written

Note: Basic understanding of Kubernetes and Istio concepts assumed.

## Lightning Talk Structure

The structure of this lightning talk is as follows:

- Lightning Talk Topics Index

1. Opening (30s)
  - Welcome, context and goals
  - Expected audience prerequisites

2. High-level Motivation (1m)
  - Why reuse upstream integration tests
  - Benefits: speed, confidence, consistency, future-proofing

3. Reusing Instead of Rewriting — The Case for Upstream Tests (2m)

4. Real-World Example: Validating Istio on OpenShift (3m)

5. Real-World Example: External Control Plane (Sail Operator) (3m)

6. Installer Script Requirements (brief checklist) (1m)
  - Support install and cleanup invocations
  - Unchanged install behavior and cleanup semantics
  - Logging, diagnostics, and exit codes

7. Giving Back: From User to Contributor (1m)
  - Using test logs for high-quality bug reports
  - How to propose portability improvements and PRs


Total: ~10 minutes

### Introduction

Did you know that you can test your Istio deployments by reusing Istio's integration test framework? Instead of starting from scratch, I'll show how you can run these existing tests to verify that your setup works as expected. By leveraging what's already available, you can quickly catch problems, save time, and improve confidence in your environment. 
We'll also cover how your experience running these tests can feed back into the Istio project, helping improve coverage and reliability for everyone.

For anyone working with Istio, validating that things are working right, especially in non-standard environments is essential. This talk shows how to use what's already there instead of building new tests from scratch. It's a simple but powerful way to save time, avoid duplication, and even give back to the project. We'll provide examples like validating on OpenShift or running on Kubernetes with external control plane installers to show what this looks like in practice.

These are some of the key things you'll take away from this talk:
- **Understand the efficiency** of reusing existing Istio integration tests to validate complex, custom environments, avoiding redundant effort.
- **Learn practical methods** for adapting Istio's integration tests to fit specific cluster configurations, including examples on OpenShift and with external control planes installations. Make it fit your environment with minimal changes.
- **Discover how contributing insights** from running these tests can directly enhance the upstream Istio project, benefiting the entire community

To illustrate this, here's a simple flowchart comparing the traditional approach of writing custom tests from scratch versus reusing the existing Istio test framework (Is in the ppt slides):


### 1. Reusing Instead of Rewriting: The Case for Upstream Tests

**The Problem with Custom Tests:**
- Hard to maintain and often miss critical edge cases
- Inconsistent validation across environments
- Time-consuming to develop comprehensive coverage
- Difficult to keep up with Istio evolution

**The Smarter Approach:** Reuse integration tests from the Istio project itself, with this a powerful framework already built to cover a wide range of scenarios in your own environment. This means you get:
- **Speed:** Leverage comprehensive test suite on day one without writing code. Minimizing setup time
- **Confidence:** Upstream tests cover vast functionality matrix: traffic shifting, policy enforcement, telemetry, multi-cluster communication. This ensures that your deployment behaves as expected.
- **Consistency:** Align validation with project's own quality gates, if it passes upstream, it should pass in your environment too.
- **Future-proof:** Rerun same tests after upgrades to catch regressions instantly.

**Example**
Prepare to run a basic Istio validation test against a standard Kubernetes cluster. We will need a kubernetes cluster running. The tests will install Istio automatically unless we specify otherwise. This means it will install Istio, run the tests, and then clean up.
```bash
# Basic validation in one command
go test -tags=integ ./tests/integration/pilot -run TestTraffic -v --istio.test.ci --test.timeout=90m
```

What does this do?
- Installs Istio core components (Istiod, ingress gateway)
- Deploys sample applications (httpbin, sleep)
- Runs traffic routing tests to ensure Istio is working as expected on that cluster

You can also use a variety of flags to customize the test run:
```bash
--istio.test.kube.loadbalancer=false          # Use NodePort instead of LoadBalancer for ingress
--istio.test.nocleanup                        # Preserve test setup for post-mortem analysis
--log_output_level=tf:debug                   # Enable verbose logging for troubleshooting
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
    --- PASS: TestTraffic/virtualservice (42.33s)
        --- PASS: TestTraffic/virtualservice/added_header (1.76s)
            --- PASS: TestTraffic/virtualservice/added_header/a (0.60s)
                --- PASS: TestTraffic/virtualservice/added_header/a/to_b (0.19s)
                --- PASS: TestTraffic/virtualservice/added_header/a/to_tproxy (0.18s)
                --- PASS: TestTraffic/virtualservice/added_header/a/to_vm (0.22s)
            --- PASS: TestTraffic/virtualservice/added_header/tproxy (0.56s)
                --- PASS: TestTraffic/virtualservice/added_header/tproxy/to_b (0.18s)
                --- PASS: TestTraffic/virtualservice/added_header/tproxy/to_tproxy (0.18s)
                --- PASS: TestTraffic/virtualservice/added_header/tproxy/to_vm (0.19s)
            --- PASS: TestTraffic/virtualservice/added_header/vm (0.58s)
                --- PASS: TestTraffic/virtualservice/added_header/vm/to_b (0.20s)
                --- PASS: TestTraffic/virtualservice/added_header/vm/to_tproxy (0.18s)
                --- PASS: TestTraffic/virtualservice/added_header/vm/to_vm (0.19s)
        --- PASS: TestTraffic/virtualservice/set_header (1.60s)
```
### 2. Real-World Example: Validating Istio on OpenShift

**OpenShift Challenge:** Stricter security posture and unique networking constructs can pose challenges for Istio

#### Customizing the Test Run

**Minimal Configuration Changes for OpenShift:**
```bash
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

### 3. Real-World Example: Verifying External Control Plane

**The Challenge:** Custom installers of Istio like Sail Operator. How can you be sure they deliver conformant Istio?
A few months ago was merged a feature that allow us to manage external Istio control plane installations using the existing Istio integration test. This is very powerful, for example on Sail Operator we can use this to validate that the Istio installation done by Sail Operator is conformant with upstream Istio.

To use this feature we just need to set:
* `--istio.test.kube.deploy=false` : This will skip the Istio installation step
* `--istio.test.kube.controlPlaneInstaller=<path_to_your_installer>` : This will point to the installer that will be used to manage the Istio control plane installation. For example: https://github.com/openshift-service-mesh/istio/blob/17574c9312708ac101c6026f2d5c9069fd580123/prow/setup/sail-operator-setup.sh

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
When you run these tests in your unique environment, you may encounter issues or gaps in coverage. Sharing these insights helps improve Istio for everyone.

#### Filing High-Quality Bug Reports
This gaps that you may find can be reported to the Istio project to help improve the test coverage and reliability. Here are some tips for filing effective reports:

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

#### Making Tests More Portable

### Portability and platform agnostic

Istio is intended to be platform‑agnostic. Tests and production code should not hard‑wire platform‑specific behavior. Instead, where platform differences are required, prefer explicit, documented adapters or configuration flags so the same upstream tests and code paths can run across environments.

Recommended guidance:
- Avoid embedding platform-specific logic in core code or test cases; use feature flags, platform detectors, or test flags (e.g., --istio.test.openshift) to select adaptations.
- Expose platform differences via configuration (Helm/iop values, env vars, or test flags) rather than code branches.
- Implement small, well-documented adapters for platform-specific actions (CNI, route objects, SCCs) that the test harness can invoke when needed.
- When you find a portability gap, prefer making the test harness/configuration pluggable; if code changes are required, open a focused PR with tests.
- Document platform-specific requirements and recommended flags so users can reproduce test runs and file actionable bug reports.

These practices keep the upstream test suite reusable while allowing minimal, controlled adaptations to ensure Istio works reliably everywhere.

### Key Messages
1. **Don't rebuild what exists** - Leverage upstream investment
2. **Tests reveal environment specifics** - Turn challenges into insights
3. **Your experience benefits everyone** - Community-driven improvement
4. **Conformance matters** - Verify custom installations work correctly
