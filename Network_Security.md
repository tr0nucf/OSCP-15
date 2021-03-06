# Network Security

## Passive vs Active Reconnaissance 

Before the dawn of computer systems and networks, in the Art of War, Sun Tzu taught, "If you know the enemy and know yourself, your victory will not stand in doubt." If you are playing the role of an attacker, you need to gather information about your target systems. If you are playing the role of a defender, you need to know what your adversary will discover about your systems and networks.

Reconnaissance (recon) can be defined as a preliminary survey to gather information about a target. It is the first step in The Unified Kill Chain to gain an initial foothold on a system. We divide reconnaissance into:

1. Passive Reconnaissance 
2. Active Reconnaissance 

In passive reconnaissance, you rely on publicly available knowledge. It is the knowledge that you can access from publicly available resources without directly engaging with the target. Think of it like you are looking at target territory from afar without stepping foot on that territory. 

Passive reconnaissance activities include many activities, for instance: 

- Lokking up DNS records of a domain from a public DNS server.
- Checking job ads related to the target website.
- Reading news articles about the target compnay 

Active reconnaissance, on the other hand, cannot be achieved so discreetly. It requires direct engagement with the target. Think of it like you check the locks on the doors and windows, among other potential entry points.

Examples of active reconnaissance activities include: 

- Connecting to one of the company server such as HTTP, FTP, SMTP
- Calling the company in an attempt to get information (social engineering)
- Entering company premises pretending to be a repairman 

## Whois 

WHOIS is a request and respose protocol that follows the RFC 3912 specification. A WHOIS server listens on TCP port 43 for incoming requests. The domain registrar is responsible for maintaining the WHOIS records for the domain names it is leasing. The WHOIS server replies with various information related to the domain requested. Of particular interest, we can learn:

- Registrar: Via which registrar was the domain name registered?
- Contact info of registrant: Name, Organization, address, phone, among other things. (Unless made hidden via privacy service) 
- Creation, update and expirartion dates: When was the domain name first registered? When was it last updated? And when does it need to be renewed?
- Name Server: Which server to ask to resolve the domain name?

To get this information, we need to use a ```whois``` client or an online service. Many online services provide ```whois``` information; however, it is generally faster and more convenient to use your local whois client. 

