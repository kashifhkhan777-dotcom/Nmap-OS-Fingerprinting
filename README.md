# Nmap OS Fingerprinting Lab

## Overview
Used Nmap's OS detection (-O) to fingerprint a target host. OS fingerprinting works by sending a series of specially crafted TCP/UDP/ICMP probes and analyzing how the target responds. Nmap compares the response patterns (window sizes, TCP options, TTL values, IP ID behavior) against its internal database of known OS signatures.

## Lab Environment
- Scanner: Kali Linux (10.0.2.15)
- Target: 10.0.2.2 (VirtualBox NAT gateway)
- Tool: Nmap 7.99
- Network distance: 1 hop

## Command Used

sudo nmap -O 10.0.2.2 -oN os_scan.txt

## Results

## Analysis

### Correct Identification
The scan correctly identified the target as an **Oracle VirtualBox Slirp NAT bridge** - this is exactly what 10.0.2.2 is in a VirtualBox NAT network. The Slirp stack uses QEMU-derived network code, which is why QEMU also appeared as a high-confidence guess.

### Port Discovery
Two Windows-specific ports are exposed:
- **135/tcp (MSRPC)** - Microsoft Remote Procedure Call
- **445/tcp (Microsoft-DS/SMB)** - Windows file sharing

These ports are being proxied through the VirtualBox NAT gateway from the Windows host machine, which is a known behavior of VirtualBox NAT networking.

### Why Nmap Guessed "AT&T BGW210 Voice Gateway" at 99%
This is a false positive from a signature collision. The AT&T BGW210 home router likely runs the same QEMU/Slirp network code underneath, so its TCP/IP fingerprint matches VirtualBox's NAT gateway. This is a great example of how OS fingerprinting can return confident-but-wrong answers.

### Why OSScan Warning Appeared
Nmap warned "we could not find at least 1 open and 1 closed port." OS detection is most reliable when Nmap can contrast an open port's behavior with a closed port's behavior. Since the gateway filtered most ports (no-response rather than closed/reset), Nmap had less data to work with.

## Key Takeaways
1. OS fingerprinting is best-effort, not definitive - even at 99% confidence, results can be wrong.
2. The technique relies on matching TCP/IP stack implementation quirks against a known database.
3. Network distance and firewall behavior strongly affect accuracy.
4. A "filtered" state (no-response) gives Nmap less data than a "closed" state (RST).
5. Real-world engagements should combine `-O` with `-sV` (service version detection) and banner grabbing for higher confidence.
6. MAC address vendors can corroborate OS guesses - but VirtualBox uses locally administered MACs, so this signal was unavailable here.

## Evidence
- os_scan.txt - Raw Nmap -O output

## Tools Used
- Nmap 7.99 (-O flag)
