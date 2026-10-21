# Recon Cheat Sheet

Flags I use most for each tool. For chained commands, see [pipelines.md](pipelines.md).

- [subfinder](#subfinder) · [dnsx](#dnsx) · [httpx](#httpx) · [katana](#katana) · [naabu](#naabu) · [nuclei](#nuclei) · [ffuf](#ffuf) · [SecLists](#seclists)

## subfinder

Passive subdomain enumeration.

```bash
subfinder -d target.com                       # basic enumeration of one domain
subfinder -d target.com -o subs.txt           # save results to a file
subfinder -dL domains.txt                     # enumerate a list of domains
subfinder -d target.com -all                  # use all sources, including slow ones (use for real work)
subfinder -d target.com -recursive            # recursively enumerate discovered subdomains
subfinder -d target.com -silent               # subdomains only, no banner (for piping)
subfinder -d target.com -nW                   # exclude wildcard subdomains
subfinder -d target.com -t 50                 # concurrency (default 10)
subfinder -d target.com -timeout 30           # per-source timeout in seconds
subfinder -d target.com -oJ -o out.json       # JSON Lines output
subfinder -d target.com -cs                   # include the source that found each result
subfinder -ls                                 # list all available sources
subfinder -s crtsh,virustotal -d target.com   # use only specific sources
subfinder -es shodan -d target.com            # exclude specific sources
```

## dnsx

Fast DNS toolkit. Its main job is cutting a raw subdomain list down to hosts that actually resolve.

```bash
dnsx -l subs.txt                              # resolve a list, print those that resolve
subfinder -d target.com -silent | dnsx        # read from stdin (the normal use)
dnsx -l subs.txt -silent                      # no banner (for piping)
dnsx -l subs.txt -a -resp                     # show A records
dnsx -l subs.txt -cname -resp                 # show CNAMEs (key for subdomain takeover hunting)
dnsx -l subs.txt -mx -resp                    # MX records
dnsx -l subs.txt -txt -resp                   # TXT records (can leak internal info, verification tokens)
dnsx -l subs.txt -ns -resp                    # nameserver records
dnsx -l subs.txt -ptr                         # reverse DNS lookups
dnsx -l subs.txt -recon                       # query all record types at once
dnsx -l ips.txt -ptr -resp-only               # reverse-resolve IPs into hostnames
dnsx -l subs.txt -r resolvers.txt             # custom resolvers (strongly recommended)
dnsx -l subs.txt -t 200                       # concurrency (default 100)
dnsx -l subs.txt -rcode noerror,servfail      # filter by DNS response code
dnsx -l subs.txt -wd target.com -wt 25        # wildcard detection and filtering
dnsx -d target.com -w words.txt               # brute-force subdomains from a wordlist
```

## httpx

Probes hosts for live web servers.

```bash
# main command
httpx -l subs.txt -sc -title -td -ip

httpx -l subs.txt                             # probe a list
subfinder -d target.com -silent | httpx       # read from stdin
httpx -l subs.txt -silent                     # clean output
httpx -l subs.txt -sc                         # status code
httpx -l subs.txt -title                      # page title
httpx -l subs.txt -td                         # tech detection
httpx -l subs.txt -cl                         # content length
httpx -l subs.txt -server                     # server header
httpx -l subs.txt -ip                         # resolved IP
httpx -l subs.txt -cname                      # CNAME
httpx -l subs.txt -location                   # redirect location
httpx -l subs.txt -mc 200,403                 # match status codes
httpx -l subs.txt -fc 404,301                 # filter status codes
httpx -l subs.txt -ms "admin"                 # match string in response
httpx -l subs.txt -fs 1234                    # filter by response size
httpx -l subs.txt -p 80,443,8080,8443         # ports to probe
httpx -l subs.txt -path /admin                # probe a specific path
httpx -l subs.txt -sr                         # store responses
httpx -l subs.txt -json -o out.json           # JSON output
httpx -l subs.txt -follow-redirects           # follow redirects
httpx -l subs.txt -t 100 -rl 150              # threads and rate limit
httpx -l subs.txt -favicon                    # favicon hash
httpx -l subs.txt -jarm                       # JARM TLS fingerprint
httpx -l subs.txt -asn                        # ASN info
httpx -l subs.txt -screenshot                 # take screenshots
```

## katana

Web crawler. Finds real URLs, endpoints, JavaScript files, and parameters instead of guessed ones.

```bash
katana -u https://target.com                  # crawl a single site
katana -list urls.txt                         # crawl multiple sites
httpx -l subs.txt -silent | katana            # crawl from stdin
katana -u target.com -silent                  # clean output
katana -u target.com -d 5                     # crawl depth (default 3)
katana -u target.com -jc                      # parse JavaScript for endpoints
katana -u target.com -kf all                  # follow known files (robots.txt, sitemap.xml)
katana -u target.com -fx                      # extract and crawl form fields
katana -u target.com -aff                     # auto form-fill during crawl
katana -u target.com -js-crawl                # crawl endpoints found in JS
katana -u target.com -headless                # headless browser for JS-heavy apps
katana -u target.com -hl -no-sandbox          # headless on a server with no GUI
katana -u target.com -ef woff,css,png,svg,jpg # skip static-asset extensions
katana -u target.com -mr "api/"               # match URLs by regex
katana -u target.com -fr "logout"             # filter out URLs by regex
katana -u target.com -f qurl                  # only URLs that have query parameters
katana -u target.com -c 20 -p 10              # concurrency and parallelism
katana -u target.com -rl 100                  # rate limit
katana -u target.com -o urls.txt              # save output
katana -u target.com -jsonl -o out.json       # JSON Lines output
```

## naabu

Fast SYN port scanner. Finds open ports quickly, then hands them to nmap for service detection.

```bash
naabu -host target.com                        # top 100 ports
naabu -host target.com -p -                   # all 65,535 ports
naabu -list hosts.txt                         # scan a list of hosts
naabu -host target.com -top-ports 1000        # top N ports
naabu -host target.com -p 80,443,8080         # specific ports
naabu -host target.com -p 1-1000              # port range
naabu -host target.com -silent                # clean output for piping
naabu -host target.com -sn                    # host discovery only (ping sweep)
naabu -host target.com -s s                   # SYN scan (fast, default, needs root)
naabu -host target.com -s c                   # CONNECT scan (no root, slower)
naabu -host target.com -rate 2000             # packets per second
naabu -host target.com -c 50                  # concurrency
naabu -host target.com -nmap                  # run nmap on found ports
naabu -host target.com -nmap-cli 'nmap -sV -sC'   # custom nmap command on results
naabu -host target.com -json -o out.json      # JSON output
naabu -host target.com -exclude-ports 80,443  # skip ports
naabu -host target.com -ep 21,22 -p -         # all ports except some
naabu -host target.com -verify                # re-verify open ports (fewer false positives)
```

## nuclei

Template-based vulnerability scanner for known CVEs, misconfigurations, exposures, default credentials, and takeovers.

```bash
nuclei -u https://target.com                  # scan one URL with all templates
nuclei -l live.txt                            # scan a list of hosts
httpx -l subs.txt -silent | nuclei            # scan from stdin (the normal use)
nuclei -u target.com -silent                  # only print findings
nuclei -l hosts.txt -t http/cves/             # run a template directory
nuclei -l hosts.txt -t http/takeovers/        # subdomain takeover checks
nuclei -l hosts.txt -tags cve,rce             # run templates with these tags
nuclei -l hosts.txt -severity critical,high   # filter by severity (cuts noise)
nuclei -l hosts.txt -etags dos,fuzz           # exclude tags
nuclei -l hosts.txt -id CVE-2021-44228        # run one template by ID
nuclei -l hosts.txt -rl 150                   # rate limit (requests/sec)
nuclei -l hosts.txt -c 50                     # templates in parallel
nuclei -l hosts.txt -o results.txt            # save findings
nuclei -l hosts.txt -json -o out.json         # JSON output
nuclei -l hosts.txt -me report/               # export a markdown report
nuclei -l hosts.txt -as                       # automatic scan: fingerprint tech, run relevant templates
nuclei -l hosts.txt -nc                       # no color (cleaner logs)
nuclei -l hosts.txt -stats                    # live progress stats
nuclei -l hosts.txt -H "X-Bug-Bounty: cfish8" # identify yourself (many programs require it)
nuclei -t my-templates/ -u target.com         # run your own templates
nuclei -l hosts.txt -itags fuzz               # include normally excluded tags
```

### Scans I run often

```bash
nuclei -u https://example.com -tags exposure,misconfig -severity info,low,medium,high,critical -rl 2
nuclei -u https://example.com -tags panel,kev,sqli,exposure
```

## ffuf

Web fuzzer for directories, files, parameters, subdomains, and vhosts.

The main skill is filtering false positives. Sites that return 200 for everything, or a custom 404 page, flood the results.

```bash
# auto-calibration learns the "not found" response and filters it
ffuf -u https://target.com/FUZZ -w list.txt -ac

# filter a known junk response size once you spot it
ffuf -u https://target.com/FUZZ -w list.txt -fs 1234

# common directories
ffuf -u https://target.com/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -ac

# common files
ffuf -u https://target.com/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-files.txt -ac

# API endpoints
ffuf -u https://target.com/api/FUZZ -w /usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt -ac

# two wordlists at once
ffuf -u https://target.com/W1/W2 -w dirs.txt:W1 -w files.txt:W2 -mode clusterbomb
```

## SecLists

The standard wordlist collection used by ffuf, dnsx, and most guides.

```bash
git clone --depth 1 https://github.com/danielmiessler/SecLists.git /usr/share/seclists
# or, older version: sudo apt install seclists
```

| Folder in `/usr/share/seclists` | Contents |
| --- | --- |
| `Discovery/Web-Content/` | Directory and file brute-force lists |
| `Discovery/Web-Content/api/` | API endpoint lists |
| `Discovery/DNS/` | Subdomain wordlists |
| `Usernames/` | Username lists |
| `Passwords/` | Password lists (default credentials, leaks) |
| `Fuzzing/` | Injection fuzzing payloads |
| `Pattern-Matching/` | Regex and grep patterns |

Lists I use most:

```
Discovery/Web-Content/raft-medium-directories.txt
Discovery/Web-Content/raft-medium-files.txt
Discovery/Web-Content/directory-list-2.3-medium.txt
Discovery/Web-Content/api/api-endpoints.txt
Discovery/DNS/subdomains-top1million-5000.txt
Discovery/DNS/subdomains-top1million-110000.txt
```
