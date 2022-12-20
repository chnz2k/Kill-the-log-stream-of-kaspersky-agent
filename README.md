# Killing the Log Stream from Kaspersky Endpoint Security Agent

# Why i did it ? 
 * short answer is "Simplly beacause i can".
 * The truth is "that i can't uninstall the agent from the machine because of the company has a compliance certificate ;)"

# Product informations
  * Product name : Kaspersky Endpoint Security
  * Version number : 11.6.0.394 AES56
  * Operating system : Windows 10 Pro (22H2) 19045.2364, Windows Feature Experience Pack 120.2212.4190.0

# One more thing you might see hidden/blured parts of pictures below, it's because the demo was tested in a real infastructure (not a LAB).

1. ##### Introduction
2. ##### Kaspersky Agent State
3. ##### Default Log Stream
4. ##### Enumeration and Analysis
5. ##### Kill Stream configuration
6. ##### Demo
7. ##### Conclusion


# 1. Introduction
> Using a local admin account, I have successfully killed the log stream between Kaspersky agent and QRadar SIEM, using a basic way to prevent the connection to the server but most importantly the Agent has not reported the stream issue to the SIEM, so in this use case we shall see how a Cybercriminal could blind the soc team from the activities on a certain machine.

# 2. Kaspersky Agent State
![](https://github.com/chnz2k/Kill-the-log-stream-of-kaspersky-agent/blob/main/agent-state.jpg?raw=true)

# 3. Default Log Stream

![](https://github.com/chnz2k/Kill-the-log-stream-of-kaspersky-agent/blob/main/log_stream.JPG?raw=true)

> The log has been provided by Kaspersky Agent and as we see the process where blocked and show in our SIEM at the time of the offense this represent our default SIEM workflow.

  • So how can we can kill the communication between the agent and the SIEM ?

# 4. Enumeration and Analysis

> Let’s start by a small network monitoring using “wireshark” to see what’s happening !

![](https://github.com/chnz2k/Kill-the-log-stream-of-kaspersky-agent/blob/main/wireshark_1.JPG?raw=true)

> While deploying a payload I have spotted a specific ip[:]port when Kaspersky agent send the logs data to the SIEM in order to alert the soc team.

![](https://github.com/chnz2k/Kill-the-log-stream-of-kaspersky-agent/blob/main/port_13000.JPG?raw=true)

> Meanwhile after a quick search I found out that the Kaspersky communication goes trough multiple ports one of them is “13000” as figure below shows.

  • So now we have the server ip and port !

> Using the correct filters on wireshark for dstport = “13000” and ip = “10.254.20.114”, show exactly what we need, following the TCP STREM now !

![](https://github.com/chnz2k/Kill-the-log-stream-of-kaspersky-agent/blob/main/wireshark_2.JPG?raw=true)

> We can see a domain in the TCP STREAM string which is the ip that Kaspersky agent report to on the port 13000.

# 5. Kill Stream configuration

> Let’s try some basic way to kill the log stream between the agent and our SIEM.

> Since we have the domain of the server ip that the agent reports to, we can edit the hosts file to point the domain we found in the TCP STREAM to our localhost.

![](https://github.com/chnz2k/Kill-the-log-stream-of-kaspersky-agent/blob/main/hosts.JPG?raw=true)
![](https://github.com/chnz2k/Kill-the-log-stream-of-kaspersky-agent/blob/main/trace.JPG?raw=true)

> Checking the advanced Firewall rules (INBOUND), I found that Kaspersky has already a defined rules, that the only way to contact the server for updates (INBOUND).

![](https://github.com/chnz2k/Kill-the-log-stream-of-kaspersky-agent/blob/main/rules_1.JPG?raw=true)

> So I taught about reversing those rules and made them Blocking instead of allowing (Still Enabled).

Change “Allow the connection” to “Block the connection” :

![](https://github.com/chnz2k/Kill-the-log-stream-of-kaspersky-agent/blob/main/rules_details_1.JPG?raw=true)

> The 3 scopes “Domain”, “Private”, “Public” shall be selected :
![](https://github.com/chnz2k/Kill-the-log-stream-of-kaspersky-agent/blob/main/rules_details_2.JPG?raw=true)

> After applying same configuration to all “Kaspersky” rules the output should be something similar to this :
![](https://github.com/chnz2k/Kill-the-log-stream-of-kaspersky-agent/blob/main/rules_details_3.JPG?raw=true)

> Now for the OUTBOUND Rules we should consider adding two main rules for both protocoles “TCP” and “UDP” :

Go to “New Rule” :

![](https://github.com/chnz2k/Kill-the-log-stream-of-kaspersky-agent/blob/main/rules_details_4.JPG?raw=true)

Select “port” :

![](https://github.com/chnz2k/Kill-the-log-stream-of-kaspersky-agent/blob/main/rules_details_5.JPG?raw=true)

Edit “Specific remote ports” to “13000” :

![](https://github.com/chnz2k/Kill-the-log-stream-of-kaspersky-agent/blob/main/rules_details_6.JPG?raw=true)

Chose “Block the connection” :

![](https://github.com/chnz2k/Kill-the-log-stream-of-kaspersky-agent/blob/main/rules_details_7.JPG?raw=true)

Make sure the 3 scopes are selected :

![](https://github.com/chnz2k/Kill-the-log-stream-of-kaspersky-agent/blob/main/rules_details_8.JPG?raw=true)

> Then you repeat the same thing for “UDP”.

The output should be like something similar to this :
![](https://github.com/chnz2k/Kill-the-log-stream-of-kaspersky-agent/blob/main/rules_details_9.JPG?raw=true)

# 6. Demo
After an execution of a malicious payload the agent did popup a notification about the detection in the local machine :
![](https://github.com/chnz2k/Kill-the-log-stream-of-kaspersky-agent/blob/main/rules_details_10.JPG?raw=true)

Back to our “SIEM” nothing was detected or in other words the log stream could not reach the server :
![](https://github.com/chnz2k/Kill-the-log-stream-of-kaspersky-agent/blob/main/rules_details_11.JPG?raw=true)

> Well, Nothing reported to our “SIEM” no detection of the payload execution on the “SIEM” solution neither the evasion technique applied !

# 7. Conclusion
> Any local admin account can cause this log stream issue and I believe that “kaspersky” agent should setup some kind of measure to protect or at least detect these techniques, It’s so simple but still effective !
