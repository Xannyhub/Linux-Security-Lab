# Incident Report: VirtualBox Network Troubleshooting

## 1. Incident Summary

**Incident type:** Network connectivity and scan-result inconsistency  
**Environment:** Isolated VirtualBox cybersecurity lab  
**Attacker/test machine:** Kali Linux — `192.168.56.102`  
**Target machine:** Ubuntu Server — `192.168.56.101`  
**Tools:** Nmap, tcpdump, VirtualBox, Linux networking utilities  
**Status:** Resolved during testing

## 2. Initial Symptoms

During authorized network reconnaissance against my Ubuntu lab machine, Nmap initially reported ports 81 and 8080 as filtered, and some scans took longer than expected.

This was inconsistent with my expected results because the target was not configured to intentionally drop traffic to those ports.

## 3. Investigation

I used a systematic troubleshooting approach:

1. Checked the Ubuntu firewall status using UFW and inspected the iptables policies.
2. Captured packets on the Ubuntu target using:

   sudo tcpdump -ni any port 81

3. Observed incoming TCP SYN packets and TCP RST replies, consistent with closed-port behaviour.
4. Checked the Kali network interfaces and confirmed that the Host-only interface was active.
5. Verified the target's Host-only IP address and repeated the scan over the intended private lab network.

## 4. Root Cause Assessment

The scans were not consistently using the intended Host-only network path. This produced misleading scan results during the initial troubleshooting.

The network-path issue was the working diagnosis based on the investigation. The direct Host-only scan subsequently returned the expected closed-port states and completed much faster.

## 5. Remediation

I corrected the scan path so that Nmap targeted the Ubuntu machine directly over the Host-only network.

Validation command:

nmap -p 22,80,81,8080 192.168.56.101

The subsequent results showed:

- TCP 22: open — SSH
- TCP 80: open — HTTP
- TCP 81: closed
- TCP 8080: closed

These results were consistent with the services configured on the target at the time of testing.

## 6. Lessons Learned

- Confirm the source and destination interfaces before troubleshooting network scans.
- Understand the differences between open, closed, and filtered port states.
- Use packet captures to validate what is happening on the network.
- Verify routing and packet paths before concluding that a firewall is responsible.
- Re-test after remediation to confirm that the observed behaviour has changed.

## 7. Security Impact

The issue affected the reliability and speed of reconnaissance results in the lab. No evidence from this exercise established a compromise of either virtual machine.

## 8. Evidence

Related evidence is available in the repository screenshots:

- `nmap-service-scan.png.png`
- `nmap-closed-ports-fixed.png.png`
- `tcpdump-rst-replies.png.png`
- `target-firewall-check.png.png`
- `slow-full-scan.png.png`

## 9. Next Steps

- Complete a full TCP port scan and document the results.
- Review SSH authentication and Apache access logs.
- Configure and validate UFW and Fail2ban.
- Create a separate incident report for a simulated authentication event.

## Ethics and Scope

All testing was performed against virtual machines I own within an isolated VirtualBox lab network. No third-party systems were targeted.
