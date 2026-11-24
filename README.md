# NetMirror / net77.cc — Public Infrastructure R&D

**Investigation date:** 24 November 2026  
**Method:** Read-only DNS, WHOIS, TLS, and HTTP observations from a Mac on an Indian internet connection.  
**Scope:** `netmirror.gg`, `net77.cc`, `net52.cc`, and media-related hostnames observed during browsing.

> **Evidence standard:** “Confirmed” below means **directly observed in the supplied command output at the time of testing**, not a permanent guarantee or independent proof of server ownership/location. In particular, Cloudflare edge locations are **not** origin-server locations.

## 1. Confirmed observations (direct command output)

### DNS and routing

| Hostname | Observed result | Confidence / limitation |
|---|---|---|
| `netmirror.gg` | Nameservers `alexis.ns.cloudflare.com`, `ingrid.ns.cloudflare.com` | **Confirmed DNS lookup result**; DNS can change. |
| `net77.cc` | Nameservers `rosalyn.ns.cloudflare.com`, `sergi.ns.cloudflare.com` | **Confirmed DNS lookup result**. |
| `net52.cc` | Nameservers `ariadne.ns.cloudflare.com`, `pranab.ns.cloudflare.com` | **Confirmed DNS lookup result**. |
| `net77.cc` | Cloudflare-proxied IPv4 answers observed: `172.67.73.17`, `104.26.7.150`, `104.26.6.150` | **Confirmed observed DNS answers**; these are not proven origin IPs. |
| `net52.cc` | Observed A records: `172.67.71.108`, `104.26.0.214`, `104.26.1.214` | **Confirmed observed DNS answers**. |
| `imgcdn.kim` | Observed A records: `104.26.3.88`, `172.67.72.14`, `104.26.2.88` | **Confirmed observed DNS answers**. |
| `subscdn.top` | Observed A records: `104.21.78.96`, `172.67.219.239` | **Confirmed observed DNS answers**. |
| `f2.freecdn4.top` | Resolved to `67.159.12.10` | **Confirmed observed DNS answer**; association with the app came from browser-network observations, not ownership proof. |

**Cloudflare trace / response headers:** `netmirror.gg` was observed with `MRS` (Marseille) edge; `net77.cc` with `BLR`, `BOM`, and `DEL` edges on different tests; `net52.cc` and `imgcdn.kim` with `BOM`; `subscdn.top` with `MRS`. `loc=IN` in `/cdn-cgi/trace` described the **client's** country, not the server country. These edge identifiers are **confirmed as observed**, but their physical interpretation is based on Cloudflare's published airport-code conventions.

### `net77.cc` HTTP endpoint responses

| Endpoint | Observed status | Observed content type / body | Observed Cache-Control |
|---|---|---|---|
| `/` | `301` | `text/html; charset=UTF-8`; 0 response-body bytes in one GET | `max-age=604800` |
| `/post.php` | `200` | `application/json`; `{"status":"n","error":"Invalid User"}` | `max-age=120` |
| `/play.php` | `200` | `application/json`; `null` | `no-cache, no-store, must-revalidate` |
| `/recentplay.php` | `200` | `application/json`; `null` | `max-age=604800` |

**Confirmed:** All four HEAD responses included `server: cloudflare` and `cf-cache-status: DYNAMIC`. The `/` response redirected to a `/login_...` path; its visitor-specific suffix is intentionally omitted. `200` does **not** imply authenticated or successful application functionality. `DYNAMIC` does **not** prove a request reached a particular origin.

### Timing measurements

Twelve sequential GET requests to `net77.cc` produced these time-to-first-byte (TTFB) measurements:

| Endpoint | TTFB samples (ms) | Average (ms) |
|---|---|---:|
| `/` | 331, 359, 258 | ~316 |
| `/post.php` | 328, 306, 240 | ~291 |
| `/play.php` | 276, 237, 253 | ~256 |
| `/recentplay.php` | 322, 383, 307 | ~337 |

A separate earlier test measured connection setup around **24–28 ms**, TLS completion around **50–57 ms**, and TTFB around **313–321 ms**. These are **observed performance measurements only**. They cannot identify origin geography or backend architecture.

### TLS certificate for `net77.cc`

Observed with `openssl s_client` and `openssl x509`:

- **Subject:** `CN=net77.cc`
- **Issuer:** `Google Trust Services`, `WE1`
- **Valid from:** `2026-11-02 18:07:57 GMT`
- **Valid until:** `2027-01-31 19:07:38 GMT`
- **Subject alternative names:** `net77.cc`, `*.net77.cc`

**Confirmed:** This is the certificate presented on the tested HTTPS connection. It is **not proof that Google hosts the website** and does not reveal the origin-server certificate.

### Domain registration results

| Domain | Directly observed registration data | Caveat |
|---|---|---|
| `net77.cc` | Creation `2026-07-07 18:27:24Z`; registrar **Internet Domain Service BS Corp**; registrant organization redacted; registrant country `IS`; expiry `2027-07-07` | `IS` is a registration-contact field, **not server location**. |
| `net52.cc` | Creation `2025-12-07 18:21:52Z`; registrar **NameCheap, Inc.**; privacy service shown; registrant country `US`; expiry `2026-12-07` | `US` is a registration-contact field, **not server location**. |
| `netmirror.gg` | WHOIS grep returned only `created: 1996-08-07` and an empty `Registrar:` line | **Not enough evidence** for the individual domain's creation date or registrar. The date likely concerns registry metadata. |

No MX or TXT answers were returned for the three root domains in the user's queries. That is a **negative observation for those queries**, not proof no email infrastructure exists.

### Media-related host `f2.freecdn4.top`

- DNS resolved to **`67.159.12.10`**.
- WHOIS returned **`FDCservers.net`**, network `FDCSERVERS`; Team Cymru returned **AS30058**, prefix `67.159.12.0/24`, country `US`.
- `curl -I https://f2.freecdn4.top` returned **`404 Not Found`** with **`Server: nginx`**.
- A traceroute to the IP ended with **`!X`** (administratively prohibited), not a verified geographic endpoint.

**Confirmed:** The observed IP is registered/announced with FDCservers / AS30058 in the queried records. **Not confirmed:** the server's physical location, its owner, or whether it hosts `net77.cc`'s application backend. WHOIS's `Destin, US` field is registration information, **not a geolocation measurement**.

### Certificate transparency query

The `crt.sh` JSON queries failed to parse. A follow-up `curl` for `net77.cc` timed out after 20 seconds with **HTTP code `000` and zero bytes**. This establishes only that the lookup **failed from the tested connection**; it does not establish that certificate transparency records are absent.

## 2. Possible interpretations — NOT 100% verified

- PHP-named endpoints **may** be part of a PHP application or PHP-compatible routing layer; filenames alone do not establish implementation language.
- `/post.php` **appears** to perform some form of user validation because it returned `Invalid User`; its authentication mechanism is unknown.
- `/play.php` and `/recentplay.php` **may** require parameters or session context; their bare GET responses were `null`.
- Similar response times **may** indicate shared processing paths, but do not establish shared origin servers.
- The media hostname **may** be involved in playback delivery, but this does not establish a direct hosting relationship with the application origin.

## 3. Unknown / not established

- The **real application origin IP address** for `netmirror.gg` or `net77.cc`.
- The **physical country, city, or data center** of the application backend.
- The **origin hosting provider** and whether the two application domains share an origin.
- The actual application stack, authentication design, ownership, or relationships between the domains.
- Whether historical DNS or certificate records disclose additional infrastructure.

## 4. Safe, reproducible commands (macOS zsh)

**Important zsh pitfall:** Do **not** use `path` as a loop variable. In zsh, `path` is tied to `PATH` and can overwrite the shell's command search directories. The earlier `for path in ...` command caused `PATH` to become `/recentplay.php`, which broke commands like `clear`. The session was restored with:

```zsh
export PATH="/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin"
```

To inspect response metadata without revealing redirect tokens:

```zsh
for endpoint in / /post.php /play.php /recentplay.php; do
  echo "===== $endpoint ====="
  curl -4 -sS -I --max-time 10 "https://net77.cc$endpoint" \
    | grep -Ei '^(HTTP/|server:|cf-ray:|cf-cache-status:|cache-control:|content-type:)'
done
```

To inspect public DNS nameservers:

```zsh
for domain in netmirror.gg net77.cc net52.cc; do
  echo "===== $domain ====="
  dig +short NS "$domain"
done
```

To inspect the currently presented TLS certificate:

```zsh
echo | openssl s_client -connect net77.cc:443 -servername net77.cc 2>/dev/null \
  | openssl x509 -noout -subject -issuer -dates -ext subjectAltName
```

These are read-only diagnostics. Avoid publishing complete network logs, cookies, authentication headers, account-specific redirect URLs, or session tokens.

## 5. Bottom line

**High-confidence, directly observed facts:** Cloudflare DNS and proxy responses; specific DNS results; HTTP statuses, JSON bodies, caching headers, measured timings; TLS certificate fields; the stated WHOIS/ASN registration results; and the certificate lookup timeout.

**Not verified:** Any claim that the **actual NetMirror / net77.cc backend is physically hosted in India, France, Iceland, the United States, or any other specific country**. Cloudflare edge codes, domain registration countries, and IP registration addresses are not proof of origin-server location.

---

*Compiled from Terminal outputs supplied in this conversation. No independent access to the user's Mac or private backend systems was used.*
