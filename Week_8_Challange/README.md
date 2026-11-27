# SOC - Grouping noisy alerts and tuning a detection rules

## NB: - All samples used are fictional
## Ticket:  - A large file upload rule fired 24 times last week and team started ignoring it and can be dangerous. Fixing the rule so that it is useful again

## Knowledge base:

- 𝐓𝐨𝐝𝐚𝐲'𝐬 𝐝𝐚𝐭𝐞: 23 November 2026. The alerts cover Monday 16 to Sunday 22 November (times in GMT).
- 𝐓𝐡𝐞 𝐫𝐮𝐥𝐞: `LARGE-UPLOAD-01`: alert when any internal host sends 𝐦𝐨𝐫𝐞 𝐭𝐡𝐚𝐧 𝟓𝟎𝟎 𝐌𝐁 to an external destination within one hour.
- 𝐀𝐩𝐩𝐫𝐨𝐯𝐞𝐝 𝐝𝐞𝐬𝐭𝐢𝐧𝐚𝐭𝐢𝐨𝐧𝐬: `backup.vaultly.example` (backup), `registry.buildhub.example` (software build images), `video.clipshare.example` (webinar videos, Marketing only), `demo.webconf.example` (customer demo recordings, Sales), `payroll.paywise.example` (payroll), `git.codehost.example` (source code), and `sftp.clinicgroup-a.example` (a weekly billing export every Friday at 06:00, change ticket CHG-4388).
- `drive.freefilebox.example` is a personal file-sharing site. It is 𝐧𝐨𝐭 approved.
- `198.51.100.77` belongs to a hosting provider. Northwind has no contract with it.
- The nightly backup normally sends about 40 GB at around 02:00, every night.
- `NW-RPT-01` is an internal reporting server. Over the last 90 days it has sent less than 5 MB a day to the internet, only to the operating-system update server. It has 𝐧𝐨 𝐄𝐃𝐑 𝐚𝐠𝐞𝐧𝐭.
- HR says `a.novak` (Finance) gave notice on Monday 16 November. Their last day is Friday 27 November.

### 𝐓𝐡𝐞 𝟐𝟒 𝐚𝐥𝐞𝐫𝐭𝐬


| # | Day, time | Source host | Role | Destination | MB sent | Context from the logs |
|---|---|---|---|---|---|---|
| 1 | Mon 16 02:12 | NW-BKP01 | Backup server | `backup.vaultly.example` | 39,800 | Process: vaultly-agent, scheduled job |
| 2 | Mon 16 10:14 | NW-CI01 | Build server | `registry.buildhub.example` | 820 | Account: svc-ci, process: docker push |
| 3 | Mon 16 13:20 | NW-MAC-0212 | Marketing laptop | `video.clipshare.example` | 1,150 | Browser, user t.brooks |
| 4 | Mon 16 17:02 | NW-HR-APP | HR app server | `payroll.paywise.example` | 580 | Account: svc-payroll, scheduled job |
| 5 | Tue 17 02:12 | NW-BKP01 | Backup server | `backup.vaultly.example` | 40,450 | Process: vaultly-agent, scheduled job |
| 6 | Tue 17 14:30 | NW-CI01 | Build server | `registry.buildhub.example` | 905 | Account: svc-ci, process: docker push |
| 7 | Tue 17 15:10 | NW-MAC-0212 | Marketing laptop | `video.clipshare.example` | 1,350 | Browser, user t.brooks |
| 8 | Tue 17 16:20 | NW-DEV-0931 | Developer laptop | `git.codehost.example` | 612 | git push, user r.cho |
| 9 | Wed 18 02:12 | NW-BKP01 | Backup server | `backup.vaultly.example` | 41,200 | Process: vaultly-agent, scheduled job |
| 10 | Wed 18 10:02 | NW-LT-0215 | Sales laptop | `demo.webconf.example` | 610 | Browser, user m.dube |
| 11 | Wed 18 11:05 | NW-CI01 | Build server | `registry.buildhub.example` | 870 | Account: svc-ci, process: docker push |
| 12 | Wed 18 16:42 | NW-CI01 | Build server | `registry.buildhub.example` | 940 | Account: svc-ci, process: docker push |
| 13 | Wed 18 22:41 | NW-LT-0447 | Finance laptop | `drive.freefilebox.example` | 2,140 | Browser, user a.novak, home internet connection |
| 14 | Thu 19 11:40 | NW-MAC-0212 | Marketing laptop | `video.clipshare.example` | 2,210 | Browser, user t.brooks |
| 15 | Thu 19 15:20 | NW-CI01 | Build server | `registry.buildhub.example` | 880 | Account: svc-ci, process: docker push |
| 16 | Thu 19 23:05 | NW-LT-0447 | Finance laptop | `drive.freefilebox.example` | 1,960 | Browser, user a.novak, home internet connection |
| 17 | Fri 20 02:12 | NW-BKP01 | Backup server | `backup.vaultly.example` | 40,100 | Process: vaultly-agent, scheduled job |
| 18 | Fri 20 06:04 | NW-APP-03 | Billing app server | `sftp.clinicgroup-a.example` | 640 | Account: svc-export, SFTP |
| 19 | Fri 20 09:55 | NW-CI01 | Build server | `registry.buildhub.example` | 910 | Account: svc-ci, process: docker push |
| 20 | Fri 20 10:01 | NW-LT-0215 | Sales laptop | `demo.webconf.example` | 640 | Browser, user m.dube |
| 21 | Fri 20 14:05 | NW-MAC-0212 | Marketing laptop | `video.clipshare.example` | 1,780 | Browser, user t.brooks |
| 22 | Sat 21 02:12 | NW-BKP01 | Backup server | `backup.vaultly.example` | 41,900 | Process: vaultly-agent, scheduled job |
| 23 | Sat 21 04:20 | NW-RPT-01 | Reporting server | `198.51.100.77` | 1,420 | Process unknown (no EDR agent), no signed-in user |
| 24 | Sun 22 02:12 | NW-BKP01 | Backup server | `backup.vaultly.example` | 38,700 | Process: vaultly-agent, scheduled job |


