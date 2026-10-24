# Google Dorks

Search operators for finding exposed pages and files. Combine them with `site:` to stay inside a program's scope, for example `site:target.com inurl:admin`.

| Dork | Finds |
| --- | --- |
| `inurl:admin` | Admin panel login pages |
| `intext:password` | Documents that contain the word "password" |
| `filetype:xls inurl:budget` | Excel files with "budget" in the URL |
| `site:.gov filetype:pdf` | PDF files on .gov websites |
| `site:.edu inurl:login` | Login pages on .edu websites |

## Useful operators

| Operator | Does |
| --- | --- |
| `site:` | Limit results to a domain |
| `inurl:` | Word must appear in the URL |
| `intext:` | Word must appear in the page text |
| `filetype:` / `ext:` | Limit to a file type |
| `-` | Exclude a term, for example `site:target.com -www` |

## Good for SSRF 
```
site:\*.target.com inurl:"url="  
site:\*.target.com inurl:"redirect="  
site:\*.target.com inurl:"?next=http"  
site:\*.target.com inurl:"img_url="  
site:\*.target.com inurl:"load.php?url="  
site:\*.target.com inurl:"api.php?url="  
site:\*.target.com inurl:"fetch?uri="  
site:\*.target.com inurl:"file=http"  
site:\*.target.com inurl:"src=http"  
site:\*.target.com inurl:"devUrl="  
site:\*.target.com inurl:"returnUrl="  
site:\*.target.com inurl:"/redirect?target="  
site:\*.target.com inurl:"/api/load?uri="  
site:\*.target.com inurl:"?open=http"  
site:\*.target.com inurl:"proxy?path="  
site:\*.target.com inurl:"?data=http"  
site:\*.target.com inurl:"external?link="  
site:\*.target.com inurl:"endpoint=http"  
site:\*.target.com inurl:"external?url="  
site:\*.target.com inurl:"src=http" filetype:xml  
site:\*.target.com inurl:"url=" filetype:pdf  
site:\*.target.com inurl:"callbackUrl=http"  
site:\*.target.com inurl:"?background=http"  
site:\*.target.com inurl:"?continue=http"  
site:\*.target.com inurl:"?referenceUrl=http"_
```
