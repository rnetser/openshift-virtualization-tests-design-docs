# Openshift-virtualization-tests Test plan

## **[Feature Title — SIG-Specific Scope] - Quality Engineering Plan**

### **Metadata & Tracking**

- **Feature Tracking:** [Link to the relevant feature in Jira]
- **Epic Tracking:** [Link to the SIG-specific tracking Jira Epic, if separate from the parent]
- **QE Owner(s):** [Name (@github-handle, email)]
- **Owning SIG:** [The SIG that owns the feature and the parent STP]
- **Participating SIG:** [This SIG — the one this child STP covers]

**Document Conventions (if applicable):** [Define terms specific to this SIG's scope only]

### **SIG Scope**

<!-- 2-4 sentences. State what this SIG is responsible for testing within the parent feature.
Do NOT repeat the Feature Overview from the parent — reference it instead. -->

This STP covers [SIG name]'s test scope for [feature name].
See the parent STP for the full Feature Overview, requirements, and acceptance criteria.

[Briefly describe what this SIG tests and why it's a separate scope.]

---

### **I. SIG-Specific Review**

<!-- This section covers only SIG-specific review items. Feature-wide requirements,
acceptance criteria, and NFRs are defined in the parent STP. -->

#### **1. SIG-Specific Requirements**

- [x] **Review Requirements**
  - *SIG-specific requirements:* [List requirements from the parent that this SIG owns or extends]

- [x] **SIG-Specific Acceptance Criteria**
  <!-- Only list acceptance criteria that are specific to this SIG's scope.
  Feature-wide acceptance criteria belong in the parent STP. -->
  - [List SIG-specific acceptance criteria]

- [x] **Testability**
  - *Note any SIG-specific requirements that are unclear or untestable:* [List or "None"]

- [x] **Non-Functional Requirements (NFRs)**
  <!-- Only list NFRs specific to this SIG's scope. If all NFRs are covered by the parent, state that. -->
  - *SIG-specific NFRs:* [List or "None — all NFRs are covered by the parent STP"]
  - *NFRs not covered and why:* [List with justification]

#### **2. Known Limitations**

<!-- List only SIG-specific limitations. Feature-wide limitations belong in the parent STP.
If none, state: "None — reviewed and confirmed with [Name/Date]." -->

- **[SIG-specific limitation]**
  - *Sign-off:* [Name/Date]

#### **3. Technology and Design Review**

- [x] **Developer Handoff/QE Kickoff**
  - *Key takeaways and concerns:* [SIG-specific technical concerns]

- [x] **Technology Challenges**
  - *List identified challenges:* [SIG-specific challenges]
  - *Impact on testing approach:* [Impact on this SIG's tests]

- [x] **API Extensions**
  - *List new or modified user-facing APIs:* [SIG-specific APIs, or "None — see parent STP"]
  - *Testing impact:* [Impact]

- [x] **Test Environment Needs**
  - *See environment requirements in Section II.3 and testing tools in Section II.3.1*

- [x] **Topology Considerations**
  - *Describe topology requirements:* [SIG-specific topology needs]
  - *Impact on test design:* [Impact]

### **II. Software Test Plan (STP)**

#### **1. Scope of Testing**

**Testing Goals**

<!-- List only this SIG's testing goals. Feature-wide goals belong in the parent STP.
Order by priority: P0 first, then P1, then P2. -->

- **[P0]** [SIG-specific testing goal]
- **[P1]** [SIG-specific testing goal]
- **[P2]** [SIG-specific testing goal]

**Out of Scope (Testing Scope Exclusions)**

<!-- List items this SIG explicitly will not test. If something is out of scope for the
entire feature, it belongs in the parent STP — not here. -->

- **[Item]**
  - *Rationale:* [Why this SIG won't test it]
  - *PM/Lead Agreement:* [Name/Date]

**Test Limitations**

<!-- Constraints on this SIG's testing approach. -->

- **[Test Limitation]**
  - *Sign-off:* [Name/Date]

#### **2. Test Strategy**

<!-- Mark items that this SIG will test. For items covered by the parent STP or other SIGs,
uncheck and state "Covered by parent STP" or "Covered by [SIG name]". -->

**Functional**

- [ ] **Functional Testing**
  - *Details:* [SIG-specific functional testing approach]

- [ ] **Automation Testing**
  - *Details:* [SIG-specific automation plan]

- [ ] **Regression Testing**
  - *Details:* [Which existing SIG test suites run on the feature cluster]

**Non-Functional**

- [ ] **Performance Testing**
  - *Details:* [SIG-specific, or "Covered by parent STP"]

- [ ] **Scale Testing**
  - *Details:* [SIG-specific, or "Covered by parent STP"]

- [ ] **Security Testing**
  - *Details:* [SIG-specific, or "Covered by parent STP"]

- [ ] **Usability Testing**
  - *Details:* [SIG-specific, or "Covered by parent STP"]

- [ ] **Monitoring**
  - *Details:* [SIG-specific, or "Covered by parent STP"]

**Integration & Compatibility**

- [ ] **Compatibility Testing**
  - *Details:* [SIG-specific, or "Not applicable for this STP"]

- [ ] **Upgrade Testing**
  - *Details:* [SIG-specific, or "Covered by parent STP"]

- [ ] **Dependencies**
  - *Details:* [SIG-specific blocking dependencies]

- [ ] **Cross Integrations**
  - *Details:* [SIG-specific, or "Covered by parent STP"]

**Infrastructure**

- [ ] **Cloud Testing**
  - *Details:* [SIG-specific, or "Not applicable"]

#### **3. Test Environment**

<!-- Covered in the parent STP; make sure it includes the SIG's specific requirements. -->

#### **3.1. Testing Tools & Frameworks**

- **Test Framework:** Standard

- **CI/CD:** [SIG-specific CI lanes, or N/A]

- **Other Tools:** N/A

#### **4. Risks**

<!-- List only SIG-specific risks. Feature-wide risks belong in the parent STP.
Include only categories where this SIG has a specific risk — omit categories with no SIG-specific risk. -->

**[Risk Category]**

- **Risk:** [SIG-specific risk]
  - **Mitigation:** [Mitigation plan]
  - *[Category-specific field]:* [Details]
  - *Sign-off:* [Name/Date]

---

### **III. Test Scenarios & Traceability**

<!-- List only this SIG's test scenarios. Each scenario must trace to a Jira requirement. -->

- **[Jira-ID]** — As a [role], I want [action] so that [benefit]
  - *Test Scenario:* [Description]
  - *Tier:* [1 or 2]
  - *Priority:* [P0/P1/P2]

---

### **IV. Sign-off and Approval**

This Software Test Plan requires approval from the following stakeholders:

* **Reviewers:**
  - [Name / @github-handle]
* **Approvers:**
  - [QE Lead Name / @github-handle]
  - [PM Name / @github-handle]
  - [Dev Lead Name / @github-handle]
