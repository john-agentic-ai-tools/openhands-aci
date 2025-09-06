# OpenHands ACI Mainframe Support Specification

## 1. Overview

The Mainframe ACI is a Python-based code editing assistant designed to run on zLinux and interact with IBM z/OS mainframe environments.
It provides developers with modern code editing workflows (similar to cloud-native environments) while supporting the unique file systems, source control, and security requirements of mainframe systems.

The ACI may be built as a fork or extension of the OpenHands ACI framework, customized for mainframe use.

### 1.1 Purpose & Goals

* Provide seamless access to mainframe code artifacts across:

  * MVS datasets (PDS/PDSE)

  * z/OS UNIX System Services (USS, zFS/HFS)

  * VSAM datasets (read-only support initially)

* Support multiple file access methods to accommodate different enterprise security policies.

* Integrate with mainframe source control products (Endevor, ISPW, RTC, Git on z/OS).

* Understand code migration workflows between regions (Dev → Test → QA → Prod).

* Detect and report security and access errors, providing remediation guidance.

* Provide an AI-assisted coding interface with syntax awareness for COBOL, PL/I, JCL, REXX, and other common mainframe languages.

### 1.2 Scope Limitations

The following items are not in scope.

* The ACI will not replace existing mainframe IDEs (e.g., IDz, Topaz).
* The ACI will not bypass enterprise RACF/ACF2/Top Secret security policies.
* The ACI will not handle production deployments directly; it focuses on developer productivity

### 1.3 Target Environment

The ACI will initially be run on zLinux and would be constrained by the versions of Python supported on zLinux versions:
The following table lists recent, currently supported zLinux (IBM Z) distributions and the corresponding supported Python versions:

| zLinux Distribution         | Supported Versions (as of 2024) | Supported Python Versions      |
|----------------------------|----------------------------------|-------------------------------|
| Red Hat Enterprise Linux   | 8.6, 8.8, 9.2, 9.3               | 3.6, 3.8, 3.9, 3.11           |
| SUSE Linux Enterprise      | 15 SP4, 15 SP5                    | 3.6, 3.9                      |
| Ubuntu (IBM Z)             | 20.04 LTS, 22.04 LTS              | 3.8, 3.10                     |

**Notes:**

* Only LTS and enterprise distributions officially supported by IBM are listed.
* Python 2.x is deprecated and not recommended for new development.
* For the most up-to-date compatibility, refer to IBM's official documentation and the respective Linux distribution's support matrix.

## 2. Requirements

### 2.1 Functional Requirements

#### 2.1.1 File Access

R1. Support read/write of PDS/PDSE members.

R2. Support USS file operations (create, read, update, delete).

R3. Support read-only access to VSAM datasets (future: edit APIs if available).

R4. Provide multiple adapters (z/OSMF, Zowe, FTP, NFS).

R5. Automatically detect encoding (EBCDIC/ASCII/UTF-8).

Security & Error Handling

R6. Capture and classify security errors (RACF/ACF2/Top Secret).

R7. Provide remediation suggestions for access issues.

R8. Log security-related events (without exposing credentials).

Source Control Integration

R9. Support check-in/check-out with Endevor.

R10. Support task operations with ISPW.

R11. Support Git (both USS-based repos and remote GitHub/Bitbucket).

R12. Track element promotion across regions (Dev → Test → QA → Prod).

R13. Detect conflicts between regions (e.g., “member changed in QA since last promotion”).

#### 2.1.2 Developer Experience

R14. Enable AI-assisted code suggestions (COBOL, JCL, PL/I).

R15. Provide syntax-aware diffing between dataset versions.

R16. Allow side-by-side local vs mainframe comparison.

R17. Cache members locally for editing, with safe sync-back.

R18. Provide real-time hints about region-specific migration practices.

### 2.2 Non-Functional Requirements

Performance: Must handle large PDS libraries efficiently (≥10,000 members).

Security: Must comply with enterprise IAM policies (RACF/ACF2/Top Secret).

Portability: Runs on zLinux (s390x), compatible with RHEL, SUSE, Ubuntu on Z.

Extensibility: File adapters and SCM integrations should be pluggable.

Usability: Developers should not need mainframe terminal emulators (ISPF) for basic code editing.

### 2.3 System Requirements

#### 2.3.1 — Developer (desktop) requirements

* Supported client platforms
  * Any modern desktop OS (Windows 10/11, macOS, Linux) with a modern browser.
  * Supported browsers: Chrome, Edge, Firefox (latest stable).

* Network & access
  * HTTPS access to the ACI endpoint (TLS certificate trusted by client).
  * VPN or corporate network access when required by policy.
  * Ability to reach z/OSMF, Zowe, or SFTP/FTP endpoints from the ACI host (no direct mainframe access required from developer desktop).

* Tools (recommended)
  * Git client (≥2.30) for local repo operations.
  * Zowe CLI (optional, recommended for direct dataset/USS access and debugging).
  * SSH client/SFTP client for file transfer when required.
  * VS Code (or other editor) with the OpenHands ACI extension or editor integration (optional).
  * Lightweight JSON/REST client (for troubleshooting API calls).

* Local environment
  * Local disk for member cache (typical developer cache: 100–500 MB; size grows with workspace).
  * Optional Docker if running local ACI development instances or testing containers.

* Credentials & authentication
  * Valid mainframe service/user credentials (z/OSMF user, Git on z/OS credentials, Endevor/ISPW service account) or federated SSO access as configured.
  * SSH keys or certificate material if SFTP/SSH or mTLS is required.
  * MFA/SSO client if enterprise requires it.

* Usability
  * No ISPF terminal emulator required for normal editing workflows.
  * Basic familiarity with Git and the chosen editor recommended.

#### 2.3.2 — Mainframe administrator (installation & enablement) requirements

* Host platform (zLinux)
  * Supported s390x distribution (RHEL/SUSE/Ubuntu on Z) matching organization policy and the ACI Python support matrix.
  * Minimum sizing (reference): 4 vCPU, 8–16 GB RAM, 50–100 GB persistent disk (adjust for scale and local cache needs).
  * systemd service to run the ACI process; container/runtime options supported (Docker/Podman).

* Language/runtime & packages
  * Python 3.8–3.11 installed system-wide or in virtualenv.
  * pip and build tools; recommended to install in isolated virtualenv or container image.
  * TLS libraries and OS packages required by dependencies (openssl, libffi, etc.).

* Network & TLS
  * Public hostname and TLS certificate (CA-trusted) or internal PKI for HTTPS.
  * Ports: 443 (HTTPS), 22 (SFTP/SSH if used), additional ports as required by monitoring or admin interfaces.
  * Reverse proxy (nginx/Apache) recommended for TLS termination, access control, and static content caching.
  * Optional mTLS for z/OSMF or z/OS endpoints that require client certs.

* z/OS connectivity & services
  * z/OSMF REST API enabled and reachable; dedicated z/OSMF user account with scoped privileges.
  * z/OS FTP/SFTP service enabled if using FTP-based adapters.
  * OMVS UID mapping and USS permissions configured for any file-based operations.
  * NFS mounts or shared filesystem availability when using NFS adapter (proper export configuration and security).
  * Endevor / ISPW / RTC integration endpoints and service accounts with documented API credentials and privileges.
  * Git on z/OS (if used) or access to remote Git servers configured.

* Security & authorization
  * Service accounts (ACI service ID) provisioned in RACF/ACF2/Top Secret with least privilege: dataset READ/UPDATE/ALLOC as required, USS permissions, and any package/tool-specific entitlements.
  * Credential management: secure secret store (e.g., HashiCorp Vault, KMS) for storing API keys, service credentials, and certificates; avoid storing plaintext credentials.
  * Audit logging and SMF integration for record of dataset access and promotions.
  * Key rotation and credential lifecycle policy.

* Integration & middleware
  * API access and credentials for Endevor/ISPW; sample test package/environment for smoke tests.
  * Git repositories and hooks (if promoting code via Git).
  * Message/queue or webhook endpoints if integrating with CI/CD pipelines.

* Observability & operations
  * Centralized logs (syslog, ELK, or Splunk) and metrics export (Prometheus exporter integration recommended).
  * Backup strategy for configuration and local caches.
  * Health checks and monitoring dashboards, alerting for high latency, auth failures, or adapter errors.
  * Upgrade and rollback procedures documented; staging/test instance recommended before production rollout.

* Hardening & compliance
  * Enforce TLS, disable weak ciphers, and require mTLS where policy requires.
  * Run ACI under a dedicated, non-root service account with limited filesystem access.
  * Penetration test and vulnerability scanning as part of installation validation.
  * Documentation of required RACF/ACF2/Top Secret rules to grant the service ID minimal required access.

* Deployment checklist (quick)
  * Provision zLinux host with recommended sizing.
  * Install Python, create virtualenv/container image, deploy ACI service.
  * Configure TLS, reverse proxy, and firewall rules.
  * Provision mainframe service accounts and verify z/OSMF, FTP/SFTP, and Endevor/ISPW access.
  * Integrate credential store and configure logging/monitoring.
  * Run integration smoke tests (dataset read/write, USS file ops, SCM check-in/out).

## 3. Technical Architecture

### 3.1 High-Level Components

* Core ACI Engine (Python-based, fork of OpenHands):
* Code editing UI - Similar Experience to IBM QuickEdit
* Syntax intelligence (COBOL, PL/I, JCL)
* AI-driven suggestions
* File Access Layer (pluggable adapters):
  * z/OSMF REST API Adapter → datasets, USS
  * Zowe CLI/SDK Adapter → datasets, USS
  * FTP/SFTP Adapter → legacy dataset access
  * NFS Mount Adapter → USS as mounted FS

* Local Cache Manager → temporary sync of members to zLinux files

* Security & Error Handling Layer:

  * Detects common mainframe security errors:
  * RACF "not authorized" (e.g., ICH408I)
  * Dataset in-use contention
  * USS permission denied
  * Provides human-readable explanation + remediation advice (e.g., “You need READ access to HLQ.PROD.COBOL via RACF. Contact security admin.”)

* Source Control Integration Layer:

* Endevor → APIs to check out/in elements

* ISPW → APIs for task management

* RTC/Jazz → repository access

* Git (z/OS + USS) → direct integration with git commands

* Awareness of migration flows (DEV→TEST→PROD) to track where code should be promoted

* Developer Experience Layer:

  * Local editing with automatic EBCDIC↔UTF-8 encoding conversion

  * Line/record integrity for datasets (respect RECFM/LRECL)

  * Side-by-side diffing between dataset versions

  * Region-aware migration hints (e.g., “This member is in DEV region; to promote, use Endevor package XYZ”)

### 3.1 File System Support

This platform must support the full variety of z/OS/mainframe file kinds and access semantics. For each file system below, list of supported operations and important behavioral details are provided.

#### 3.1.1 MVS datasets (primary focus)

Supported dataset types

* PS (Physical Sequential)
* PDS (Partitioned Data Set)
* PDSE (Partitioned Data Set Extended)
* GDG (Generation Data Groups)
* Concatenated datasets (multi-volume / concatenation)
* Dataset attributes (RECFM, LRECL, BLKSIZE, DSNTYPE, DSORG)

Required operations & behaviors (all types)

* Read/list/stream file contents with correct record semantics (respect RECFM/LRECL).
* Create/allocate and delete datasets (via z/OSMF, Zowe, FTP or adapter that can request allocation).
* Read and present dataset metadata (VOLSER, VSAM/DSNTYPE, RECFM/LRECL, created date).
* Automatic encoding detection and safe conversion (EBCDIC ↔ UTF‑8) with options to treat files as binary.
* Respect dataset-level security and provide mapped, actionable error messages for RACF/ACF2/Top Secret failures.
* Local caching with record/line integrity and safe sync-back (atomic replace, detect concurrent changes).