![image](https://user-images.githubusercontent.com/79100627/170852179-904b22b8-d619-4133-bc48-d14cede065ff.png)

We can see plenty of information; we will inspect them in the order displayed. First, we notice that we were redirected to ```whois.namecheap.com``` to get our information. In this case and at the time being, ```namecheap.com``` is maintaining the WHOIS record for this domain name. Furthermore, we can see the creation date along with the last update date and expiration date.

Next, we obtain information about registrar and registrant. We can find the registrant's name and contact information unless they are using some privacy service. Although not displayed above, we get the admin and tech contracts for this domain. Finally, we see the domain name servers that we should query if we have any DNS records to look up.

The information collected can be insepcted to find new attack surfaces, such as social engineering or technical attacks. For instance, depending on the scope of the penetration test, you might consider an attack against the email server of the admin user or the DNS servers, assuming they are owned by your client and fall within the scope of penetration test.

It is important to note that due to automated tools abusing WHOIS queries to harvest email addresses, many WHOIS services take measures against this. They might redact email addresses, for instance. Moreover, many registrants subscribe to privacy services to avoid their email addresses being harvested by spammers and keep their information private.

## NSLOOKUP and Dig 

In the previous task, we used WHOIS protocol to get various information about the domain name we were looking up. In particular, we were able to get the DNS servers from the registrar. 

Find the IP address of a domain name using ```nslookup```, which stands for Name Server Look Up. You need to issue the command ```nslookup DOMAIN_NAME```, for example, ```nslookup tryhackme.com```. Or, more generally, you can use ```nslookup OPTIONS DOMAIN_NAME SERVER```. The three main parameters are:

- OPTIONS contains the query type as shown in the table below. For instance, you can use ```A``` for IPv4 addresses and ```AAAA``` for IPv6 addresses. 
- DOMAIN_NAME is the domain name you are looking up. 
- SERVER is the DNS server that you want to query. You can choose any local or public DNS server to query. Cloudflare offers ```1.1.1.1``` and ```1.0.0.1```, Google offers ```8.8.8.8``` and ```8.8.4.4```, and Quad9 offers ```9.9.9.9``` and ```149.112.112.112```. There are many more public DNS servers that you can choose from if you want alternatives to your ISP's DNS servers. 

![image](https://user-images.githubusercontent.com/79100627/170885371-450769f6-2015-41a6-ab87-ac11c53a2474.png)

For instance, ```nslookup -type=A tryhackme.com 1.1.1.1``` (or ```nslookup -type=a tryhackme.com 1.1.1.1``` as it is case-insensitive) can be used to return all the IPv4 addresses used by tryhackme.come 

![image](https://user-images.githubusercontent.com/79100627/170885458-763f342c-2cca-489a-b752-2088d879d73b.png)

The A and AAAA records are used to return IPv4 and IPv6 addresses, respectively. This lookup is helpful to know from a penetration testing perspective. In the example above, we started with one domain name, and we obtained three IPv4 addresses. Each of these IP addresses can further checked for insecurities, assuming they lie when the scope of the penetration testing.

Let's say you want to learn about the email servers and configuration for a particular domain. You can issue ```nslookup -type=MX tryhackme.com```. Here is an example:

![image](https://user-images.githubusercontent.com/79100627/170885799-da2c80ba-9864-4583-898a-3cb51cd31fc2.png)

We can see that tryhackme.com's current email configuration uses Google. Since MX is looking up the MailExchange servers, we notice that when a mail server tries to deliver email ```@tryhackme.com```, it will try to connect to the ```aspmx.l.google.com```, which has order 1. If it is busy or unavailable, the mail server will attempt to connect to the next in order mail exchange servers, ```alt1.aspmx.l.googke.com``` or ```alt2.aspmx.l.google.com```. Google provides the listed mail servers; therefore, we should not expect the mail servers to be running a vulnerable server version. However, in other cases, we might find mail servers that are not adequately secured or patched. 

Such pieces of information might prove valuable as you continue the passive reconnaissance of your target. You can repeat similar queries fro other domain names and try different types, such as ```-type=txt```. Who knows what kind of information you might discover along your way 

For more advanced DNS queries and additional functionality, you can use ```dig```, the acronym for "Domain Information Groper". we can use ```dig DOMAIN_NAME```, but to specify the record type, we would use ```dig DOMAIN_NAME TYPE```. Optionally, we can select the server we want to query using ```dig@SERVER DOMAIN_NAME TYPE```.

- SERVER is the DNS server that you want to query 
- DOMAIN_NAME is the domain name you are looking up 
- TYPE contains the DNS record type, as shown in the table provided earlier

![image](https://user-images.githubusercontent.com/79100627/170886338-328dd966-846d-4dd3-be65-940b1023415c.png)

A quick comparison between the output of ```nslookup``` and ```dig``` shows that ```dig``` returned more information, such as the TTL (Time to Live) by default. If you want query a ```1.1.1.1``` DNS server, you can execute ```dig @1.1.1.1 tryhackme.com MX```.

## DNSDumpster

DNS lookup tools, such as nslookup and dig, cannot find subdomains on their own. The domain you are inspecting might include a different subdomain that can reveal much information about the target. For instance, if tryhackme.com has the subdomains wiki.tryhackme.com and webmail.tryhackme.com, you want to learn more about these two as they can hold a trove of information about your target. There is a possibility that one of these subdomains has been set up and is not updated regularly. Lack of proper regular updates usually leads to vulnerable services. But how can we know that such subdomain exists?

We can consider using multiple search engines to compile a list of publicly known subdomains. One search engine won't be enough; moreover, we should expect to go through at least tens of results to find interesting data. After all, you are looking for subdomains that are not explicitly advertised, and hence it is not necessary to make it to first page of search results. Another approach to discover such subdomains would be rely on brute-forcing queries to find which subdomains have DNS records. 

To avoid such a time-consuming search, one can use an online service that offers detailed answer to DNS queries, such as DNSDumpster. If we search DNSDumpster for ```tryhackme.com```, we will discover the subdomain ```blog.tryhackme.com```, which a typical DNS query cannot provide. In addition, DNSDumpster will return the collected DNS information in easy to read tables and graph. DNSDumpster will also provide any collected information about listening servers.

We will search for ```tryhackme.com``` on DNSDumpster to give you a glimpse of the expected output. Among the results, we got a list of DNS servers for the domain we are looking up. DNSDumpster also resolved the domain names to IP addresses and even tried to geolocate them. We can also see the MX records; DNSDumpster resolved all five mail exchange servers to their respective IP addresses and provided more information about the owner and location. 

![image](https://user-images.githubusercontent.com/79100627/170887007-7e569d90-faa5-415a-8422-f013874c3373.png)

DNSDumpster will also represent the collected information graphicaly. DNSDumpster displayed the data from the table earlier as a graph. You can see the DNS and MX branching to their respective server and also showing the IP addresses. 

## Shodan.io 

When you are tasked to run a penetration test against specific targets, as part of the passive reconnaissance phase, a service like Shodan.io can be helpful to learn various pieces of information about client's network, without actively connecting to it. Furthermore, on the defensive side, you can use different services from Shodan.io to learn about connected and exposed devices belonging to your organization.

Shodan.io tries to connect every device reachable online to build a search engine of connected "things" in contrast with a search engine for web pages. Once it gets a response, it collects all the information releated to the service and saves it in the database to make it searchable. Consider the saved record of one of tryhackme.com's servers. 

![image](https://user-images.githubusercontent.com/79100627/170887283-06757b0d-723f-45ab-8685-23fd1171f2e6.png)

This record shows a web server; however, as mentioned already, Shodan.io collects information related to any device it can find connected online. Searching for ```tryhackme.com``` on Shodan.io will display at least the record shown in the screenshot above. Via this Shodan.io search result, we can learn several things related to our search such as:

- IP address
- Hosting company
- Geographic location
- Server type and version 

You may also try searching for the IP addresses you have obtained from DNS lookups. There are, of course, more subject to change. On their help page, you can learn about all the search options available at Shodan.io, and you are encouraged to join TryHackMe's Shodan.io.

![image](https://user-images.githubusercontent.com/79100627/170887472-45688313-849a-4a5d-b511-c81b2da5efe1.png)

## Active Reconnaissance 

We learn to use a web browser to collect more information about our target. Moreover, we discuss using simple tools such as ```ping```, ```traceroute```, ```telnet```, and ```nc``` to gather information about the network, system, and services. 

Active reconnaissance requires you to make some kind of contact with your target. This contact can be a phone call or a visit to the target company under some pretence to gather more information, usually as part of social engineering. Alternatively, it can be direct connection to the target system, whether visiting their website or checking if their firewall has SSH port open. Think of it like you are closely inspecting windows and door locks. Hence, it is essential to remember not to engage in active reconnaissance work before getting signed legal authorization from the client. 

We focus on active reconnaissance. Active reconnaissance begins with direct connections made to the target machine. Any such connection might leave information in the logs showing the client IP address, time of the connection, and duration of the connection among other things. However, not all connections are suspicious. It is possible to let your active reconnaissance appear as regular client activity. Consider web browsing; no one would suspect a browser connected to a target web server among hundreds of other legitimate users. You can use such techniques to your advantage when working as part of the red team (attacker) and don't want to alarm the blue team (defenders).

## Web Browser 

The web browser can be a convenient tool, especially that it is readily available on all systems. There are several ways where you can use a web browser to gather information about a target. 

On the transport level, the browser connects to:
- TCP port 80 by default when website is accessed over HTTP
- TCP port 443 by default when the website is accessed over HTTPS

Since 80 and 443 are default ports for HTTP and HTTPS, the web browser does not show them in the address bar. However, it is possible to use custom ports to access a service. For instance, http://127.0.0.1:8834/ will connect to 127.0.0.1 (localhost) at port 8834 via HTTPS protocol. If there is an HTTPS server listening on that port, we will receive a web page.

While browsing a web page, you can press ```CTRL+SHIFT+I``` on a PC or ```Option + Command + I``` to open Developer Tools on Firefox. Similar shortcuts will also get you started with Google Chrome or Chromium. Developer Tools lets you inspect many things that your browser has received and exchanged with the remote server. For instance, you can view and even modify the JavaScript (JS) files, inspect the cookies set on your system and discover the folder structure of the site content. 

There are also plenty of add-ons for Firefox and Chrome that can help in penetration testing. Here are few examples:
- FoxyProxy: lets you quickly change the proxy server you are using to access the target website. This browser extension is convenient when you are using a tool such as Burp Suite or if you need to switch proxy servers regularly. 
- User-Agent Switcher and Manager: gives you the ability to pretend to be accessing the webpage from a different operating system or different web browser. In other words, you can pretend to be browsing a site using an iPhone when in fact, you are accessing it from Mozila Firefox. You can download User-Agent Switcher and Manager for Firefox here. 
- Wappalyzer: provides insights about the technologies used on the visited websites. Such extension is handy, primarily when you collect all this information while browsing the website like any other user. A screenshot of Wappalyzer is shown below

![image](https://user-images.githubusercontent.com/79100627/170897573-ef20fa39-79ff-465f-bd29-5aac07d73ef6.png)

## Ping 

In simple terms, the ping command sends a packet to a remote system, and the remote system replies. This way, you can conclude that the remote system is online and that the network is working between the two systems. If you prefer a pickier definition, the ping is a command that sends an ICMP Echo packet to a remote system. If the remote system is online, and the ping packet was correctly routed and not blocked by any firewall, the remote system should send back an ICMP Echo Reply. Similarly, the ping reply should reach the first system if appropriately routed and not blocked by any firewall. 

The objective of such a command is to ensure that the target system is online before we spend time carrying out more detailed scans to discover the running operating system and services. 

![image](https://user-images.githubusercontent.com/79100627/170897801-3d2560a8-8039-4031-84b0-621903145a3a.png)

In the example above, we saw clearly that the target system is responding. The ping output is an indicator that it is online and reachable. We have transmitted five packets, and we received five replies. We notice that, on average, it took 0.475 ms (millisecond) for the reply to reach our system, with the maximum being 0.636 ms. 

From a penetration testing point of view, we will try to discover more about this target. We will try to learn as much as possible, for example, which ports are open and which services are running.

Let's consider the following case: we shut down the target virtual machine and then tried to ping . As you would expect in the following example, we don;t recieve any reply.

![image](https://user-images.githubusercontent.com/79100627/170897948-1e968066-a515-4350-9f78-8d2e351bc47e.png)

In this case, we already know that we have shut down the target computer. For each ping, the system we are using, is responding with "Destination Host Unreachable". We can see that we have transmitted five packets, but none was received, resulting in a %100 packet loss. 

Generally speaking, when we don't get a ping reply back, there are a few explanations that would explain why we didn't get a ping reply, for example:

- The destination computer is not responsibe; possibly still booting up or turned off, or the OS has crahsed
- It is unplugged from the network, or there is a faulty network device accross the path.
- A firewall is configuredto block such packets. The firewall might be a piece of software running on the system itself or a separate network appliance. MS Windows firewall blocks ping by default.
- Your system is unplugged from the network. 

## Traceroute 

As the name suggests, the traceroute command traces the route taken packets from your system to another host. The purpose of a traceroute is to find the IP addresses of the routers or hops that a packet traverses as it goes from your system to a target host. This command also reveals the number of routers between the two systems. It is helpful as it indicates the number of hops (routers) between your system and the target host. However, note that the route taken by the packets might change as many routers use dynamic routing protocol that adapt to network changes. 

On Linux and macOS, the command to use is ```traceroute MACHINE_IP```, and MS Windows, it is tracert MACHINE_IP```. ```traceroute``` tries to discover router across the path from your system to the target system.

There is no direct way to discover the path from your system to a target system. We rely on ICMP to "trick" the routers into revealing their IP addresses. We can accomplish this by using a small Time To Live (TTL) in the IP header field. Although the T in TTL stands for time, TTL indicates the maximum number of routers/hops that packet can pass through before being dropped; TTL is not a maximum number of time units. When a router receives a packet, it decrements the TTL by one before passing it to the next router. The following figure shows that each time the IP packet passes through a router, its TTL value is decremented by 1. Initially, it leaves the system with a TTL value of 64; it reaches the target system with a TTL value of 60 after passing through 4 routers.

![image](https://user-images.githubusercontent.com/79100627/171254308-1a757212-04bd-40b3-ae52-488891134b16.png)

![image](https://user-images.githubusercontent.com/79100627/171254356-7c592c46-49f5-4a02-95a6-d3a6652c8b6f.png)

However, if the TTL reaches 0, it will droppped, and an ICMP Time-To-Live exceeded would be sent to the original sender. In the following figure, the system set TTL to 1 before sending it to the router. The first router on the path decrements the TTL by 1, resulting in a TTL of 0. consequently, this router will discard the packet and send an ICMP time exceeded in-transit error message. Note that some routers are configured not to send such ICMP messages when discarding a packet. 

![image](https://user-images.githubusercontent.com/79100627/171254967-e2e6182b-f1d3-4c2d-a8bb-7c232a08bca6.png)
![image](https://user-images.githubusercontent.com/79100627/171255008-16ceb2d9-d873-4217-a3e1-ecc88bc41546.png)

On Linux, ```traceroute``` will start by sending UDP datagrams within IP packets of TTL being 1. Thus, it causes the first router to you. Then it will send another packet with TTL=2; this packet will be dropped at the second router. And so on. Let's try this on live systems

In the following examples, we run the same command, ```traceroute tryhackme.com``` from TryHackMe's AttackBox. We notice that different runs might lead to different routes taken by the packets. 

Traceroute A

![image](https://user-images.githubusercontent.com/79100627/171255486-62c6cc5d-ee7e-4733-a114-d1c6c6864e8b.png)

In the traceroute output above, we have 14 numbered lines; each line represents one router/hop. Our system sends three packets with TTL set to 1, then three packets with TTL set to 2, and so forth. Depending on the network topology, we might get replies from up to 3 different routers, depending on the route taken by the packet. Consider line number 12, the twelfth router with the listed IP address has dropped the packet three times and sent an ICMP time exceeded in transit message. The line ```12, 99.83.69.207 (99.83.69.207) 17.603ms 15.827 ms 17.351 ms``` shows the time in millieseconds for each reply to reach our system. 

On the other hand, we can see that we recieved only a single reply on the third line. The two starts in the output ```3 * 100.66.16.176 (100.66.16.176) 8.006 ms * ``` indicate that our system didn't receive two expected ICMP time exceeded in transit messages.

Finally, in the first line of the output, we can see that the packets leaving the AttackBox take different rotues. We can see two routers that responded to TTL 

TraceRoute B

![image](https://user-images.githubusercontent.com/79100627/171257112-bbc15825-8047-4062-a8eb-3e6e64f2fac8.png)

In the second run of the traceroute program, we noticed that the packets took a longer route this time, passing through 26 routers. If you are running a traceroute to a system within your network, the route will be unlikely to chagne. However, we can not expect the route to remain fixed when the packets need to go via other routers outside our network.

To Summarize, we can notice the following:
- THe number of hops/routers between your system and the target system depends on the time you are running tracerroute. There is no guarantee that your packets will always follow the same route, even if you are on the same network or you repeat the traceroute command within a short time.
- Some routers return a public ip address. You might examine a few of these routers based on the scope of the intended penetration testing.
- Some routers don't return a reply.

## Telnet 

The TELNET (Teletype Network) protocol was developed in 1969 to communicate with a remote system via a command-line interface (CLI). Hence, the command ```telnet``` uses the TELNET protocol for remote administration. The default port used by telnet is 23. From a security perspective, ```telnet``` sends all the data, including usernames and passwords, in cleartext. Sending in cleartext makes it easy for anyone, who has access to the communication channel, to steal the login credentials. The secure alternative SSH (Secure SHell) protocol.

However, the telnet client, with its simplicity, can be used for other purposes. Knowing that telnet client relies on the TCP protocol, you can use Telnet to connect to any service and grab its banner. Using ```telnet MACHINE_IP Port```, you can connect to any service running on TCP and even exchange a few messages unless it uses encryption.

Let's say we want to discover more information about a web server, listening on port 80. We connect to the server at port 80, and then we communicate using the HTTP protocol. You don't need to dive into the HTTP protocol; you just need to issue ```GET/HTTP/1.1```. To specify something other that we want to use HTTP version 1.1 for communication. To get a valid response, insted of an error, you need to input some value for the host ```host: example``` and hit enter twice. Executing these steps will provide the requested index page. 

![image](https://user-images.githubusercontent.com/79100627/171259330-fbfa2d6b-5670-41fe-939d-e33120142699.png)

Of particular intrest for us is discovering the type and version of the installed web server, ```Server: nginx/1.6.2```. In this example, we communicated with a web server, so we used basic HTTP commands. If we connect to a mail server, we need to use proper commands based on the protocol, such as SMTP and POP3.

## Netcat 

Netcat or simply nc has different applications that can be great value to a pentester. Netcat supports both TCP and UDP protocols. It can function as client that connects to a listening port; alternatively, it can act as a server that listens on a port of your choice. Hence, it is a convenient tool that you can use as a simple client or server over TCP or UDP.

First, you can connect to a server,as you did with Telnet, to collect its banner using ```nc 10.10.154.156 PORT```, which is quite similar to our previous ```telnet 10.10.154.156 PORT```

![image](https://user-images.githubusercontent.com/79100627/171264014-474e8c97-00cd-45ca-b70b-9a4e28080750.png)

In the terminal shown above, we used netcat to connect to 10.10.154.156 port 80 using ```nc 10.10.154.156 80```. Next, we issued a get for the default page using ```GET / HTTP/1.1```; we are specifying to the target server that our client supports HTTP version 1.1. Finally, we need to give a name to our host, so we added on a new line, ```host: netcat```; you can name your host anything as this has no impact on this exercise.

Based on the output ```Server: nginx/1.6.2``` we received, we can tell that on port 80, we have Nginx version 1.6.2 listening for incoming connections. 

You can use netcat to listen on a TCP port and connect to a listening port on another system. 

On the server system, where you want to open a port and listen on it, you can issue ```nc -lp 1234``` or better yet, ```nc -vnlp 1234```, which is equivalent to ```nc -v -l -n -p 1234```, as you would remember from the Linux room, The exact order of the letters does not matter as long as the port number is preceded directly by ```-p```.

![image](https://user-images.githubusercontent.com/79100627/171265381-fb8db003-a442-4dfb-8e69-fa5a3fbae950.png)

Notes: 
- The option ```-p``` should appear just before the port number you want to listen on.
- The option ```-n``` will avoid DNS lookups and warnings. 
- port numbers less 1024 require root privileges to listen on. 

On the client-side, you would issue ```nc 10.10.154.156 PORT```. Here is an example of using ```nc``` to echo. After you successfully establish a connection to the server, whatever you type on the client-side will be echoed on the server-side and vice versa.

Consider the following example. On the server side, we will listen on port 1234. We can achieve this with the command ```nc -vnlp 1234``` (same as ```nc -lvnp 1234```). In our case, the listening server has the IP address ```10.10.154.156```, so we can connect to it from the client-side by executing ```nc 10.10.154.156 1234```. This setup would echo whatever you type on the other side of the TCP tunnel. You can find a recording of the process below.

## Putting All together 

![image](https://user-images.githubusercontent.com/79100627/171268464-cee3d800-d91a-470d-8dce-92c9cd285417.png)

## Nmap Live Host Discovery 

When we want to target a network, we want to find an efficient tool to help us handle repetitive tasks and answer the following questions:

1. Which systems are up?
2. What services are running on these systems?

The tool that we will reply on is Nmap. The first question about finding live computers is answered in this room. This room is the first in a series of four rooms dedicated to Nmap. The second question about discovering running services is answered in the next Nmap rooms that focus on port-scanning. 

We present the different approaches that Nmap uses to discover live hosts. In particular we cover:

1. ARP scan: This scan uses ARP requests to discover live hosts
2. ICMP scan: This scan ICMP requests to identify live hosts 
3. TCP/UDP ping scan: This scan sends packet to TCP ports and UDP ports to determine live hosts 

## Subnetworks 

Let's review a couple of terms before we move on the main tasks. A network segment is a group of computers connected using a shared medium. For instance, the medium can be the Ethernet switch of WiFi access point. In an IP network, a subnetwork is usually the equivalent of one or more network segments connected together and configured to use the same router. The network segment refers to a physical connection, while a subnetwork refers to a logical connection. 

In following network diagram, we have four network segments or subnetworks. Generally speaking, your system would be connected to one of these network segments/subnetworks. A subnetwork, or simply a subnet, has its own IP address range and is connected to a more extensive network via a router. There might be a firewall enforcing security policies depending on each network. 

![image](https://user-images.githubusercontent.com/79100627/171719993-e972d496-e71f-4bfe-a4a9-c90bd04d31d5.png)

The figure above shows two types of subnets:

- subnet with ```/16```, which means that subnet mask is ```255.255.0.0```. This subnet can have around 2^16 hosts 
- subnet with ```/24```, which indicates subnet mask is ```255.255.255.0``` This subnet can have around 2^8 hosts (-2 for 0.0.0.0 (network ID) 255.255.255.255 (broadcasting)

As part of active reconnaissance, we want to discover more information about a group of hosts or about a subnet. If you are connected to the same subnet, you would expect your scanner to rely on ARP (Address Resolution Protocol) queries to discover live hosts. An ARP query aims to get the hardware address (MAC address) so that communication over the link-layer becomes possible; however, we can use this to infer that the host is online. 

If you are in Network A, you can use ARP only to discover the devices within that subnet (10.1.100.0/24). Suppose you are connected to a subnet different from the subnet of the target system. In that case, all packets generated by your scanner will be routed via the default gateway to reach the systems on another subnet; however the ARP queries won't be routed and hence cannot cross the subnet router. ARP is a link-layer protocol, and ARP packets are bound to their subnet. 

## Enumerating Targets 

We mentioned the different techniques we can use for scanning in Task 1. Before we explain each in detail and put it into use against a live target, we need to specify the targets we want to scan. Generally speaking, you can provide a list, a range or a subnet. Examples of target specification are:

- List: ```MACHINE IP scanme.nmap.org example.com``` will scan 3 IP addresses. 
- range: ```10.11.12.15-20``` will scan 6 IP addresses: ```10.11.12.15```, ```10.11.12.13.16```,...and ```10.11.12.13.20```. 
- subnet: ```MACHINE_IP/30``` will scan 4 IP addresses

You can also provide a file as input for your list of targets, ```nmap -iL list_of_host.txt```.

If you want to check the list of hosts that Nmap will scan, you can use ```nmap -sL TARGETS```. This option will give you a detailed list of the hosts that Nmap will scan without scanning them; however, Nmap will attempt a reverse-DNS resolution on all the targets to obtain their names. Names might reveal various information to the pentester. (if you don't want Nmap to DNS server, you can add ```-n```)

![image](https://user-images.githubusercontent.com/79100627/171724732-3eb87172-498d-47be-b297-342a392a6039.png)

## Discovering Live Hosts

Let's revisit the TCP/IP layer shown in the figure next. We will leverage the protocols to discover the live hosts. Starting from bottom to top we can use 

- ARP from Link Layer
- ICMP from network layer
- TCP from Transport Layer
- UDP from Transport Layer

![image](https://user-images.githubusercontent.com/79100627/171725192-9738d706-1bb5-4f94-953d-0ff3ecad90f5.png)

Before we discuss how scanners can use each in deatil, we will brefly review these four protocols. ARP has one purpose: sending a frame to braodcast address on the network segment and asking the computer with a specific IP address to respond by providing its MAC (hardware) address. 

ICMP has many types. ICMP ping uses Type 8 (Echo) and Type 0 (Echo Reply) 

If you want to ping a system on the same subnet, an ARP request should precede the ICMP Echo. 

Although TCP and UDP are transport layers, for network scanning purposes, a scanner can send a specially-crafted packet to common TCP or UDP ports to check whether the target will respond. This method is efficient, especially when ICMP Echo is blocked. 

## Nmap Host Discovery Using ARP 

How would you know which hosts are up and running? It is essential to avoid wasting our time port-scanning an offline or an IP address not in use. There are various ways to discover online hosts. When no host discovery options are provided, Nmap follows the following approaches to discover live hosts:

1. When a prvileged user tires scan targets on a local network (Ethernet), Namp uses ARP requests. A privileged user is ```root``` or a user who belongs to ```sudoers``` and can run ```sudo```.
2. When a privileged user tries to scan target outside the local network, Nmap uses ICMP echo requests, TCP ACK to port 80, TCP SYN to port 443, and ICMP timestamp request. 
3. When an unprivileged user treis to scan targets outside the local network, Nmap resorts to a TCP 3-way handshake by sending SYN packets to port 80 and 443. 

Nmap, by defaultm uses a ping scan to find live hosts, then proceeds to scan live hosts only. If you want to use Nmap to discover online hosts without port-scanning the live system, you can issue ```nmap -sn TARGETS```. 

ARP scan is possible only if you are on the same subnet as the target systems. On a Ethernet (802.3) and WiFi(802.11), you need to know the MAC address of any system before you can communicate with it. The MAC address is necessary for the link-layer header; the haeader contains the source MAC address and destination MAC address among other fields. To get the MAC address, the OS sends an ARP query. A host that replies ARP quieres is up. The ARP queries only owkrs if the target is on the same subnet as yourself. You should expect to see many ARP queries generated during a Nmap scan of a local network. if you want Nmap only to perfrom an ARP scan without port-scanning, you can use ```nmap -PR -sn TARGETS```, where ```-PR``` indicates that you only want an ARP scan. The following example shows Nmap using ARP for host discovery without any port scanning. We run nmap -PR -sn MACHINE_IP/24 to discover all the live systems on the same subnet as our target machine. 

![image](https://user-images.githubusercontent.com/79100627/171728423-d3ee6894-d4b5-4ea8-8f7e-5d810ff223bc.png)

In this case, the AttackBox had the IP address 10.10.210.6 and it used ARP requests to discover the live hosts on the same subnet. ARP scan works, as shown in the figure below. Nmap sends ARP requests to all the target computers and those online should send an ARP reply back 

![image](https://user-images.githubusercontent.com/79100627/171728581-d0546f55-d7c6-4d64-9edb-1bff51bee778.png)

If you look at the packets generated using a tool such as tcpdump or Wireshark, we will see network traffic similar to the figure below. In the figure below, Wireshark displays the source MAC address, destination MAC address, protocol, and query related to each ARP request. The source address is the MAC address of our Attackbox, while the destination is the broadcast address as we don't know the MAC address of the target. However, we see the target's IP address, which appears in the Info column. In the figure, we can see that we are requesting the MAC addresses of all the IP addresses on the subnet. starting with ```10.10.210.1```. The host with the IP address we are asking about will send an ARP reply with its MAC address, and that's how we will know that it is online. 

![image](https://user-images.githubusercontent.com/79100627/171729078-094de4e4-ccff-46eb-930f-71b52897b477.png)

Talking about ARP scans, we should mention a scanner built around ARP queries: ```arp-scan```; it provides many options to customize your sacn. Visit the arp-scan wiki for detailed information. One popular choice is ```arp-scan --localnet``` or simply ```arp-scan -l```. This command will send ARP queries to all valid IP addresses on your local networks. Moreover, if your system has more than one interfae and you are intrested in discovering the live hosts on one of them, you can specify the interface using ```-I```. For instance, ```sudo arp-scan -I eth0 -l``` will send ARP queires for all valid IP addresses on the ```eth0``` inteface. 

Note that arp-scan is not installed on the AttackBox; however, it can be installed using ```apt install arp-scan```.

In the exmaple below, we canned the subnet of the AttackBox using ```arp-scan Attackbox_IP/24```. Since we ran this scan at a time frame close to the previous one ```nmap -PR -sn AttackBox_IP/24```, we obtain the same three live targets 

![image](https://user-images.githubusercontent.com/79100627/171729783-b4f9767c-9845-418f-b2cc-93f9162eb056.png)

Similarly, the command ```arp-scan``` will generate many ARP queries that we can see using tcpdump, Wireshark, or a similar tool. We can notice that the packet capture for ```arp-scan``` and ```nmap -PR -sn``` yield similar traffic patterns. 

## Nmap Host Discovery Using ICMP
 
 We can ping every IP address on a target network and see who would respond to our ```ping``` (ICMP Type8/Echo) 
 
 requests with a ping reply (ICMP Type 0). Simple isn't it? Although this would be the most straingtforward approach, it is not always reliable. Many firewalls block ICMP echo; New versions of MS Windows are configured with a host firewall that blocks ICMP echo requests by default. Remember that an ARP query will precede the ICMP request if your target is on the same subnet. 
 
 To use ICMP echo request to discover live hosts, add the option ```-PE```. (Remember to add ```-sn``` if you don't want to follow that with a port sacn.) As shown in the following figure, an ICMP echo scan works by sending an ICMP echo request and expects the target to reply with an ICMP echo reply if it is online. 
 
 ![image](https://user-images.githubusercontent.com/79100627/171732197-4cd3f2e6-2495-4da3-9d4f-253d017234ba.png)

In the example below, we canned the target's subnet using ```nmap -PE -sn MACHINE_IP/24```. This scan will send ICMP echo packets to every IP address on the subnet. Again, we expect live hosts to reply; however, it is wise to remember that many firewalls block ICMP. The output below shows the result of scanning the virtual machine's class C subnet using ```sudo nmap -PE -sn MACHINE_IP/24``` from the Attackbox.

![image](https://user-images.githubusercontent.com/79100627/171732623-9c8fbc4f-466a-45f3-a724-7f04aa46477f.png)

The scan output shows that eight hosts are up; moreover, it shows their MAC addresses. Generally Speaking, we don't expect to learn the MAC addresses of the targets unless they are on the same subnet as our system. The output above indicates that Nmap didn't need to send ICMP packets as it confirmed that these hosts are up based on the ARP responses it received. 

we will repeat the scan above; however, this time, we will scan from a system that belongs to a different subnet. The results are similar but without the MAC addresses. 

![image](https://user-images.githubusercontent.com/79100627/171733412-d303226c-592c-4794-8f18-22b7a1de7572.png)

If you look at the network packets using a tool like Wireshark, you will see something similar to the image below. You can see that we  have one source IP address on a different subnet than that of the destination subnet, sending ICMP echo requests to all the IP addresses in the target subnet to see which one will reply. 

![image](https://user-images.githubusercontent.com/79100627/171733920-3a48f11c-7970-4159-9ba1-b2f261557a13.png)

Because ICMP echo requests tend to be blocked, you might also consider ICMP Timestamp or ICMP Address Mask requests to tell if a system is online. Nmaps timestamp request (ICMP Type 13) and checks whether it will get a Timestamp reply (ICMP Type 14). Adding ```-PP``` option tells Nmap to use ICMP timestamp requests. As shown in the figure below, you expect live hosts to reply. 

![image](https://user-images.githubusercontent.com/79100627/171734943-482acc7b-32e8-4d24-a547-aab5ca42e167.png)

In the following example we run ```nmap -PP -sn Machine_IP/24``` to discover the online computers on the target machine subnet. 

![image](https://user-images.githubusercontent.com/79100627/171735133-69dbe5b8-abf8-423b-8dda-7ba66e89644c.png)

Similar to the previous ICMP scan, this scan will send many ICMP timestamp requests to every valid IP address in the target subnet. In the Wireshark screenshot below, you can see one source IP address sending packets to every possible IP address to discover online hosts.

![image](https://user-images.githubusercontent.com/79100627/171735333-d2d6d409-25b9-4d42-a8aa-059ea36d1964.png)

Similarly, Nmap uses address mask queries (ICMP Type 17) and checks whether it gets an address mask reply (ICMP Type 18). This scan can be enabled with the option ```-PM``` . As shown in the figure below, live hosts are expected to reply to ICMP address mask requests. 

![image](https://user-images.githubusercontent.com/79100627/171735569-dcea06d9-37d6-4ed2-8430-9dc2722f67d3.png)

In attempt to discover live hosts using ICMP address mask queries, we run the command ```nmap -PM -sn MACHINE_IP/24```. Although based on earlier scans, we know that at least eight hosts are up, this scan returned none. The reason is that the target system or a firewall on the route is blocking this type of ICMP packet. Therefore, it is essential to learn multiple approaches to achieve the same result. If one type of packet is being blocked, we can always choose another to discover the target network and services. 

![image](https://user-images.githubusercontent.com/79100627/171735946-f3114dbb-88dc-46ed-82b3-ec6c52f675cc.png)

Although we didn't get any reply and could not figure out which hosts are online, it is essential to note that this scan sent ICMP address mask requests to every valid IP address and waited for a reply. Each ICMP request was sent twice as we can see in the screenshot below.

## TCP SYN Ping 

We can send a packet with the SYN (Synchronize) flag set to a TCP port, 80 by default, and wait for response. An open port should reply with at SYN/ACK (Acknowledge); a closed port would result in an RST(Reset). In this case, we only check whether we will get any response to infer whether the host is up. The specific state of the port is not significant here. The figure below is reminder of how a TCP 3-way handshake usually works.

![image](https://user-images.githubusercontent.com/79100627/171737644-302506d5-c02f-4b7b-b695-7d9bebdab0e6.png)

If you want to Nmap to use TCP SYN ping, you can do so via the option ```-PS``` followed by the port number, range list, or a combination of them. For example ```-PS21``` will target port 21, while ```-PS21-25``` will target port 21,22,23,24,25. Finally ```-PS80,443,8080``` will target three ports: 80,443 and 8080. 

Privileged users (root and sudoers) can send TCP/SYN packets and don't need to complete the TCP 3-way handshake even if the port is open, as shown in the figure below. Unprivileged users have no choice but to complete the 3-way handshake if the port is open 

![image](https://user-images.githubusercontent.com/79100627/171738017-0a7f9e23-e454-4037-a80d-c1fd874122d5.png)

We will run ```nmap -PS -sn MACHINE_IP/24``` to scan the target VM subnet. 

![image](https://user-images.githubusercontent.com/79100627/171738157-917d336d-b779-4af5-8243-a1067408eb01.png)

Let's take a closer look at what happened behind the scenes by looking at the network traffic on Wireshark in the figure below. Technically speaking, since we dind't specify any TCP ports to use in the TCP ping scan, Nmap used common ports; in this case, it is TCP port 80. Any service listening on port 80 is expected to reply, indirectly indicating that the host is online. 

![image](https://user-images.githubusercontent.com/79100627/171738598-50aae544-2cdc-4d61-8f3b-1465027452c5.png)

## TCP Ack Ping 

As you have guessed, this sends a packet with an ACK flag set. You must be running Nmap as a privileged user to be able to accomplish this. If you try it as an unprivileged user, Nmap will attempt a 3-way Handshake.

By default, port 80 is used. The syntax is similar to TCP SYN ping. ```-PA``` should be followed by a port number,range,list or a combination of them. For example, consider ```-PA21```, ```-PA21-25``` and -PA80,443,8080```. If no port is specified, port 80 will be used. 

The following figure shows that any TCP packet with an ACK flag should get a TCP packet back with an RST flag set. The target responds with the RST flag set because the TCP packet with the ACK flag is not part of any ongoing connection. The expected response is used to detect if the target host is up. 

The following figure shows that any TCP packet with an ACK flag should get a TCP packet back with an RST flag set. The target responds with the RST flag set because the TCP packet with the ACK flag is not part of any ongoing connection. The expected reponse is used to detect if the target host is up. 

![image](https://user-images.githubusercontent.com/79100627/171739514-edeacc74-8959-40a5-a7e9-03205129fdc4.png)

we can ```sudo nmap -PA -sn MACHINE_IP/24``` to discover the online hosts on the target's subnet. 

![image](https://user-images.githubusercontent.com/79100627/171739604-c4266829-4598-4952-bca0-b52dd9e90bf8.png)

If we peek at the network traffic, we will discover many packets with the ACK flag set and sent to port 80 of the target. Nmap sends each packet twice. The system don't respond are offilne or inaccessible.

![image](https://user-images.githubusercontent.com/79100627/171739747-fc27e187-5e0a-42c5-a8f5-98c16edd4330.png)

## UDP Ping 

Finally we can use UDP to discover if the host is online. Contrary to TCP SYN ping, sending UDP packet to an open port is not expected to lead to any reply. However, if we send a UDP packet to a closed UDP packet, we expect to get an ICMP port unreachable packet; this indicates that the targetsystem is up and available. 

In the following figure, we see a UDP packet sent to an open UDP port and not triggering any response. However, sending a UDP packet to any closed UDP port can trigger a response indirectly indicating that the target is online. 

![image](https://user-images.githubusercontent.com/79100627/171740056-960abe15-22d4-4404-83cd-a3b503bdc6be.png)

The syntax to specify the port is similar to the TCP SYN ping and TCP ACK; Nmap uses ```-PU``` for UDP ping. In the following example, we use UDP scan, and we discover five live hosts. 

![image](https://user-images.githubusercontent.com/79100627/171740181-85eedc56-76aa-4167-b62c-59a71d36ef9d.png)

Let's inspect the UDP packets generated. In the following Wireshark screenshot, we notice Nmap sending UDP packets to UDP ports that are most likely closed . The Nmap uses an uncommon UDP port to trigger an ICMP destination unreachable (port unreachable error). 

![image](https://user-images.githubusercontent.com/79100627/171740686-cecb4a86-90be-4389-a503-038d738ff32c.png)

## USING Reverse - DNS Look Up

Nmap's default behaviour is to use reverse-DNS online hosts. Because the hostnames can reveal a lot, this can be a helpful step. However, if you don't want to send such DNS queries, you use ```-n``` to skip this step.

By defult Nmap will look up online host; however, you can use option ```-R``` to query the DNS server even for offline hosts. If you want to use specific DNS server, you can add the ```--dns-servers DNS_SERVER``` option.


## Summary 

![image](https://user-images.githubusercontent.com/79100627/171741488-187534bf-7b38-4ec4-810e-b10b6bd96e28.png)
![image](https://user-images.githubusercontent.com/79100627/171741526-ee397509-9ecf-4efc-a393-6a39461b61f4.png)


## TCP and UDP ports 

In the same sense that an IP address specifies a host on a network among many others, a TCP port or UDP port is used to identify a network service running on that host. A server provides the network service, and it adheres to a specific network protocol. Examples include providing time, responding to DNS queries, and serving web pages. A port is usually linked to a service using that specific port number. For instance, an HTTP server would bind to TCP port 80 by default; moreover, if the HTTP server supports SSL/TLS, it would listen to TCP port 443. (TCP port 80 and 443 are default ports for HTTP and HTTPS; however, the webserver administrator might choose other port numbers if necessary.) Furthermore, no more than one service can listen on any TCP or UDP port (On the same IP address). 

At the risk of oversimplification, we can classify ports in two states:

1. Open port indicates that there is some service listening on that port.
2. Closed port indicates that there is no service listening on that port. 

However, in practical situations, we need to consider the impact of firewalls. For instance, a port might be open, but a firewall might be blocking the packets. Therefore, Nmap considers the following six states:

1. Open: indicates that a service is listening on the specified port. 
2. Closed: indicates that no service is listening on the specified port, although the port is accessible. By accessible, we mean that it is reachable and is not blocked by a firewall or other security appliances/programs. 
3. Filtered: means that Nmap cannot determine if the port is open or closed because the port is not accessible. This state is usually due to a firewall preventing Nmap from reaching that port. Nmap's packets may be blocked from reaching the port. Alternatively, the responses are blocked from reaching Nmap's host. 
4. Unfiltered: means that Nmap cannot determine if the port is open or closed although the port is accessible. This state is encountered when using an ACK scan ```-sA```
5. Open|Filtered: This means that Nmap cannot determine whether the port is open or filtered
6. Closed|Filtered: This means that Nmap cannot decide whether a port is closed or filtered

DNS - UDP Port 53
SSH - TCP Port 22

## TCP Flags 

Nmap supports different types of TCP port scans. To understand the difference between these port scans, we need to review the TCP header. The TCP header is the first 24 bytes of a TCP segment. The following figure shows the TCP header as defined in RFC 793. This figure looks sophisticated at first; however, it is pretty simple to understand. In the first row, we have the source TCP port number and the destination port number. We can see that the port number is allocated 16 bits (2bytes). In the second and third rows, we have the sequence number and the acknowledgement number. Each row has 32 bits (4 bytes) allocated, with six rows total making up 24 bytes

![image](https://user-images.githubusercontent.com/79100627/171971524-9ca25151-eade-4dc8-99aa-e85d8291533b.png)

In particular, we need to focus on the flags that Nmap can set or unset. We have highlighted the TCP flags in red. Setting a flag bit means setting its value to 1. From left to right, the TCP header flags are: 

1. URG: Urgent flag indicates that the urgent pointer filed is significant. The urgent pointer indicates that incoming data is urgent, and that a TCP segment with URG flag set is processed immediately without consideration of having to wait on previously sent TCP segments. 
2. ACK: Acknowledgement flag indicates that the acknowledgement number is significant. It is used to acknowledge the recipet of a TCP segment. 
3. PSH: Push flag asking TCP to pass the data to the application promptly.
4. RST: Reset flag is used to reset the connection. Another device, such as a firewall, might send it to tear a TCP connection. This flag is also used when data is sent to a host and there is no service on the receiving end to answer. 
5. SYN: Synchronize flag is used to initiate a TCP-3way handshake and synchronize sequence numbers with the other host. The sequence number should be set randomly during TCP connection establishment.
6. FIN: The sender has no more data to send. 

## TCP Connection Scan 

TCP connect scan works by completing the TCP- 3way handshake. In standard TCP connection establishment, the client sends a TCP packet with SYN flag set, and the server responds with SYN/ACK if the port is open; finally, the client completes the 3-way handshake by sending an ACK.

![image](https://user-images.githubusercontent.com/79100627/171972065-3def5370-c3cb-47f1-97a9-7fb6c26f8b2f.png)

We are interested in learning whether the TCP port is open, not establishing a TCP connection. Hence the connection is torn as soon as its state is confirmed by sending a RST/ACK. You can choose to run TCP connect scan using ```-sT```.

![image](https://user-images.githubusercontent.com/79100627/171972147-0e46543f-d521-42c8-aeb4-a239e09718d4.png)

It is important to note that if you are not a privileged user (root or sudoer), a TCP connect scan is the only possible option to discover open TCP ports. 

In the following Wireshark packet capture window, we see Nmap sending TCP packets with SYN flag set to various ports, 256, 443, 143, and so on. By default, Nmap will attempt to connect to the 1000 most common ports. A closed TCP port responds to a SYN packet with RST/ACK to indicate that it is not open. This pattern will repeat for all the closed ports as we attempt to initiate a TCP 3-way handshake with them.

![image](https://user-images.githubusercontent.com/79100627/171972350-2a6a107a-db32-42d3-8f6a-461a49310226.png)

We notice that port 143 is open, so it replied with a SYN/ACK, and Nmap completed the 3-way handshake by sending an ACK. The figure below shows all the packets exhcanged between our Nmap host and the target system's port 143. The first three packets are the TCP 3-way handshake being completed. Then, the fourth packet tears it down with an RST/ACK packet. 

![image](https://user-images.githubusercontent.com/79100627/171972462-7051e424-b036-4ac8-8eda-c551e24da05c.png)

To illustrate the ```-sT``` (TCP connect scan), the following command example returned a detailed list of the open ports. 

![image](https://user-images.githubusercontent.com/79100627/171972510-572ee954-c0b0-4e0e-a1a7-e6d2111c6d41.png)

Note that we can use ```-F``` to enable fast mode and decrease the number of scanned ports from 1000 to 100 most common ports.

It is worth mentioning that the ```-r``` option can also be added to scan the ports in consecutive order instead of random order. This option is useful when testing whether ports open in a consistent manner, for instance, when a target boots up. 

## TCP SYN Scan 

Unprevileged users are limited to connect scan. However, the default scan mode is SYN scan, and it requires a privileged (root or sudoer) user to run it. SYN scan does not need to complete the TCP 3-way handshake; instead, it tears down the connection once it receives a response from the server. Because we didn't establish a TCP connection, this decreases the chances of the scan being logged. We can select this scan type by using the ```-sS``` option. The figure below shows thow the TCP SYN scan works without completing the TCP 3-way handshake. 

![image](https://user-images.githubusercontent.com/79100627/171973779-f08cd2d5-df10-488c-9678-222fddd15933.png)

The following screenshot from Wireshark shows a TCP SYN scan. The behaviour in the case of closed TCP ports is similar to that of the TCP connect scan.

![image](https://user-images.githubusercontent.com/79100627/171973945-ca9163aa-d6d7-4e55-9d66-6bf9e82e2267.png)

To better see the difference between the two scans, consider the following screenshot. In the upper half of the following figure, we can see a TCP connect scan ```-sT``` traffic. Any open TCP port will require Nmap to complete the TCP 3-way handshake before closing the connection. In the lower half of the following figure, we see how a SYN scan ```-sS``` does not need to complete TCP 3-way handshake; instead Nmap sends an RST packet once a SYN/ACK packet is received. 

![image](https://user-images.githubusercontent.com/79100627/171974350-bb30abdf-8bba-428c-83e3-739947e8704e.png)

TCP SYN scan is the default scan mode when running Nmap as a privileged user, running as root or using sudo, and it is very reliable choice. It has successfully discovered the open ports you found earlier with the TCP connect scan, yet no TCP connection was fully established with the target 

![image](https://user-images.githubusercontent.com/79100627/171974400-c56307a3-4d2d-46dc-9575-5733e20ff78f.png)

## UDP Scan

UDP is a connectionless protocol, and hence it does not require any handshake for connection establishment . We cannot guarantee that a service listening on a UDP port would respond to our packets. However, if a UDP packet is sent to a closed port, an ICMP port unreachable error (type 3, code 3) is returned. You can select UDP scan using the ```-sU``` option; moreover, you can combine it with another TCP scan. 

The following figure shows that if we send a UDP packet to an open UDP port, we cannot expect any reply in return. Therefore, sending a UDP packet to an open port won't tell us anything 

![image](https://user-images.githubusercontent.com/79100627/171974576-2e80b603-1af5-4d51-aa44-424d0b5b2407.png)

However, as shown in the figure below, we expect to get an ICMP packet of type 3, destination unreachable, and code 3, port unreachable. In otherwords, the UDP ports that don't generate any response are the ones that Nmap will state as open.

![image](https://user-images.githubusercontent.com/79100627/171974617-87063d74-fdb8-40ea-9a71-86e0c9d91c9f.png)

In the Wireshark capture below, we can see that every closed port will generate an ICMP packet destination unreachable (port unreachable) 

![image](https://user-images.githubusercontent.com/79100627/171974641-716d6005-27d2-4e35-92a5-69a3200929d0.png)

Launching a UDP scan against this Linux server proved valuable, and indeed, we learned that port 111 is open. On the other hand, Nmap cannot determine whether UDP port 68 is open or filtered. 

![image](https://user-images.githubusercontent.com/79100627/171974866-95c96294-4fbf-4e56-8c48-fc77ff1f07e9.png)

## Fine-Tuning Scope and Performance 

You can specify the ports you want to scan instead of the default 1000 ports. Specifying the ports is intuitive by now. Let's see some examples:

- Ports list: ```-p22,80,443``` will can port 22, 80, and 443.
- Port range: ```-p1-1023``` will scan all ports between 1 and 1023 inclusive, while ```-p20-25``` will scan ports between 20 and 25 inclusive. 

You can request the scan of all ports by using ```-p-```, which will scan all 65535 ports. If you want to scan the most common 100 ports, add ```-F```. Using ```--top-ports 10``` will check the ten most common ports.

You can control the scan timing using ```-T<0-5>```, ```-T0``` is the slowest (paranoid), while ```-T5``` is the fastest. According to Nmap manual page, there are six templates:

- Paranoid (0)
- Sneaky (1)
- Polite (2)
- Normal (3) 
- Aggressive (4)
- Insane (5) 

To avoid IDS alert, you might consider ```-T0``` or ```-T1```. For instance, ```-T0``` scans one port at a time and waits 5 minutes between sending each probe, so you can guess how how long scanning one target would take to finish. If you don't specify any timing, Nmap uses normal ```-T3```. Note that ```-T5``` is the most aggressive in terms of speed; howeverm, this can affect the accurarcy of the sacn results due to the increased likelyhood of packet loss. Note that ```-T4``` is often used during CTFs and when learning to scan on practice targets, where as ```-T1``` is often used during real engagements where stealth is more important. 

Alternatively, you can choose to control the packet rate using ```--min-rate <number>``` and ```-max-rate<number>```. For example, ```--max-rate 10``` or ```--max-rate=10``` ensures that your scanner is not sending more than ten packets per second.

Moreover, you can control probing parallelization using ```-min-parallelism <numprobes>``` and ```--max-parallelism <numprobes>```. Nmap probes the targets to discover which hosts are live and which ports are open; probing parallelization specifies the number of such probes that can be run in parallel. For instance, ```--min-parallelism=512``` pushes Nmap to maintain at least 512 probes in parallel; these 512 probes are related to host discovery and open ports 

## Summary

![image](https://user-images.githubusercontent.com/79100627/171976709-2a6ca0c6-9c01-4860-a93c-4b926b9fe444.png)
![image](https://user-images.githubusercontent.com/79100627/171976718-a9a6d4e3-177d-4274-9fc5-e31529ff9e8f.png)

## TCP Null Scan, FIN Scan and Xmas Scan

### NULL Scan 

The null scan does not set any flag; all six flag bits are set to zero. You can choose this scan using ```-sN``` option. A TCP Packet with no flags set will not trigger any response when it reaches an open port, as shown in the figure below. Therefore, from Nmap's perspective, a lack of reply in a null scan indicates that either the port is open or firewall is blocking the packet. 

![image](https://user-images.githubusercontent.com/79100627/172093226-9b8fc8b4-a7f9-49c5-8998-6085502f9518.png)

However, we expect the target server to respond with an RST packet if the port is closed. Consequently, we can use the lack of RST response to figure out the ports that are not closed: open or filtered.

![image](https://user-images.githubusercontent.com/79100627/172093380-362b494f-18c2-4f5a-af1c-ef60f3346215.png)

Below is an example of a nul scan against a Linux server. The null scan we carried out has successfully identified the six open ports on the target system. Beacuse the null scan relies on the lack of response to infer that the port is not closed, it cannot indicate with certainty that these  ports are open; there is a possibility that the ports are not responding due to a firewall rule. 

![image](https://user-images.githubusercontent.com/79100627/172093884-50772ebe-0b20-40bd-baba-45d6b1d1344b.png)

### FIN scan 

The FIN scan sends a TCP packet with the FIN flag set. You can choose this scan type using the ```-sF``` option. Similarly, no response will be sent if the TCP port is open. Again, Nmap cannot be sure if the port is open or if a firewall is blocking the traffic related to this TCP port. 

![image](https://user-images.githubusercontent.com/79100627/172094090-504b1b13-0b41-4eed-a897-4d99d4b4848b.png)

However, the target system should respond with RST if the port is closed. Consequently, we will be able to know which port are closed and use this knowledge to infer the ports that are open or filtered. It's worth nothing some firewalls will `silently` drop the traffic without sending RST. 

![image](https://user-images.githubusercontent.com/79100627/172094265-1b12393b-8c3b-43b9-8c62-e3f663814898.png)

![image](https://user-images.githubusercontent.com/79100627/172095526-6341b02b-fcfa-41be-8689-a0715ae40fdf.png)

### Xmas Scan

The Xmas scan gets its name after Chirstmas tree lights. An Xmas scan sets the FIN,PSH, and URG flags simultaneously. You can select Xmas scan with the option ```-sX```. Like the Null scan and FIN scan, if an RST packet is received, it means that the port is closed. Otherwise, it will be reported as open|filtered.

![image](https://user-images.githubusercontent.com/79100627/172095662-7676dec8-ec5b-4e90-bbd9-60e0ca874a17.png)

![image](https://user-images.githubusercontent.com/79100627/172096021-8ad2afd9-ec3d-4d95-b61b-65d1b53fdb5b.png)

## TCP Maimon Scan 

Uriel Maimon first described this scan in 1996. In this scan, the FIN and ACK bits are set. Th target should send an RST packet as a response. However, certain BSD-derived systems drop the packet if it is an open port exposing the open ports. This scan won't work on most targets encountered in modern networks; however, we include it in this room to better understand the port scanning mechanism and the hacking mindset. To select this scan type use the ```-sM``` option.

Most target systems respond with an RST packet regardless of whether the TCP port is open. In such a case, we won't discover the open ports. The figure below shows the expected behaviour in the cases of both open and closed TCP ports. 

![image](https://user-images.githubusercontent.com/79100627/172096394-e6931ac7-7cf3-43bd-9747-4a83ed0c6b6d.png)

## TCP ACK, Window and Custom Scan 

### TCP ACK Scan 

an Ack scan will send a TCP packet with ACK flag set. use ```-sA``` option to choose this scan. As we show in the figure below, the target would respond to the ACK with RST regardless of the state of the port. This behaviour happnes because a TCP pacekt with the ACK flag set should be sent only in response to received TCP packet to acknowledge the receipt of some data, unlike our case. Hence this scan won't tell us whether the target port is open in a simple set up. 

![image](https://user-images.githubusercontent.com/79100627/172096724-1abbc34d-03f4-438a-bb72-a506d0638f64.png)

![image](https://user-images.githubusercontent.com/79100627/172097770-866a1830-2729-46f5-a29d-bd63a3d938e5.png)

This kind of scan would be helpful if there is a firewall in front of the target. Consequently, based on which ACK packets resulted in responses, you will learn which ports were not blocked by firewall. In other words, this type of scan is more suitable to discover firewall rule sets and configuration.

After setting up the target VM ```IP_ADDRESS``` with a firewall, we repeated the ACK scan. This time, we received some interesting results. As seen in the console output below, we have three ports that aren't being blocked by firewall. This result indicates that the firewall is blocking all other ports except for these three ports. 


![image](https://user-images.githubusercontent.com/79100627/172097770-866a1830-2729-46f5-a29d-bd63a3d938e5.png)

### Window Scan

Another similar scan is the TCP window scan. The TCP window scan is almost the same as the ACK scan; however, it examines the TCP Window field of the RST packets returned. On specific systems, this can reveal that the port is open. You can select this scan type with the option ```-sW```. As shown in the figure below, we expect to get an RST packet in reply our "uninvented" ACK packets, regardless of whether the port is open or closed 

![image](https://user-images.githubusercontent.com/79100627/172098336-2d069870-6cbb-42ff-995a-1d818301132c.png)

However, as you would expect, if we repeat our TCP window scan against a server behind a firewall, we expect to get more satisfying results. In the console output shown below, the TCP window scan pointed that three ports are detected as closed. (This is in contrast with ACK scan that labelled the same three ports as unfiltered). Although we know that these three ports are not closed, we realize they responded differently, indicating that firewall does not block them. 

![image](https://user-images.githubusercontent.com/79100627/172098968-d6dcffc7-fea9-4ee2-8c91-a0ceadd975b0.png)

### Custom Scan 

If you want to experiment with a new TCP flag combination beyond the built in TCP scan type, you can do so using ```--scanflags```. For instance, if you want to set SYN,RST, and FIN simultaneously, you can do so using ```--scanflags RSTSYNFIN```. As shown in the figure below, if you develop your custom scan, you need to know how the different ports will behave to interpret the results in different scenarios correctly. 

![image](https://user-images.githubusercontent.com/79100627/172099184-562eb9cf-7606-4b16-818e-e67349591263.png)

Finally, it is essential to note that the ACK scan and window scan were very efficient at helping us map out the firewall rules. However, it is vital to remember that just because a firewall is not blocking a specific port, it does not necessarily mean that a service is listening on that port. For example, there is a possibility that the firewall rules need to be updated to reflect recent service cahanges, hence ACK and window scans are exporsing the firewall rules, not the service. 

## Spoofing and Decoys 

In some network setups, you will be able to scan a target system using a spoofed IP address and even a spoofed MAC address. Such a scan is only beneficial in a situation where you can guarantee to capture the response. If you try to scan a target from some random network using a spoffed IP address, chaces are you won't have any response routed to you, and the scan results could be reliable.

The following figure shows the attacker launching the command ```nmap -S SPOOFED_IP MACHINE_IP```. Consequently, Nmap will craft all the packets using the provided source IP address ```SPOOFED_IP```. The target machine will respond to the incoming packets sending the replies to the destination IP address ```SPOOFED_IP```. For this scan to work and give accurate results, the attacker needs to monitor the network traffic to analyze the replies. 

![image](https://user-images.githubusercontent.com/79100627/172279233-75b934ac-1245-4b4d-8b55-ac0eccb1809d.png)

In brief, scanning with a spoofed IP address is three steps:
1. Attacker sends a packet with a spoofed IP address to the target machine 
2. Target machine replies to the spoofed IP address as the destination.
3. Attacker captures the replies to figure out open ports.

In general, you expect to specify the network interface using ```-e``` and to explicitly disable ping scan ```-Pn```. Therefore, instead of ```nmap -S SPOOFED_IP MACHINE_IP```, you will need to issue ```nmap -e NET_INTERFACE -Pn -S SPOOFED_IP MACHINE_IP``` to tell Nmap explicitly which network interface to use and not to expect to recieve a ping reply. It is worth repeating that this scan will be useless if the attacker system cannot monitor the network for responses. 

When you are on the same subnet as the target machine, you would be able to spoof your MAC address as well. You can specify the source MAC addressing using ```--spoof-mac SPOOFED_MAC```. This address spoofing is only possible if the attacker and the target machine are on the same Ethernet (802.3) network or same WiFi (802.11). 

Spoofing only works in a minimal number of cases where certain conditions are met. Therefore, the attacker might resort to using ```--spoof-mac SPOOFED_MAC```. This address spoofing is only possible if the attacker and the target machine are on the same Ethernet (802.3) network or same WiFi (802.11).

Spoofing only works in a minimal number of cases where certain conditions are met. Therefore, the attacker might resort to using decoys to make it more challenging to be pinpointed. The concept is simple, make the scan appears to be coming from many IP addresses so that the attacker's IP address would be lost among them. As we see in the figure below, the scan of the target machine will appear to be coming from 3 different sources, and consequently, the replies will go the decoys as well. 

![image](https://user-images.githubusercontent.com/79100627/172281351-e7bdd60d-d3c1-4ca7-be77-3d4f1afbdca2.png)

You can launch a decoy scan by specifying a specific or random IP address after ```-D```. For example, ```nmap -D 10.10.0.1,10.10.0.2,ME MACHINE_IP``` will make the scan. For example, ```nmap -D 10.10.0.1, 10.10.0.2, ME MACHINE_IP will make the sacn of MACHINE_IP appear as comming from the IP address 10.10.0.1, 10.10.0.2, and then ```ME``` to indicate that your IP address should appear in the thrid order. Another example command would be ```nmap -D 10.10.0.1, 10.0.0.2, RND, RND, ME MACHINE_IP, where the thrid and fourth source IP addresses are assigned randomly, while the fifth source is going to be attacker's IP address. In other words, each time you execute the latter command, you would expect two new random IP address to be the thrid and fourth decoy sources. 

## Fragmented Packets 

### Firewall

A firewall is a piece of software or hardware that permits packets to pass through or blocks them. It functions based on firewall rules, summarized as blocking all traffic with exceptions or allowing all traffic with exceptions. For instance, you might block all traffic to your server except those coming to your web server. A traditional firewall inspects, at least IP Header and the transport layer header. A more sophisticated firewall would also try to examine the data carried by the transport layer. 

### IDS

An intrusion detection system (IDS) inspects network packets for select behavioural patterns or specific content signatures. It raises an alert whenever a malicious rule is met. In addition to the IP header and transport layer header, an IDS would inspect the data contents in the transport layer and check if it matches any malicious patterns. How can you make it less likely for a traditional firewall/IDS to detect your Nmap activity? It is not easy to answer this; however, depending on the type of firewall/IDS, you might benefit from dividing the packet into smaller packets

### Fragmented Packets

Nmap provides the option ```-f``` to fragment packets. Once chosen, the IP data will be divdided into 8 bytes or less. Adding another ```-f```(```-f -f``` or ```-ff```) will split the data into 16 byte-fragments instead of 8. You can change the default value using the --mtu; however, you should always choose a multiple of 8.

To properly understand fragmentation, we need to look at the IP header in the figure below.Notice the source address taking 32 bits (4 bytes) on the fourth row, while the destination address is taking another 4 bytes on the fifth row. The data that we will fragment across multiple packets is highlighted in red. To aid in the reassembly n recipient side, IP uses the identification (ID) and fragment offset, shown on the second row of the figure below.

![image](https://user-images.githubusercontent.com/79100627/172438678-41ae5f19-b6cb-46be-a71f-d81171c5a924.png)

Let's compare running ```sudo nmap -sS -p80 10.20.30.144``` and ```sudo nmap -sS -p80 -f 10.20.30.144```. As you know by now, this will use stealth TCP SYN scan on port 80; however, in the second command, we are requesting Nmap to fragment the IP packets.

In the first two lines, we can see an ARP query and response. Nmap issued an ARP query because the target is on the same Ethernet. The second two lines show a TCP SYN ping and a reply. The fifth line is the beginning of the port scan; Nmap sends a TCP SYN packet to port 80. In this case, the IP header is 20 bytes, and the TCP header is 24 bytes. Note that the minimum size of the TCP header is 20 bytes.

![image](https://user-images.githubusercontent.com/79100627/172439830-aff8133e-7773-4646-a778-d15a659773e1.png)

With fragmentation requested via ```-f```, the 24 bytes of the TCP header will be divided into multiple of 8 bytes, with the last fragment containing 8 bytes or less of the TCP header. Since 24 is divisible by 8, we got 3 IP fragments; each has 20 bytes of IP header and 8 bytes of TCP header. We can see the three fragments 

![image](https://user-images.githubusercontent.com/79100627/172440168-8d49cd37-acca-4477-8dc9-b18ac7531acc.png)

Note that if added ```-ff``` (or ```-f -f```), the fragmentation of the data will be multiples of 16. In other words, the 24 bytes of the TCP header, in this case, would be  divided over two IP fragments, the first containing 16 bytes and the second containing 8 bytes of the TCP header. 

On the other hand, if you prefer to increase the size of your packets to make packets to make them look innocuous, you can the option ```--data-length NUM```, where num specifies the number of bytes you want to append to your packets.

## Idle / Zombie Scan 

Spoofing the source IP address can be a great approach to scanning stealthily. However, spoofing will only work in specific network setups. It requires you to be in a position where you can monitor the traffic. Considering these limitations, spoofing your IP address can have little use; however, we can give it an upgrade with the with the idle scan. 

The idle scan, or zombie scan, requires an idle system connected to the network that you can communicate with. Practically, Nmap will make each probe appear as if coming from the idel (zombie) host, then it will check for indicators whether the idle (zombie) host received any response to the spoofed probe. This is accomplished by checking the IP identification (IP ID) value in the IP header. You can run an idel scan using ```nmap -sI ZOMBIE_IP MACHINE_IP```, where ```ZOMBIE_IP``` is the IP address of the idle host (zombie). 

The idle (zombie) scan requires the following three steps to discover whether a port is open: 

1. Trigger the idle host to respond so that you can record the current IP ID on the idle host. 
2. Send a SYN packet to a TCP port on the target. The packet should be spoofed to appear as if it was coming from the idle host (zombie) IP address. 
3. Trigger the idle machine again to respond so that you can compare the new IP ID with the one received earlier.

Let's explain with figures. In the figure below, we have the attacker system probing an idle machine, a multi-function printer. By sending a SYN/ACK, it responds with an RST packet containing its newly incremented IP ID.

![image](https://user-images.githubusercontent.com/79100627/172442214-d4d79861-0a8b-4747-87d0-e964556321da.png)

The attacker will send a SYN packet to the TCP port they want to check on the target machine in the next step. However, this packet will use the idel host (zombie) IP address as the source. Three scenarios would arise. In the first scenario, shown in figure below, the TCP port is closed; therefore, the target machine responds to the idle host with an RST packet. The idle host does not respond; hence its IP ID is not incremented.

![image](https://user-images.githubusercontent.com/79100627/172442771-7cbf02f3-2096-4f71-a5b2-db950d77a38a.png)

In the second scenario, as shown below, the TCP port is open, so the target machine responds with a SYN/ACK to the idle host (zombie). The idle host responds to this unexpected packet with an RST packet, thus incrementing its IP ID.

![image](https://user-images.githubusercontent.com/79100627/172443132-09f994f8-ac57-416a-b081-dcb36e2c336c.png)

In the thrid scenario, the target machine does not respond at all due to firewall rules. This lack of response will lead to the same result as with the closed port; the idle host won't increase the IP ID.

For the final step, the attacker sends another SYN/ACK to the idle host. The idle host responds with an RST packet, incrementing the IP ID by one again. The attacker needs to compare the IP ID of the RST packet received in the first step with the IP ID of the RST packet received in this thrid step. If the difference is 1, it means the port on the target machine was closed or filtered. However, if the difference is 2, it means that the port on the target was open. 

It is worth repeating that this scan is called an idle scan because choosing an idle hosting is indispensable for the accuracy of the scan. If the "idle host" is busy, all the returned IP IDs would be useless. 

You might consider adding ```--reason``` if you want Nmap to provide more details regarding its reasoning and conclusions. Consider the two scan below to the system; however, the latter adds ```--reason```

## Getting more details 

![image](https://user-images.githubusercontent.com/79100627/172444595-14bfa0ae-92a0-432b-91c2-7b0c4097ee32.png)

![image](https://user-images.githubusercontent.com/79100627/172444715-8b9b484b-f488-49f9-b1ee-eeb6834ea799.png)

Providing the ```--reason``` flag gives us the explicit reason why Nmap concluded that the system is up or a particular port is open. In this console output above, we can see that this system is considered online because Nmap "recieved arp-response." On the other hand, we know that the SSH port is deemed to be open because Nmap received a "SYN-ACK" packet back. 

For more detail output, you can consider using ```-v``` for verbose output or ```-vv``` for even more verbosity 

![image](https://user-images.githubusercontent.com/79100627/172445422-a7293b2f-993f-466c-b27d-7100da6ee022.png)

if ```-vv``` does not satisfy your curiousity, you can use ```-d``` for debugging details or ```-dd``` for more details.

## Summary

![image](https://user-images.githubusercontent.com/79100627/172445751-3bfe529d-a21a-468f-9cb5-fd5cfefb0ba2.png)

![image](https://user-images.githubusercontent.com/79100627/172445818-181c1d40-869d-42f2-9956-171210bb8712.png)

## Service Detection 

Once Nmap discovers open ports, you can probe the available port to detect the running service. Further investigation of open ports is an essential piece of information as the pentester can use it to learn if there are any known vulnerabilities of the service. 

Adding ```-sV``` to your Nmap command will collect and determine service and version information for the open ports. You can control the intensity with ```--version-intensity LEVEL``` where the level ranges between 0, the lightest, and 9, the most complete. ```-sV --version-light``` has an intensity of 2, while ```-sV --version-all``` has an intensity of 9. 

It is important to note that using ```-sV``` will force Nmap to proceed with the TCP 3-way handshake and establish the connection. The connection establishment is necessary because Nmap cannot discover the version without establishing a connection fully and communicating with the listening service. In other words, stealth SYN scan ```-sS``` is not possible when ```-sV``` option is chosen 

The console output below shows a simple Nmap stealth SYN scan with the ```-sV``` option. Adding the ```-sV``` option leads to a new column in the output showing the version is for each detected service. For instance, in the case of TCP port 22 being open, instead of ```22/tcp open ssh```, we obtain ```22/tcp open ssh OpenSSH 6.7p1 Debian 5+deb8u8 (protocol 2.0)```. Notice that the SSH protocol is guessed as the service because TCP port 22 is open; Nmap didn't need to connect to port 22 to confirm. However, ```-sV``` required connecting to this open port to grab the service banner and any version information it can get, such as ```nginx 1.6.2```. Hence, unlike the service column, the version column is not a guess. 

![image](https://user-images.githubusercontent.com/79100627/172459554-2f09f120-0c66-4a4d-a8e3-53c5ed84ca9e.png)

Note that many Nmap options require root privileges. Unless you are running Nmap as root, you need use ```sudo``` as in the example above.

## OS Detection 

Nmap can detect the Operating System (OS) based on its behaviour and any telltale signs in its responses. OS detection can be enable using ```-O```; this is an uppercase O as in OS. 

![image](https://user-images.githubusercontent.com/79100627/172460850-2be9ad9e-7954-43ae-b374-35d6ec5363d2.png)

The system that we scanned and attempted to detect its OS version is running kernel version 3.16. Nmap was able to make a close guess in this case. In another case, we scanned a Fedora Linux system with kernel 5.13.14; however, Nmap detected it as Linux 2.6.X. The good news is that Nmap detected the OS correctly; the not-so-good news is that the kernel version was wrong. 

The OS detection is very convenient, but many factors might affect its accuracy. First and foremost, Nmap needs to find at least one open and one closed port on the target to make a reliable guess. Furthermore, the guest OS fingerprints might get distorted due to the rising use of virtualization and similar technologies. Therefore, always take the OS version with a grain of salt. 

## Traceroute 

If you want Nmap to find the routers between you and the target, just add ```-traceroute```. In the following example, Nmap appended a traceroute to its scan results. Note that Nmap's traceroute works slightly different than the ```traceroute``` command found on Linux and macOS or ```tracert``` found on MS windows. Standard ```traceroute``` starts with a packet of low TTL (Time to Live) and keeps increasing until it reaches the target. Nmap's traceroute starts with a packet of high TTL and keeps decreasing it. 

In the following example, we executed ```nmap -sS --traceroute 10.10.99.125``` on the Attackbox 

![image](https://user-images.githubusercontent.com/79100627/172461872-61cd2de2-dc8f-4b24-af9e-017b45532908.png)

It is worth mentioning that many routers are configured not to send ICMP Time-to-Live exceeded, which would prevent us from discovering their IP addresses. 

A script is a piece of code that does not need to be complied. In other words, it remains in its original human-readable form and does not need to be converted to machine language. Many programs provide additional functionality via scripts; moreover, scripts make it possible to add custom functionality that did not exist via the built in commands. Similarly, Nmap provides support for scripts using the Lua language. A part of Nmap, Nmap Scripting Engine (NSE) is a Lua interpreter that allows Nmap to execute Nmap scripts written in Lua language. However, we don;t need to learn Lua to make use of Nmap scripts 

Your Nmap default installation can easily contain close to 600 scripts. 

![image](https://user-images.githubusercontent.com/79100627/172470815-f1244dde-190c-487d-bf4e-ad1af368a06b.png)

You can specify to use any or a group of these installed scripts; moreover, you can install other user's scripts and use them for your scans. Let's begin with the default scripts. You can choose to run the scripts in the default category using ```--script=default``` or simply adding ```-sC```. In addition to default, categories include auth, broadcast, brute, default, discovery, dos, exploit, external, fuzzer, intrusive, malware, safe, version, and vuln. 

![image](https://user-images.githubusercontent.com/79100627/172471120-b58e4078-bb80-4e8b-81ac-b928dd6c172c.png)

Some scripts belong to more than one category. Moreover, some scripts launch brute-force attacks against services, while other launch DoS attack and exploit systems. Hence, it is crucial to be careful when selecting scripts to run if you don't want to crash service or exploit them.

We use Nmap to run a SYN scan against ```10.10.190.98``` and execute the default scripts in the console shown below. The command is ```sudo nmap -sS -sC 10.10.190.98```, where ```-sC``` will ensure that Nmap will execute the default scripts following the SYN scan. There are new details that appear below. Take a look at the SSH service at port 22; Nmap recovered all four public keys related to the running server. Consider another example, the HTTP service at port 80; Nmap retrieved the default page title. We can see that the page has been left as default. 

![image](https://user-images.githubusercontent.com/79100627/172472840-059cf5f8-d583-4d44-a812-800ee6563fbb.png)

You can also specify the script by name using ```--script "SCRIPT-NAME"``` or a pattern such as ```--script "ftp*"```, which would ```ftp-brute```. If you are unsure what a script does, you can open the script file with a text reader, such as ```less```, or a text editor. In the case of ```ftp-brute```, it states: "Performs brute force password auditing against FTP servers." You have to becareful as some scripts are pretty intrusive. Moreover, some script might be for a specific server and, if chosen at random, will waste your time with no benefit. As usual, make sure that you are authorized to launch such tests on the target server. 

Get the date from HTTP-likes services. Also it prints how much the date differs from loacl time 

![image](https://user-images.githubusercontent.com/79100627/172474018-07f0dd22-8c68-413f-bd97-97d01888d367.png)

## Svaing the Output 

Whenever you run a Nmap scan, it is only reasonable to save the results in a file. Selecting and adopting a good naming convention for your filenames is also crucial. The number of files can quickly grow and hinder your ability to find a previous scan result. The three main formats are:

1. Normal 
2. Grepable (```grep```able)
3. XML 

There is a fourth one that we cannot recommend:

- Script Kiddie 

## Telnet 

The telnet protocol is an application layer protocol used to connect to a virtual terminal of another computer. Using Telnet, a user can log into another computer and access its terminal (console) to run programs, start batch processes, and perform system administration tasks remotely. 

Telnet protocol is relatively simple. When a user connects, they will be asekd for a username and password. Upon correct authentication, the user will access the remote system's terminal. Unfortunately, all this communication between the Telnet client and Telnet server is not encrypted, making it an easy target for attackers. 

A Telnet server uses the Telnet protocol to listen for incoming connection on port 23. Let's consider the example shown below. A user is connecting to the ```telnetd```, a Telnet server. The steps are as follows:

1. First, he is asked to provide his login name (username). We can see the user entering ```frank```
2. Then, he is asked for the password, ```D2jiij```. The password is not shown; however, we display it below for demonstration purposes. 
3. Once the system checks his login credentials, he is greeted with a welcom message. 
4. And the remote server grants him a command prompt, ```~$```. The $ indicates that this is not root terminal. 

![image](https://user-images.githubusercontent.com/79100627/172756838-b3ff7dfd-c913-4c74-8848-817dc8a97922.png)

Although Telnet gave us access to the remote system's terminal in no time, it is not a reliable protocol for remote administration as all the data are sent in cleartext. In the figure below, we captured the traffic generated by Telnet, and it was so easy to find the password. The figure below shows the ASCII data exchanged between our computer and the remote system. The text in red is the text that we are sending to the remote system. while the text in blue is the text that the remote system is sending. Notice how the user name was sent back (echoed at us to display them in our terminal); however, the password was not. In other words, if someone is watching us type, they wont' able to see the passwrod characters on the screen. 

![image](https://user-images.githubusercontent.com/79100627/172757234-44aa6c54-8b55-47a9-a3a8-26bc88204a35.png)

## HTTP (Hypertext Transfer Protocol)

Hypertext Transfer Protocol (HTTP) is the protocol used to transfer web pages. Your web browser connects to the webserver and uses HTTP to request HTML pages and images among other files and submit forms and upload various files. Anytime you browse the World Wide Web (WWW), you are certainly using the HTTP protocol.

The image below shows a client requesting the HTML ```index.html```, which the webserver provides. Then the client requests an image ```logo.jpg``` and the web server sends it 

![image](https://user-images.githubusercontent.com/79100627/172757581-e8058460-7167-4efc-b7ef-6b12b0b7bd96.png)

HTTP sends and receives data as cleartext (not encrypted); therefore, you can use a simple tool, such as Telnet (or Netcat), to communicate with a web server and act as a "Web Browser". The key difference is that you need to input the HTTP- related commands instead of the web browser doing that for you. 

In following example, we will see how we can request a page from a web server; morever, we will discover the webserver version. To accomplish this, we will use the Telnet client. We chose it because Telnet is a simple protocol; futhermore, it uses cleartext for communication. We will use ```telnet``` instead of a web browser to request a file from the webserver. The steps will be as follows:

1. First, we connect port 80 using ```telnet 10.10.41.241 80```.
2. Next we need to type ```GET /index.html HTTP/1.1``` to retrieve the page ```index.html``` or ```GET / HTTP/1.1``` to retrieve the default page. 
3. Finally, you need to provide some value for the host like ```host: telnet``` and hit the Enter/Return key twice

![image](https://user-images.githubusercontent.com/79100627/172758529-1973165f-1c63-4638-a9d4-e51adbca0052.png)

Of particular intrest in the output above is that the user needs only to type a couple of commands to get the page they need: ```GET /index.html HTTP/1.1``` followed by ```host:telnet```.

We need an HTTP server (webserver) and an HTTP client (web browser) to use the HTTP protocol. The webserver will "serve" a specific set of files to the requesting web browser.

Three popular chocies for HTTP servers are:
- Apache 
- Internet Information Service 
- nginx 

Apache and Nginx are free

## File Transfer Protocol (FTP) 

File Transfer Protocol (FTP) was developed to make the transfer of files between different computers with different systems efficient. 

FTP  also sends and receives data as cleartext; therefore, we can use Telnet (or Netcat) to coomunicate with an FTP server and act as an FTP client. In the example below, we carried out the following steps:

1. We connected to an FTP server using a Telnet client. Since FTP servers listen on port 21 by default Telnet port.
2. We needed to provide the username with the command ```USER frank```.
3. Then, we provide the password with the command ```PASS D2xc9CgD```.
4. Beacuse we supplied the correct username and password, we got logged in

A command like ```STAT``` can provide some added information. The ```SYST``` command shows the System Type of the target (UNIX in this case). ```PASV``` switches the mode to passive. It is worth nothing that there are two modes for FTP: 

- Active: In the active mode, the data is sent over a separate channel originating from the FTP server's port 20. 
- Passive: In the passive mode, the data is sent over a separate channel originating from an FTP client's port above port number 1023. 


The command ```TYPE A``` switches the file transfer mode to ASCII, while ```TYPE I``` switches the file transfer mode to binary. However, we cannot transfer a file using a simple client such as Telnet because FTP creates a separate connection for file transfer. 

![image](https://user-images.githubusercontent.com/79100627/172899079-73823014-b170-48f8-9f62-c9f1f6ed155a.png)

The image below shows how an actual file transfer would be conducted using FTP. To keep things simple in this figure, let's only focus on the fact that the FTP client will initiate a connection to an FTP server, which listens on port 21 by default. All commands will be sent over the control channel. Once the client requests a file, another TCP connection will be established between them. (The details of establishing the data connection/channel is beyond the scope of this room).

![image](https://user-images.githubusercontent.com/79100627/172900211-fdd4fbd7-d629-47cb-b2d9-32ab1ca457b8.png)

Considering the sophistication of the data transfer over FTP, let's use an actual FTP client to download a text file. We only needed a small number of commands to retrieve the file. After logging in successfully, we get the FTP prompt, ```ftp>```, to execute various FTP commands. We used ```ls``` to list the files and learn the file name; then, we switched to ```ascii``` since it is a text file (not binary). Finally ```get FILENAME``` made the client and server establish another channel for file transfer. 

![image](https://user-images.githubusercontent.com/79100627/172900955-80ab1b1c-5137-4faa-a3ac-56bd57a273f0.png)

FTP servers and FTP clients use the FTP protocol. There are various FTP server software that you can select from if you want to host your FTP server. Example of FTP servers:

- vsftpd
- ProFTPD
- uFTP

For FTP clients, in addition to the console FTP client commonly found on Linux systems, you can use an FTP client with GUI such as FileZilla. Some web browsers also support FTP protocol.

## Simple Mail Transfer Protocol (SMTP)

Email is one of the most used services on the Internet. There are various configurations for email servers; for instance you may set up an email system to allow local users to exchange emails with each other with no access to the Interent. However, we will consider the more general setup where different email servers connect over the Internet.

Email delivery over the Internet requires the following components:

1. Mail Submission Agent (MSA) 
2. Mail Transfer Agent (MTA)
3. Mail Delivery Agent (MDA)
4. Mail User Agent (MUA) 

The above four terms may look cryptic, but they are more straightforward than they appear. We will explain these terms using the figure below. 

![image](https://user-images.githubusercontent.com/79100627/172902404-d1f12fed-5a07-4cb6-89e2-6828041a7ac6.png)

The figure shows the following five steps that an email needs to go through to reach the recipient's inbox:

1. A Mail User Agent (MUA), or simply an email client, has an email message to be sent. The MUA connects to a Mail Submission Agent (MSA) to send its message.
2. The MSA receives the message, checks for any errors before transferring it to the Mail Transfer Agent (MTA) server, commonly hosted on the same server. 
3. The MTA will send the email message to the MTA of the recipient. The MTA can also function as a Mail SUbmission Agent (MSA).
4. A typical setup would have the MTA server also functioning as a Mail Delivery Agent (MDA).
5. The recipient will collect its email from the MDA using their email client. 

If the above steps sound confusing, consider the following analogy:

1. You (MUA) want to send postal mail.
2. The post office employee (MSA) checks the postal mail for any issues before your local post (MTA) accepts it.
3. The local post office checks the mail destination and ssends it to the post office (MTA) in the correct country. 
4. The post office (MTA) delivers the mail to the recipient mailbox (MDA).
5. The recipient (MUA) regularly checks the mailbox for new mail. They notice the new mail, and they take it. 

In the same way, we need to follow a protocol to communicate with an HTTP server, and we need to rely on email protocols to talk with an MTA and an MDA. The protocols are:

1. Simple Mail Transfer Protocol (SMTP)
2. Post Office Protocol Version 3 (POP3) or Interent Message Access Protocol (IMAP) 

SMTP is used to communicate with MTA server. Because SMTP uses cleartext, where all commands are sent without encryption, we can use basic Telnet client to connect to an SMTP server and act as an email client (MUA) sending a message. 

SMTP server listens on port 25. To see basic communication with an SMTP server, we used Telnet to connect to it. Once connected, we issue ```helo hostname``` and then start typing our email. 

![image](https://user-images.githubusercontent.com/79100627/172903969-6f60debb-e361-4aa5-9a7e-4c5ec2c4882a.png)

After ```helo```, we issue ```mail from:```, ```rcpt to:``` to indicate the sender and the recipient. When we send our email message, we issue the command ```data``` and type our message. We issue ```<CR><LF>,<CR><LF>``` (or ```Enter . Enter``` to put it in simpler terms). The SMTP server now queues the message. 

## POST Office Protocol Version 3

Post Office Protocol Version 3 (POP3) is a protocol used to download the email messages from a Mail Delivery Agent (MDA) server, as shown in the figure below. The mail client connects to the POP3 server, authenticates, downloads the new email messages before (optionally) deleting them. 

![image](https://user-images.githubusercontent.com/79100627/172905674-729910a5-a743-49c2-b626-004813e213f3.png)

The example below shows what a POP3 session would like if conducted via a Telent client. First, the user connects to the POP3 server at the POP3 default port 110. Authentication is required to access the email messages; the user authenticates by providing his user name ```USER frank``` and password ```PASS D2xc9CgD```. Using the command ```STAT```, we get the reply ```+OK 1 179```; based on RFC 1939, a positive response to STAT has the format ```+OK nn mm```, when nn is the number of email messages in the inbox, and mm is the size of the inbox in octets (bytes). The command ```LIST``` provided a list of new messages on the server, and ```RETR 1``` retrieved the first message in the list. We don't need to concern ourselves with memorizing these commands; however, it is helpful to strengthen our understanding of such protocol. 

![image](https://user-images.githubusercontent.com/79100627/172906532-6edaa061-36fd-4323-8742-68aa1c0383b1.png)

The example above shows that the commands are sent in cleartext. Using telnet was enough to authenticate and retrieve an email message. As the username and password are sent in the cleartext, any thrid party watching the network traffic can steal the login credentials. 

In general, your mail client (MUA) will connect to the POP3 server (MDA), authenticate, and download the messages, Although the communication using the POP3 protocol will be hidden behind a sleek interface, similar commands will be issued, as shown in the Telnet session above. 

Based on the default settings, the mail client deletes the mail message after it downloads it. The default behaviour can be changed from the mail client settings if you wish to download the emails again from another mail client. Accessing the same mail account via multiple client's using POP3 is usually not very convenient as one would lose track of read and unread messages.

## Interent Message Access Protocol (IMAP)

Internet Message Access Protocol (IMAP) is more sophisticated than POP3. IMAP makes it possible to keep your email synchronized accross multiple devices (and mail clients). In other words, if you mark an email message as read when checking your email on your smartphone, the change will be saved on IMAP server (MDA) and replicated on your laptop when you synchronize your inbox. 

Let's take a look at simple IMAP commands. In the console output below, we use Telnet to connect to the IMAP server's default port, and then we authenciate using ```LOGIN username password```. IMAP requires each command to be preceded by a random string to be able to track the reply. So we added ```c1```, then ```c2```, and so on. Then we listed our mail folders using ```LIST "" "*"```, before checking if we have any new messages in the inbox using ```Examine INBOX```. We don;t need to memorize these commands; however, we are simply  providing the example below to give a vivid image of what happens when the mail client communicates with an IMAP server.

![image](https://user-images.githubusercontent.com/79100627/172912211-bfceaf37-13e9-48a4-9ea9-61e4aca221a0.png)

## Summary 

![image](https://user-images.githubusercontent.com/79100627/172912418-53236e9f-286d-4dd5-b3a5-8d75fab63a56.png)

## Sniffing Attack 

Sniffing attack refers to using a network packet capture tool to collect information about the target. When a protocol communicates in cleartext, the data exchanged can be captured by a third party to analyse. A simple network packet capture can reveal information, such as the content of private messages and login credentials, if the data isn't encrypted in transit.

A sniffing attack can be conducted using an Ethernet (802.3) network card, provided that the user has proper permissions (root permissions on Linux and administrator privileges on MS Windows). There are many programs available to capture network packets. We consider the following:

1. TCPdump is a free open source command-line interface (CLI) Program that has been ported to work on many operating systems.
2. Wireshark is a free open source graphical user interface (GUI) program available for several  operating systems, including Linux, macOS and MS Windows. 
3. Tshark is a CLI alternative to Wireshark 

There are several specialized tools for capturing passwords and even complete messages; however, this can still be acheived by TCPdump and Wireshark with some added effort. 

Consider a user checking his email messages using POP3. First, we are going to use Tcpdump to attempt to capture the username and password. In the terminal output below, we used the command ```sudo tcpdump port 110 -A```. Before explaining this command, we should mention that this attack requires access to the network traffic, for example, via a wiretap or a switch with port mirroring. Alternatively, we can access the traffic exchanged if we launch a successful Man in the Middle attack. 

We need ```sudo``` as packet captures required root privileges. We wanted to limit the number of captured and displayed packets to those exchanged with the POP3 server. We know that POP3 uses port 110, so we filtered our packets using ```port 110```. Finally, we wanted to display the contents of the captured packets in ASCII format, so we added ```-A```.

![image](https://user-images.githubusercontent.com/79100627/172916375-1d1ec13a-b482-4293-8842-ce844b055773.png)

In the terminal output above, we have removed the unimportant packets to help you better focus on the ones that matter. In particular, the username and password were each sent int their own packet. The first packet explicitly displays "USER frank", while the last packet reveals the password "PASS D2xc9CgD"

We could also use the Wireshark to achieve the same result 
![image](https://user-images.githubusercontent.com/79100627/172916746-f6a0eaa9-2174-47f5-a5a0-49994bbeeb34.png)

## Man In the Middle (MITM) Attack 

A Man in The Middle (MITM) attack occurs when a victim (A) believes they are communicating with a legitimate destination (B) but is unkowingly communicating with an attacker (E). In the figure below, we have A requesting the transfer of $20 to M; however, E altered this message and replaced the original value with a new one. B received the modified messaged and acted on it. 

![image](https://user-images.githubusercontent.com/79100627/172917870-19cab646-f898-451a-8004-0b6895efd15c.png)

This attack is relatively simple to carry out if the two parties do not confirm the authenticity and integrity of each message. In some cases, the chosen protocol does not provide secure authentication or integirty checking; moreover some protocols have inherent insecurities that make them susceptible to this kind of attack.

Any time you browse over HTTP, you are susceptible to a MITM attack, and the scary thing is that you cannot recognize it. Many tools would aid you in carrying out such an attack, such as Ettercap and Bettercap.

MITM can also affect other cleartext protocols such as FTP, SMTP, and POP3. Mitigation against this attack requires the use of cryptography. The solution lies in proper authentication alogn with encryption or signing of the exchanged messages. With the help of Public Key Infrastructure (PKI) and trusted root certificates, Transport Layer Security (TLS) prototects from MITM attacks.

## Transport Layer Security (TLS)

SSL (Secure Socket Layer) started when the world wide web started to see new applications, such as online shopping and sending payment information. Netsacpe introduced SSL in 1994, with SSL 3.0 being released in 1996. But eventually, more security was needed, and TLS (Transport Layer Security) protocol was introduced in 1999. Before we explain what TLS and SSL provide, let's see how they fit the networking model.

The common protocols we have covered so far send the data in cleartext; this makes it possible for anyone with access with access to the network to capture, save and analyze the exchanged messages. The image below shows the ISO/OSI network layers. The protocols we have covered so far in this room are on the application layer. Consider the ISO/OSI model; we can add encryption to our protocols via the presentation layer. Consequently, data will be presented in an encrypted format (ciphertext) instead of its original form. 

![image](https://user-images.githubusercontent.com/79100627/172920516-46453f2e-348b-43f2-9bdd-bce16bf2b41d.png)

Beacuse of the close relation between SSL and TLS, one might be used instead of other. However, TLS is more secure than SSL, and it has partically replaced SSL. We could have droped SSL and just written TLS instead of SSL/TLS, but we will continue to mention the two to avoid any ambiguity because the term SSL is still in wide use. However, we can expect all modern servers to be using TLS.

An existing cleartext protocol can be updagraded to use Encryption via SSL/TLS. We can use TLS to upgrade HTTP,FTP,SMTP,POP3, and IMAP, to name a few. The following table lists the protocols we have covered and their default ports before and after the necryption upgrade via SSL/TLS. The list is not exhaustive;

![image](https://user-images.githubusercontent.com/79100627/172921007-43da67ce-ae7b-4bed-83e7-e1ee6fa0eec6.png)

Considering the case of HTTP. initially to retrieve a web page over HTTP, the web browser would need at least perform the following two steps:

1. Establish a TCP connection with the remote webserver
2. Send HTTP requests to the web server, such as GET and POST requests. 

HTTPS reuqires an additional step to encrypt the traffic. The new step takes palce after establishing a TCP connection and before sending HTTP requests. This extra step can be inferred from the ISO/OSI model in the image presented ealier. Consequently , HTTPS requires at least the following three steps: 

1. Establish a TCP connection
2. Establish SSL/TLS connection
3. Send HTTP requests to the webserver 

To establish an SSL/TLS connection, the client needs to perform the proper handshake with the server. Based on RFC 5101, the SSL connection establishment will look like the figure below. 

![image](https://user-images.githubusercontent.com/79100627/172921979-b02d8001-15b1-4ff5-bfa7-0b5985a17e0c.png)

After establishing a TCP connection with the server, the client establishes an SSL/TLS connection, as shown in the figure above. The terms might look complicated depending on your knowledge of cryptography, but we can simplify the foru steps as:

1. The client sends a ClientHello to the server to indicate its capabilities, such as supported algorithms.
2. The server responds with a ServerHello, idnicating the selected connection parameters. The server provides its certificate if server authentication is required. The certificate is a digital file to identify itself; it is usually digitally signed by a thrid party. Moreover, it might send additional information necessary to generate the master key, in its ServerKeyExchange message, before sending the ServerHelloDone Message to indicate that it is done with the neogtiation.
3. The client responds with a ClientKeyExchange, which contains additional information required to generate the master key. Furthermore, it switches to use encryption and informs the server using the ChangeCipherSpec message. 
4. The server swithces to use encryption as well and informs the client in the ChangeCipherSpec message 

If this still sounds sophisticated, dont worry; we only need the gist of it. A client was able to agree on a secret key with a swerver that has a public certificate. This secret key was securely generated so that a thrid party monitoring the channel wouldn't be able to discover it. Further communication between the client and the server will be encrypted using the generated key.

Consequently, once an SSL/TLS handshake has been established, HTTP requests and exchanged data won't be accessible to anyone watching the communication channel. 

As a final note, for SSL/TLS to be effective, especially when browsing the we over HTTPS, we rely on public certificates signed by certificate authorities trusted by our systems. In other words, when we browse to TryHackMe over HTTPS, our browser expect the TryHackMe web server to provide a signed certificate from a trusted certificate authroity, as per the example below. This way, our browser ensures that it is communicating with the correct server, and a MITM attack cannot occur

![image](https://user-images.githubusercontent.com/79100627/172923846-bdbc0b9c-232a-4c6d-8ce3-b015f007a04e.png)

## Secure Shell (SSH)

Secure Shell (SSH) was created to provide a secure way for remote system administration. In other words, it lets you securely connect to another system over the network and execute commands on the remote system. Put simply, the "S" in SSH stands for secure, which can be summarized simply as:

1. You can confirm the identity of the remote server
2. Exchanged messages are encrypted and can only be decrypted by the intended recipient 
3. Both sides can detect any modification in the messages

The above three points are ensured by cryptography. In more technical terms, they are part of confidentiality and integrity, made possible through the proper use of different encryption algorithms.

To use SSH, you need an SSH server and an SSH client. The SSH server listens on port 22 by default. The SSH client can authenticate using:

- A username and a password
- A private and public key (after the SSH server is configured to recognize the corresponding public key) 

On Linux, macOS, and MS Window builds after 2018, you can connect an SSH server using following command ```ssh username@macine_IP```. This command will try to connect to the server of IP address ```MACHINE_IP``` with the login name ```username```. If an SSH server is listening on the default port, it will ask you to provide the password for ```username```. Once authenticated the user will have to access the target server's terminal. The terminal output below is an example of using SSH to access a Debian Linux server. 

![image](https://user-images.githubusercontent.com/79100627/173194599-7784fb21-6454-4842-8962-bc513db10a5e.png)

In the example above, we issued command ```ssh mark@MachineIP```. Then once, we entered the correct password, we got access to the remote system's terminal. SSH is very reliable for remote administration because our username and password were sent encrypted; moreover, all commands we execute on the remote system will be sent over an encrypted channel.

Note that if this is the first time we connect ot the system, we will need to confirm the fingerprint of the SSH server's public key to avoid man-in-the-middle attacks. As explained earlier, MITM takes place when a malicious party, E, situates itself between A and B, and communicates with A, pretending to be B, and communicates with B pretending to be A, while A and B think they are communicating directly with each other. In the case of SSH, we don't ususally have a thrid party to check if the public key is valid, so we need to do this manually.

![image](https://user-images.githubusercontent.com/79100627/173194701-11a96594-bf0a-413a-9109-75e4b39cb930.png)

We can use SSH to transfer files using SCP (Secure Copy Protocol) based on the SSH protocol. An example of the syntax is as follows:

```scp mark@10.10.17.190:/home/mark/archive.targ.gz ~```. This command will copy a file named ```archive.tar.gz``` from the remote system located in the ```/home/mark``` directory to ```~```, i.e, the root of the home directory of the currently logged-in user. 

Another example syntax is ```scp backup.tar.bz2 mark@10.10.17.190:/home/mark/```. This command will copy the file ```backup.tar.bz2``` from the local system to the directory ```/home/mark/``` on the remote system. 

![image](https://user-images.githubusercontent.com/79100627/173194808-7e8b0340-c955-435a-8555-eb6e03f9bacd.png)

As a closing note, FTP could be secured using SSL/TLS by using the FTPS protocol which uses port 990. It is worth mentioning that FTP can also be secured using the SSH protocol which is the SFTP protocol. By default this service listens on port 22, just like SSH. 

## Password Attack 

Many protocols require you to authenticate. Authentication is proving who you claim to be. When we are using protocols such as POP3, we should not be given access to the mailbox before verifying our identify. The POP3 example from the Protocols and Servers room is repeated below for your convenience. In this example, we are identified as the user frank, and the server authenticated us because we provided the correct password. In other words, the password is one way to authentication. 

![image](https://user-images.githubusercontent.com/79100627/173195129-b31ed12a-d7bb-49f1-8a0f-875cec480630.png)

Authentication, or providing your identity can be achieved through one of the following, or a combination of two:

1. Something you know, such as password and PIN code.
2. Something you have, such as SIM card, RFID card, and USB dongle 
3. Something you are, finger prints and iris 

Attack against passwords are usually carried out by: 

1. Password Guessing: Guessing a password requires some knowledge of the target, such as their pet's name and birth year. 
2. Dictionary Attack: This approach expands on password guessing and attempts to include all valid words in a dictionary or wordlist.
3. Brute Force Attack: This attack is most exhaustive and time-consuming where an attacker can go as far as trying all possible character combinations, which grows fast (exponential growth with the number of characters).

Let's focus on dictonary attacks. Over time, hackers have complied list after list containing leaked password from data breaches. One example is RockYou's list of breached passwords. The choice of word list should depend on your knowledge of the target. For instance, a French user might use French word instead of an English one. 

We want an automated way to try the common passwords or the entries from a word list; here comes THC Hydra. Hydra supports many protocols. including FTP, POP3, IMAP, SMTP, SSH and all methods related to HTTP. The general command-line syntax is: ```hydra -l username -P wordlist.txt server service``` where we specify the following options:

- ```-l username```: ```-l``` should preceded the ```username```, ie the login name of the target
- ```-P wordlist.txt```: ```-P``` preceds the ```wordlist.txt``` file, which is a text file containing the list of passwords you want to try with the provided username 
- ```server``` is the hostname or IP address of the target server
- ```service``` indicates the service which you are trying to launch the dictionary attack.

Consider the following example:

```hydra -l mark -P /usr/share/wordlists/rockyou.txt 10.10.17.190 ftp``` will use ```mark``` as the username as it iterates over the provided password against the FTP server. 

```hydra -l mark -P /usr/share/wordlists/rockyou.txt ftp://10.10.17.190``` is identical to the previous example. ```10.10.17.190 ftp``` is the same as ```ftp://10.10.17.190```. 

```hydra -l frank -P /usr/share/wordlists/rockyou.txt 10.10.17.190 ssh``` will use ```frank``` as the user name as it tries to login via SSH using the different passwords. 

There are some extra optional arguments that you can add:

- ```-s PORT``` to specify a non-default port for the service in the question.
- ```-V``` or ```-vV```, for verbose, makes Hydra show the username and password comibnation that are being tried. The verbosity is very convenient to see the progress
- ```-t n``` where n is the number of parallel connections to the target ```-t 16``` will create 16 threads used to connect to the target.
- ```-d```, for debugging to get more detailed information about progress. The debugging output can save you much frustration; for instance, if Hydra tries to connect to a closed port and timing out, ```-d``` will reveal this right away.

Once the password is found, you can issue CTRL-C to end the process. 

![image](https://user-images.githubusercontent.com/79100627/173196142-5666e553-0b62-46ea-9821-013ccb6e4a0a.png)

