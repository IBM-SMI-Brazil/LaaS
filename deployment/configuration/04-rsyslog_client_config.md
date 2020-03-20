# RSYSLOG Client Configuration Steps

The following steps should be performed on Syslog servers to configure it to send the events to the central Graylog server.  Please check the sections below for instructions on how to do this configurations with TLS certificates or without encryption (**not recommended**).


## Configuration using TLS

To ensure that your events are encrypted during the transmission, copy the key and certificate to Syslog server at the following path:

- /etc/rsyslog.d/keys/ca.d/server.crt
- /etc/rsyslog.d/keys/ca.d/server.key

Be sure to create a file with the following content on your syslog server at `/etc/rsyslog.d/10-rsyslog_client_forward.conf`:

```
global(
DefaultNetstreamDriver="gtls"
DefaultNetstreamDriverCertFile="/etc/rsyslog.d/keys/ca.d/server.crt"
DefaultNetstreamDriverKeyFile="/etc/rsyslog.d/keys/ca.d/server.key"
)
# set up the action for all messages
*.* action(type="omfwd" Target="192.168.0.20" Port="1515" Protocol="tcp" StreamDriver="gtls" StreamDriverMode="1" StreamDriverAuthMode="anon" Template="RSYSLOG_SyslogProtocol23Format")
```

After creating the file, perform a restart of the rsyslog service.


## Configuration without TLS

If no encription is required, create a file with the following content on your syslog server at `/etc/rsyslog.d/10-rsyslog_client_forward.conf` and them restart the rsyslog services:

```
*.* action(type="omfwd" Target="192.168.0.20" Port="1515" Protocol="tcp" Template="RSYSLOG_SyslogProtocol23Format")
```

**Please notice that by not using certificates to encrypt your messages is not recommended.**

## [ Optional ] Sending all executed commands to Graylog

As an example on what can be achieved with the proposed solution, one add the following to a user profile so that all the commands executed by the user are automatically sent to the Graylog server:

Append this to the user `/etc/profile` to log shell commands to local6 syslog facility:

```
# TTY to syslog
readonly CURRENT_TTY=`tty | sed 's/\/dev\///g'`
readonly CURRENT_SESSION_IP=`pinky | grep ${CURRENT_TTY} | awk '{ print $NF }'`
export CURRENT_TTY
export CURRENT_SESSION_IP
readonly PROMPT_COMMAND='RETRN_VAL=$?;logger -p local6.debug "$(whoami) [$$]: $(history 1 | sed "s/^[ ]*[0-9]\+[ ]*//" ) [$RETRN_VAL] [$CURRENT_TTY] [$CURRENT_SESSION_IP]"'
export PROMPT_COMMAND
```

The message will be sent to syslog at the local6 facility according to the format shown in the example below:

```
hprado [1122]: traceroute 8.8.8.8 [0] [pts/0] [192.168.0.7] 
hprado [22407]: netstat -na | grep -i established [0] [pts/0] [192.168.0.7]
root [3841]: tail -1000 /var/log/messages | grep cmd [0] [pts/0] [192.168.0.7]
hprado [1122]: cat  /etc/sysconfig/network [0] [pts/0] [192.168.0.7]  
hprado [18401]: docker ps -a | grep grafana | awk '{print $1}' [0] [pts/0] [192.168.0.7] 
```