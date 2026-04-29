# prscan

A network scanner written in [Praia](https://praia.sh) that discovers hosts, NetBIOS names, MAC addresses, and reverse DNS hostnames on a local network. Requires Praia >= 0.4.5.

## Installation

Requires [Praia](https://praia.sh/getting-started/installation/) and the [arp](https://github.com/viggou/praia-arp) grain.

```sh
git clone https://github.com/praia-lang/prscan.git
cd prscan
sand install
chmod +x prscan
```

## Usage

```sh
./prscan <target> [options]
```

### Targets

```sh
./prscan 192.168.1.1              # single host
./prscan 192.168.1.1-50           # range (short form)
./prscan 192.168.1.1-192.168.1.50 # range (full form)
./prscan 192.168.1.0/24           # CIDR notation
```

### Options

| Flag | Description |
|------|-------------|
| `-t`, `--timeout <ms>` | Timeout per host in milliseconds (default: 1500) |
| `-v`, `--verbose` | Show all hosts including down ones |
| `-q`, `--quiet` | Only print IP and hostname (for scripting) |
| `-o`, `--output <file>` | Save results to file (`.json`, `.csv`, or `.txt`) |
| `--no-progress` | Hide the progress bar |

### Examples

Scan a subnet with a shorter timeout:

```sh
./prscan 192.168.1.0/24 -t 1000
```

Export results as JSON:

```sh
./prscan 10.0.0.0/24 -o results.json
```

Export as CSV for spreadsheets:

```sh
./prscan 192.168.1.0/24 -o hosts.csv
```

Quiet mode for piping to other tools:

```sh
./prscan 192.168.1.0/24 -q | grep "server"
```

## How it works

For each IP in the target range, prscan:

1. **NetBIOS NBSTAT query** (UDP 137) — retrieves the hostname, domain/workgroup, and MAC address from Windows/Samba hosts
2. **Reverse DNS** — falls back to `host` or `dig -x` for hosts without NetBIOS
3. **ARP lookup** — pings the host then reads the ARP table to get the MAC address

Ctrl+C gracefully stops the scan and prints a summary of results so far.

## Library usage

prscan can also be used as a Praia grain in your own scripts:

```sh
sand install github.com/praia-lang/prscan
```

```praia
use "prscan"

// Scan a range
let results = prscan.scan("192.168.1.0/24", {timeout: 1000})
for (r in results) {
    if (r.status == "up") {
        print(r.ip + " " + r.hostname + " " + r.mac)
    }
}

// Scan a single host
let host = prscan.scanHost("192.168.1.1", 1500)
if (host.status == "up") {
    print(host.hostname)
}
```

Each result is a map with: `ip`, `hostname`, `domain`, `mac`, `status` (`"up"` or `"down"`).

## Cross-platform

Works on macOS and Linux. Platform differences are handled automatically:

- **ARP**: uses `arp -a` on macOS, `/proc/net/arp` or `ip neigh` on Linux
- **Ping timeout**: `-W 1000` (ms) on macOS, `-W 1` (s) on Linux
- **Reverse DNS**: tries `host`, falls back to `dig`

## License

MIT
