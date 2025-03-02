# DNS Resolver

A Python-based DNS resolver that performs both **iterative** and **recursive** DNS lookups, resolving domain names to IP addresses by querying DNS servers directly.

## Features
- Supports **iterative** DNS resolution by querying root, TLD, and authoritative DNS servers.
- Supports **recursive** DNS resolution using the system's default resolver.
- Uses **root servers** as the starting point for iterative resolution.
- Extracts and resolves **NS (Nameserver) records**.
- Implements **timeout handling** to avoid long waits on unresponsive servers.

## Requirements
- Python 3.x
- `dnspython` package

## Installation
It is recommended to use a virtual environment to manage dependencies.

```bash
# Create a virtual environment
python3 -m venv dns_env

# Activate the virtual environment (Linux/macOS)
source dns_env/bin/activate

# Activate the virtual environment (Windows)
dns_env\Scripts\activate

# Install dependencies
pip install dnspython
```

## Usage
Run the script using the following format:
```bash
python3 dns_resolver.py <iterative|recursive> <domain>
```
where:
- `<iterative|recursive>` specifies the resolution method.
- `<domain>` is the domain name to resolve.

### Example Commands
```bash
python3 dns_resolver.py iterative example.com
python3 dns_resolver.py recursive example.com
```

## How It Works
### Iterative DNS Lookup
1. Queries a **root DNS server**.
2. Retrieves **TLD nameservers** and queries them.
3. Retrieves **authoritative nameservers** and queries them until an IP is found.
4. If successful, prints the resolved IP address.

### Recursive DNS Lookup
1. Uses the system’s default resolver (or a configured public DNS like Google or Cloudflare).
2. Queries for **NS (Nameserver) records** first.
3. Queries for **A (IPv4 Address) records** and prints the final result.

## Example Output
### Iterative Lookup
```bash
[Iterative DNS Lookup] Resolving example.com
[DEBUG] Querying ROOT server (198.41.0.4) - SUCCESS
[DEBUG] Querying TLD server (192.33.4.12) - SUCCESS
[DEBUG] Querying AUTH server (93.184.216.34) - SUCCESS
[SUCCESS] example.com -> 93.184.216.34
Time taken: 1.234 seconds
```

### Recursive Lookup
```bash
[Recursive DNS Lookup] Resolving example.com
[SUCCESS] example.com -> ns1.example.com
[SUCCESS] example.com -> 93.184.216.34
Time taken: 0.567 seconds
```

## Error Handling
- If a server does not respond within the timeout, it moves to the next available nameserver.
- If resolution fails at any stage, an error message is displayed.



