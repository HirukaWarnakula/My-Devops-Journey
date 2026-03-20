# Incident Report: Apache Port 6400 Connectivity Fix

## 🛠 Problem Description
The Apache service on `stapp01` was unreachable on port **6400**. 
Initial diagnostics showed the service was in a `failed` state.

## 🔍 Diagnostics & Discovery
I used the following tools to identify the root cause:
1. systemctl status httpd: Confirmed the service failed with error `(98)Address already in use`.
2. ss -tulpn: Identified that `sendmail` (PID 16207) was already listening on port 6400.

## The Fix

### 1. Evicting the Port Conflict
First, I had to stop the service occupying our target port:
```bash
sudo systemctl stop sendmail
sudo systemctl disable sendmail

## 2.Securing with Firewall(iptables)

Instead of disabling the firewall , I added a specific "Security Setting" to allow traffic while keeping the system protected:

   #Added above the REJECT lines in /etc/sysconfig/iptables:
-A INPUT -p tcp -m state --state NEW tcp --dport 6400 -j ACCEPT

Key Lessons I Learned 

* Port Ownership:Always  check ss or netstat first .Multiple Services Cannot bind to the same      IP/Port combination.
*Firewall Persistence :Disabling a firewall is a "quick fix" but not a proffesional one. 
*Syntax Matters :iptable requires a blank newline at the end of the config file to parse correctly .
*Fleet Health: In a cluster, checking one node isn't enough. Use a for loop to verify the whole fleet.