PDS and PDSE — member model, specifics

* Member-level operations:
  * List members (directory listing).
  * Read member contents.
  * Create/add member.
  * Update/replace member.
  * Delete member.
  * Rename/move/copy member (where supported by adapter).
* Directory considerations:
  * For PDS, track directory block limits and provide guidance/automatic reorg suggestions when directory is full.
  * For PDSE, support dynamic directory management and larger member counts.
* Allocation & extension:
  * Support allocation parameters (primary/secondary extents) and report allocation failures.
  * For PDS, warn when directory full and provide remediation steps (reorg, copy to PDSE).
* Member metadata:
  * Preserve member attributes when editing (record format, text vs binary, sequence numbers if present).
  * Detect and respect load modules vs source members (binary vs text).
* Concurrency & checkout:
  * Provide advisory locking / check-out semantics to avoid lost updates (integrate with SCM or use dataset exclusive allocation when available).
  * Detect "dataset in use" conditions and report clear remediation steps.
* Performance:
  * Efficient listing and random access for very large PDS/PDSE libraries (≥10k members).

GDG support

* List GDG base and available generations.
* Read specific generation or latest generation.
* Support retention and expiration metadata display.
* Support allocation of new GDG generation via standard APIs.

Concatenated datasets & allocation semantics

* Recognize concatenated dataset chains and present unified view for sequential reads.
* Report multi-volume or concatenation issues and provide guidance.

#### 3.1.2 VSAM datasets (initially read-only)

Supported types

* KSDS (Key-Sequenced)
* ESDS (Entry-Sequenced)
* RRDS (Relative Record)
Support details
* Read access to records and record ranges via key or relative record number (adapter permitting).
* Present VSAM control interval/record organization info.
* Initial scope: read-only (R3). Note future plan for write/update APIs if environment permits and security allows.
* Handle alternative indexes where queries require it (read-only traversal).

#### 3.1.3 z/OS UNIX System Services (USS) / zFS / HFS

Supported behaviors

* Regular file operations: stat, read, write, create, delete, rename, chmod/chown (subject to credentials).
* Directory listing and traversal with POSIX semantics.
* Preserve file permissions, ownership and extended attributes.
* Support for symlinks and pipes where present.
* Support for text/binary detection and line ending normalization when requested.
* Integration with NFS mounts (treat mounted USS/zFS as local FS when NFS adapter used).

#### 3.1.4 JES Spool (JES2/JES3) — read-only

* Ability to list job output and retrieve SYSOUT classes via z/OSMF or JES APIs.

* Present job logs and return codes for diagnostics related to dataset allocations or job runs.

#### 3.1.5 Other artifacts & integrations

* DB2 files (metadata only) — surface dataset names and access info; actual DB access out of scope unless explicitly integrated.

* IMS/Message Queues — metadata integration only unless adapter provided.
* Tape datasets (where visible via catalog) — read semantics and allocation metadata reporting; write operations only via explicit admin workflows.

#### 3.1.6 Adapters and correspondence

* z/OSMF adapter: preferred for REST-based dataset and USS operations (allocate, list members, read/write).

* Zowe CLI/SDK adapter: full feature parity for datasets, USS, JES where available.
* FTP/SFTP adapter: legacy access for PS/PDS/PDSE and USS files (note FTP limitations for PDSE and VSAM).
* NFS adapter: treat mounted zFS/USS as local FS (suitable for read/write file operations).
* Agent-based adapters: allow privileged dataset operations that cannot be performed over standard protocols (must run under a controlled service account).

#### 3.1.7 Operational & UX considerations

* Always surface dataset-level metadata before edits (RECFM/LRECL/encoding) and ask user to confirm if conversion or format changes are needed.

* Provide contextual remediation guidance for allocation, locking, and security errors (targeted messaging referencing common z/OS messages).
* Ensure that any write/allocate operation requires explicit confirmation and documents the exact z/OS allocation parameters that will be used.
* Maintain audit trails for dataset access and promotion activities (SMF messages or adapter-provided logs).
* Pluggable file adapter model so support for additional file types (e.g., direct VSAM write, IMS) can be added in future phases.

These file system support details ensure correct record semantics, safe edits, clear error guidance for mainframe users, and extensibility for future adapters and dataset types.

### 3.2 Encoding Support

* Automatic detection and explicit configuration
  * Default detection order: EBCDIC variants (cp1047 / cp037) → UTF-8 → ISO-8859-1 (latin-1) → ASCII fallback.
  * Allow explicit override per dataset/member (dataset attributes, user preference, or adapter-provided metadata).
  * Provide a "treat as binary" option to bypass any character conversion for non-text or packed/binary members.

* EBCDIC variants (primary for legacy COBOL)
  * Support common z/OS EBCDIC code pages: cp1047, cp037, cp500 and others as needed.
  * Rationale: most COBOL source files created decades ago on z/OS are in EBCDIC; cp1047 and cp037 are the most frequent for US/Western sites.
  * Handle EBCDIC-specific characters used in copybooks, literals and comment areas; preserve column-oriented semantics (columns 1–6 sequence area, column 7 indicator, 8–72 source).
  * Use IBM/ICU/iconv mappings that are proven for z/OS ↔ UTF-8 round-trip.

* ASCII / Latin and Unicode
  * Support UTF-8 for modernized or cross-platform files.
  * Support single‑byte Western encodings (ISO-8859-1 / latin-1) for cases where files were converted or exchanged with ASCII hosts.
  * Preserve multi-byte sequences and validate well-formed UTF-8 when detected.

* Control bytes, record framing, and non-text data
  * Respect dataset RECFM/LRECL and do not treat RDW or system-level record headers as text.
  * Detect and preserve packed decimal, binary, and other non-text content; flag files that contain high probability of binary data so no conversion is applied.
  * Preserve trailing spaces and fixed-record lengths; do not trim records during encoding conversion.

* Heuristics and validation
  * Use heuristics: byte-distribution analysis (high-bit density), presence of common EBCDIC tokens (e.g., COBOL keywords in EBCDIC values), absence/presence of UTF-8 BOM, and adapter-supplied metadata to select encoding.
  * Provide a validation pass showing sample lines and a "confidence" score before auto-converting on open/save.
  * Offer a quick "preview conversion" view and a one-click "force original encoding" option.

* Mapping edge-cases and NLS
  * Support site-configurable mapping tables for local national characters or historic custom codepages.
  * Provide a mechanism to map or warn about characters that have no direct mapping (show hex values and suggested substitutes).

* Round-trip safety and audit
  * Ensure round-trip conversion tests (EBCDIC → UTF-8 → EBCDIC) to detect lossy transformations; log any lossy conversions.
  * Store per-member encoding metadata in the local cache and include it in audit logs for restores/sync-backs.

* Implementation notes / codec names (implementation-friendly)
  * Typical Python codec names: "cp037", "cp1047", "cp500", "utf-8", "latin-1".
  * Prefer using robust libraries (iconv, Python codecs, ICU) and test mappings against actual z/OS samples.
  * Expose conversion options in CLI/API (source_encoding, target_encoding, binary_mode, force).

* Recommended defaults and UX
  * Default: assume EBCDIC cp1047 or cp037 for MVS/PDS/PDSE unless adapter metadata indicates otherwise.
  * Show detected encoding prominently in the editor status bar and include a one-click switch to change encoding and re-load.
  * When saving, require explicit confirmation if the target encoding differs from the original and could lose information.

* Testing guidance
  * Validate with a corpus of legacy COBOL: copybooks, JCL, DATA DIVISION literals, and files containing special characters.
  * Include tests for: round-trip fidelity, record-length preservation, detection accuracy, and behavior on undecodable bytes.

This set of encoding behaviors ensures safe, predictable handling of decades-old COBOL sources while allowing modern Unicode workflows and site-specific exceptions.

### 3.3 Language Support

This section lists the programming and scripting languages the ACI will understand, edit, and assist with, plus a compact support matrix for common mainframe tools (e.g., Easytrieve).

Supported languages / artifacts

* COBOL family
  * IBM Enterprise COBOL for z/OS (modern Enterprise COBOL dialects)
  * COBOL II / legacy mainframe COBOL variants (column-oriented fixed-format)
  * Micro Focus COBOL (mainframe-hosted variants / Visual COBOL artifacts)
  * Copybooks (COPY / COPYREPL resolution, copybook include paths)
  * Free-format vs fixed-format handling; column semantics (columns 1–6 seq, 7 indicator, 8–72 source)
* JCL
  * JES2 / JES3 JOB/PROC/EXEC statements, DD parameters and utilities (IEFBR14, IDCAMS, IEBCOPY, SORT, etc.)
  * JES output parsing (SYSOUT classes, return codes)
* DB2 SQL
  * Embedded SQL (EXEC SQL … END-EXEC) in host languages (COBOL, PL/I)
  * Static SQL (plans, BIND artifacts) and Dynamic SQL handling
  * SQL DDL/DML, cursors, host-variable diagnostics, SQLCODE/SQLSTATE awareness
  * Stored procedures and SQL PL artifacts
* PL/I (mainframe PL/I dialects)
* HLASM (High-Level Assembler / IBM Assembly)
* REXX (TSO/ISPF REXX, batch REXX)
* CLIST (legacy TSO CLIST scripts)
* Easytrieve (report programs and field definitions)
* USS scripting & languages
  * POSIX shell scripts (sh, ksh, bash on z/OS USS), Python/Perl/Node.js on z/OS USS
* Java on z/OS (source and manifest/deployment artifacts)
* IMS artifacts (DDL / PSB / DBD metadata exposure; runtime integration optional)
* VSAM metadata (queries/read-only access; not a language but often accessed via programs above)

Support matrix (features vs languages/tools)

| Feature \ Language / Tool | Enterprise COBOL | COBOL II / Legacy | Micro Focus COBOL | Copybooks | JCL (JES2/JES3) | DB2 SQL (embedded/static) | PL/I | HLASM | REXX | Easytrieve | USS shell |
|--------------------------:|:---------------:|:-----------------:|:-----------------:|:---------:|:---------------:|:-------------------------:|:----:|:-----:|:----:|:---------:|:--------:|
| Syntax highlighting      | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Parsing & AST            | ✓ (column-aware) | ✓ (column-aware) | ✓ | ✓ | ✓ (DD/JCL tree) | ✓ (embedded-aware) | ✓ | ✓ | ✓ | ✓ | ✓ |
| Encoding (EBCDIC↔UTF‑8)  | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Column/record semantic checks | ✓ | ✓ | ✓ | ✓ | n/a | ✓ (host var mapping) | ✓ | ✓ | n/a | ✓ | ✓ |
| Linting / static analysis| ✓ (rulesets) | ✓ | ✓ | ✓ | ✓ (DD/param checks) | ✓ (SQLCODE checks) | ✓ | ✓ | ✓ | limited | limited |
| Formatter / pretty-print | ✓ (fixed/free aware) | limited | ✓ | n/a | limited | n/a | limited | n/a | n/a | limited | ✓ (shell) |
| Refactoring (rename, extract) | ✓ (identifiers/copybook-aware) | limited | ✓ | ✓ | n/a | partial (table/column-aware) | limited | limited | limited | limited | limited |
| Copybook/include resolution | ✓ | ✓ | ✓ | ✓ | n/a | ✓ (host integration) | n/a | n/a | n/a | n/a | n/a |
| Embedded SQL awareness   | ✓ (EXEC SQL) | ✓ | ✓ | n/a | n/a | ✓ (bind/explain integration) | ✓ | n/a | n/a | n/a | n/a |
| Debugging integration    | integration via batch/JCL | basic | Micro Focus debugger integrations | n/a | job submit / spool feedback | DB2 explain / plan mapping | limited | limited | n/a | n/a | process-level debug |
| Batch/run integration (submit, capture SYSOUT) | ✓ | ✓ | ✓ | n/a | ✓ (submit/monitor) | ✓ (DB2 job flows) | limited | limited | ✓ | ✓ | ✓ |
| AI-assisted suggestions  | ✓ (fixes, idioms, migrate hints) | ✓ | ✓ | ✓ | ✓ (allocation/param hints) | ✓ (SQL tuning hints) | ✓ | limited | ✓ | ✓ | ✓ |
| Limitations / Notes      | Must preserve column semantics; codepage-sensitive | Legacy constructs require relaxed parsing | Vendor runtime differences; test on target | Copybook circular includes require configured paths | Some site-specific DD/PROC patterns may need customization | BIND/PLAN metadata required for full static checks | Language dialects vary by compiler version | Macro expansions and relocatables are complex | Host environment variants (ISPF vs batch) | Dialect differences between Easytrieve versions | Line endings / permissions vary on USS |

Quick integration notes

* Embedded SQL: tool will surface SQLCODE/SQLSTATE and map DB2 errors to actionable guidance; support both static (BINDed) and dynamic usage.
* Copybooks: resolve via configured include paths and dataset mappings; show origin dataset/member and preservation of formatting.
* JCL utilities: surface common utility parameter templates and validation warnings (e.g., IDCAMS syntax, IEBCOPY datasets).
* Easytrieve: provide syntax highlighting, field/report preview, and batch submit via JCL templates; advise on mapping to COBOL copybooks where applicable.
* Debug/run: job submit and SYSOUT retrieval via z/OSMF/Zowe with integrated parsing of return codes and common messages.

Limitations and roadmap

* Some vendor-specific runtime/debugger integrations (Micro Focus, IBM Debug tools) may require separate connectors or licensed integrations.
* Advanced refactoring across multiple datasets and promotion flows will be phased (Phase 2–4).
* VSAM and IMS deep-editing/debugging are out of scope initially (read-only / metadata-first); future adapters may expand write/debug support.

Configuration knobs

* Per-project dialect settings (COBOL compiler version, free/fixed format, copybook dataset paths).
* DB2 connection/BIND metadata endpoints for improved SQL validation.
* Easytrieve dialect/version selection for accurate parsing.

This coverage ensures the ACI can safely edit, analyze, and help modernize common mainframe sources (COBOL variants, JCL, DB2 SQL) while supporting ancillary languages and tools that appear in typical z/OS development workflows.

### 3.4 Integration Points

This section describes how the Mainframe fork integrates with the existing OpenHands ACI, what new capabilities are added, and which existing modules are modified for mainframe semantics.

#### 3.4.1 Overview

* Integration approach: extend OpenHands ACI by introducing a pluggable adapter layer and dataset-aware file model while preserving core ACI engine, AI suggestion pipelines, and UI frameworks. Design emphasizes backward compatibility: existing cloud/posix adapters remain unchanged unless explicitly listed below.
* Deployment model: new adapters and services load via the ACI plugin registry/config. Feature flags allow enabling mainframe behavior per deployment or per-project.

#### 3.4.2 New operations and capabilities added

* Dataset-aware file model
  * Add dataset/member abstraction (PDS/PDSE/PS/GDG) with member-level CRUD.
  * Expose dataset metadata (RECFM, LRECL, BLKSIZE, DSORG, VOLSER).
  * Support concatenated datasets and unified sequential views.
* PDS/PDSE member operations
  * List members, read member, create/add member, update/replace member, delete member, rename/copy (adapter-dependent).
  * Directory block monitoring and PDSE/PDS reorg guidance.
* Encoding & record semantics
  * Automatic encoding detection pipeline (EBCDIC variants → UTF-8 → latin-1).
  * Record-framed read/write preserving RECFM/LRECL and RDW semantics.
  * Binary-mode flag and preview-conversion flow in editor/API.
* z/OS adapters
  * z/OSMF REST adapter: allocate, list, read/write datasets & USS files, JES spool retrieval.
  * Zowe CLI/SDK adapter: parity alternative to z/OSMF where required.
  * FTP/SFTP adapter: legacy PS/PDS access and USS support (with PDSE caveats).
  * NFS adapter: treat mounted zFS/USS as local FS.
  * Agent-based adapter (optional): privileged operations not possible via standard protocols.
* VSAM and JES integrations (initial scope)
  * Read-only VSAM operations (KSDS/ESDS/RRDS) with key/record reads and metadata exposure.
  * JES spool listing and SYSOUT retrieval via z/OSMF/Zowe.
* Source control & promotion integrations
  * Endevor: check-out/check-in APIs, promotion flow awareness, mapping to CI events.
  * ISPW: task operations and status management.
  * Git on z/OS + USS Git: repo-aware operations, push/pull hooks for promotion flows.
  * Promotion tracking: Dev → Test → QA → Prod state mapping, conflict detection.
* Developer UX features
  * Local member cache manager with safe sync-back (atomic replace, conflict detection).
  * Side-by-side local vs mainframe compare, syntax-aware diffing preserving column semantics.
  * Editor status indicators: detected encoding, dataset attributes, region/state.
  * Migration hints surfaced by AI based on promotion state and SCM metadata.
* Security, auditing & error mapping
  * Map RACF/ACF2/Top Secret and z/OS messages to human-readable remediation guidance.
  * Audit logging of dataset access and promotions (SMF-compatible logs or adapter-provided audit entries).
  * Integration with secret stores for mainframe credentials (Vault/KMS).

#### 3.4.3 Modifications to existing OpenHands components

* File Access Layer
  * Replace/simple-extend existing filesystem abstraction to accept "file kinds" (posix, dataset-member, vsam, jes-spool) and record-oriented APIs.
  * Add adapter registration APIs and capability discovery so core engine can route operations to correct adapter.
* Core ACI Engine
  * Extend open/read/write APIs to accept dataset parameters (RECFM, LRECL, encoding) and to return dataset metadata alongside content.
  * Add conflict detection hooks and optional advisory locking semantics tied to SCM or adapter-exclusive allocation.
  * Update caching subsystem to support per-member encoding metadata and round-trip safety checks.
* UI / Editor Integration
  * Editor protocol extended to show dataset attributes, encoding toggles, and fixed-column displays for column-aware languages.
  * Add explicit save/allocate confirmation dialogs for operations that change allocation or dataset attributes.
* Language/Parsing Layer
  * Enhance parsers to preserve fixed-column semantics (columns 1–6, indicator area, columns 8–72) for COBOL/legacy sources.
  * Hook copybook resolution to dataset-member lookups via adapters.
* SCM & Promotion Layer
  * Add Endevor and ISPW connectors; extend existing Git connector to support z/OS-specific repo models and promotion metadata.
  * Add promotion state model and conflict detection service integrated with ACI telemetry and UI prompts.
* Error handling & messaging
  * Centralize mainframe error classification and remediation library; modify existing error renderer to include mainframe-specific guidance and actionable next steps.
* Telemetry & Logging
  * Enhance telemetry schema with dataset access metrics (member counts, large PDS scan times) and adapter-specific error codes.
  * Ensure logs do not contain plaintext credentials; add adapter hooks for SMF/audit forwarding where available.

#### 3.4.4 Compatibility and configuration controls

* Feature flags
  * Per-host or per-project flags to enable mainframe adapters and dataset model. Default disabled on existing installs to avoid behavioral changes.
* Backward compatibility
  * Existing POSIX and cloud adapters behave as before; only projects configured for mainframe mode see dataset semantics.
  * APIs extended in a backward-compatible manner: new fields in responses are optional; old clients continue to function unless they opt-in.
* Admin controls
  * Adapter capability discovery allows admins to restrict allocate/delete operations.
  * Permission model integrates with enterprise IAM and adapter-level service accounts.

#### 3.4.5 Operational and deployment notes

* Packaging
  * Adapters ship as separate Python packages/plugins to keep core ACI small and permit independent updates.
* Testing
  * Integration tests for each adapter (z/OSMF, Zowe, FTP, NFS) and end-to-end flows (edit → check-in → promote).
* Security
  * Require service account provisioning guidance and secret-store integration; document least-privilege requirements for each adapter.
* Rollout
  * Recommend rollout in staging with feature-flag gating; smoke-tests for large PDS libraries and encoding round-trip fidelity.

#### 3.4.6 Summary of operations added vs modified

* Added (new)
  * Dataset/member CRUD (PDS/PDSE/PS/GDG), member listing
  * Encoding auto-detection and record-aware read/write
  * z/OSMF, Zowe, FTP/SFTP, NFS, Agent adapters
  * VSAM read-only, JES spool read-only
  * Endevor & ISPW SCM connectors, Git on z/OS enhancements
  * Promotion-tracking, conflict detection, audit/SMF hooks
  * Local member cache with per-member metadata
* Modified
  * File access abstraction to accept dataset semantics
  * Core read/write APIs to include dataset metadata and encoding options
  * Editor/UI to present dataset attributes and column-aware editing
  * Error handling to map mainframe messages to remediation guidance
  * Telemetry/logging schemas to include adapter and dataset metrics

This integration plan keeps the existing OpenHands core intact while adding explicitly pluggable mainframe-aware capabilities and preserving backward compatibility through opt-in configuration and plugin packaging.

## 4. File System Operations

### 4.1 Path Conventions

