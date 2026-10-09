# AEGIS Stack Profile

Last updated: 2026-09-15

## AEGIS's own stack (highest priority)
- Docker / Docker Compose (Docker Engine, containerd)
- PostgreSQL
- nginx
- Oracle Cloud Infrastructure (the VM AEGIS runs on)
- GitHub Actions (CI/CD)

## Customer-typical SMB infrastructure (secondary priority)
- SSO providers: Okta, Azure AD / Entra
- Common CMS/ERP software
- Popular cloud providers: AWS, GCP, Azure

## Adjacent watch (context, not primary)
- The LLM/agent-security space itself — new prompt-injection or agent-security disclosures relevant to AEGIS's own product category

## Accuracy Upgrade Proposal: Extension & Third-Party Package Filtering
- **Context:** Previous scans surfaced third-party extensions/MCP wrappers (e.g. `postgres-mcp-server`, PostGIS `address_standardizer`, `docker/mcp-gateway`) which match core stack keywords but are maintained separately and do not represent core vulnerabilities in Docker Engine, PostgreSQL, nginx, or OCI.
- **Rule / Refinement:** In `/scan-threats` filtering (Step 3), explicitly categorize matching CVEs into *Core Stack* vs *Third-Party Wrapper / Extension / MCP*, automatically discarding the latter unless verified as actively installed/deployed in AEGIS's production manifest (`template.yaml` / compose files).
- **How to apply:** During Step 3 keyword query evaluation, check package namespace and maintainer; ignore auxiliary plugins/MCP tools unless explicitly listed in `template.yaml`.

## Accuracy Upgrade Proposal 2: Container Escape vs Host Kernel Distinction
- **Context:** Container engine (Docker/containerd) CVEs often trigger high CVSS scores, but container escape vulnerabilities specifically require local root access inside the container, uncontained cgroups, or specific kernel/namespace configurations not present in AEGIS's hardened rootless/resource-limited Oracle Cloud VM setup.
- **Rule / Refinement:** In `/scan-threats` analysis, distinguish between host kernel CVEs affecting the Linux 6.12 kernel on OCI vs container runtime daemon CVEs. Escalate only if the attack vector requires no prior privileges or if container isolation is directly bypassed without local container access.
- **How to apply:** Cross-reference runtime CVE advisories against container privilege configuration and kernel version (Linux 6.12.76) before escalating.

## Accuracy Upgrade Proposal 3: GitHub Actions & CI/CD Action Pinning / Supply Chain Integrity
- **Context:** AEGIS stack includes GitHub Actions for CI/CD automation. Vulnerabilities and supply-chain attacks frequently target third-party actions using floating tags or compromised maintainer credentials.
- **Rule / Refinement:** When polling CI/CD and GitHub Actions advisories in `/scan-threats`, prioritize actions utilizing unpinned references (floating tags like `@v2` or branch references) or verified action repo compromises over theoretical or local-only workflow issues.
- **How to apply:** Filter GitHub Actions CVEs to specifically target action execution or runner runner-environment escapes when checking CI pipeline components.

## Accuracy Upgrade Proposal 4: Oracle Cloud Infrastructure (OCI) Network Gateway & Load Balancer Configuration
- **Context:** AEGIS runs on Oracle Cloud Infrastructure. While OCI manages core hypervisor security, tenant-configured virtual network security lists, VCN gateways, and OCI load balancer SSL/TLS cipher policies require tenant-side configuration review.
- **Rule / Refinement:** Distinguish between OCI physical hypervisor/infrastructure CVEs (automatically patched by Oracle) and tenant-serviceable VCN security rule or load-balancer configuration advisories.
- **How to apply:** Filter OCI security bulletins to tenant-configurable networking and load-balancing components before flagging for ceo escalation.

## Accuracy Upgrade Proposal 5: Nginx Reverse Proxy Upstream Security & HTTP Request Smuggling
- **Context:** AEGIS utilizes nginx as a reverse proxy fronting its web and API services. Vulnerabilities in HTTP parsing, chunked transfer encoding, and upstream proxy header handling pose direct risk to proxy-level security.
- **Rule / Refinement:** Prioritize nginx CVEs affecting HTTP/1.1 or HTTP/2 request smuggling, buffer overflow in header parsing, or upstream connection spoofing over local denial-of-service vulnerabilities requiring pre-authenticated administrative access.
- **How to apply:** Filter nginx CVE advisories to assess impact specifically on public-facing reverse proxy proxy-pass configurations and TLS termination.

## Accuracy Upgrade Proposal 6: PostgreSQL Extension Privileges & Connection Handling
- **Context:** AEGIS uses PostgreSQL for backend data persistence. Vulnerabilities often arise in non-core extensions (e.g. contrib modules, untrusted languages) or connection pooling/authentication mechanisms.
- **Rule / Refinement:** Distinguish between core PostgreSQL engine vulnerabilities requiring superuser execution and lower-risk extension/contrib module issues unless untrusted users have CREATE privilege on target databases.
- **How to apply:** Filter PostgreSQL CVEs to focus on remote code execution, unauthenticated connection pool exhaustion, or privilege escalation via untrusted extensions.

## Change log
- 2026-09-15: Initialized default stack profile for fleet test.
- 2026-09-16: Added stack-profile accuracy upgrade proposal for extension/MCP false-positive filtering.
- 2026-09-17: Added stack-profile accuracy upgrade proposal 3 for GitHub Actions supply chain integrity.
- 2026-09-18: Added stack-profile accuracy upgrade proposal 4 for OCI network gateway and load balancer configuration filtering.
- 2026-09-19: Added stack-profile accuracy upgrade proposal 5 for Nginx reverse proxy upstream security and request smuggling filtering.
- 2026-09-20: Added stack-profile accuracy upgrade proposal 6 for PostgreSQL extension privileges and connection handling.
