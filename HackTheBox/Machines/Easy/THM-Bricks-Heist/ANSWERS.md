# THM Bricks Heist - Answers

## Completed:
1. **What is the content of the hidden .txt file in the web folder?**
   - Answer: `THM{fl46_650c844110baced87e1606453b93f22a}`
   - Location: `/data/www/default/650c844110baced87e1606453b93f22a.txt`

2. **What is the name of the suspicious process?**
   - Answer: `badr`
   - Found via: `systemctl list-units --type=service`

3. **What is the service name affiliated with the suspicious process?**
   - Answer: `badr.service`
   - Location: `/etc/systemd/system/badr.service`

## Still Need:
4. **What is the log file name of the miner instance?**
   - From service: `/var/log/badr.log`
   - Run: `ls -la /var/log/badr.log`

5. **What is the wallet address of the miner instance?**
   - Check: `cat /var/log/badr.log`
   - Or: `grep -r "wallet\|pool" /var/log/ 2>/dev/null`
   - Or: Check process cmdline: `cat /proc/1896/cmdline`

6. **The wallet address used has been involved in transactions between wallets belonging to which threat group?**
   - Once wallet found, search on blockchain explorer or threat intelligence databases

## Commands to run in shell:
```bash
# Stabilize
python3 -c 'import pty;pty.spawn("/bin/bash")'
export TERM=xterm

# Check log
cat /var/log/badr.log

# If log empty, check process
ps aux | grep badr
cat /proc/$(pgrep badr)/cmdline | tr '\0' ' '

# Find wallet
grep -r "4[0-9A-Za-z]" /var/log/ 2>/dev/null | head -20
```
