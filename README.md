# relayx - msmtp + mailx Setup for Gmail SMTP Relay on RHEL 8 / Rocky / AlmaLinux

This guide helps you install and configure msmtp and mailx to send emails via Gmail SMTP relay on RHEL 8 and its derivatives (Rocky Linux, AlmaLinux). Ideal for system alerts, cron jobs, and script-based notifications.

🛠️ Installation
Enable EPEL repository and install required packages:

```bash
sudo dnf install epel-release
sudo dnf install msmtp mailx
```

⚙️ Configure msmtp for Gmail
Create ~/.msmtprc and set permissions:

touch ~/.msmtprc
chmod 600 ~/.msmtprc

Example configuration:

defaults
auth           on
tls            on
tls_trust_file /etc/ssl/certs/ca-bundle.crt
logfile        ~/.msmtp.log

account gmail
host           smtp.gmail.com
port           587
from           your.email@gmail.com
user           your.email@gmail.com
password       your_app_password

account default : gmail

NOTE: 🔐 Gmail requires an App Password if 2FA is enabled. 
Check it: https://support.google.com/accounts/answer/185833?hl=en

📤 Configure mailx to use msmtp

Create ~/.mailrc:

set sendmail="/usr/bin/msmtp"
set use_from=yes
set from="your.email@gmail.com"


✅ Send a test email

echo "Test message body" | mailx -s "Test Subject" recipient@example.com

🔀 Using multiple Gmail accounts

Add multiple accounts to ~/.msmtprc:

account gmail1
host smtp.gmail.com
port 587
from first.email@gmail.com
user first.email@gmail.com
password app_password1

account gmail2
host smtp.gmail.com
port 587
from second.email@gmail.com
user second.email@gmail.com
password app_password2

Send using a specific account:

echo "Message" | msmtp -a gmail2 recipient@example.com


🧪 Example: Sending System Info via Script

You can use msmtp directly in a Bash script to send system diagnostics or alerts. Here's a real-world example that collects basic system information and emails it using Gmail SMTP relay:

#!/bin/bash

INFOFILE="/tmp/system_info.txt"

# Collect system information
{
  echo "=== Hostname ==="
  hostname
  echo
  echo "=== OS ==="
  cat /etc/os-release
  echo
  echo "=== CPU ==="
  lscpu | grep 'Model name'
  echo
  echo "=== Memory ==="
  free -h
} > "$INFOFILE"

# Send email with MIME headers
{
  echo "From: your.email@gmail.com"
  echo "To: recipient@example.com"
  echo "Subject: System Info - $(hostname)"
  echo "Content-Type: text/plain; charset=\"UTF-8\""
  echo
  cat "$INFOFILE"
} | msmtp --debug --from=default -t

🚀 How to Use

Save the script to a file:
nano send_sysinfo.sh
# Paste the script above, then save and exit

Make it executable:
chmod +x send_sysinfo.sh

Run the script:
./send_sysinfo.sh


✅ The recipient will receive a clean, readable email with proper headers, subject line, and UTF-8 encoding — no raw dumps or broken formatting.

✅ Make sure your ~/.msmtprc is properly configured with Gmail credentials and default account.

This approach is ideal for cron jobs, monitoring scripts, or remote diagnostics. You can extend it to include disk usage, mount point checks, or alert thresholds.



