# Openshift-virtualization-tests Test plan

## **Multi-Architecture Support - Quality Engineering Plan**

### **Metadata & Tracking**

- **Enhancement(s):**
  [VEP: Heterogeneous Cluster Support](https://github.com/kubevirt/enhancements/blob/main/veps/sig-storage/dic-on-heterogeneous-cluster/dic-on-heterogeneous-cluster.md)
- **Feature Tracking:** [VIRTSTRAT-494](https://redhat.atlassian.net/browse/VIRTSTRAT-494)
- **Epic Tracking:**
  - [CNV-67900](https://redhat.atlassian.net/browse/CNV-67900) — HCO multi-arch support
  - [CNV-26818](https://redhat.atlassian.net/browse/CNV-26818) — Multi-arch worker node support
  - [CNV-76714](https://redhat.atlassian.net/browse/CNV-76714) — Infra multi-arch support
  - [CNV-76732](https://redhat.atlassian.net/browse/CNV-76732) — Storage multi-arch support
  - [CNV-76741](https://redhat.atlassian.net/browse/CNV-76741) — Network multi-arch support
  - [CNV-61832](https://redhat.atlassian.net/browse/CNV-61832) — UI multi-arch support
- **QE Owner(s):** Haim Meir (hmeir@redhat.com)
- **Owning SIG:** sig-iuo
- **Participating SIGs:** sig-virt, sig-infra, sig-storage, sig-network

**Document Conventions:**

- **Multi-Arch:** Multi-Architecture, heterogeneous clusters with mixed CPU architectures (AMD64 + ARM64).
- **Boot Source:** A pre-configured VM image (also known as a golden image) that is automatically imported and made available for VM creation per supported architecture.

### **Feature Overview**

Multi-architecture support enables OpenShift Virtualization to operate on heterogeneous clusters with mixed AMD64 and ARM64 worker nodes.
This feature targets customers adopting ARM-based infrastructure alongside existing AMD64 deployments.

---

### **I. Motivation and Requirements Review (QE Review Guidelines)**

#### **1. Requirement & User Story Review Checklist**

- [x] **Review Requirements**
  - *List the key D/S requirements reviewed:*
    - Architecture-aware VM scheduling; users can select a VMs target architecture
    - All VM-related operations (lifecycle, migration, networking, storage) are supported on both architectures
    - Workload observability across architectures
    - Existing VMs continue to be fully functional without regression

- [x] **Understand Value and Customer Use Cases**
  - *Describe the feature's value to customers:* Customers can run ARM64 and AMD64 VMs in a single cluster,
     reducing operational overhead and enabling gradual ARM adoption.
  - *List the customer use cases identified:*
    - A VM admin deploys VMs on ARM64 nodes from available boot sources after ARM64 nodes are added to the cluster.

- [x] **Testability**
  - *Note any requirements that are unclear or untestable:*
    All requirements are testable on a heterogeneous AWS cluster.

- [x] **Acceptance Criteria**
  - *List the acceptance criteria:*
    - Boot sources are available per supported architecture when multi-arch support is enabled
    - Fully functional, running VM on an AMD64 node in a heterogeneous cluster with no behavioral difference from a homogeneous
      cluster
    - Fully functional, running VM on an ARM64 node in a heterogeneous cluster with no behavioral difference from a homogeneous
      cluster
    - Same-architecture live migration completes successfully (ARM64→ARM64 and AMD64→AMD64)
    - Cross-architecture live migration is rejected with a clear, user-facing error message
    - Existing AMD64 workloads continue to operate without disruption after enabling multi-arch support
    - Multi-arch configuration is preserved across cluster upgrades
    - Adding a node of a different architecture to a homogeneous cluster makes architecture-specific boot sources available for that architecture
    - Removing all nodes of an architecture from a heterogeneous cluster cleans up the corresponding architecture-specific boot sources
    - Existing templates referencing boot sources continue to function after enabling multi-arch support
    - Custom (user-provided) boot sources work for VM creation on both architectures  -hmeir - iirc, this will not work out of the box - if so, move to feature limitations and add that needs to be documented
    - Network connectivity between VMs running on different architectures works correctly
    - Alerts fire when boot image provisioning fails for a specific architecture

  - *Note any gaps or missing criteria:* None.

- [x] **Non-Functional Requirements (NFRs)**
  - *List applicable NFRs and their targets:*
    - Monitoring: alerts for boot image provisioning failures
    - Observability: workload metrics across both architectures
  - *Note any NFRs not covered and why:* Performance/scale deferred to a future cycle.

#### **2. Known Limitations**

The limitations are documented to ensure alignment between development, QA, and product teams.
The following topics will not be tested or supported.

- Cross-architecture live migration is not supported (AMD64↔ARM64)
- Multi-arch support is opt-in in 4.22 (default-enabled in 4.23+)
- hmeir: What is the mitigation for already running VMs on a cluster that becomes heterogeneous? (e.g., existing AMD64 VMs when ARM64
  nodes are added)

#### **3. Technology and Design Review**

- **Developer Handoff/QE Kickoff**
  - *Key takeaways and concerns:*
    - Cross-SIG kickoff conducted with all 5 participating SIGs
    - Each SIG confirmed ownership of their multi-arch test scope

- **Technology Challenges**
  - *List identified challenges:*
    - Heterogeneous cluster provisioning requires AWS with Graviton instances
    - CPU model configuration per architecture needs validation (hmeir: please check with the virt team)
  - *Impact on testing approach:*
    - Tests are limited to AWS platform

- **API Extensions**
  - *List new or modified APIs:*
    - `spec.template.spec.architecture` — new field on VM spec to target a specific CPU architecture
    - Multi-arch support enable/disable configured through HCO
  - *Testing impact:*
    - VM creation with explicit architecture targeting must be tested.
    - Enable/disable behavior must be tested.

- **Test Environment Needs**
  - *See environment requirements in Section II.3 and testing tools in Section II.3.1*

- **Topology Considerations**
  - *Describe topology requirements:* Multi-node cluster with at least 2 AMD64 and 2 ARM64 workers.
  - *Impact on test design:* Live migration tests require at least 2 nodes of the same architecture.

### **II. Software Test Plan (STP)**

#### **1. Scope of Testing**

Tests verify that OpenShift Virtualization operates correctly on heterogeneous clusters with AMD64 and ARM64 nodes.
AMD64 and ARM64 are already tested as a homogeneous cluster; this STP focuses on heterogeneous-specific behavior and regression validation.

**Testing Goals**

New functional flows specific to heterogeneous multi-arch clusters:

- **[P0]** Verify cross-architecture live migration is blocked with a clear, user-facing error message
- **[P0]** Enable multi-arch support and verify boot sources become available for each supported architecture
- **[P0]** Disable multi-arch support and verify architecture-specific boot sources are removed - hmeir: what is the risk here vs the value? consider moving to out of scope +explain why
- **[P1]** Drain an ARM64 node and verify VMs migrate only to other ARM64 nodes, not to AMD64 nodes
- **[P0]** Verify network connectivity between VMs running on different architectures (ARM64 VM ↔ AMD64 VM)
- **[P1]** Verify alerts fire when boot image provisioning fails for a specific architecture
- **[P1]** Verify workload metrics distinguish between AMD64 and ARM64 VM workloads - hmeir: relevant?
- **[P2]** Verify multi-arch configuration and VMs are preserved across cluster upgrade
- **[P0]** Add/remove ARM64 nodes and verify boot sources update accordingly - hmeir: what is the risk here vs the value? consider moving to out of scope +explain why
- **[P1]** Verify existing VMs and custom templates that reference boot sources continue to work after enabling multi-arch support
- **[P1]** Verify custom (user-provided) boot sources work on both architectures  (as mentioned above - limitation?)

**Regression Goals**

Existing test suites to run on the heterogeneous cluster to verify no degradation:

- **[P0]** sig-virt Tier 1 and Tier 2 on both architectures
- **[P0]** sig-iuo Tier 1 and Tier 2
- **[P1]** sig-network Tier 1 and Tier 2 (currently zero ARM64 coverage)
- **[P1]** sig-storage Tier 1 and Tier 2 (currently zero ARM64 coverage)
- **[P1]** sig-infra Tier 1 and Tier 2
- **[P2]** Observability tests on multi-arch cluster (currently zero ARM64 coverage)

**Per-SIG Testing Responsibilities**

> **NOTE:** Each SIG is responsible for ensuring their Tier 1 and Tier 2 executed regression suites cover all scenarios
> that may be affected by the multi-arch feature on heterogeneous clusters.

- **sig-iuo (owning SIG):** Multi-arch support configuration, boot image provisioning lifecycle, alerts. Owns overall coordination.
  - *Regression:*
    - Tier 1 and Tier 2 (iuo and observability)

- **sig-virt:** VM deployment on ARM64 and AMD64 nodes, same-arch live migration, cross-arch blocking, scheduling, CPU model per architecture.
  - *Regression:*
    - Tier 1 and Tier 2

- **sig-infra:** Architecture-specific VM templates and backward compatibility, instance types.
  - *Regression:* Tier 1 and Tier 2

- **sig-storage:** Boot image import per architecture — verify correct images are imported and available for each supported architecture.
  - *Regression:* Tier 1 and Tier 2

- **sig-network:** Network connectivity between VMs on different architectures.
  - *Regression:* Tier 1 and Tier 2

**Out of Scope (Testing Scope Exclusions)**

- [ ] **Performance/scale testing**
  - *Rationale:* Initial focus is functional correctness.
  - *PM/Lead Agreement:* [Name/Date]

- [ ] **s390x architecture**
  - *Rationale:* s390x architecture is not in the scope of this feature.
  - *PM/Lead Agreement:* [Name/Date]

- [ ] **GPU passthrough for ARM64**
  - *Rationale:* Not in scope per VIRTSTRAT-494.
  - *PM/Lead Agreement:* [Name/Date]

**Test Limitations**

- The tests are limited to AWS; no tests are done on bare metal clusters.
- 12-hour ARM64 cluster availability window — extension to be investigated (hmeir: can there be an extension?)
- No Windows guest tested on ARM64
- IPv6 single-stack and dual-stack not available on AWS — multi-arch testing is IPv4 only

#### **2. Test Strategy**

**Functional**

- **Functional Testing** — Validates that the feature works according to specified requirements and user stories
  - *Details:* Validate heterogeneous cluster-specific functionality.

- **Automation Testing** — Confirms test automation plan is in place for CI and regression coverage
  - *Details:* All tests automated.

- **Regression Testing** — Verifies that new changes do not break  existing functionality
  - *Details:* Run existing Tier 1 and Tier 2 test suites on heterogeneous cluster
    for both architectures. See Section II.1 for per-SIG testing responsibilities.

**Non-Functional**

- **Performance Testing** — Validates feature performance meets requirements
  - *Details:* Out of scope for initial release. - hmeir: is there a plan to test in the future?

- **Scale Testing** — Validates feature behavior under increased load
  - *Details:* Out of scope for initial release. - hmeir: is there a plan to test in the future?

- **Security Testing** — Verifies security requirements, RBAC, authentication, authorization
  - *Details:* N/A — No security-specific requirements for this feature.

- **Usability Testing** — Validates user experience and accessibility requirements
  - *Details:* UI work was completed under [CNV-61832](https://redhat.atlassian.net/browse/CNV-61832). UI testing is owned by
the UI team.

- **Monitoring** — Does the feature require metrics and/or alerts?
  - *Details:* Alerts for boot image provisioning failures on multi-arch clusters.

**Integration & Compatibility**

- **Compatibility Testing** — Ensures feature works across supported platforms, versions, and configurations
  - *Details:* Tests will be executed on AWS only.

- **Upgrade Testing** — Validates upgrade paths from previous versions
  - *Details:* Upgrade from single-arch to multi-arch; verify existing VMs and boot images are preserved.
     Verify multi-arch support is preserved across OCP upgrades.

- **Dependencies** — Blocked by deliverables from other components/products
  - *Details:* Each participating SIG owns their test automation.

- **Cross Integrations** — Does the feature affect other features or require testing by other teams?
  - *Details:* This STP coordinates testing across 5 SIGs. Each SIG is responsible
    for running their regression suites on the heterogeneous cluster.

**Infrastructure**

- **Cloud Testing** — Does the feature require multi-cloud platform testing?
  - *Details:* No; tests are executed on AWS only.

#### **3. Test Environment**

- **Cluster Topology:** 3 AMD64 control-plane, 2 AMD64 workers, 2 ARM64 workers
- **CPU Virtualization:** AMD64 workers use hardware virtualization (VT-x), ARM64 Graviton workers use ARM
  virtualization extensions
- **OCP & OpenShift Virtualization Version(s):** OCP 4.22 with OpenShift Virtualization 4.22
- **Compute Resources:** Minimum per worker node: 16 vCPUs, 32GB RAM
- **Special Hardware:** N/A
- **Storage:** io2-csi
- **Network:** OVN-Kubernetes, IPv4
- **Required Operators:** N/A
- **Platform:** AWS (Graviton instances for ARM64 workers)
- **Special Configurations:** HCO configured with multi-arch support enabled

#### **3.1. Testing Tools & Frameworks**

- **Test Framework:** The framework supports running on a heterogeneous cluster; each sig must make sure that their tests run the VMs on the desired architecture.
- **CI/CD:** Jenkins job to deploy a heterogeneous cluster - Done.
- **Other Tools:** N/A

#### **4. Entry Criteria**

The following conditions must be met before testing can begin:

- [ ] Requirements and design documents are **approved and merged**
- [x] Test environment can be **set up and configured** (see Section II.3 - Test Environment)

#### **5. Risks**

**Timeline/Schedule**

- **Risk:** Cross-SIG coordination across 5 SIGs may delay test completion.
  - **Mitigation:** Establish clear ownership per SIG early; track in Jira.
  - *Estimated impact on schedule:* x (hmeir to complete) delay if not proactive.

**Test Coverage**

- **Risk:** OS coverage on ARM64 is narrower than AMD64 --hmeir: - leaving it here for discussion; network, storage, observability do not have tests marked with `arm` - what will they run on arm?
  - **Mitigation:** hmeir - based on the inputs , this needs to be updated.
  - *Areas with reduced coverage:* sig-network, sig-storage, observability.

**Test Environment**

- **Risk:** 12-hour ARM64 cluster availability window limits test execution time.
  - **Mitigation:** Prioritize test execution.
  - *Missing resources or infrastructure:* Longer cluster availability for full regression runs.

**Dependencies**

- **Risk:** Feature spans multiple components — changes in any can affect multi-arch behavior.
  - **Mitigation:** Shared CI signals; cross-team communication.
  - *Dependent teams or components:* All participating SIG engineering teams.

---

### **III. Test Scenarios & Traceability**

**sig-iuo — New Functional Tests**

- **[CNV-67900]** — As a cluster admin, I want to disable multi-arch support and have architecture-specific boot sources removed
  - *Test Scenario:* [Tier 2] Verify disabling multi-arch removes ARM64-specific boot sources  -hmeir: see ny comment above on this
  - *Priority:* P0

- **[CNV-67900]** — As a cluster admin, I want to be alerted when boot image provisioning fails for a specific architecture
  - *Test Scenario:* [Tier 2] Verify alerts fire on architecture-specific boot image provisioning failure
  - *Priority:* P1

- **[CNV-67900]** — As a cluster admin, I want workload metrics to distinguish between architectures - hemir?
  - *Test Scenario:* [Tier 2] Verify observability metrics differentiate AMD64 and ARM64 VM workloads
  - *Priority:* P1

- **[CNV-67900]** — As a cluster admin, I want multi-arch support preserved after upgrade
  - *Test Scenario:* [Tier 2] Verify upgrade preserves multi-arch configuration and VMs on both architectures
  - *Priority:* P2

- **[CNV-67900]** — As a cluster admin, I want boot sources to update when ARM64 nodes are added or removed
  - *Test Scenario:* [Tier 2] Verify adding ARM64 nodes makes ARM64 boot sources available; removing them cleans up accordingly - hmeir: see my comment above
  - *Priority:* P0

- **[CNV-67900]** — As a VM admin, I want my existing VMs and custom templates to keep working after multi-arch is enabled
  - *Test Scenario:* [Tier 2] Verify existing VMs and templates referencing boot sources continue to function after enabling - hmeir: see my comment above
    multi-arch support
  - *Priority:* P1

- **[CNV-67900]** — As a VM admin, I want to use my own custom boot sources on both architectures
  - *Test Scenario:* [Tier 2] Verify user-provided custom boot sources work for VM creation on both AMD64 and ARM64 nodes
  - *Priority:* P1

**sig-network — New Functional Tests**

- **[CNV-76741]** — As a VM admin, I want VMs on different architectures to communicate with each other
  - *Test Scenario:* [Tier 2] Verify network connectivity between an ARM64 VM and an AMD64 VM
  - *Priority:* P0

**sig-virt — New Functional Tests**

- **[CNV-26818]** — As a VM admin, I want to run VMs from all available boot sources on each supported architecture
  - *Test Scenario:* [Tier 2] Deploy and verify a running VM from each available boot source on AMD64 and ARM64 nodes
  - *Priority:* P0

- **[CNV-75737]** — As a cluster admin, I want cross-architecture migration to be blocked
  - *Test Scenario:* [Tier 2] Verify cross-arch migration fails with clear, user-facing error
  - *Priority:* P0

- **[CNV-26818]** — As a cluster admin, I want VMs to stay on the correct architecture when a node is drained
  - *Test Scenario:* [Tier 2] Verify draining an ARM64 node migrates VMs only to other ARM64 nodes
  - *Priority:* P1

**sig-infra — New Functional Tests**

- **[CNV-76714]** — As a VM admin, I want to create VMs from architecture-specific templates
  - *Test Scenario:* [Tier 2] Verify VMs can be created from architecture-specific templates on both AMD64 and ARM64 nodes
  - *Priority:* P0

**sig-storage — New Functional Tests**

- **[CNV-76732]** — As a cluster admin, I want boot images imported for the correct architecture
  - *Test Scenario:* [Tier 2] Verify boot image import pulls the correct architecture-specific image
  - *Priority:* P0

---

### **IV. Sign-off and Approval**

This Software Test Plan requires approval from the following stakeholders:

* **Reviewers:**
  - QE Members (sig-iuo):
  - Dev Members (sig-iuo):

  - QE Members (sig-network):
  - Dev Members (sig-network):

  - QE Members (sig-storage):
  - Dev Members (sig-storage):

  - QE Members (sig-virt):
  - Dev Members (sig-virt):

  - QE Members (sig-infra):
  - Dev Members (sig-infra):
* **Approvers:**
  - QE Architect: [Ruth Netser](@rnetser)
  - Principal Developer (sig-iuo):
  - Product Manager/Owner:
