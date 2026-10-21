# Pipelines

One-liners that chain the tools in [recon-cheatsheet.md](recon-cheatsheet.md) together.

## Discover and probe

```bash
# the standard recon pipeline: find, resolve, probe
subfinder -d target.com -all -silent | dnsx -silent | httpx -silent -sc -title -td -o live.txt

# several domains, de-duplicated into a running master list
subfinder -dL scope.txt -all -silent | anew all_subs.txt

# non-standard web ports
httpx -l subs.txt -p 80,443,3000,8000,8080,8443,8888,9000 -silent
```

## Ports

```bash
# resolve subdomains to IPs, then port scan
subfinder -d target.com -silent | dnsx -silent -a -resp-only | naabu -silent
dnsx -l subs.txt -a -resp-only -silent | sort -u | naabu -silent

# find web ports, then probe them with httpx
naabu -host target.com -p 80,443,8080,8443,3000,8000 -silent | httpx -silent -title -td
```

## Crawl

```bash
# crawl every live host
httpx -l subs.txt -silent | katana -silent -jc -o all_urls.txt

# parameterized URLs only, into an XSS candidate list
katana -u target.com -f qurl -silent | gf xss | tee xss_candidates.txt

# collect JavaScript files for secret and endpoint hunting
katana -u target.com -silent | grep '\.js$' | httpx -silent -sr -srd js_files/
```

## Scan

```bash
# live hosts into nuclei
httpx -l subs.txt -silent | nuclei -t http/ -severity medium,high,critical

# the full one-liner
subfinder -d target.com -all -silent | dnsx -silent | httpx -silent | \
  nuclei -severity high,critical -H "X-Bug-Bounty: cfish8" -o hits.txt

# subdomain takeover sweep
nuclei -l subs.txt -t http/takeovers/ -silent
dnsx -l subs.txt -cname -resp -silent | grep -iE "s3|azure|herokudns|github.io|cloudfront"

# continuous monitoring: alert on new findings only
nuclei -l live.txt -silent | anew seen_findings.txt | notify
```

## Secrets in responses and JavaScript

```bash
# save every response body, then search them offline
httpx -l subs.txt -sr -srd responses/
grep -ri "api_key\|secret\|password" responses/
```

```bash
H='X-HackerOne-Research: cfish8'

# A. fastest: nuclei exposure and secret templates across all live hosts
nuclei -l livesubs.txt -tags exposure,tokens,secret,config,disclosure -o exposures.txt

# B. crawl all JavaScript, then search each file for high-signal secrets
cat livesubs.txt | grep -oE 'https?://[^ ]+' | httpx -silent -mc 200 \
  | katana -silent -jc -d 2 -ef woff,woff2,png,jpg,svg,css,ico > urls.txt
grep -iaE '\.js($|\?)' urls.txt | sort -u > jsurls.txt

while read u; do
  curl -sk --max-time 20 -H "$H" "$u" \
  | grep -aoiE '(AKIA[0-9A-Z]{16}|aws_?secret[a-z_]*["'\'' :=]+[A-Za-z0-9/+]{20,}|client_?secret["'\'' :=]+[A-Za-z0-9_\-]{12,}|x?-?api[_-]?key["'\'' :=]+[A-Za-z0-9_\-]{16,}|-----BEGIN [A-Z ]*PRIVATE KEY-----|xox[baprs]-[A-Za-z0-9-]{10,}|ghp_[A-Za-z0-9]{36})' \
  | sed "s#^#$u => #"
done < jsurls.txt | sort -u | tee js-secrets.txt
```

## Passive subdomain sources

```bash
# subfinder: fast, API-powered
subfinder -d target.com -all -recursive -o subs_subfinder.txt

# assetfinder: related domains and subdomains
echo target.com | assetfinder -subs-only > subs_assetfinder.txt

# amass: deep OSINT engine
amass enum -d target.com -o subs_amass.txt
amass enum -d target.com -brute -w $DNS_WORDLIST -o subs_amass_brute.txt

# findomain: multi-source
findomain -t target.com -u subs_findomain.txt

# chaos: ProjectDiscovery's dataset (needs an API key; keep it in an environment variable)
export PDCP_API_KEY=YOUR_KEY
chaos -d target.com -o subs_chaos.txt

# github-subdomains: mines public code
github-subdomains -d target.com -t $GITHUB_TOKEN -o subs_github.txt
```

### Certificate transparency (crt.sh)

```bash
curl -s "https://crt.sh/?q=%25.target.com&output=json" \
  | jq -r '.[].name_value' \
  | sed 's/\*\.//g' \
  | tr ',' '\n' \
  | grep -oE "[A-Za-z0-9._-]+\.target\.com" \
  | sort -u > subs_crt.txt
```

### Run everything at once (subenum)

```bash
./subenum.sh -d target.com \
  -u wayback,crt,abuseipdb,Findomain,Subfinder,Amass,Assetfinder \
  -o subs_subenum.txt

# for a list of targets
./subenum.sh -l targets.txt \
  -u wayback,crt,abuseipdb,Findomain,Subfinder,Amass,Assetfinder \
  -o subs_subenum.txt
```
