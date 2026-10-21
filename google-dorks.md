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
