```markdown
# CiteFlow Protocol Specification v0.1 — README

This file is the **canonical README for the CiteFlow Protocol Specification v0.1**, maintained under `spec/citeflow-protocol-v0.1.md` and other future **public specification documents** (e.g., `spec/citeflow-protocol-v0.2.md`, `spec/citeflow-policy-schema.md`, etc.).

It is **not** the README of the implementation project, SDK, or any hosted product. It belongs only to the **spec / protocol documents** themselves.

---

## What this document is

This README describes **how to read, version, interpret, and extend the CiteFlow Protocol Specification family of documents**, starting with `citeflow-protocol-v0.1.md`.

These specs define:

- the machine‑readable rules for **citation, access, and attribution** of web content;
- the **protocol messages** between AI crawlers, verifiers, and publishers;
- the **policy schema** that publishers can attach to their resources;
- the **format of public manifests** (e.g., `/.well‑known/citeflow.json`, `/.well‑known/answer-commerce.json`).

---

## Versioning and file naming

Specifications follow a precise naming convention:

- `spec/citeflow-protocol-v0.1.md`  
- `spec/citeflow-protocol-v0.2.md`  
- `spec/citeflow-policy-schema-v0.1.md`  
- `spec/citeflow-manifest-format-v0.1.md`

Each `vX.Y` file is **immutable once released**.  
If you need to change something, create a new version with a higher number; do **not** rewrite the existing file.

---

## License and usage

- The **specifications themselves** are published under [Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0) (or as otherwise stated in each spec).
- Anyone may:
  - implement the protocol;
  - build libraries, SDKs, or tools;
  - expose compliant endpoints on their own services.
- The license **does not** grant rights to:
  - trademark usage (e.g., “CiteFlow”, “CiteFlow‑certified”);
  - use CiteFlow‑branding in derivative products without explicit permission.

---

## How to read a spec file

Every specification file is structured as:

1. **Overview** – informal description of the problem and objectives.
2. **Terms and definitions** – protocol‑specific vocabulary (e.g., `answer‑unit`, `citation‑token`, `freshness‑TTL`).
3. **Semantics** – what each part means, not just how it looks.
4. **Syntax and schema** – JSON/JSON‑LD shapes, HTTP headers, or URL patterns.
5. **Conformance** – which actors must do what (e.g., AI crawlers, website owners, verification services).
6. **Extensibility** – how to add custom fields without breaking interop.
7. **Security and privacy considerations** – notable risks and mitigations.
8. **Appendices and examples** – sample payloads, server configs, and valid/invalid cases.

---

## How to contribute to the specs

1. **Fork and open a pull request** that touches **only** the `spec/` directory.  
2. **Do not** modify implementation code, configurations, or tests in the same PR.  
3. **Reference existing GitHub issues** (e.g., `Resolves #123`) instead of starting from scratch.  
4. **Provide concrete examples** when adding new fields or message types.

Changes must:

- be **backwards compatible** wherever possible;
- include **normative descriptions** (what implementors must do), not just editorial notes.

---

## How to cite this specification

When you publish an implementation or article, refer to the spec using:

- **Persistent URL**:
  - e.g., `https://github.com/CiteFlow/papers/spec/citeflow-protocol-v0.1.md`
- **Semantic version**:
  - e.g., `CiteFlow Protocol v0.1`
- **Optional label**:
  - e.g., “CiteFlow Protocol v0.1 (2026‑03‑30)”

Do **not** rely on branch names (`main`, `dev`) in stable references; they are unstable.

---

## How future versions will be announced

- New protocol versions (`v0.2`, `v1.0`, etc.) will be announced in:
  - the `CHANGELOG.md` file of the **project root**;
  - the release notes of the main repository.
- Each spec file will point to:
  - the **previous version**;
  - the **next version**, if it exists;
  - a **compatibility matrix** in the `spec/compatability/` directory.

---

## How to expose your site’s CiteFlow compliance

Once you implement a CiteFlow‑compatible server, publish:

- a **policy manifest** at `/.well‑known/citeflow.json`  
- an optional **human‑readable description** at `/about/citeflow`  
- correct **HTTP headers** that reference the specification version.

The exact schema and response format are documented in:

- `spec/citeflow-protocol-v0.1.md`  
- `spec/citeflow-manifest-format-v0.1.md`

---

## How to verify this spec

To check if an implementation is compliant:

1. Run the **conformance tests** linked from the spec (if available).
2. Compare the **request/response shapes** against the JSON samples.
3. Validate that all **required fields** are present and that **forbidden behaviour** (per semantics) does not occur.

Compliance is **self‑declared by each publisher or crawler**, not centrally enforced.

---

## Non‑goal: this is not a product README

This README is **not**:

- a marketing page;
- a feature list of any SDK;
- an onboarding guide for a hosted SaaS;
- a license negotiation for tariffs or percentages.

Those topics are documented elsewhere (e.g., `README.md` at the project root, `docs/licenses.md`, `docs/pricing/`).

If you arrived here from a hosted product page or SDK docs, return to those for:

- hosting options;
- pricing and usage costs;
- account management and support contacts.

---

## Feedback and support

For issues with the **specification itself** (inaccurate syntax, unclear semantics, missing conformance rules):

- Open an issue in the **main repository**.
- Set the label `spec`, `protocol`, or `rfc`.

For issues with **implementations**, SDKs, or hosted services, use the appropriate support channel:

- GitHub Discussions / issues in the `sdk‑js`, `server‑core`, or `dashboard` repositories;
- the hosted product’s support portal.

---

## Maintainers and editors

The current editors of the CiteFlow Specification family are:

- `@bgrusnak` (role: protocol lead)

---

## Contact and identity

For official correspondence, use:

- `support@citeflow.cloud`  
- or the `#templates‑spec` / `#citeflow‑spec` channels in the project’s communication platform.

Do **not** use personal GitHub handles as primary contacts in specifications; those may change over time.

---

## Why this README exists here

This README exists **inside the `spec/` directory** so that every future spec file (`citeflow-protocol-v0.2.md`, `citeflow‑policy‑schema‑v1.0.md`, etc.) can link to a consistent explanation of:

- how specs are versioned;
- how to contribute;
- how to cite and implement;
- where to find non‑spec documentation.

You may safely copy and adapt this file into new `spec/*.md` READMEs, as long as you keep it focused on **the specification documents**, not the whole project.
```