## Deliverable 1 - Grouping:


| S/N | Source Host | Destination | Count | Benign | Investigate | Evidence |
|---|---|---|---|---|---|---|
| 1 | NW-BKP01 | backup.vaultly.example | 6 | No | Yes | Sent 39,800 + 40,450 MB of files between Mon 16 02:12 and Tue 17 02:12 which should be sent by NW-RPT-01 and more than 500 MB per hour |
| 2 | NW-CI01 | registry.buildhub.example | 6 | Yes | No | Sent 820 + 905 MB between Mon 16 10:14 and Tue 17 14:30 which in range of 500 MB per hour |
| 3 | NW-MAC-0212 | video.clipshare.example | 4 | Yes | No | Sent 1,150 + 1,130 MB between Mon 16 13:20 and Tue 17 15:10 which in range of 500 MB per hour |
| 4 | NW-HR-APP | payroll.paywise.example | 1 | Yes | No | Sent 580 MB over the week which in 500 MB per hour |
| 5 | NW-DEV-0931 | git.codehost.example | 1 | Yes | No | Sent 612 MB over the week which in 500 MB per hour |
| 6 | NW-LT-0215 | demo.webconf.example | 2 | Yes | No | Sent 610 MB over the week which in range of 500 MB per hour |
|7 | NW-LT-0447 | drive.freefilebox.example | 2 | Yes | Yes | Sent 2,140 MB to non-approved destination in range 500 MB per hour |
| 8 | NW-APP-03 | sftp.clinicgroup-a.example | 1 | Yes | No | Sent 640 MB over the week which in 500 MB per hour |
| 9 | NW-RPT-01 | 198.51.100.77 | 1 | Yes | Yes | Northwind has no contract with it |


## Deliverable 2 - Investigation:

- From deliverable 1, NW-BKP01 should be investigated and configured to pass through NW-RPT-01 because NW-RPT-01 is an internal reporting server and over the last 90 days it has sent less than 5 MB a day to the internet, only to the operating-system update server and has 𝐧𝐨 𝐄𝐃𝐑 𝐚𝐠𝐞𝐧𝐭
- Again, NW-LT-0447 and NW-RPT-01 data sent to drive.freefilebox.example and 198.51.100.77 respectively should be investigated because of non-compliance.
  

## Deliverable 3 - Setting New Rules:
The new should include normal 500 MB per hour and less than 1GB no alert. This will cover the noise that occurred at number 2, 3, 4, 5, 6, and 8 in deliverable 1 table as well alert for number 1 for more than 500 MB sent and non-compliance destinations.


## Deliverable 4 - The Blind Spot:
The new rule will alert when more than 500 MB are sent per hour and create noise when it is from different sources over week but setting controls over the week will mitigate any noise that may occur.
