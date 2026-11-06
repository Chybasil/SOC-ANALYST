# Designing a detection rule and testing it against real-lookin noise

## Allowed set of IP range/VPN usage
Normal staff sign in from office range (192.0.2.0/24) or the VPN

## Alerts Table (Between 14:00 - 15:00)

| Time  | Source IP    | type     | Account    | Status     |
|-------|--------------|----------|------------|------------|
| 14:03 | 203.0.113.77 | External | a.morgan   | Failed     |
| 14:06 | 203.0.113.77 | External | b.lee      | Failed     |
| 14:09 | 203.0.113.77 | External | c.patel    | Failed     |
| 14:13 | 203.0.113.77 | External | d.nguyen   | Failed     |
| 14:52 | 203.0.113.77 | External | k.yoon     | Successful |
|  -    | 192.0.2.5    | Internal | VPN        | Successful |
|  -    | 192.0.2.30   | Internal | Server 1   | Succesful  |
|  -    | 192.0.2.44   | Internal | j.thompson | successful |



## Deliverable 1: Why the old rule missed alerting;

Old rule failed to flag VPN usage. This is because IP 203.0.113.77 is an external and has a failed login then bypassed using VPN which on allowed IP, 192.0.2.5 then successfully moving laterally by gaining more access to other accounts. 

## Deliverable 2: New rules;

I would group IP addresses 192.0.2.0/24 range as normal. Any other IPs like 203.0.113.77 as abnormal immediate alerting. Then set more than 3 failed logins alerts to any IP not in subnet range. After any successful login and screen time is more than 5 minutes with data transfer size limit be flagged. Again, set rules for lateral movement and abnormal behavours.

## Deliverable 3: False positives;

To handle false positives, IP 203.0.113.77 has not been seen before and is automatic flag. Then 192.0.2.5 is address range under VPN would be checked for identity, device and access pattern. Also, set location base rule and time allowed for access (work). Again, for expired password with IP 192.0.2.30 audit the account and force password reset.

## Deliverable 4: First three steps mitigation;

First line of action is blocking IP, 203.113.77. Second, for successful login at 14:52 audit the account and restrict privilege permissions. Then investigate the account on k.yoon, disable account and force password reset.

## MITRE ATT&CK Technique and Control (https://attack.mitre.org/techniques/T1078)

Technique - Valid account

Control - Control is active directory configuration

