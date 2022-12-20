# Killing the Log Stream from Kaspersky Endpoint Security Agent


Table of Contents

[TOCM]

[TOC]

#Introduction
> Using a local admin account, I have successfully killed the log stream between Kaspersky agent and QRadar SIEM, using a basic way to prevent the connection to the server but most importantly the Agent has not reported the stream issue to the SIEM, so in this use case we shall see how a Cybercriminal could blind the soc team from the activities on a certain machine.

#Kaspersky Agent State

![](https://github.com/chnz2k/Kill-the-log-stream-of-kaspersky-agent/blob/main/agent-state.jpg?raw=true)

#Default Log Stream

![](https://github.com/chnz2k/Kill-the-log-stream-of-kaspersky-agent/blob/main/log_stream.JPG?raw=true)

> The log has been provided by Kaspersky Agent and as we see the process where blocked and show in our SIEM at the time of the offense this represent our default SIEM workflow.

  • So how can we can kill the communication between the agent and the SIEM ?

#Enumeration and Analysis

> Let’s start by a small network monitoring using “wireshark” to see what’s happening !

![](https://github.com/chnz2k/Kill-the-log-stream-of-kaspersky-agent/blob/main/wireshark_1.JPG?raw=true)

> While deploying a payload I have spotted a specific ip[:]port when Kaspersky agent send the logs data to the SIEM in order to alert the soc team.

![](https://github.com/chnz2k/Kill-the-log-stream-of-kaspersky-agent/blob/main/port_13000.JPG?raw=true)

> Meanwhile after a quick search I found out that the Kaspersky communication goes trough multiple ports one of them is “13000” as figure below shows.

  • So now we have the server ip and port !

> Using the correct filters on wireshark for dstport = “13000” and ip = “10.254.20.114”, show exactly what we need, following the TCP STREM now !

![](https://github.com/chnz2k/Kill-the-log-stream-of-kaspersky-agent/blob/main/wireshark_2.JPG?raw=true)

> We can see a domain in the TCP STREAM string which is the ip that Kaspersky agent report to on the port 13000.

#Kill Stream configuration

> Let’s try some basic way to kill the log stream between the agent and our SIEM.

> Since we have the domain of the server ip that the agent reports to, we can edit the hosts file to point the domain we found in the TCP STREAM to our localhost.

![](https://github.com/chnz2k/Kill-the-log-stream-of-kaspersky-agent/blob/main/hosts.JPG?raw=true)
![](https://github.com/chnz2k/Kill-the-log-stream-of-kaspersky-agent/blob/main/trace.JPG?raw=true)

#Demo
#Conclusion
