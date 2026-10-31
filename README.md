# tryhackme-intro-to-logs-writeup

**TryHackMe profile:** [prajesh.shrestha250](https://tryhackme.com/p/prajesh.shrestha250?utm_campaign=social_share&utm_medium=social&utm_content=profile&cardVersion=d54c0f25bcaf&utm_source=copy&sharerId=6ae16e4554080224cf72b648)

---

# Intro to Logs: TryHackMe Write-up

| | |
|---|---|
| **Platform** | TryHackMe |
| **Room** | Intro to Logs |
| **Focus** | Log collection, configuration, rotation and analysis |
| **Completed** | October 2026 |

## Overview

This room teaches how logs are generated, formatted, collected, stored and analysed, and why each step matters for detecting and investigating attacks. It is built around a realistic scenario in which I act as a systems administrator responding to suspicious activity on a company's public-facing web server.

## Scenario

A public-facing GitLab server was being repeatedly scanned by an unknown adversary. The server was poorly maintained, and log forwarding to the SIEM had never been configured. My task was to set up proper logging, collect the logs manually, and work out what the attacker was doing.

## Concepts Covered

- **Log types:** system, application, web server and security logs
- **Log formats:** e.g. Common and Combined Log Format, and how to identify which one a file uses by reading its fields (IP, timestamp, request, status code, referrer, user agent)
- **Log collection and forwarding:** the role of rsyslog and a SIEM
- **Log storage and rotation:** retention, rotation frequency and compression with logrotate
- **Log integrity:** hashing rotated files so tampering can be detected
- **Log analysis:** reading raw logs, filtering, and the limits of unparsed logs in a log viewer

## What I Did

1. **Reviewed the environment.** Checked my limited sudo permissions and read the handover note to identify which logs mattered.
2. **Identified log types and formats.** Examined the Nginx access log of the GitLab server and confirmed it uses the Combined Log Format by breaking down a sample entry.
3. **Configured rsyslog.** Wrote rules that send sshd and cron messages to dedicated log files, and reviewed an existing config that had log forwarding to the SIEM commented out.
4. **Configured logrotate.** Created a rotation policy with a set frequency, number of retained copies and compression, plus a post-rotation script that records SHA-256 hashes of archived logs.
5. **Analysed the logs.** Reviewed web, SSH and cron logs for suspicious behaviour.
6. **Used a log viewer.** Loaded the raw logs and saw that unparsed logs lack a date field, so they can't be filtered or merged until they are parsed.

## Findings

| Stage | Evidence | Log source |
|---|---|---|
| Reconnaissance | Repeated automated requests with an outdated, spoofed browser user agent | Nginx access log |
| Credential attack | Many parallel failed SSH logins against a non-existent username | sshd log |
| Persistence | A root cron job launching a reverse shell every minute | cron log |

All three activities came from the **same source IP**, showing a single adversary moving through reconnaissance, initial access attempts and persistence. Correlating logs from multiple sources made this visible. No single log told the whole story.

## Key Takeaways

- Logs are only useful if they are collected, retained and protected. Rotation and hashing preserve evidence.
- Parsing and normalising logs is what makes them searchable and mergeable in tools like a SIEM.
- Attacks leave traces across several logs, so correlation is essential.
- Spoofed or outdated user agents, bursts of failed logins, and unexpected cron jobs are simple but effective indicators of compromise.

## Commands and Tools Used

`sudo`, `cat`, `head`, `tail`, `grep`, `ls`, `systemctl`, `logrotate`, `rsyslog`, `sha256sum`, a web-based log viewer

## Possible Improvements

- Enable log forwarding to the SIEM so logs no longer need manual collection
- Block the offending IP and add rate limiting or `fail2ban` for SSH
- Disable password authentication for SSH and use keys instead
- Remove the malicious cron entry and investigate how it was planted
- Alert on new cron jobs and on repeated failed logins

## Skills Demonstrated

Linux command line, log analysis, rsyslog and logrotate configuration, threat identification, incident investigation, technical documentation

---
