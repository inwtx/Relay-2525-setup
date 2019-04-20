# Relay-2525-setup
These instructions show how to setup a remailer server to be a port 2525 relay.

Most ISPs block traffic to TCP port 25, therefore your only option is to use the ISP's outgoing SMTP servers.  Direct MX elsewhere is prohibited.  This can allow your ISP to monitor and/or log that you are sending messages to remailers.  Some remailer operators have therefore, opened port 2525 on their servers to allow sending message to known remailers, using the remailer server's MTA.

<b><i>I. Place this code into a file called RemailerAccess.sh:</i></b>
  
#!/bin/bash  
&#35;  
&#35; This script updates a remailer_access file in order for    
&#35; a server to be used as a relay to all remailers (only).  
&#35; It determines the remailer addresses by downloading the  
&#35; mlist2.txt file from the chosen echolot stats pinger  
&#35; pointed to in the 'StatsURL=' parameter below.  
&#35; Create the file remailer_access in your server's  
&#35; /etc/postfix folder.  If you choose to use curl instead  
&#35; of wget, undelimiter 'curl' and delimiter 'wget' below.  
&#35; https://paste.debian.net/1078252  

&#35; Put URL of mlist2.txt to download here after 'StatsURL=':  
StatsURL=https://www.sec3.net/echolot/mlist2.txt

&#35; Location of remailer_access file:  
DEST=/etc/postfix/remailer_access

filePath=${0%/*}  # current file path

&#35;curl https://www.sec3.net/echolot/mlist2.txt > $filePath/mlist2.aux  
wget --no-check-certificate https://www.sec3.net/echolot/mlist2.txt -O $filePath/mlist2.aux  
grep \$remailer $filePath/mlist2.aux | cut -f 2 -d \< | cut -f 1 -d \> | xargs printf "%-60s OK\n" > $DEST

&#35;curl https://cloaked.pw/yamn/mlist2.txt >> $filePath/mlist2.aux  
wget --no-check-certificate https://cloaked.pw/yamn/mlist2.txt -O $filePath/mlist2.aux  
grep \$remailer $filePath/mlist2.aux | cut -f 2 -d \< | cut -f 1 -d \> | xargs printf "%-60s OK\n" >> $DEST

$(which postmap) $DEST

rm $filePath/mlist2.aux

exit 0
  
<b>II. Create this cronjob:</b>  
0 6 * * * /etc/Servstats/RemailerAccess.sh &> /dev/null
  
<b>III. Create this file in /etc/postfix/:</b>  
remailer_access
  
<b>IV. Place this code in /etc/postfix/main.cf:</b>  
smtpd_relay_restrictions =  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;permit_mynetworks,  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;permit_sasl_authenticated,  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;check_recipient_access hash:/etc/postfix/remailer_access,  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;reject_unauth_destination  
  
<b>IIV. Place this line in /etc/postfix/main.cf within 'smtpd_recipient_restrictions =':</b>  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;check_recipient_access hash:/etc/postfix/remailer_access,  


