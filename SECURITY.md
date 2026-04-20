# Security Policy

## Supported Versions

Only the latest minor release is supported with security updates.
Please upgrade to the most recent version listed on the
[Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=anzory.vscode-gppl-ide).

| Version      | Supported          |
| ------------ | ------------------ |
| 1.1.x        | ✅                 |
| 1.0.2, 1.0.3 | ✅ (security only) |
| 1.0.0, 1.0.1 | ❌ **withdrawn**   |
| < 1.0.0      | ❌                 |

## Withdrawn Versions

### v1.0.0 and v1.0.1 — withdrawn due to two vulnerabilities

Fixed in **v1.0.2** (2026-04-20). Details:

1. **XML External Entity (XXE) and billion-laughs DoS** in the `.vmid`
   parser. A malicious machine-description file could read local files,
   exhaust memory, or cause denial of service.

2. **Path traversal in `inc` document links**. The `inc "filename"`
   directive accepted absolute paths, parent-directory segments, and
   UNC paths, enabling information disclosure and potential NTLM-hash
   leaks over SMB.

If you are running v1.0.0 or v1.0.1, **please upgrade immediately**.
The extension auto-updates by default; a full VS Code restart forces
the update to apply.

See the [CHANGELOG](./CHANGELOG.md#102--2026-04-20) for the technical
description of the fixes and the regression tests that guard them.

## Reporting a Vulnerability

If you discover a security issue in this extension, please report it
privately — **do not open a public GitHub issue**.

**Preferred channel:**
[GitHub Private Vulnerability Reporting](https://github.com/anzory/SolidCAM-GPPL-IDE/security/advisories/new)

**Alternative:** email `andrey.a.zorin@gmail.com` with subject
`[SECURITY] vscode-gppl-ide: <short summary>`.

Please include:

- Extension version (from `Help → About` in VS Code, or `code --list-extensions --show-versions`)
- OS and VS Code version
- Reproduction steps, ideally with a minimal `.gpp` or `.vmid` sample
- Impact you were able to demonstrate
- Your disclosure timeline preference

**What to expect:**

- Acknowledgement within **3 business days**
- Initial severity assessment within **7 business days**
- Fix and coordinated disclosure targeted within **30 days** for
  high-severity issues, longer for lower severities
- Credit in the CHANGELOG and the advisory (unless you prefer to
  remain anonymous)

## Disclosed Vulnerabilities

| Advisory                                                                                   | Issue                                                                | Severity | Affected     | Fixed in |
| ------------------------------------------------------------------------------------------ | -------------------------------------------------------------------- | -------- | ------------ | -------- |
| [GHSA-92vg-f4fq-fxm9](https://github.com/anzory/SolidCAM-GPPL-IDE/security/advisories/GHSA-92vg-f4fq-fxm9) | XML External Entity (XXE) + billion-laughs DoS in `.vmid` parser     | High     | 1.0.0, 1.0.1 | 1.0.2    |
| [GHSA-xvpx-9p39-g62m](https://github.com/anzory/SolidCAM-GPPL-IDE/security/advisories/GHSA-xvpx-9p39-g62m) | Path traversal in `inc` directive (file probing, NTLM-leak over UNC) | High     | 0.7.0–1.0.1  | 1.0.2    |

See also [CHANGELOG § v1.0.2](./CHANGELOG.md#102--2026-04-20) for the
technical description of each vulnerability and the tests that guard
the fixes. CVE IDs will be added to the advisories above once the
GitHub CNA assigns them (typically 1–3 business days after the
initial `CVE request`).
