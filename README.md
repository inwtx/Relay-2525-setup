# Relay-2525-setup
These instructions show how to setup a remailer server to be a port 2525 relay.

Most ISPs block traffic to TCP port 25, therefore your only option is to use the ISP's outgoing SMTP servers.  Direct MX elsewhere is prohibited.  This can allow your ISP to monitor and/or log that your are sending messages to remailers.  Some remailer operators have therefore, opened port 2525 on their servers to allow sending message to known remailers, using the remailer server's MTA.
