# Incident Report -- Internal Lateral Movement and HTTPS Data Exfiltration

## Scenario Overview

The analyzed PCAP represents a suspected internal compromise followed by
lateral movement and encrypted data exfiltration over HTTPS.

Unlike phishing-based intrusion, this case suggests that the attacker
already had initial access to one internal workstation and leveraged SMB
and Windows authentication mechanisms to move laterally before staging
and exfiltrating sensitive data.

All IP addresses and hostnames have been sanitized for public release.

------------------------------------------------------------------------

# 1. Initial Indicators of Compromise

During initial triage in Wireshark, abnormal internal traffic was
observed between workstations within the same subnet.

### Protocol Hierarchy (Statistics → Protocol Hierarchy)

High volume of: - SMB (TCP/445) - Kerberos (TCP/88) - LDAP (TCP/389) -
HTTPS (TCP/443)

Unusual finding: One internal workstation initiated SMB connections to
multiple internal hosts in short succession.

------------------------------------------------------------------------

# 2. Lateral Movement Analysis

## SMB Enumeration Activity

Filter used:

tcp.port == 445

Observed behavior:

-   Repeated SMB session setups to multiple internal hosts
-   NTLM authentication attempts
-   Access to administrative shares (C$, ADMIN$)

Indicators:

-   High-frequency authentication attempts
-   Sequential targeting of internal IP addresses
-   Access attempts to file shares without normal user browsing pattern

This behavior is consistent with credential reuse or automated lateral
movement tools.

------------------------------------------------------------------------

# 3. Authentication Analysis

Filter used:

kerberos \|\| ntlmssp

Findings:

-   Multiple Kerberos TGS requests
-   Service ticket requests for CIFS services
-   NTLM authentication negotiation

Notable pattern:

Service ticket requests were generated shortly before SMB sessions to
new hosts, suggesting automated privilege escalation or lateral spread
using valid credentials.

------------------------------------------------------------------------

# 4. Data Staging Behavior

Before external communication began, one internal host initiated:

-   Large SMB file transfers from a file server
-   Read operations on multiple document shares
-   No corresponding user browsing traffic

Filter used:

smb2

Observation:

Large read requests followed by data responses, consistent with file
aggregation.

This suggests internal staging of sensitive data prior to exfiltration.

------------------------------------------------------------------------

# 5. Encrypted Data Exfiltration

Following staging activity, the compromised host initiated HTTPS
communication to a previously unseen external server.

Filter used:

tcp.port == 443

Observed:

-   Long-lived TLS sessions
-   Significant outbound data volume
-   No corresponding inbound payload size
-   Self-signed or uncommon certificate metadata

JA3 fingerprinting indicated a non-standard TLS client fingerprint
inconsistent with typical browser traffic.

Traffic characteristics:

-   Large outbound encrypted payloads
-   Minimal response data
-   Periodic beacon-like intervals

This strongly suggests HTTPS-based encrypted data exfiltration.

------------------------------------------------------------------------

# 6. Communication Flow (Sanitized)

INTERNAL_WORKSTATION → MULTIPLE_INTERNAL_HOSTS (SMB / Kerberos)

INTERNAL_WORKSTATION → FILE_SERVER (SMB large reads)

INTERNAL_WORKSTATION → EXTERNAL_SERVER (HTTPS large outbound transfer)

------------------------------------------------------------------------

# 7. Likely Attack Chain

1.  Initial compromise of internal workstation
2.  Credential harvesting or reuse
3.  Lateral movement via SMB and Kerberos
4.  File staging from internal file server
5.  Encrypted data exfiltration via HTTPS

------------------------------------------------------------------------

# 8. Attacker Sophistication Assessment

Moderate to High.

Indicators supporting this assessment:

-   Controlled lateral movement pattern
-   Use of legitimate authentication mechanisms
-   Avoidance of cleartext protocols for exfiltration
-   Staging before outbound transfer
-   Encrypted C2/exfiltration channel

The attacker leveraged native Windows protocols, making detection more
difficult in environments without deep traffic inspection.

------------------------------------------------------------------------

# 9. Key Technical Takeaways

-   Kerberos service ticket spikes may indicate lateral movement
-   Internal file staging before outbound HTTPS is a common exfiltration
    pattern
-   Long-lived TLS sessions with high outbound ratio are suspicious
-   JA3 fingerprinting can assist in identifying non-browser TLS clients

------------------------------------------------------------------------

End of Report
