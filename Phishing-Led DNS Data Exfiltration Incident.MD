🕵️ Network Forensics Report

**Scenario: Phishing Campaign with DNS Data Exfiltration**

> All real infrastructure details (IP addresses, domains, hashes) have
> been intentionally redacted for public publication.

------------------------------------------------------------------------

# 1. Executive Summary

The analyzed PCAP file reveals a structured multi-stage attack
involving:

-   A typosquatted phishing domain\
-   Delivery of suspicious artifacts\
-   DNS-based data exfiltration\
-   Separated attacker infrastructure

The victim host interacted with a fake banking portal, downloaded
external artifacts, and subsequently transmitted encoded data via
structured DNS queries.

The attack demonstrates deliberate staging and moderate attacker
sophistication.

------------------------------------------------------------------------

# 2. Tools Used

## Network Analysis

-   Wireshark\
-   Wireshark Statistics tools\
-   HTTP Object Export

## Data Reconstruction & Analysis

-   CyberChef\
-   Linux `file` command\
-   Base64 decoding\
-   Text editor / Excel for data cleanup\
-   Audio playback tools\
-   Spectrogram analysis software\
-   Steghide

------------------------------------------------------------------------

# 3. Initial Network Triage

## Protocol Identification

Using: Statistics → Protocol Hierarchy

Observed protocols: - HTTP (TCP/80)\
- HTTPS/TLS (TCP/443)\
- DNS (UDP/53)

------------------------------------------------------------------------

## Suspicious Domain Identification

Filter used: dns

Then narrowed down with: dns.qry.name contains "phishing-domain"

Observed: - High volume of unusually long DNS queries\
- Repetitive structure\
- Sequential identifiers in subdomains

------------------------------------------------------------------------

# 4. Indicators of Compromise (Sanitized)

## Malicious Infrastructure

-   example-phishing-domain.com -- phishing banking portal\
-   ATTACKER_WEB_SERVER -- staging server hosting artifacts\
-   ATTACKER_DNS_SERVER -- external DNS resolver used for exfiltration

All encoded DNS queries were directed to a single external DNS server,
suggesting possible DNS configuration manipulation on the victim host.

------------------------------------------------------------------------

## Files Observed

HTTP filter: http

Suspicious URIs: http.request.uri contains "audio"

Observed files: - /audio.wav (HTTP 200 OK)\
- /audio_steg.exe (HTTP 304 Not Modified -- cached locally)\
- /accounts.html\
- /cards.html\
- /loans.html\
- /contact.html

------------------------------------------------------------------------

# 5. Timeline of Events

  Time (UTC)   Event
  ------------ --------------------------------------------------------------
  T1           Victim accessed phishing portal
  T2           Interaction with multiple fake banking pages
  T3           Request for executable artifact (HTTP 304 -- cached locally)
  T4           Download of WAV file (HTTP 200 OK)
  T5           Series of structured DNS queries
  T6           Repeated communication with single external DNS server

------------------------------------------------------------------------

# 6. HTTP Artifact Recovery

Wireshark →\
File → Export Objects → HTTP

Recovered: - audio.wav

Not recovered: - audio_steg.exe (HTTP 304 response -- not transmitted in
capture)

------------------------------------------------------------------------

# 7. DNS Data Exfiltration Analysis

## Detection of Anomalous Pattern

Filter: dns.qry.name contains "phishing-domain"

Observed structure:

`<chunk_id>`{=html}.`<encoded_data>`{=html}.example-phishing-domain.com\
END.`<id>`{=html}.example-phishing-domain.com

Characteristics: - Sequential chunk identifiers\
- Identifiers in hexadecimal format\
- Constant-length encoded segments\
- High frequency of queries\
- Explicit termination marker (END.`<id>`{=html})

------------------------------------------------------------------------

## Data Reconstruction Process

### Step 1 -- Extract DNS Query Names

Copied DNS query names from packet details.

### Step 2 -- Extract Encoded Segments

Removed: - Chunk identifiers\
- Root domain\
- Transmission markers

Kept only encoded labels.

### Step 3 -- Reassemble in Correct Order

Important observation: Chunk identifiers were in hexadecimal format.

Fragments were reordered according to hex sequence.

### Step 4 -- Normalize and Decode

-   Concatenated fragments\
-   Removed formatting artifacts\
-   Performed Base64 decoding (CyberChef / Linux base64 -d)

Resulting file header: RIFF....WAVE

Verified using: file output.bin

Confirmed as: RIFF (little-endian) data, WAVE audio

------------------------------------------------------------------------

# 8. Recovered Artifact

Recovered file details:

-   Type: PCM WAV\
-   Size: \~72 KB\
-   Duration: \~0.75 seconds\
-   Content: short voice message ("hello")

SHA256: \[REDACTED_HASH_FOR_PUBLIC_VERSION\]

------------------------------------------------------------------------

# 9. Steganography Analysis Attempts

## Spectrogram Analysis

Observed anomaly around \~7 kHz frequency band.

## Steghide Attempts

Command: steghide info audio.wav

Attempted extraction with: steghide extract -sf audio.wav

Tried: - Common passwords\
- Wordlists\
- Strings extracted from PCAP

No successful extraction.

## Morse Code Hypothesis

After cleaning decoded data, fragment resembling: NOT THERE?

Likely intentional rabbit hole.

------------------------------------------------------------------------

# 10. Communication Flow (Sanitized)

INTERNAL_HOST → PHISHING_WEB_SERVER\
INTERNAL_HOST → STAGING_SERVER\
INTERNAL_HOST → ATTACKER_DNS_SERVER

All exfiltration traffic was directed to a single external DNS resolver.

------------------------------------------------------------------------

# 11. Attack Hypothesis

Probable attack chain:

1.  Victim visits typosquatted phishing portal\
2.  User interacts with fake banking pages\
3.  Auxiliary artifact delivered from staging infrastructure\
4.  Local data prepared\
5.  Data exfiltrated via DNS tunneling\
6.  Transmission terminated with explicit END marker

------------------------------------------------------------------------

# 12. Attacker Sophistication Assessment

Moderate.

Techniques combined:

-   Typosquatting\
-   Phishing portal\
-   Infrastructure separation\
-   DNS tunneling\
-   Chunked hexadecimal indexing\
-   Transmission termination marker

Individually common techniques, but structured into a coordinated
multi-stage attack.

------------------------------------------------------------------------

# 13. Key Technical Takeaways

-   DNS can be abused as a covert exfiltration channel\
-   Long structured subdomains are strong indicators\
-   Hex-based indexing is common in chunked transmission\
-   Single external DNS resolver usage is highly suspicious\
-   HTTP 304 responses may indicate pre-existing malicious binaries

------------------------------------------------------------------------

**End of Report**