* Canonical forms
  * MVS dataset (PS/PDSE/PDS): HLQ.PROJ.LIB or HLQ.PROJ.LIB(MEMBER)
  * Sequential PS dataset: HLQ.PROD.FILE
  * GDG generation: HLQ.BACKUP.GDG(+0) or HLQ.BACKUP.GDG(0001) (adapter-dependent display)
  * VSAM dataset: HLQ.VSAM.NAME (referenced as a dataset; adapter may accept vsam://HLQ.VSAM.NAME)
  * USS path: absolute POSIX path, e.g. /u/john/project/script.sh or /etc/config.cfg
  * JES spool (read-only): adapter-specific URI, e.g. zosmf://jobs/{JOBNAME}/{JOBID}/sysout/{CLASS}
  * Adapter URI examples: zosmf://datasets/HLQ.PROJ.LIB(MEMBER), zowe://datasets/HLQ.PROJ.LIB(MEMBER), ftp://'HLQ.PROJ.LIB(MEMBER)'

* Character set, case and validation
  * Follow z/OS dataset rules: qualifiers separated by dots, total length up to 44 characters, allowed chars typically A–Z, 0–9 and special qualifiers ($ # @). Member names typically 1–8 chars and uppercase by convention.
  * Normalization: adapters SHOULD convert dataset and member names to uppercase unless explicitly quoted by user.
  * Validation: reject names that violate dataset/member rules and return a clear error referencing z/OS naming constraints.

* Quoting and escaping
  * Parentheses denote members: use parentheses only for member syntax: HLQ.LIB(MEMBER)
  * When used in URIs or shells, percent-encode or single-quote dataset names as required (e.g., ftp://'<DSNAME>' or zosmf://datasets/HLQ.PROJ.LIB%28MEMBER%29).
  * Local UI should display the human-friendly canonical form (as above) and the raw adapter URI for troubleshooting.

* Mapping to local cache
  * Canonical local path pattern: <cache-root>/datasets/HLQ/PROJ/LIB/MEMBER (replace dots with path segments; store original names in metadata)
  * Preserve metadata (RECFM, LRECL, encoding, dataset type) in a sidecar file (.meta.json) to guarantee round-trip fidelity.
  * Avoid lossy filename transforms; when collisions or illegal filesystem chars occur, use a reversible encoding (percent-encode) and show the original mainframe path in the UI.

* Special cases & semantics
  * Concatenated datasets: present a unified read view and show underlying chain in metadata (list of VOLS/parts).
  * Binary/load modules: treat as binary paths (e.g., HLQ.LOAD.MEMBER) and expose a binary_mode flag; do not auto-convert encoding.
  * GDG generations: support relative (+n), absolute generation numbers and timestamped generation names; adapters must present generation metadata (creation date, retention) with the path.

* Best-practice display & APIs
  * Always return both:
    * canonical_mainframe_path (human form: HLQ.LIB(MEMBER) or /path/on/uss)
    * adapter_uri (machine form: zosmf://..., zowe://..., ftp://...)
  * Include per-path metadata (dataset type, RECFM, LRECL, encoding, region/state) in API responses so UIs can show context before edits.
  * Provide helper functions to:
    * parse and validate dataset/member syntax,
    * produce adapter-safe URIs (escaping),
    * compute reversible local-cache filenames.

Examples
    - MVS member: HLQ.APP.SRC(PAYROLL)
    - USS file: /u/app/src/payroll.sh
    - GDG read: HLQ.BACKUPS.GDG(+1)
    - Adapter URI: zosmf://datasets/HLQ.APP.SRC%28PAYROLL%29

### 4.2 Supported Operations

* Read / List
  * List datasets, libraries, members, GDG generations, VSAM tables, USS directories and files, and JES spool entries with paginated results for very large collections.
  * Retrieve file/member contents as full streams or as ranged/record-based reads (byte ranges, record numbers, key ranges for VSAM).

* Create / Allocate
  * Allocate new PS, PDS, PDSE datasets with configurable attributes (DSORG, RECFM, LRECL, BLKSIZE, primary/secondary extents, VOLSER hints).
  * Create new members in PDS/PDSE and create files/directories on USS with permission and ownership options.
  * Create new GDG base and allocate new GDG generations (with retention and limit parameters) via supported adapters.

* Update / Write / Replace
  * Overwrite entire dataset/member or perform record-oriented writes preserving RECFM/LRECL and RDW semantics.
  * Append records to sequential PS datasets and to files on USS where semantics permit.
  * Replace a member atomically (write to temp, validate, atomic swap) to avoid partial writes and preserve round-trip encoding.

* Delete / Deallocate
  * Remove members from PDS/PDSE and delete PS datasets or USS files with confirmation and audit logging.
  * Expire or delete GDG generations and support adapter-aware safeguards for production data.

* Rename / Move / Copy
  * Member-level rename/move within same dataset type when supported by adapter; copy members between datasets, including PDS→PDSE migration.
  * Copy datasets to local cache or export to tar/zip for archive; import archives into a dataset/member set with mapping rules.

* Member & Directory Management (PDS/PDSE)
  * List member directory with attributes (size, creation/mod time, sequence numbers if present).
  * Add new members, update member attributes, detect and report directory-full conditions with reorg recommendations.
  * Monitor directory block usage and provide automated reorg/copy suggestions for PDS directory exhaustion.

* VSAM (read-only initial)
  * Read records by key or relative record number, scan key ranges, and present VSAM control interval and catalog metadata.
  * Retrieve alternate-index metadata and support read-only traversal for complex access patterns.

* GDG and Concatenation Support
  * Enumerate GDG base and generations; read specific generation or latest; report retention and expiration metadata.
  * Present concatenated dataset chains as unified read views while exposing underlying parts and volumes in metadata.

* USS / zFS Operations
  * POSIX-style operations: stat, read, write, create, delete, rename, chmod/chown, symlink handling (subject to credentials).
  * Directory traversal and recursive operations with permission propagation options and file attribute preservation.

* JES Spool Operations (read-only)
  * List jobs and SYSOUT classes, stream job output logs, and retrieve return codes and job diagnostics through z/OSMF/Zowe adapters.

* Encoding, Conversion & Binary Handling
  * Auto-detect encoding and expose detection results; open in decoded UTF-8 preview and allow explicit encoding override.
  * Provide "treat as binary" option to bypass conversion; preserve fixed-record lengths, trailing spaces, and RDW headers.
  * Preview conversion diffs and confidence scoring before applying encoding changes.

* Record-aware Operations
  * Provide record-oriented editing APIs that respect RECFM/LRECL, maintain record boundaries, and avoid trimming or padding changes.
  * Support mapping between mainframe record numbers and local line numbers for editor integrations.

* Partial / Chunked Transfer
  * Support chunked upload/download for large members or datasets, resumable transfers, and progress events for UI feedback.

* Caching & Sync
  * Local member cache management: fetch-to-cache, validate via checksums, detect local vs remote divergence, and safe sync-back with conflict detection.
  * Delta sync: calculate diffs, optionally upload only changed records/lines, and provide dry-run previews before commit.

* Locking, Checkout & Concurrency
  * Advisory checkout/check-in semantics integrated with SCM or adapter-exclusive allocations where supported.
  * Detect dataset-in-use conditions and provide clear remediation messages; support optimistic concurrency with three-way merges and conflict markers.

* Validation & Sanity Checks
  * Validate allocation parameters before create/allocate operations and perform pre-write checks (encoding, RECFM/LRECL constraints, size limits).
  * Linting/format validation hooks for language-aware files (e.g., COBOL column rules) prior to commit.

* Promotion & SCM Interops
  * Export/import operations to/from SCM workflows (Endevor, ISPW, Git): package members for check-in, attach promotion metadata, and map promotion states.
  * Produce change sets and promotion artifacts compatible with downstream promotion connectors.

* Auditing & Telemetry
  * Emit structured audit entries for read/write/allocate/delete/promote actions; include per-action metadata (user, timestamp, dataset attributes) and checksum for integrity.
  * Support SMF/audit-forwarding hooks where adapters permit.

* Error Handling & Recovery Flows
  * Provide structured error responses with actionable remediation steps (permission changes, allocation fixes, re-run guidance).
  * Recovery operations: rollback of failed replace, orphaned temp cleanup, re-try helpers for transient adapter errors.

* Administrative Operations
  * Reorg / compact suggestions for PDS (indicate commands and parameters or automated reorg steps when authorized).
  * Directory extension or PDSE conversion helpers with simulation/dry-run and impact analysis.
  * Quota and usage reports for large libraries and cache sizing recommendations.

* Search, Indexing & Metadata Queries
  * Full-text and token searches across members with language-aware tokenization (optionally limited to cached content).
  * Query dataset metadata (RECFM, LRECL, encoding, DSNTYPE, VOLSER) and filter listings by attributes.

* Export / Import & Interchange
  * Export dataset/member sets to portable formats (ZIP/TAR, sequential files) with metadata sidecars; import mappings to target dataset/member names.
  * Git-friendly export: create a workspace snapshot suitable for Git commits with encoding metadata preserved.

* Safety & Confirmation UX
  * Require explicit confirmation for destructive or allocation-changing operations; provide a "what will change" preview including adapter commands to be executed.

* Adapter-aware Limitations & Capabilities
  * Advertise per-adapter capability matrix (which operations are supported, read-only vs read-write differences) and surface adapter-specific constraints to callers.
  * Provide fallbacks or guidance when an adapter cannot perform an operation (e.g., PDSE rename unsupported via FTP).

Notes on semantics and guarantees
    - Atomicity: writes/replaces should be atomic from the editor perspective (temp-write + swap) where the adapter supports it; otherwise provide clear consistency warnings.
    - Round-trip fidelity: encoding and record framing choices are preserved in metadata and used during sync-back to avoid accidental content loss.
    - Paging & performance: listing and large scans use pagination, indexing, and server-side filters to support very large libraries efficiently.

Implementation hooks (developer-facing)
    - Each operation exposes telemetry hooks, dry-run mode, and optional preview endpoints.
    - Operations return both human-friendly canonical path and adapter URI along with full metadata to enable reliable UI presentation and troubleshooting.

### 4.3 Error Handling

Provide robust error handling with detailed information on what went wrong and how the issue can be corrected.

The users who are reading the errors will likely be experienced Mainframe programmers or Mainframe administrators. They may have limited experience with Linux operating systems and will very little experience with Python and AI systems. Error messages must provide detailed help on how to correct the problems.

| Error                 | Example Message                                                                      | Developer Guidance                                                             |
| --------------------- | ------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------ |
| RACF not authorized   | `ICH408I USER(USER1) GROUP(DEVS) LOGON ... NOT AUTHORIZED TO DATASET HLQ.PROD.COBOL` | *Request READ access to dataset HLQ.PROD.COBOL from security admin.*           |
| Dataset in use        | `IEC161I 052-084,ALLOC FAILED`                                                       | *Dataset locked by another user/job. Try again later or coordinate with team.* |
| USS permission denied | `EACCES: Permission denied`                                                          | *Check file permissions with `ls -l`. Request update from USS admin.*          |

## 5. Command Interface

### 5.1 New Commands

Below is a comprehensive command set for the Mainframe ACI CLI (example prefix: oh-aci). Commands are grouped by area. For each command: brief syntax, purpose, required adapter(s)/permissions, important flags, and notes about expected behavior/errors. Commands follow the integration model (pluggable adapters, feature flags, optional agent) and preserve backward compatibility (no change to POSIX flows unless mainframe mode enabled).

#### 5.1.1 General CLI conventions

* Prefix: oh-aci <domain>:<action> [args]
* --adapter <name> to force a specific adapter (zosmf, zowe, ftp, nfs, agent)
* --dry-run for simulation of destructive operations
* --json / --yaml for machine-readable output
* Commands return structured error codes and remediation hints when adapter/mainframe errors occur

#### 5.1.2 Adapter & Host management

* adapter:list
  * Syntax: oh-aci adapter:list
  * Purpose: enumerate installed adapters, versions, and capability matrices.
  * Notes: used by UI to display supported ops per host.

* adapter:register
  * Syntax: oh-aci adapter:register --name <name> --path <package-or-url>
  * Purpose: install/enable an adapter plugin package (admin).
  * Permissions: admin/service account.
  * Notes: validates plugin signature and capability declaration.

* adapter:capabilities
  * Syntax: oh-aci adapter:capabilities --adapter <name> --path <canonical-path>
  * Purpose: query per-path feature support (e.g., rename PDSE via z/OSMF?).
  * Notes: helps UI decide available operations.

#### 5.1.3 Feature flags & configuration

##### 5.1.3.1 admin:feature-flag

* Syntax: oh-aci admin:feature-flag --set <flag>=<on|off> [--scope host|project]
* Purpose: enable/disable mainframe dataset semantics per-host or per-project.

* admin:config-validate
  * Syntax: oh-aci admin:config-validate --project <id>
  * Purpose: validate required adapter creds, secret-store integration, and minimal perms required.

#### 5.1.4 Dataset-level operations (PS/PDS/PDSE/GDG)

* dataset:list
  * Syntax: oh-aci dataset:list --pattern <HLQ.*> [--page N] [--adapter <name>]
  * Purpose: list datasets, PDSE/PDS libraries, or GDG base names.
  * Notes: paginated, returns canonical_mainframe_path + adapter_uri + metadata.

* dataset:info
  * Syntax: oh-aci dataset:info --ds <HLQ.NAME> [--adapter <name>]
  * Purpose: fetch metadata (RECFM/LRECL/BLKSIZE, DSNTYPE, VOLSER, encoding heuristic).
  * Notes: used before edits; returns encoding confidence.

* dataset:create
  * Syntax: oh-aci dataset:create --ds <HLQ.NAME> --type <PS|PDS|PDSE|GDG> --recfm <FB|VB|...> --lrecl N --blksize N [--extents ...]
  * Purpose: allocate dataset with validated params.
  * Permissions: adapter must support allocation; service account required.
  * Flags: --simulate (dry-run), --owner, --mode for USS-style when NFS adapter.

* dataset:delete
  * Syntax: oh-aci dataset:delete --ds <HLQ.NAME> [--force] [--adapter <name>]
  * Purpose: deallocate/remove dataset with explicit confirmation and audit log.
  * Notes: requires explicit confirmation for destructive ops.

* dataset:concat-info
  * Syntax: oh-aci dataset:concat-info --ds <HLQ.NAME>
  * Purpose: show concatenated chain parts and volumes.

#### 5.1.5 Member-level operations (PDS/PDSE)

* member:list
  * Syntax: oh-aci member:list --lib <HLQ.LIB> [--page N] [--adapter <name>]
  * Purpose: list members with attributes (size, seq no, created, encoding).
  * Notes: indicates directory block usage for PDS.

* member:read
  * Syntax: oh-aci member:read --path <HLQ.LIB(MEMBER)> [--encoding <auto|cp1047|utf-8|binary>] [--records start:end]
  * Purpose: read member contents with record-aware semantics.
  * Flags: --raw to return raw bytes including RDW; --preview to show conversion confidence sample.

* member:write
  * Syntax: oh-aci member:write --path <HLQ.LIB(MEMBER)> --file <local-path> [--encoding <...>] [--atomic]
  * Purpose: create/replace member; default atomic swap (temp+validate+swap).
  * Permissions: adapter must support write; service account required.
  * Flags: --force (override encoding mismatch), --dry-run.

* member:append
  * Syntax: oh-aci member:append --path <HLQ.LIB(MEMBER)> --file <local-path>
  * Purpose: append records to sequential PS or text-like members where adapter permits.

* member:delete
  * Syntax: oh-aci member:delete --path <HLQ.LIB(MEMBER)> [--force]
  * Purpose: delete member; returns remediation guidance on directory full or in-use failure.

* member:rename
  * Syntax: oh-aci member:rename --from <HLQ.LIB(OLD)> --to <HLQ.LIB(NEW)> [--adapter <name>]
  * Purpose: rename/move when adapter supports (PDSE usually supported; FTP may not).
  * Notes: warns if operation will perform copy+delete and potential loss of attributes.

* member:export
  * Syntax: oh-aci member:export --path <HLQ.LIB(MEMBER)> --out <local-path> [--format zip|txt|git-workspace]
  * Purpose: export member(s) with sidecar metadata (.meta.json) preserving encoding/RECFM.

* member:import
  * Syntax: oh-aci member:import --in <local-path> --target <HLQ.LIB> [--mapping file]
  * Purpose: bulk import members into PDS/PDSE or PS datasets; validates allocation parameters.

* member:preview-conversion
  * Syntax: oh-aci member:preview-conversion --path <HLQ.LIB(MEMBER)> --to-encoding <utf-8>
  * Purpose: show sample lines, confidence, and any lossy bytes before saving.

#### 5.1.6 VSAM & GDG operations

* vsam:info
  * Syntax: oh-aci vsam:info --name <HLQ.VSAM.NAME>
  * Purpose: metadata (KSDS/ESDS/RRDS), control interval info; read-only.

* vsam:read
  * Syntax: oh-aci vsam:read --name <HLQ.VSAM.NAME> [--key <value>] [--range start:end] [--limit N]
  * Purpose: read records by key or relative record number (read-only initial).

* gdg:list
  * Syntax: oh-aci gdg:list --base <HLQ.GDG.BASE> [--adapter <name>]
  * Purpose: list generations with retention/expiration metadata.

* gdg:create
  * Syntax: oh-aci gdg:create --base <HLQ.GDG.BASE> --limit N --options ...
  * Purpose: create GDG base; requires adapter support.

#### 5.1.7 USS (z/OS Unix) operations

* uss:stat
  * Syntax: oh-aci uss:stat --path /u/app/file --adapter <zowe|zosmf|nfs>
  * Purpose: stat file, permissions, owner, encoding hint.

* uss:read / uss:write
  * Syntax: oh-aci uss:read --path /u/app/file
                        oh-aci uss:write --path /u/app/file --file <local-path> [--mode]
  * Purpose: standard POSIX file ops respecting chmod/chown semantics.

* uss:chmod / uss:chown
  * Syntax: oh-aci uss:chmod --path /u/app/file --mode 0644
  * Purpose: manage permissions subject to service creds.

#### 5.1.8 JES spool

* jes:list
  * Syntax: oh-aci jes:list --jobname <name|*> [--since <date>] [--adapter <zosmf|zowe>]
  * Purpose: list jobs and SYSOUT classes.

* jes:get
  * Syntax: oh-aci jes:get --jobid <JOBID> --class <A|B|...> --out <local-path>
  * Purpose: retrieve job SYSOUT; used for diagnostics and job logs.

#### 5.1.9 Checkout / Locking / SCM & Promotion workflows

* scm:checkout
  * Syntax: oh-aci scm:checkout --path <HLQ.LIB(MEMBER)> --backend <endevor|ispw|git> [--task <id>]
  * Purpose: advisory checkout; integrates with Endevor/ISPW or local advisory lock.
  * Notes: will create a lock token or adapter-exclusive allocation when available.

* scm:checkin
  * Syntax: oh-aci scm:checkin --path <HLQ.LIB(MEMBER)> --backend <endevor|ispw|git> --message "..."
  * Purpose: submit change to SCM connector, attach metadata and promotion hints.

* scm:status
  * Syntax: oh-aci scm:status --path <HLQ.LIB(MEMBER)> --backend <...>
  * Purpose: show element state across regions (Dev/Test/QA/Prod) and latest promotion history.

* scm:promote
  * Syntax: oh-aci scm:promote --path <HLQ.LIB(MEMBER)> --from <Dev> --to <Test> --backend <endevor|ispw>
  * Purpose: invoke promotion APIs; logs audit events; returns conflict detection info.

* scm:conflicts
  * Syntax: oh-aci scm:conflicts --path <HLQ.LIB(MEMBER)> --backend <...>
  * Purpose: detect cross-region divergence and suggest resolution steps.

* scm:history
  * Syntax: oh-aci scm:history --path <HLQ.LIB(MEMBER)> [--limit N]
  * Purpose: show change log and promotion events.

#### 5.1.10 Cache & sync management

* cache:fetch
  * Syntax: oh-aci cache:fetch --path <HLQ.LIB(MEMBER)> --to <cache-root> [--checksum]
  * Purpose: fetch member to local cache, record per-member metadata.

* cache:status
  * Syntax: oh-aci cache:status --path <HLQ.LIB(MEMBER)> --local <path>
  * Purpose: show divergence, checksums, and encoding metadata.

* cache:sync
  * Syntax: oh-aci cache:sync --path <HLQ.LIB(MEMBER)> [--strategy three-way|force] [--dry-run]
  * Purpose: safe sync-back with atomic replace, conflict detection, and three-way merges where possible.

* cache:clear
  * Syntax: oh-aci cache:clear --project <id> [--older-than <days>]
  * Purpose: administrative cache pruning.

#### 5.1.11 Encoding & conversion

* encoding:detect
  * Syntax: oh-aci encoding:detect --path <HLQ.LIB(MEMBER)> [--sample N]
  * Purpose: run heuristics and return confidence scores and suggested codepage.

* encoding:set
  * Syntax: oh-aci encoding:set --path <HLQ.LIB(MEMBER)> --encoding cp1047 [--roundtrip-check]
  * Purpose: persist per-member encoding in metadata; affects subsequent writes.

* encoding:preview-diff
  * Syntax: oh-aci encoding:preview-diff --path <HLQ.LIB(MEMBER)> --to-encoding utf-8
  * Purpose: show differences and flagged lossy bytes before applying conversion.

#### 5.1.12 Transfer, chunking, and resume

* transfer:upload
  * Syntax: oh-aci transfer:upload --path <HLQ.LIB(MEMBER)> --file <local-path> [--chunk-size N] [--resume-id id]
  * Purpose: chunked/resumable upload for large members; progress events emitted.

* transfer:download
  * Syntax: oh-aci transfer:download --path <HLQ.LIB(MEMBER)> --out <local-path> [--resume-id id]
  * Purpose: chunked/resumable download.

#### 5.1.13 Search & indexing

* search:index
  * Syntax: oh-aci search:index --project <id> [--force] [--paths <...>]
  * Purpose: build language-aware index for cached members (tokenization respects encodings).

* search:query
  * Syntax: oh-aci search:query --q "SELECT" --path <HLQ.*> [--limit N] [--encoding-aware]
  * Purpose: full-text and token-based queries; supports filter by dataset metadata.

#### 5.1.14 Administrative & maintenance

* admin:reorg-suggest
  * Syntax: oh-aci admin:reorg-suggest --lib <HLQ.LIB>
  * Purpose: analyze PDS directory and recommend reorg/PDSE conversion parameters and commands.

* admin:audit-query
  * Syntax: oh-aci admin:audit-query --since <date> --user <id> --action <read|write|promote>
  * Purpose: retrieve structured audit entries; supports SMF-forwarded logs.

* admin:healthcheck / test:smoke
  * Syntax: oh-aci admin:healthcheck [--adapter <name>] | oh-aci test:smoke --project <id>
  * Purpose: run integration smoke tests (dataset read/write, USS ops, SCM checkin/out).

* admin:metrics
  * Syntax: oh-aci admin:metrics --type access|performance|adapter --since <date>
  * Purpose: fetch telemetry metrics for dashboards.

#### 5.1.15 Validation & preview utilities

* validate:allocation-params
  * Syntax: oh-aci validate:allocation-params --ds <HLQ.NAME> --params '{"RECFM":"FB","LRECL":80}'
  * Purpose: ensure adapter/allocation constraints satisfied; returns remediation steps.

* preview:diff
  * Syntax: oh-aci preview:diff --path <HLQ.LIB(MEMBER)> --local <file> [--encoding-aware]
  * Purpose: show syntax-aware diff (column-preserving for COBOL) before commit.

#### 5.1.16 Export / Import / Git workspace

* export:workspace
  * Syntax: oh-aci export:workspace --paths <HLQ.LIB(M1),HLQ.LIB(M2)> --out <tar.zip> [--include-meta]
  * Purpose: produce Git-friendly snapshot with sidecar metadata.

* import:workspace
  * Syntax: oh-aci import:workspace --in <zip> --target <HLQ.LIB> [--map-paths mapping.json]
  * Purpose: bulk import with validation and allocation simulation.

#### 5.1.17 Error mapping & remediation

* errors:explain
  * Syntax: oh-aci errors:explain --code <adapter-code> --message "raw adapter message"
  * Purpose: translate z/OS/RACF/adapter errors into human-readable remediation steps and suggested commands.

#### 5.1.18 Locking & conflict resolution

* lock:acquire
  * Syntax: oh-aci lock:acquire --path <HLQ.LIB(MEMBER)> --ttl N
  * Purpose: acquire advisory lock (SCM integrated or adapter exclusive allocation).

* lock:release
  * Syntax: oh-aci lock:release --token <token>
  * Purpose: release lock; admin override possible.

* conflicts:resolve
  * Syntax: oh-aci conflicts:resolve --path <HLQ.LIB(MEMBER)> --strategy keep-local|keep-remote|three-way --editor <tool>
  * Purpose: assist conflict resolution with three-way merge helpers that honor column semantics.

#### 5.1.19 Safety, dry-run, and simulation

* simulate:operation
  * Syntax: oh-aci simulate:operation --op "dataset:create ..." --adapter <name>
  * Purpose: simulate exact adapter commands to be executed; used by admin/UI to show "what will change".

### 5.2 Implementation notes / expectations

* Each command emits structured telemetry and audit events; destructive commands require explicit confirmation or --force.
* Commands will surface adapter capability errors (e.g., "PDSE rename not supported by FTP adapter") and suggest alternative adapters or admin steps.
* Where security-sensitive, commands use secret-store references rather than raw credentials and never echo secrets.
* All operations return canonical_mainframe_path and adapter_uri where applicable, plus metadata (encoding, RECFM, LRECL, dataset type).
* UI bindings will call these commands; they must be idempotent where possible and support --dry-run for safe UX previews.

### 5.3 Backward compatibility & feature flagging

* New commands are available only when mainframe adapters or mainframe-mode flags are enabled; POSIX adapters continue to expose existing commands unchanged.
* Admin commands to enable adapters or feature flags are provided so operators can roll out incrementally.

### 5.4 Error handling contract

* Commands must return:
  * exit code (0 success, non-zero failures with categorized codes)
  * structured error object {code, message, remediation, adapter_hint}
  * optional raw_adapter_message for debugging (logged, not shown by default)
* Common mapped errors include RACF not-authorized, dataset-in-use, directory-full, encoding-lossy, allocation-failure; each response includes actionable remediation text targeted at mainframe admins/developers.

This command set implements the full functional scope (dataset/member CRUD, VSAM/JES read, USS ops), developer experience (encoding preview, previews/diffs, local cache sync), SCM/promote integrations (Endevor/ISPW/Git), and operational/admin tooling (adapter registration, reorg suggestions, audits) described in the specification while preserving the OpenHands ACI core and plugin-based adapter model.

### 5.2 Modified Commands

Note: all changes are opt-in (feature-flag or --adapter) and backward-compatible. Default behavior and existing CLI output remain unchanged for non-mainframe workloads.

* General file/FS commands (file:list, file:stat, file:read, file:write, file:delete, file:rename, file:copy)
  * Accept mainframe path forms (HLQ.LIB(MEMBER), GDG(+n), /u/path) as optional inputs.
  * Optional flags: --adapter <zosmf|zowe|ftp|nfs|agent>, --encoding <auto|cp1047|utf-8|binary>, --records start:end, --raw.
  * When mainframe adapter used, responses include additional optional fields: canonical_mainframe_path, adapter_uri, dataset metadata (RECFM, LRECL, BLKSIZE, DSNTYPE, encoding_confidence).
  * Preserve original output shape for POSIX adapters; new fields are optional and absent unless mainframe mode/adapter selected.

* Transfer / chunking commands (transfer:upload, transfer:download)
  * Support record-aware chunking and resumable transfers for dataset/member semantics.
  * Add --atomic and --preserve-records flags for adapters that require temp-write+swap or RDW preservation.
  * Resume semantics adapt to adapter capability; report adapter_capabilities when resume unsupported.

* Member/dataset commands (dataset:list, dataset:info, member:list, member:read, member:write, member:delete, member:rename)
  * Extend to accept and validate dataset/member syntax.
  * Add encoding preview (--preview), --dry-run, and --force for encoding/attribute mismatches.
  * Member operations use atomic replace flow by default where supported; fall back behavior is clearly indicated in output.
  * Return directory-block usage and PDSE/PDS-specific warnings when applicable.

* Cache and sync commands (cache:fetch, cache:status, cache:sync, cache:clear)
  * Store per-member metadata sidecars (.meta.json) including encoding and RECFM; show these fields in cache:status.
  * cache:sync supports three-way merge strategy that preserves column semantics; default strategy unchanged unless --strategy provided.
  * cache:fetch exposes adapter_uri and canonical_mainframe_path when fetching from mainframe adapters.

* SCM and promotion commands (scm:checkout, scm:checkin, scm:status, scm:promote, scm:conflicts)
  * Integrate with adapter-backed checkout semantics (Endevor/ISPW/Git on z/OS) when --backend is a mainframe SCM.
  * Add lock token metadata and advisory-lock integration; existing SCM flows remain unchanged for non-mainframe backends.
  * Responses include promotion-state metadata and region mapping when available.

* Encoding commands (encoding:detect, encoding:set, encoding:preview-diff, member:preview-conversion)
  * New optional output: confidence score, sample lines, and lossy-byte indicators.
  * encoding:set persists per-member metadata in cache; no changes to POSIX file behavior unless applied.

* Adapter management commands (adapter:list, adapter:capabilities, adapter:register)
  * adapter:list/adapter:capabilities report per-adapter capability matrix (dataset-level ops, vsam, jes, atomic-replace) so callers can safely choose adapters.
  * adapter:register validates plugin signature and capability declaration; existing registration UX preserved.

* JES / VSAM commands (jes:list, jes:get, vsam:info, vsam:read)
  * Marked read-only by default; same CLI names retained with optional adapter gating.
  * Return adapter-specific URIs and metadata fields; non-mainframe users unaffected.

* Search / index / export commands (search:index, search:query, export:workspace, import:workspace)
  * Tokenization becomes encoding-aware when indexing cached mainframe content; existing indexing for POSIX remains unchanged.
  * export:workspace includes sidecar metadata when exporting mainframe paths; flag-controlled to avoid changing default exports.

* Admin / health / test commands (admin:healthcheck, admin:config-validate, test:smoke)
  * Health checks and smoke tests include adapter-specific tests when adapter selected or mainframe-mode enabled.
  * No additional checks run by default to avoid impacting non-mainframe deployments.

* Error handling changes (errors:explain and all commands)
  * Commands optionally return structured error object {code, message, remediation, adapter_hint} and raw_adapter_message (logged, not shown by default).
  * Mainframe-specific error translations (RACF, IEC, ICH messages) surfaced only when adapter/mainframe mode used.
  * Exit codes extended with new categorized codes for dataset-in-use, encoding-lossy, directory-full — existing exit codes preserved for backwards compatibility.

* Locking & concurrency (lock:acquire, lock:release, conflicts:resolve)
  * Support adapter-backed advisory locks and return lock token metadata; default (non-mainframe) locking unchanged.
  * conflicts:resolve provides three-way merge helpers that honor column semantics when mainframe files are involved.

### 5.2.1 Implementation constraints and guarantees

* All modifications are additive and opt-in: core command signatures remain valid; new flags are optional.
* Feature-flag: mainframe-mode and per-host feature flags ensure changes are enabled by admin/project; CLI usage without mainframe flags behaves as before.
* Capability discovery: clients should call adapter:capabilities before invoking mainframe-specific operations; commands will return clear "adapter not supported" responses rather than silent failures.
* Telemetry and audit: new adapter/encoding fields are emitted to telemetry but not required by existing clients.
* Documentation: updated CLI help and machine-readable schemas will be provided; backward-compatible JSON/YAML outputs preserve prior fields.

This scoped, additive approach enables mainframe semantics where needed while minimizing or eliminating impact to the broad base of non-mainframe OpenHands users.

## 6. Security Considerations

Security is paramount for any Mainframe integration. Mainframe systems typically host mission‑critical, highly sensitive workloads and must meet strict operational, legal, and regulatory constraints. The ACI must enforce defense‑in‑depth controls (authentication, authorization, encryption, auditability), minimize attack surface, preserve data integrity and availability, and provide traceable auditable actions. Design and implementation choices should explicitly account for DoD/SRG, NIST (SP 800‑53/800‑171), FedRAMP, FIPS, and other applicable regulatory frameworks; evidence and configuration guidance for compliance (hardening checklists, control mappings, and audit artifacts) should be produced as part of the delivery.

### 6.1 Authentication

#### 6.1.1 Supported and recommended authentication methods

* Federated SSO (preferred for interactive users): SAML 2.0 and OpenID Connect (OIDC) with MFA enforced by identity provider (IdP).
* MFA: time‑based one‑time passwords (TOTP), hardware tokens, enterprise MFA providers, or IdP‑enforced adaptive MFA for high‑risk operations.
* x.509 client certificates and mutual TLS (mTLS) for service‑to‑service and adapter authentication (z/OSMF, Zowe, agent).
* Kerberos and LDAP integration where enterprise policy requires direct directory integration for admin/automation users.
* SSH key authentication for SFTP/agent channels; keys must be centrally managed, rotated, and stored in a credential store.
* Service accounts: scoped, least‑privilege service IDs for adapters (RACF/ACF2/Top Secret entries); use short‑lived tokens where supported.
* z/OS native mechanisms: support for RACF/ACF2/Top Secret authentication flows and z/OSMF token exchange when available.

#### 6.1.2 Implementation notes

* Centralize credential issuance and validation via enterprise IdP; prefer short‑lived tokens and avoid long‑lived static credentials.
* Enforce FIPS‑approved cryptographic algorithms and TLS 1.2/1.3 with strong cipher suites for all channels.
* Do not store raw credentials in code or config; require secret-store references (Vault, KMS).
* Provide configuration guidance and automation for IdP integration, certificate provisioning, and MFA enrollment to support DoD/FedRAMP/NIST control requirements.

### 6.2 Authorization

#### 6.2.1 Supported and recommended authorization approaches

* Role‑Based Access Control (RBAC): map organizational roles to coarse and fine‑grained permissions (dataset read/update/allocate/promote).
* Attribute‑Based Access Control (ABAC) / policies: support contextual rules (time, source IP, region, promotion state) for sensitive ops.
* Mainframe native authorization: integrate with RACF/ACF2/Top Secret for dataset‑level enforcement and to reflect existing enterprise policies.
* Principle of least privilege: default deny, grant minimal required privileges for service accounts, and segregate duties for promotion/production actions.
* Scoped service tokens: adapter/service tokens with limited scope and TTL for automation and CI/CD integrations.
* Dataset/member ACLs and adapter capability gating: expose per‑adapter capability discovery to prevent unauthorized allocation or destructive operations.
* Approval workflows and separation of duties for promotion actions (Endevor/ISPW): enforce multi‑actor approvals where required by policy.

#### 6.2.2 Implementation notes

* Provide mapping and synchronization utilities between Linux/ACI roles and mainframe security identities; log any identity mapping decisions for audit.
* Enforce authorization checks in both UI/CLI and adapter layers; fail closed on missing permissions.
* Support admin overrides and emergency workflows with strict audit trails and temporary elevated session lifetimes to meet compliance requirements.

### 6.3 Secure Coding & Development Practices

#### Minimum development and delivery controls

* Secure SDLC: threat modeling, documented security requirements, and security sign‑off gates per release.
* Static and dynamic analysis: integrate SAST, DAST, and dependency vulnerability scanning (SCA) into CI pipelines; fail builds on critical/high findings.
* Secrets management: prevent secrets in source control; enforce scanning for accidental secrets and require secret references to approved stores (Vault/KMS).
* Input validation and canonicalization: strict validation of all mainframe path inputs (dataset/member syntax), encoding handling (EBCDIC ↔ UTF‑8), and adapter parameters to prevent injection or malformed requests.
* Encoding and record semantics safety: implement rigorous tests and validators for encoding conversions, round‑trip fidelity checks, and reject risky automatic conversions without explicit user confirmation.
* Safe defaults: secure configuration defaults (TLS required, minimal exposed services, non‑root service account).
* Review & testing: mandatory code reviews, unit/integration tests that include negative/error cases, fuzzing for parser components (JCL/COBOL copybooks), and regular regression tests against representative mainframe samples.
* Dependency control and reproducible builds: pin versions, use lockfiles, and produce build artifacts with provenance (SBOM).
* Release hardening: artifact signing, reproducible builds, and staged rollouts with smoke/security tests before production.

### 6.4 Ecosystem & Dependency Security

#### 6.4.1 Controls for runtime and third‑party components

* Supply chain security: maintain an SBOM for all components, require signed artifacts, and adopt verification (e.g., Sigstore) for CI/CD artifacts.
* Vulnerability management: continuous CVE monitoring, prioritized patching SLAs, and automated alerts for critical vulnerabilities in OS, containers, and libraries.
* Container and host hardening: use minimal base images, apply CIS/organization hardening benchmarks, run under non‑root service accounts, and enforce kernel/sysctl restrictions where applicable.
* Third‑party adapter & plugin governance: require vetting, code signing, capability declarations, and runtime sandboxing for plugins and agent components.
* Runtime protections: implement network segmentation, firewalling, host‑based IDS/IPS, and process isolation for adapters that perform privileged dataset operations.
* Secrets & key lifecycle: centralized secret store (Vault/KMS) with automated rotation, access audit, and per‑adapter issuance of short‑lived credentials; manage TLS keys with HSMs when required by policy.
* Logging, audit, and monitoring: forward structured audit events and SMF mappings to SIEM (Splunk, ELK, or cloud SIEM) with retention policies aligned to regulatory requirements; include tamper‑evident logs for promotion and allocation events.
* Compliance evidence and controls mapping: provide documentation and automation to map controls to NIST/FedRAMP/DoD requirements (control implementations, test evidence, and configuration checklists).
* Regular assurance: scheduled penetration tests, periodic third‑party security assessments, and red/blue team exercises focusing on adapter surfaces and promotion workflows.

#### 6.4.2 Summary of compliance posture

* The platform must be configurable to meet DoD, NIST, and FedRAMP baselines: enforce FIPS‑compliant crypto, strong identity providers with MFA, hardened hosts/containers, auditable promotion workflows, and documented control implementations. Provide operator playbooks and audit artefacts (SBOM, logs, control mappings) to support accreditation/authorization processes.

## 7. Testing Strategy

Stability is critical: mainframe environments tolerate almost no unplanned downtime and many sites require predictable, auditable behavior. The testing strategy prioritizes correctness, round‑trip fidelity (encoding & record framing), safe failure/recovery, and long‑running stability so the ACI behaves deterministically in production. Tests are layered (unit → integration → performance/soak) and separated by risk (local/emulator tests for community contributors; gated real‑mainframe tests for integration with production-like systems).

### 7.1 Unit Testing

This module will extend the project’s existing unit test strategy with focused, fast unit tests that validate dataset/member semantics, adapter contracts, encoding conversions, and caching/atomic-swap behaviors.

#### 7.1.1 Key points

* Frameworks/tools: pytest, unittest.mock, hypothesis (property-based tests) and coverage enforcement in CI.
* Focus areas:
  * Encoding round‑trip: cp1047/cp037 ↔ UTF‑8, preservation of trailing spaces, fixed‑record lengths, and detection of lossy bytes.
  * Record framing: respect RECFM/LRECL/RDW and mapping between record numbers and editor line numbers.
  * Adapter contracts: adapter interface unit tests using mocked network/file backends (requests-mock / responses for HTTP, pytest‑asyncio for async adapters).
  * Cache & atomic replace: temp-write + validate + atomic swap semantics, checksums and metadata sidecar handling.
  * Error mapping: map common z/OS error patterns to remediation objects without requiring a real mainframe.
* CI policies:
  * Fast unit test suite run on every push.
  * Strict thresholds for new code (coverage gates for modified modules).
  * Tests must not require network or real mainframe credentials.

### 7.2 Integration Testing

Integration tests run in two classes: simulated/community-friendly environments, and gated tests against real mainframes. Tests are automated where possible and clearly labeled by required environment.

#### 7.2.1 Simulated / open-source integration harness (for community contributors)

* Goals: provide realistic but legal and easily reproducible testbeds so contributors without mainframe access can validate behavior.
* Recommended free/open tools and approaches:
  * Hercules (System/370/ESA/390 emulator) + public-domain MVS images (e.g., MVS 3.8j) for basic PDS/PS behaviors and job spool testing. Note: do not distribute proprietary z/OS images.
  * z390 (IBM/360/370 simulator) for smaller legacy workloads and assembler testing.
  * QEMU system-s390x running Linux on s390x for USS-like behavior and running Zowe CLI/SDK locally.
  * WireMock or MockServer to emulate z/OSMF/Zowe/SCM REST APIs and return configurable responses for allocation, list, read/write and JES endpoints.
  * Containerized mock SFTP/FTP servers (OpenSSH, pyftpdlib) to simulate file/PDSE access over legacy protocols.
  * Local file-backed dataset simulators: scripts that create directory structures and sidecar .meta.json files to emulate RECFM/LRECL/encoding semantics for fast functional tests.
  * Zowe CLI (installed in containers) pointed at a WireMock z/OSMF mock to validate Zowe adapter integration.
* Test patterns:
  * End-to-end flows using the mock stack: dataset:create → member:write → member:read → cache:sync → scm:checkin.
  * Large-scale functional scenarios: synthetic generator to create 10k+ member PDS directories to exercise listing and pagination logic.
  * Adapter capability discovery tests using mocked capability matrices to verify UI/CLI behavior when adapters report limited capabilities.
* Distribution & automation:
  * Provide docker-compose/testcontainers recipes and test fixtures so contributors can spin up the full mock environment locally or CI runners can use them.
  * Mark these tests as runnable in public CI (e.g., GitHub Actions) without secrets.

7.2.2 Real mainframe integration (gated, controlled)

* Goals: validate behavior against real z/OS, z/OSMF, Endevor/ISPW, VSAM, and JES endpoints; validate security, audit, and performance under realistic conditions.
* Requirements & process:
  * Use dedicated non‑production LPARs / test systems and scoped service accounts with least privilege.
  * Coordinate with mainframe operations for scheduling, approvals, and resource limits.
  * Use feature flags and adapter capability gating so tests only run in environments explicitly enabled.
  * Test classes: authentication flows (mTLS, token exchange), dataset allocation/delete, member atomic-replace, Endevor/ISPW check-in/out, VSAM read tests (read-only), JES spool fetches, and SMF/audit forwarding.
  * Safety: tests must be idempotent, clean up artifacts, and require explicit human approval or CI job tokens to run.
  * Logging & audit: capture SMF/audit indicators and adapter logs; do not store plaintext credentials in logs.
* CI integration:
  * Real-mainframe integration tests run in separate, gated pipelines (nightly or on-demand) with strict RBAC for who can trigger them.
  * Test results include structured artifacts (telemetry, SMF extracts, diff outputs) for post‑mortem.

### 7.3 Performance Testing

Performance testing ensures the ACI meets throughput and latency expectations both on emulator-based harnesses and real mainframes; it also verifies long-duration stability (soak tests).

#### 7.3.1 Emulator / community performance tests

* Goals: provide reproducible performance baselines contributors can run locally or in CI.
* Scenarios:
  * Large PDS/PDSE listing: generate a simulated library with ≥10,000 members and measure listing latency, pagination behavior and memory use.
  * Concurrent access: spawn many concurrent adapter clients (e.g., 50–200 simulated editors) performing reads/writes to validate adapter-side throttling and lock behavior.
  * Encoding & conversion stress: measure CPU and latency for bulk conversion workload (EBCDIC→UTF‑8) across large files.
  * Chunked transfers: validate resumable upload/download throughput and correctness under network interruptions (use tc/netem to inject latency/loss).
* Tools:
  * locust or custom asyncio-based Python load scripts for adapter API load.
  * wrk or hey for HTTP endpoint stress.
  * Prometheus + exporters to collect CPU, memory, latency; Grafana for dashboards.
* CI policy:
  * Lightweight perf smoke in PRs; nightly heavier synthetic runs against local emulator stacks.

#### 7.3.2 Real mainframe performance testing

* Goals: validate real-world throughput, interaction with z/OS resource managers, and impact on production-like systems.
* Scenarios:
  * End-to-end promotion workflows at realistic concurrency: multiple check-in/promote operations involving SCM connectors, measuring time-to-completion and SMF/RMF impact.
  * Large-member edits and bulk exports/imports simulating release packaging.
  * Long-duration soak tests (24–72 hours) to catch memory leaks, resource exhaustion, and lock-contention behaviors.
* Data collection:
  * Use SMF/RMF metrics, z/OS performance traces, adapter logs, and ACI telemetry exported to internal monitoring (Prometheus/Splunk).
  * Correlate ACI events with z/OS metrics to identify hot paths and bottlenecks.
* Safety & governance:
  * Execute only on test LPARs or with explicit approval windows.
  * Run throttled ramps with rollback triggers when thresholds are exceeded.
  * Produce performance reports and recommended configuration/scale guidance (vCPU, memory, cache sizing) for operators.

#### 7.3.3 General test governance

* Test tagging: clearly mark tests as unit/integration/emulator/real-mainframe/perf; automate gating based on tags.
* Secrets & creds: tests reference credentials via secret-store references; CI runners inject tokens at runtime only for gated jobs.
* Reproducibility: provide reproducible datasets, fixtures, and automation scripts (docker-compose, terraform for test infra where allowed).
* Acceptance criteria: define minimal SLAs (latency, throughput, resource usage) and regression alerts; require sign-off for each release for production deployments.

This layered approach lets community contributors validate correctness locally while giving operations teams rigorous, auditable tests against real z/OS systems before production rollout — meeting the extreme‑stability needs of mainframe environments.

## 8. Documentation Requirements

### 8.1 User Documentation

#### 8.1.1 Mainframe developers (primary audience: application devs, release engineers)

##### 8.1.1.1 Deliverables

* Quickstart guide
  * Getting started (connect to ACI, authenticate, open a dataset/member, edit, save, sync-back).
  * Example end-to-end flow: fetch → edit → preview conversion → check-in → promote.
* Workflow guides
  * Typical developer workflows (Dev→Test→QA→Prod promotion patterns, check-out/check-in with Endevor/ISPW, Git-on-z/USS workflows).
  * Local cache usage and offline editing workflow.
* Editor & CLI reference
  * Editor UI elements (encoding status, dataset attributes, column/record mode).
  * CLI reference for relevant commands (member:read/write, cache:sync, scm:checkout/checkin).
  * Examples and common flags (--adapter, --encoding, --dry-run, --atomic).
* Encoding & record semantics guide
  * EBCDIC↔UTF‑8 heuristics, when to use binary_mode, handling RECFM/LRECL/RDW, preserving trailing spaces and column semantics.
  * Preview-conversion and round‑trip checks with sample outputs.
* Language-specific guidance
  * COBOL/JCL/PL/I copybook resolution, column-aware editing rules, common lint/formatter guidance.
* Error mapping & remediation
  * Common mainframe errors (RACF not-authorized, dataset-in-use, directory full): raw sample messages, human-friendly explanations, step-by-step remediation and required contact/privileges.
* Best practices & patterns
  * Safe-editing checklist, conflict-avoidance (checkout/locks), promotion hygiene, copybook management.
* Troubleshooting recipes
  * Quick fixes for common failures, how to collect logs/diagnostics (what to include for an ops ticket).
* Examples & patterns
  * Sample projects, JCL submission templates, copybook include path examples.
* FAQ and policy notes
  * Answer common questions about encoding, permissions, and supported dataset types.
Format & delivery
* Markdown-based docs in repo (docs/) with short runnable examples.
* CLI man pages and OpenAPI/JSON Schema for machine consumption.
* Short video demos / step-by-step screenshots for common tasks.

#### 8.1.2 Mainframe administrators (primary audience: sysadmins, security, ops)

##### 8.1.2.1 Deliverables

* Installation & enablement guide
  * zLinux host provisioning, minimal sizing, recommended systemd/container deployment, reverse-proxy/TLS setup, certificate provisioning.
  * Step-by-step adapter installation (zosmf, Zowe, FTP, NFS, agent).
* Network & connectivity guide
  * Required ports, TLS/mTLS config, firewall rules, NFS/export considerations, z/OSMF/Zowe endpoints and test commands.
* Service account & RACF/ACF2/Top Secret provisioning
  * Least-privilege entitlement matrix per adapter operation (read, update, allocate, promote); sample RACF rules or templates.
* Secret-store integration
  * Vault/KMS wiring, recommended patterns, rotation procedures and examples.
* Hardening & compliance checklist
  * TLS/FIPS settings, non-root service account, file permissions, plugin governance, SBOM and artifact signing requirements.
* Adapter capability & deployment matrix
  * Which adapters support which operations; mapping to capability flags and how to restrict allocations.
* Observability & monitoring playbook
  * Telemetry schema, Prometheus metrics, log forwarding (SMF/audit mapping), alerting thresholds, dashboards and healthchecks.
* Backup, restore & disaster recovery
  * Configuration backup, local cache snapshot strategy, recovery steps, and emergency rollback procedures.
* Upgrade, rollback & change-control runbooks
  * Safe upgrade steps, smoke tests to run post-upgrade, rollback steps.
* Operational runbooks & troubleshooting
  * Common failure scenarios, diagnostic commands, log locations, how to gather traces for vendor support.
* Audit & compliance artifacts
  * How to prove control mappings (NIST/FedRAMP), retention guidance for audit logs, sample SMF/audit-forward configurations.
Format & delivery
* Step-by-step runbooks (procedural), architecture diagrams, YAML/Ansible/Terraform examples for automated deployments, checklists and one‑page playbooks for on-call staff.

### 8.2 Developer Documentation (OpenHands contributors, adapter/plugin developers)

Audience: OpenHands core contributors, adapter implementers, integrators writing Endevor/ISPW/Git connectors or agent-based adapters.

#### 8.2.1 Core deliverables

* Architecture overview
  * Component diagram (core ACI engine, plugin registry, adapter model, cache manager, telemetry), data flows for open/read/write/sync, and error handling flow.
* Adapter SDK / Plugin development guide
  * Adapter interface contract (method signatures, expected behaviors, capability discovery), sample adapter skeleton, packaging & registration process, capability matrix declaration.
  * Adapter capability testing checklist and stub/mocking patterns.
* API specification
  * Public REST/CLI API reference (OpenAPI), payload schemas, canonical_mainframe_path vs adapter_uri fields, structured error object schema {code,message,remediation,adapter_hint}.
* Data models & metadata
  * Dataset/member metadata schema (.meta.json), encoding/confidence model, RECFM/LRECL/BLKSIZE fields, lock token metadata and promotion-state model.
* Encoding & record semantics API
  * Libraries/utilities for encoding detection, conversion helpers, record-framing utilities, round‑trip check helpers, and guidance for binary detection.
* Caching & sync contracts
  * Cache metadata sidecar format, atomic-replace flow, checksum strategies, three‑way merge hooks and conflict marker formats.
* Testing & mock harnesses
  * Unit/integration test patterns, test fixtures, emulator/mock server recipes (WireMock, containerized SFTP/FTP, local dataset simulator), community-friendly test suite guidance.
* CI/CD & release process
  * Build/release pipelines, SBOM generation, artifact signing, packaging for plugin distribution, versioning policy and compatibility guarantees.
* Security requirements for contributors
  * Secure coding checklist, threat model, SAST/DAST/SCA guidance, secret-handling rules, signing and vetting for third-party adapters.
* Telemetry, logging & observability
  * Telemetry schema, metric names, log structured format, audit event shape for SMF forwarding, guidelines for redaction of sensitive fields.
* Performance & scalability guidance
  * Benchmarks, large-PDS pagination patterns, resource limits, recommended thread/async models for adapters and memory patterns for large listings.
* Contribution & governance
  * Coding standards, linting/formatting rules, review checklist, plugin approval workflow, security review checklist for adapter submissions.
* Examples & reference implementations
  * Complete sample adapters (zosmf, ftp, nfs), end-to-end integration example, sample smoke-test scripts and Docker-compose harness.
* Troubleshooting & debugging
  * How to enable debug traces, reproduce adapter errors with mock servers, common pitfalls (encoding loss, RDW mishandling), and remediation patterns.
Documentation formats & artifacts
* Source: Markdown in repo docs/, auto-generated OpenAPI and CLI man pages, example code in /examples, test harnesses in /test-fixtures.
* Machine-readable: JSON Schema/OpenAPI for APIs, proto/IDL if used, schema for telemetry.
* Release notes and migration notes per release describing breaking changes, new capabilities, and adapter compatibility.
Ownership & maintenance
* Document owners per area (core, adapters, security, docs) and an SLA for doc updates (docs must be updated in same PR as code changes that affect behavior).
* Doc testing: include doc-snippets in unit/integration tests where feasible (executable examples).
* Localization & accessibility: plan for English primary docs, with templates for localization as needed.
* Onboarding: a short "developer quickstart" that lets a new contributor run the mock harness and implement/validate a minimal adapter in under one hour.

### 8.3 Requirements for docs to be accepted

* Must include runnable examples and CI-verified tests.
* Must include a minimal smoke-test and a security checklist for any adapter that requests elevated capabilities.
* Must provide a compatibility table and capability declaration for each adapter package.
* Must provide explicit guidance for migrating existing OpenHands filesystem adapters to dataset-member model (mapping table, example code).
* Must expose machine-readable schemas (OpenAPI / JSON Schema) and include examples for every public API endpoint.

This set of documentation deliverables ensures clear user guidance for developers and admins and provides implementers the reference material to build, test and operate adapters and plugins consistently with OpenHands ACI mainframe semantics.

## 9. Migration and Compatibility

### 9.1 Backward Compatibility

* Opt‑in model: mainframe semantics are disabled by default. Existing OpenHands installations and workflows continue to operate unchanged unless an admin explicitly enables mainframe-mode or installs mainframe adapters.
* Additive API changes: new fields (canonical_mainframe_path, adapter_uri, dataset metadata, encoding_confidence, structured error {code,message,remediation,adapter_hint}) are optional. Clients that ignore unknown fields are unaffected.
* CLI compatibility: existing commands and default output shape are preserved. New flags (--adapter, --encoding, --records, --raw, --atomic, --dry-run) are optional and only change behavior when used or when mainframe adapters are active.
* Adapter sandboxing: adapters are delivered as separate plugins/packages and run under capability gating. Installing or registering an adapter does not alter core filesystem adapters or their behavior.
* Capability discovery: callers should consult adapter:capabilities; operations unavailable via an adapter will return clear, non‑fatal errors rather than silently changing semantics.
* Semantic versioning & deprecation policy:
  * Core changes are additive (minor/patch). Any breaking change will be announced, documented, and follow a deprecation schedule (deprecation notices in release notes and CLI warnings).
  * Plugin APIs and adapter SDKs follow their own compatibility guarantees; breaking adapter SDK changes will be versioned and documented.
* Safety & security preserved: new telemetry/metadata fields avoid including secrets and are optional; default logging behavior and redaction rules remain in effect.

### 9.2 Migration Path

High‑level upgrade steps (recommended staged rollout)

1. Prepare & validate (admin)
     * Back up current configuration, service account definitions, and local caches.
     * Ensure secret store (Vault/KMS) is configured and accessible to the ACI host.
     * Verify Python/runtime requirements on zLinux match the support matrix.
     * Schedule a staged rollout (staging → pre‑prod → production).

2. Install adapters & plugins (admin)
     * Install adapter packages as plugins (example):
         * oh-aci adapter:register --name zosmf --path /packages/oh-aci-adapter-zosmf.whl
     * Validate adapter integrity (signed packages) and confirm capability matrix:
         * oh-aci adapter:capabilities --adapter zosmf --path HLQ.PROJ.LIB

3. Enable mainframe features (admin, opt‑in)
     * Toggle per-host or per-project feature flag:
         * oh-aci admin:feature-flag --set mainframe-mode=on --scope host
     * Use feature flags progressively per project to limit blast radius.

4. Configuration & credentialing (admin)
     * Provision and store service credentials in Vault/KMS; reference them via secret-store URIs in adapter configs.
     * Create scoped service accounts on z/OS with least privilege; document RACF/ACF2/Top Secret entitlements required for adapter operations.
     * Run config validation:
         * oh-aci admin:config-validate --project <id>

5. Cache & data migration (optional)
     * If importing an existing local workspace to dataset/member model, export and re-import using export:workspace / import:workspace or use cache migration scripts.
     * Preserve .meta.json sidecars; otherwise run encoding:detect and encoding:set on critical libraries before bulk sync.

6. Smoke tests & verification (admin/dev)
     * Run adapter‑specific smoke tests in staging:
         * oh-aci test:smoke --project <id>
     * Verify common flows: member:read → member:preview-conversion → member:write → scm:checkin → scm:promote.
     * Confirm telemetry/audit entries appear in observability pipeline.

7. User guidance & client updates (developer)
     * Share user docs: quickstart, editor & CLI reference, encoding guide, and error-remediation docs.
     * Recommend users update local OpenHands ACI extensions/CLI if a new client version is published (minor client updates for UI changes; clients remain compatible if not updated).
     * Provide a short checklist for developers (how to opt into a project that uses mainframe-mode).

8. Rollback & fallback plan (admin)
     * If issues occur, disable mainframe-mode feature flag for affected host/project:
         * oh-aci admin:feature-flag --set mainframe-mode=off --scope project
     * Unregister problematic adapters:
         * oh-aci adapter:unregister --name <name>
     * Restore configuration/cache from backups if needed and reopen support ticket with logs and smoke-test artifacts.

9. Operationalization & ongoing maintenance
     * Add adapter capability checks to CI pipelines and CI smoke tests that exercise the new flows.
     * Monitor telemetry for adapter errors, encoding-lossy warnings, directory-full and dataset-in-use events.
     * Follow the documented upgrade notes in release notes; apply plugin updates separately from core ACI to minimize impact.

Quick checklist (summary for upgrade tickets)

* Backup configs and local caches.
* Ensure secret-store is in place and service accounts provisioned on z/OS.
* Install & register adapters; validate capabilities.
* Enable mainframe-mode behind feature-flag for a limited scope.
* Run admin:config-validate and test:smoke in staging.
* Roll out to projects incrementally; provide user docs and training.
* Monitor, and be ready to disable feature-flag and restore if needed.

This migration approach ensures existing OpenHands users experience no unexpected change by default, gives operators controlled, reversible steps to enable mainframe capabilities, and provides clear validation and rollback paths to minimize operational risk.

## 10. Implementation Plan

### 10.1 Phases

Phase 1: Core ACI fork + USS/NFS + z/OSMF/Zowe adapters.

Phase 2: Endevor + Git integration.

Phase 3: ISPW + RTC support.

Phase 4: Region-aware promotion hints + conflict detection.

Phase 5: Advanced AI-assisted migration planning.
