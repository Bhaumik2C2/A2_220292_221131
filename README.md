# DNS Resolver

A Python-based DNS resolver that performs both **iterative** and **recursive** DNS lookups, resolving domain names to IP addresses by querying DNS servers directly.

## Features
- Supports **iterative** DNS resolution by querying root, TLD, and authoritative DNS servers.
- Supports **recursive** DNS resolution using the system's default resolver.
- Uses **root servers** as the starting point for iterative resolution.
- Extracts and resolves **NS (Nameserver) records** to their respective IP addresses.
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
4. Extracts NS hostnames and resolves them to IP addresses.
5. Moves through the resolution hierarchy from **ROOT** → **TLD** → **AUTH**.
6. If successful, prints the resolved IP address.

### Recursive DNS Lookup
1. Uses the system’s default resolver (or a configured public DNS like Google or Cloudflare).
2. Queries for **NS (Nameserver) records** and prints them.
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

## Changes Made
### `send_dns_query()`
- Added functionality to send a DNS query using UDP:
  ```python
  query = dns.message.make_query(domain, dns.rdatatype.A)  # Construct the DNS query
  response = dns.query.udp(query, server, timeout=TIMEOUT)  # Send the query using UDP
  return response
  ```

### `extract_next_nameservers()`
- Resolved NS hostnames to IP addresses using A records:
  ```python
  for ns_name in ns_names:
      try:
          answer = dns.resolver.resolve(ns_name, "A")  # Query for A records
          for rdata in answer:
              ns_ips.append(rdata.to_text())  
              print(f"Resolved {ns_name} to {rdata.to_text()}")  # Print resolution
      except Exception:
          print(f"[WARNING] Could not resolve NS {ns_name}")  # Handle failures
  ```

### `iterative_dns_lookup()`
- Updated stage transitions in the iterative lookup process:
  ```python
  if stage == "ROOT":
      stage = "TLD"
  elif stage == "TLD":
      stage = "AUTH"
  ```

### `recursive_dns_lookup()`
- Enhanced recursive lookup to first resolve NS records, then A records:
  ```python
  answer_ns = dns.resolver.resolve(domain, "NS")
  ns_list = []
  for rdata in answer_ns:
      ns_list.append(rdata.target.to_text())  # Append them in a list
  
  # Print authoritative NS servers in the order they are received
  for rdata in ns_list:
      print(f"[SUCCESS] {domain} -> {rdata}")

  # Query for A record (IPv4 Address)
  answer_a = dns.resolver.resolve(domain, "A")
  for rdata in answer_a:
      print(f"[SUCCESS] {domain} -> {rdata}")
  ```

