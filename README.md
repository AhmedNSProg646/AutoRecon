# AutoRecon

A reconnaissance automation tool for bug bounty hunting and authorized web application testing. AutoRecon chains together popular open-source recon tools into a single pipeline — subdomain enumeration, host probing, URL/endpoint discovery, JavaScript file collection, content discovery, and parameter discovery — and optionally scaffolds an OWASP WSTG folder structure so you can keep your testing notes organized from the start.

## Features

- Subdomain enumeration via subfinder, sublist3r, amass, and an optional ffuf brute force
- Automatic deduplication and merging of subdomains from every source into one list
- Live host probing with httpx
- URL/endpoint discovery via katana and waybackurls
- Automatic download of every discovered `.js` file into its own folder for offline review
- JavaScript endpoint/secret extraction with LinkFinder
- Optional directory/content brute forcing with feroxbuster
- Optional parameter discovery with Arjun
- Optional OWASP WSTG folder scaffolding (`notes.txt` / `tests.txt` per testing category) to organize your findings
- Missing tools are skipped with a warning instead of crashing the whole run

## Requirements

Python 3.8+ and the `requests` library:

```bash
pip install requests
```

AutoRecon shells out to the following external tools. You only need to install the ones you intend to use — anything missing from your `PATH` is skipped automatically.

| Tool | Purpose | Install |
|---|---|---|
| subfinder | Subdomain enumeration | `go install -v github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest` |
| sublist3r | Subdomain enumeration | `pip install sublist3r` |
| amass | Subdomain enumeration | `go install -v github.com/owasp-amass/amass/v4/...@master` |
| ffuf | Subdomain brute forcing | `go install -v github.com/ffuf/ffuf/v2@latest` |
| httpx | Live host probing | `go install -v github.com/projectdiscovery/httpx/cmd/httpx@latest` |
| katana | URL/endpoint crawling | `go install -v github.com/projectdiscovery/katana/cmd/katana@latest` |
| waybackurls | Historical URL discovery | `go install -v github.com/tomnomnom/waybackurls@latest` |
| LinkFinder | JS endpoint extraction | clone from [GerbenJavado/LinkFinder](https://github.com/GerbenJavado/LinkFinder) |
| feroxbuster | Content/directory discovery | see [epi052/feroxbuster](https://github.com/epi052/feroxbuster) |
| Arjun | Parameter discovery | `pip install arjun` |

## Installation

```bash
git clone https://github.com/<your-username>/autorecon.git
cd autorecon
pip install requests
```

Make sure the tools you want to use (see table above) are installed and available on your `PATH`.

## Usage

```bash
python3 autorecon.py -u <target> -js <yes/no> -wstg <yes/no> [options]
```

### Arguments

| Flag | Long form | Required | Description |
|---|---|---|---|
| `-u` | `--url` | Yes | Target URL, e.g. `https://example.com` |
| `-js` | `--download-js` | Yes | `yes`/`no` — download all discovered `.js` files into a separate `js_files` folder |
| `-wstg` | `--owasp-wstg-folders` | Yes | `yes`/`no` — create OWASP WSTG category folders with `notes.txt` and `tests.txt` for note-taking |
| `-ws` | `--word-list-subs` | No | Wordlist path for ffuf subdomain brute forcing |
| `-wu` | `--word-list-urls` | No | Wordlist path for feroxbuster content discovery |
| `-pw` | `--parameters-word-list` | No | Wordlist path for Arjun parameter discovery |

### Example

```bash
python3 autorecon.py \
  -u https://example.com \
  -js yes \
  -wstg yes \
  -ws wordlists/subdomains.txt \
  -wu wordlists/content.txt \
  -pw wordlists/params.txt
```

This runs the full pipeline, downloads every discovered JS file, brute forces subdomains and content with the supplied wordlists, runs parameter discovery, and scaffolds a WSTG note-taking structure.

To run a lighter pass without brute forcing or JS downloads:

```bash
python3 autorecon.py -u https://example.com -js no -wstg no
```

## Output Structure

All results are written under `results/<domain>/`:

```
results/example.com/
├── subfinder.txt
├── sublist3r.txt
├── amass.txt
├── ffuf_subs.csv
├── ffuf_subs.txt
├── all_subdomains.txt
├── alive_hosts.txt
├── katana_urls.txt
├── wayback_urls.txt
├── all_urls.txt
├── js_files/
│   └── *.js
├── linkfinder_results.txt
├── feroxbuster_results.txt
├── arjun_results.txt
└── WSTG/
    ├── WSTG-INFO/
    │   ├── notes.txt
    │   └── tests.txt
    ├── WSTG-ATHN/
    ├── WSTG-ATHZ/
    └── ... (one folder per WSTG category)
```

## OWASP WSTG Folders

When `-wstg yes` is set, AutoRecon creates a folder for each [OWASP Web Security Testing Guide](https://owasp.org/www-project-web-security-testing-guide/) category (e.g. `WSTG-INPV` for Input Validation Testing, `WSTG-ATHZ` for Authorization Testing). Each folder gets a `notes.txt` and `tests.txt` so you can log your observations and test cases as you work through the application, keeping your manual testing organized alongside the automated recon output.

## Disclaimer

This tool is intended for authorized security testing only — your own assets, or targets where you have explicit permission or are operating within the scope of a bug bounty program. Running these tools against systems you don't have authorization to test may be illegal. You are responsible for ensuring you have proper authorization before using this tool against any target.

## License

[MIT](LICENSE)

## Author

Created by Ahmed Nasser (shadow64)
