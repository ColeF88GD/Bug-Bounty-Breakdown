# Bug Bounty Toolkit

My working notes for bug bounty recon: the tools I run, the flags I use most, and the pipelines that chain them together. Everything here runs from a dedicated DigitalOcean Droplet that I keep separate from my home network.

**Author:** Cole Fisher · [Bugcrowd](https://bugcrowd.com/h/CFish8) · [HackerOne](https://hackerone.com/cfish8) · [LinkedIn](https://www.linkedin.com/in/cole-fisher-6980a7230/)

## Workflow

```
subfinder ──► dnsx ──► httpx ──► katana ──► nuclei ──► manual testing
(find subs)  (resolve) (probe)   (crawl)    (scan)     (Burp Suite / Caido)
                          │
                          └──► naabu ──► nmap  (open ports → service detection)
```

1. **Discover** subdomains passively with subfinder, crt.sh, and other sources.
2. **Resolve** them with dnsx so only hosts that exist move on.
3. **Probe** live web servers with httpx for status codes, titles, and tech stack.
4. **Crawl** live hosts with katana to collect real URLs, parameters, and JavaScript files.
5. **Scan** with nuclei templates for known CVEs, misconfigurations, and exposures.
6. **Test manually** in Burp Suite and Caido, which is where most real findings come from.

## Contents

| File | What's in it |
| --- | --- |
| [recon-cheatsheet.md](recon-cheatsheet.md) | Flags and examples for subfinder, httpx, katana, naabu, dnsx, nuclei, ffuf, and SecLists |
| [pipelines.md](pipelines.md) | One-liners that chain the tools together, plus passive subdomain sources |
| [droplet-setup.md](droplet-setup.md) | How the Droplet is set up: key-only SSH, VS Code Remote-SSH, tmux, and a temporary file server |
| [google-dorks.md](google-dorks.md) | Search operators for finding exposed pages and files on in-scope targets |

## Rules I follow

- Test only assets that are **in scope** for a program, and follow each program's rules on rate limits and automation.
- Identify my traffic with the header a program asks for, for example `X-HackerOne-Research: cfish8`.
- Keep rate limits low (`-rl`) on shared or fragile targets.
- Never publish details of a finding before the program allows it.

`target.com` and `example.com` in these notes are placeholders.
