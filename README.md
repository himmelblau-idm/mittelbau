# Proposal: Entra ID-Compatible Enterprise Cloud Authentication Layer

## Overview

We propose the creation of a **modular, shared enterprise authentication layer** that implements Microsoft Entra ID-compatible protocols—including OAuth2 extensions, device trust, and Primary Refresh Token (PRT) issuance—based on the \[MS-OAPX] and \[MS-OAPXBC] specifications.

This shared layer will form the foundation for multiple downstream projects, including:

1. **Integration into Kanidm** to enable enterprise features (device trust, PRTs, etc.) within the Kanidm identity server.
2. **A new standalone project** (code-named **`blaud`**) that allows organizations to layer modern, cloud-compatible authentication services on top of **any** on-prem Active Directory environment (Samba or Windows Server).

## Motivation

Several overlapping concerns and initiatives prompted this proposal:

1. **Himmelblau needs a server** component to provide automated CI testing.
2. **Kanidm is a strong candidate**, but integrating tightly into it could risk the stability and independence of that project:
  * Tying it too deeply into Kanidm will risk the whole project. A failure of this effort will translate into a whole project failure.
3. **Duplication already exists** between Kanidm and Himmelblau.
4. **Samba Active Directory (ADDC)** includes proven infrastructure that shares lineage with Entra ID and could be extended into the cloud authentication space.
5. **A modular shared layer** could allow the Himmelblau, Kanidm, and Samba projects to collaborate on the same code, each benefiting from improvements without being tightly coupled.

---

## Project Breakdown

### 1. **Enterprise Auth Layer (Shared Library/Module)**

This foundational layer will implement:

* OAuth2 authorization server logic (with Entra ID extensions)
* Device registration and attestation
* Primary Refresh Token (PRT) generation and validation
* Token signing and cryptographic handling (configurable)
* Support for \[MS-OAPX] and \[MS-OAPXBC] protocol flows

This layer will be:

* **Backend-agnostic**: it delegates identity lookups and credential validation to an external directory (e.g., Kanidm, Samba, Windows AD)
* **Consumer-oriented**: meant to be integrated into multiple downstream projects
* **Security-conscious**: allows enforcement of stronger defaults than Microsoft's cloud offering

### 2. **Kanidm Enterprise Extensions (Integration Project)**

A new set of enterprise features will be developed in Kanidm by **integrating the shared enterprise layer**. These features include:

* Device enrollment and trust bootstrapping
* Hello for Business-style PIN authentication
* PRT issuance and validation

This integration will remain **modular and optional**, ensuring Kanidm’s core identity server is not burdened by this new complexity unless explicitly enabled.

### 3. **`blaud`: Bridge Layer for On-Prem AD**

We will develop a **new standalone service**, codenamed **`blaud`**, that integrates with the shared enterprise layer and connects it to **on-prem Active Directory instances**, including:

* **Samba ADDC**
* **Windows Server AD**

`blaud` will:

* Authenticate against the existing AD database
* Issue PRTs and OAuth2 tokens
* Perform device registration, policy evaluation, and cloud-style access control
* Operate as a **companion cloud identity proxy**, running on-prem alongside AD

Importantly, `blaud` can be deployed independently and **SHOULD NOT require changes to the AD domain controller** itself. This allows:

* Retrofitting modern login and compliance flows onto existing AD deployments
* Hybrid environments where on-prem identity integrates with modern cloud workflows
* Organizations to adopt cloud-native identity patterns without vendor lock-in

---

## Development Strategy

To encourage cooperation without tight coupling, the shared enterprise layer will follow a collaborative model:

### Shared Layer Contribution Rules

* **`main` branch**: for unstable, shared development. Commits require only a **courtesy notice** to the other team if breaking changes are introduced.
* **Stable release branches**: tied to downstream projects (e.g., `kanidm-vX`, `blaud-vX`). Each project **owns** its branch; contributions require approval from the owning team.
* CVE and security patches may be coordinated across branches.

---

## Long-Term Vision

The combination of Himmelblau, the enterprise auth layer, and `blaud` positions the ecosystem to:

* Deliver a fully Entra ID-compatible login experience for Linux and Windows clients
* Modernize traditional directory services (like Samba or Windows Server) with minimal disruption
* Provide open-source, privacy-respecting alternatives to proprietary identity platforms
* Enable organizations to run **secure, cloud-native identity infrastructure on-prem**

---

## Next Steps

1. Formalize the module boundaries for the enterprise layer.
2. Stand up a minimal prototype of device registration + token issuance for Himmelblau clients.
3. Define initial integration points with Kanidm (API contracts, user/device schema extensions).
4. Begin scaffolding for `blaud`, targeting Samba and Windows Server integration.
5. Publish a roadmap and RFC-style documentation for public discussion and feedback.
