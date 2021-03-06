# Relay-2525-setup
These instructions show how to setup a remailer server to be a port 2525 relay.

Most ISPs block traffic to TCP port 25, therefore your only option is to use the ISP's outgoing SMTP servers.  Direct MX elsewhere is prohibited.  This can allow your ISP to monitor and/or log that you are sending messages to remailers.  Some remailer operators have therefore, opened port 2525 on their servers to allow sending message to known remailers, using the remailer server's MTA.

<b><i>I. Place this code into a file in /etc/postfix/ called RemailerAccess.sh:</i></b>
  
  ```
#!/bin/bash  
#  
# This script updates a remailer_access file in order for    
# a server to be used as a relay to all remailers (only).  
# It determines the remailer addresses by downloading the  
# mlist2.txt file from the chosen echolot stats pinger  
# pointed to in the 'StatsURL=' parameter below.  
# Create the file remailer_access in your server's  
# /etc/postfix folder.  

# Put URLs of mlist2.txt to download here:  
StatsURL=https://www.sec3.net/echolot/mlist2.txt  
YamnURL=https://cloaked.pw/yamn/mlist2.txt

# Location of remailer_access file:  
DEST=/etc/postfix/remailer_access

filePath=${0%/*}  # current file path

wget --no-check-certificate $StatsURL -O $filePath/mlist2.aux  
grep \$remailer $filePath/mlist2.aux | cut -f 2 -d \< | cut -f 1 -d \> | xargs printf "%-60s OK\n" > $DEST

wget --no-check-certificate $YamnURL -O $filePath/mlist2.aux  
grep \$remailer $filePath/mlist2.aux | cut -f 2 -d \< | cut -f 1 -d \> | xargs printf "%-60s OK\n" >> $DEST

$(which postmap) $DEST

rm $filePath/mlist2.aux

exit 0
  
```
<b><i>II. Create this cronjob:</i></b>  
0 6 * * * /etc/postfix/RemailerAccess.sh &> /dev/null
  
<b><i>III. Execute /etc/postfix/RemailerAccess.sh once manually to create this file in /etc/postfix/:</i></b>  
remailer_access
  
<b><i>IV. Place this code in /etc/postfix/main.cf:</i></b>  
smtpd_relay_restrictions =  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;permit_mynetworks,  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;permit_sasl_authenticated,  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;check_recipient_access hash:/etc/postfix/remailer_access,  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;reject_unauth_destination  
  
<b><i>IIV. Place this line in /etc/postfix/main.cf within 'smtpd_recipient_restrictions =':</i></b>  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;check_recipient_access hash:/etc/postfix/remailer_access,  


