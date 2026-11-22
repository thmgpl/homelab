# 02 – Checking OPNsense for signs of a break-in after a late patch

## What happened

In October 2026 two critical bugs in PHP itself were published, known as PHP2Shell (CVE-2026-109618 and CVE-2026-109619). They let someone run code on the server without logging in, and within days they were being exploited on a large scale.

The OPNsense web GUI runs on PHP. I installed the fix about a week after it came out (PHP 8.5.11 → 8.5.12 on the 26.7 branch), so I wanted to know whether anything had happened in that time.

## How exposed was I

Before digging through logs, I checked what could even reach the GUI from the internet:

- WireGuard, which doesn't answer anything without a valid key
- a port forward to the Minecraft server, which sits in the isolated DMZ
- no WAN rule or port forward to the GUI itself

So the GUI was not reachable from the internet. The bigger risk was inside the network: at that time the GUI listened on all interfaces, so in theory any device on the IoT or guest network could reach it.

## What I checked

1. Configuration history (System → Configuration → History), looking for changes I didn't make
2. Users and SSH keys, looking for new accounts or keys
3. Cron jobs
4. Running processes, especially shells started by PHP or the web server
5. Listening ports and active connections (`sockstat`)
6. Recently changed files
7. Web GUI logs, looking for requests from public IPs

## What I found

Everything was clean. The only thing that looked odd at first was a group of files with new timestamps. They matched an update I had run myself a few days before.

## If something had turned up

The plan was not to clean it by hand. A router you can't trust gets reinstalled and restored from a config backup made before the attack window. After that, all passwords and keys change, including the VPN keys.

## What went wrong

The update itself took a few minutes. The problem was the delay: a critical, actively exploited bug sat unpatched on my firewall for about a week.

## What changed afterwards

- SSH only with a YubiKey, GUI only on the LAN and ADMIN VLAN: [01 – Hardening admin access](01-admin-access-hardening.md)
- A review of the firewall rules (next write-up)

## Next steps

- Get notified about OPNsense security updates instead of finding out by accident
- Keep regular config backups, so there is always a known-good copy to restore
