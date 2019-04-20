# Relay-2525-setup
These instructions show how to setup a remailer server to be a port 2525 relay.

Most ISPs block traffic to TCP port 25, therefore your only option is to use the ISP's outgoing SMTP servers.  Direct MX elsewhere is prohibited.  This can allow your ISP to monitor and/or log that your are sending messages to remailers.  Some remailer operators have therefore, opened port 2525 on their servers to allow sending message to known remailers, using the remailer server's MTA.

I. Place this code into a file called RemailerAccess.sh:
  
#!/bin/bash  
#!  
#! This script updates a remailer_access file in order for    
#! a server to be used as a relay to all remailers (only).  
#! It determines the remailer addresses by downloading the  
#! mlist2.txt file from the chosen echolot stats pinger  
#! pointed to in the 'StatsURL=' parameter below.  
#! Create the file remailer_access in your server's  
#! /etc/postfix folder.  If you choose to use curl instead  
#! of wget, undelimiter 'curl' and delimiter 'wget' below.  
#! https://paste.debian.net/1078252  

#! Put URL of mlist2.txt to download here after 'StatsURL=':  
StatsURL=https://www.sec3.net/echolot/mlist2.txt

#! Location of remailer_access file:  
DEST=/etc/postfix/remailer_access

filePath=${0%/*}  # current file path

#!curl https://www.sec3.net/echolot/mlist2.txt > $filePath/mlist2.aux
wget --no-check-certificate https://www.sec3.net/echolot/mlist2.txt -O $filePath/mlist2.aux
grep \$remailer $filePath/mlist2.aux | cut -f 2 -d \< | cut -f 1 -d \> | xargs printf "%-60s OK\n" > $DEST

#!curl https://cloaked.pw/yamn/mlist2.txt >> $filePath/mlist2.aux
wget --no-check-certificate https://cloaked.pw/yamn/mlist2.txt -O $filePath/mlist2.aux
grep \$remailer $filePath/mlist2.aux | cut -f 2 -d \< | cut -f 1 -d \> | xargs printf "%-60s OK\n" >> $DEST

$(which postmap) $DEST

rm $filePath/mlist2.aux

exit 0
  
II. Create this cronjob:  
0 6 * * * /etc/Servstats/RemailerAccess.sh &> /dev/null
  
III. Create this file in /etc/postfix/  
remailer_access
  
IV. Place this code in /etc/postfix/main.cf  

