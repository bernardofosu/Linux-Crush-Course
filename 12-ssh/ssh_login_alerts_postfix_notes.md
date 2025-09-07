# 📧 SSH Login Alerts & Postfix Configuration – Notes

## 1️⃣ Check logins manually (on demand)
Run:
```bash
last -a | grep ubuntu
```
Shows:
- Username  
- Time of login/logout  
- Source IP/hostname  

Example:
```
ubuntu   pts/0   154.161.161.208   Sat Sep  6 19:16   still logged in
ubuntu   pts/1   154.161.161.208   Sat Sep  6 19:47   still logged in
```

---

## 2️⃣ View in real time from logs
Logins are recorded in `/var/log/auth.log`.  
Run:
```bash
sudo tail -f /var/log/auth.log | grep sshd
```
Example line:
```
Accepted publickey for ubuntu from 154.161.161.208 port 52234 ssh2
```

---

## 3️⃣ Add a login notification message
Append to `~/.bashrc`:
```bash
echo "⚠️ Login detected on $(hostname) at $(date) from $SSH_CONNECTION" |   mail -s "SSH Login Alert: ubuntu@$(hostname)" you@example.com
```
- **echo "..."** → body of the email (hostname, time, IP).  
- **mail -s "..."** → subject line.  
- **you@example.com** → destination email.  

💡 Every SSH login as `ubuntu` will trigger an email.  

---

## 4️⃣ Use logger to record logins
Add to `~/.bash_profile` (or `~/.profile`):
```bash
logger "SSH LOGIN: user=$USER from=$SSH_CONNECTION on $(hostname)"
```
This writes to `/var/log/syslog` at every login.  

---

## 5️⃣ Fancy tools
- **Fail2Ban** → can monitor SSH and send alerts on logins/failures.  
- **auditd** → track specific user logins and log them centrally.  

✅ Fastest for you: **tail auth.log** (manual check) or **email alert via mailutils**.  

---

## 🛠️ How to enable email alerts
1. Install mailutils:
```bash
sudo apt update
sudo apt install mailutils -y
```

2. Add the command in `~/.bashrc`.  

3. Reload:
```bash
source ~/.bashrc
```

4. Test:
```bash
echo "Test mail" | mail -s "Test subject" you@example.com
```

---

## ⚠️ Caveats
- 📧 `mailutils` uses Postfix by default. EC2 blocks **port 25** → use **587** (Gmail/SES).  
- 🌐 Ensure outbound port 587 is open in your AWS Security Group.  
- 🛡️ You’ll get an email for **each login session** (even multiple tabs).  

---

## 📮 Postfix Configuration Options
When installing mailutils, you’ll see these choices:

- **No configuration** → Postfix disabled.  
- **Internet Site** ✅ → send mail directly via SMTP (simple).  
- **Internet with smarthost** → relay via Gmail, AWS SES, or corporate SMTP.  
- **Satellite system** → all mail forwarded to one relay machine.  
- **Local only** → mail stays on the server (not useful for alerts).  

---

## 🖊️ System Mail Name
Postfix will ask for the **system mail name**:
- Default → `ip-172-31-28-101.ec2.internal`  
- If you own a domain → use `mydomain.com` (looks more trusted).  

---

## ⚡ AWS EC2 Caveat
- Port **25** blocked by AWS (anti-spam).  
- Use port **587** with Gmail/SES for reliable delivery.  
- Or request AWS to unblock port 25 (not common for small projects).  

---

## ✅ Recommendation
- Choose **Internet Site** during Postfix setup.  
- Leave mail name as **ip-172-31-28-101.ec2.internal** for now.  
- Later → configure Postfix with Gmail/SES relay on **port 587** for reliable inbox delivery.  
