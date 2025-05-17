# Azure Honeypot

The purpose of this project was to deploy a [honeypot](https://www.fortinet.com/resources/cyberglossary/what-is-honeypot) on a web server to analyze traffic patterns and logs. Using the [T-Pot](https://github.com/telekom-security/tpotce) honeypot platform, I set up a Microsoft Azure [Virtual Machine (VM)](https://azure.microsoft.com/en-us/products/virtual-machines) and configured inbound port rules to allow extensive traffic into the simulated server.

What I found was surprising. I've heard of crawlers before, but witnessing the sheer amount of unsolicited traffic hitting a public-facing network firsthand was eye-opening. Most of the traffic was harmless, but some requests originated from malicious IP addresses—verified through [AbuseIPDB](https://www.abuseipdb.com/)—highlighting just how vulnerable an unsecured network can be.

Follow along below for a step-by-step breakdown of the setup process and key insights from this experiment!

1. The first thing I needed to do was create a VM server to host the trap. Microsoft Azure's cloud platform was a perfect choice for this. I chose an Ubuntu Linux server to host the software on and got to work.

![01 - Create VM](https://github.com/user-attachments/assets/9374506f-ff9b-4ea3-a031-b469579ea659)

2. Before we got the server was up and running, I needed to allow traffic ingress onto the server. By default, Azure has decent security properties for people who may be unfamiliar with cloud security, so we needed to strip some of those away to ensure a very wide net.

![02 - Inbound Port Rules](https://github.com/user-attachments/assets/21200d72-6472-49f3-adbc-3410b81f21a7)

3. After we finished the configuration tutorial, the server was up and running!

![03 - VM Running](https://github.com/user-attachments/assets/fc4d4ec8-6f91-4120-a65d-40622b688633)

4. In order to install the T-Pot software on the server, I chose to use a Windows application called [PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/). This allowed me to use SSH to connect to the server, since during server setup I created an asymmetric key pair. This allowed the public to access the site, but only I could access the management of the server, since I was the holder of the private and public keys.

![04 - Putty Git Install](https://github.com/user-attachments/assets/aff28c94-7da6-4923-8d79-8920c430c12c)

5. Once I was connected to the server via PuTTY, I was able to use the command line to begin the installation. First, I had to make sure all software on the Linux host server was up-to-date. This required running the commands: sudo apt update, which updated the list of local packages on the VM, and sudo apt upgrade, which upgraded any of the packages installed that might have been out of date. After that, I cloned the T-Pot repository from GitHub and began in installation.

![05 - CloneGit](https://github.com/user-attachments/assets/8d6468e3-27a9-49d5-b48f-6453b4cfbd40)

6. To ensure the install went smoothly and everything was operational, I went back to the Azure VM and copied the public IP address assigned to the server. Using my browser and login credentials, I was able to log into the server and begin monitoring.

![06 - TPotRunning](https://github.com/user-attachments/assets/4b5dd443-d7ec-4848-aa5f-f0006210dbf2)

7. The first tool on the list, and the one I first made use of, was the Attack Map. Initially, there was no traffic, as this was a fresh site that had never been advertised. After a short time, though, the crawlers began coming. They came from all over the place! The most popular visitor I had, at least as far as countries go, was the US, as that's where the server was hosted. However, I also had traffic from Turkey, Norway, China, Hong Kong, Malaysia, and many, many other countries. Each time someone hit the node, it recorded their IP address, country of origin, and more.

![07 - AttackMap](https://github.com/user-attachments/assets/eb963986-d697-4260-9827-ea129ba6711c)

8. At random, I chose one of the IP addresses and ran it through the Abuse IP Database (AbuseIPDB) to see if it was malicious, and it definitely was. The server originated in Hong Kong and AbuseIPDB rated it malicious with a 100% certainty. It had been reported over 4,000 times!

![08 - AbuseIPDB](https://github.com/user-attachments/assets/95e53022-9fdf-4c52-87e8-6732189d7776)

9. The next tool I used was [Elasticvue](https://elasticvue.com/features). This tool served for more in-depth log analysis. Before, I could see what services were being hit, which countries those hits were originating from, the IP addresses of visitors, and more, but all the information was disjointed. It was more of an aggregator and simply kept counts. Elasticvue paired all that information together, so that, line by line, you could see all of that information in correlation. One thing I noticed was that someone had tried to log into the server using basic credentials. It showed the country of origin of those login attempts, their IP provider information, and even credentials used.

![09 - Elasticvue](https://github.com/user-attachments/assets/d1fee6e5-c599-46f7-bb22-0b6546e7b26e)

![10 - Failed Logins](https://github.com/user-attachments/assets/060c0589-8782-4799-95cd-2e939dfa8525)

10. After spending some time reading through all the different log data, I backed out and ran [Kibana](https://www.elastic.co/kibana). This neat tool aggregated all the data from both of the previous tools and put it together in a grid format with graphics to visualize traffic impacts. There were many different types of software running on the honeypot, and different crawlers were visiting different subdirectories, so this tool was able to take all that data and create infographics based on that information, which made it easier to view and understand. It even gave a neat little graphic of login attempts and the credentials used. I'm sure if I had let the honeypot run for longer, we would have been able to see the more popular passwords attempted appear in larger font, indicating the most frequently used passwords by these crawler tools.

![11 - Elastic Attacks](https://github.com/user-attachments/assets/c966ae1d-7ac7-437c-90c1-675b60832ec9)

![12 - Login Attempt Info](https://github.com/user-attachments/assets/e0e1af90-8ebb-4f8d-a12d-7b1eaa2923aa)

11. Finally, the last tool used was [SpiderFoot](https://github.com/smicallef/spiderfoot). This tool is used for [OSINT](https://www.sans.org/blog/what-is-open-source-intelligence/) (Open-Source Intelligence). It allows you to input various forms of identification, such as an IP address or phone number, and get more information as to who or what is attached to that ID marker. For example, I used the Hong Kong IP address from earlier, and the data SpiderFoot returned gave the IP service provider name, their address, their CIDR range, it labeled them as suspicious, gave a description of why the address was suspicious, and the source from which they pulled all this information - AbuseIPDB.

![13 - SpiderFoot](https://github.com/user-attachments/assets/cff29d05-1b98-4ff6-a737-ce8115a86437)

12. Since all was said and done and the project was complete, the next step was to shut down the honeypot and Azure VM. There was no need to do any uninstallation, since the VM itself was being deleted and all storage was local to that machine. There was no storage volume attached that would have been needed to be cleaned up as well.

![14 - RGCleanup](https://github.com/user-attachments/assets/e8168819-55e0-454d-a27c-8332df36be17)

All in all, it was a very fun and enjoyable project. As mentioned in the intro, I was very surprised at how much traffic we generated unsolicited in a short amount of time. In about 20-30 minutes, we got over 500 hits. That's just incredible! It shows me that, even though it doesn't look like it on the surface, underneath, the internet is always moving, and whatever you put out there, you can bet your butt some crawler has come by and at least seen it, if they haven't saved it. Watch your butt out there!
