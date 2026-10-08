Preliminary Cybersecurity Practical

PrestaShop Deployment on Microsoft Azure

1. Introduction

PrestaShop is an open-source e-commerce platform that can be deployed on cloud infrastructure.

For this practical, PrestaShop was deployed on Microsoft Azure using a two-tier architecture. The application and database were hosted on separate virtual machines, as required by the assignment.

The deployment consisted of:

* An Application Server running PrestaShop, Apache and PHP.
* A separate Database Server running MariaDB.
* A private Azure Virtual Network connecting both servers.
* A public IP address for accessing the PrestaShop storefront.
* Network Security Group rules controlling access to the servers.

The database was deliberately kept separate from the application server to improve network segmentation and reduce direct exposure of the database service.

⸻

2. Deployment Architecture

The final architecture was:

                         INTERNET
                            │
                            │ HTTP :80
                            ▼
                 ┌──────────────────────┐
                 │  PrestaShop VM        │
                 │  Ubuntu 24.04 LTS    │
                 │  Apache 2.4          │
                 │  PHP 8.3             │
                 │  PrestaShop 9.1.5    │
                 │                      │
                 │ Public IP             │
                 │ 9.160.105.142        │
                 │                      │
                 │ Private IP            │
                 │ 172.16.0.4           │
                 └──────────┬───────────┘
                            │
                            │ TCP 3306
                            │ Private Network
                            ▼
                 ┌──────────────────────┐
                 │  MariaDB VM          │
                 │  Ubuntu 24.04 LTS    │
                 │  MariaDB 10.11       │
                 │                      │
                 │ Private IP            │
                 │ 172.16.0.5           │
                 │                      │
                 │ Database              │
                 │ prestashop_db         │
                 └──────────────────────┘

The architecture separates the application layer from the database layer.

⸻

3. Azure Resources

3.1 PrestaShop Application Server

The first virtual machine was created to host the PrestaShop application.

Configuration	Value
VM Name	prestashop-vm
Operating System	Ubuntu Server 24.04 LTS
VM Size	Standard B2ats_v2
vCPUs	2
RAM	1 GiB
Region	Belgium Central
Availability Zone	Zone 2
Private IP	172.16.0.4
Public IP	9.160.105.142

The application server hosts Apache, PHP and PrestaShop.

⸻

3.2 Database Server

A second virtual machine was created specifically for the database.

Configuration	Value
VM Name	prestashop-db-vm
Operating System	Ubuntu Server 24.04 LTS
VM Size	Standard B2ats_v2
vCPUs	2
RAM	1 GiB
Region	Belgium Central
Private IP	172.16.0.5
Database Server	MariaDB 10.11
Database Port	3306

The database server does not host the PrestaShop application.

⸻

4. Azure Virtual Network

Both virtual machines were connected to the same Azure Virtual Network.

Resource	Configuration
Virtual Network	vnet-belgiumcentral-1
Subnet	snet-belgiumcentral-1
Application VM	172.16.0.4
Database VM	172.16.0.5

The Virtual Network allows the two servers to communicate using their private IP addresses.

The application therefore connects to MariaDB using:

172.16.0.5:3306

rather than connecting through a public database address.

⸻

5. Application Server Configuration

The application server was configured with Ubuntu Server 24.04 LTS.

The required web-server software was installed:

* Apache 2.4
* PHP 8.3
* Required PHP extensions
* PrestaShop 9.1.5

Apache was configured to serve the PrestaShop application over HTTP.

The Apache service was checked using:

sudo systemctl status apache2 --no-pager

The service returned:

Active: active (running)

This confirmed that Apache was running successfully.

⸻

6. Database Server Configuration

The database server was configured separately from the application server.

MariaDB 10.11 was installed on the database VM.

A dedicated database was created:

Database: prestashop_db

A dedicated database user was also created:

User: prestashop_user

The application was configured to connect to:

Database server: 172.16.0.5
Database port:   3306
Database name:   prestashop_db
Database user:   prestashop_user

This ensured that the PrestaShop application and database were hosted independently.

⸻

7. Network Security Configuration

Azure Network Security Groups were used to control access to the virtual machines.

7.1 Application Server

HTTP traffic was permitted on port 80.

Setting	Value
Protocol	TCP
Port	80
Source	Any
Action	Allow

This allowed users on the Internet to access the PrestaShop storefront.

⸻

7.2 Database Server

MariaDB uses TCP port 3306.

Access to this port was restricted to the application server.

Source:      172.16.0.4
Destination: 172.16.0.5
Protocol:    TCP
Port:        3306

The database server was therefore not intended to accept database connections directly from the public Internet.

⸻

8. Database Connectivity Test

After configuring MariaDB, connectivity from the application server was tested.

The following command was used:

mysql -h 172.16.0.5 -u prestashop_user -p -e "SHOW DATABASES;"

The command returned:

+--------------------+
| Database           |
+--------------------+
| information_schema |
| prestashop_db      |
+--------------------+

This confirmed that:

1. The application server could reach the database server.
2. Communication was occurring through the private network.
3. The database user was able to authenticate.
4. The prestashop_db database was accessible.

⸻

9. PrestaShop Installation

PrestaShop version 9.1.5 was installed on the application server.

During the installation process, the following database information was provided:

Database server: 172.16.0.5
Database name:   prestashop_db
Database user:   prestashop_user
Database port:   3306

The PrestaShop installation wizard successfully connected to the MariaDB database hosted on the separate database VM.

The installation was completed successfully.

⸻

10. Public Access to PrestaShop

The application VM was assigned the public IP address:

9.160.105.142

The PrestaShop storefront was accessed through:

http://9.160.105.142

The request flow was:

Internet
   │
   │ HTTP :80
   ▼
9.160.105.142
   │
   ▼
PrestaShop VM
172.16.0.4
   │
   │ TCP :3306
   ▼
MariaDB VM
172.16.0.5

This demonstrated that the application was publicly accessible while the database remained on a separate server.

⸻

11. Website Verification

The application was tested from the application server using:

curl -I http://9.160.105.142

The server returned:

HTTP/1.1 200 OK

The response also identified the web server as:

Server: Apache/2.4.58 (Ubuntu)

The 200 OK response confirmed that the web server was responding successfully.

The PrestaShop storefront was also opened through a web browser using:

http://9.160.105.142

The storefront loaded successfully.

⸻

12. Ubuntu Firewall Check

The Ubuntu firewall status was checked using:

sudo ufw status

The result was:

Status: inactive

Azure Network Security Groups were used to control network access to the virtual machines.

⸻

13. Security Considerations

Several security considerations were incorporated into the deployment.

Separate Application and Database Servers

The database was not installed on the same server as PrestaShop.

This provides separation between the application and database layers.

Private Database Communication

The application connects to MariaDB using the database VM’s private IP:

172.16.0.5

Restricted Database Port

MariaDB’s port 3306 was restricted to communication from the application server.

No Intended Public Database Access

The MariaDB service was not intended to be exposed directly to the public Internet.

SSH Authentication

SSH authentication was used for server administration.

Network Segmentation

The two virtual machines were connected through an Azure Virtual Network, allowing private communication between the application and database layers.

⸻

14. Deployment Verification

The following components were successfully implemented and tested:

* Azure virtual machines deployed.
* Application and database servers separated.
* Azure Virtual Network configured.
* Private IP communication established.
* Apache installed and running.
* PHP configured.
* PrestaShop 9.1.5 installed.
* MariaDB 10.11 installed.
* prestashop_db database created.
* prestashop_user database account created.
* Application-to-database connectivity verified.
* MariaDB communication performed through the private network.
* Azure Network Security Group configured.
* HTTP port 80 configured.
* PrestaShop storefront made publicly accessible.
* HTTP request returned 200 OK.

⸻

15. Public PrestaShop URL

The completed PrestaShop installation was accessible at:

http://9.160.105.142

This URL provides public access to the PrestaShop storefront hosted on the Azure application server.

⸻

16. Screenshots and Evidence

The following screenshots document the implementation and configuration of the Azure deployment.

The evidence includes relevant Azure resources, server configuration, network configuration, database configuration, installation steps and verification of the working PrestaShop application.

Screenshots should be placed in chronological order in the final submission to demonstrate the implementation process.

⸻

17. Conclusion

This practical demonstrated the deployment of an open-source PrestaShop application on Microsoft Azure using a two-tier cloud architecture.

The application server and database server were deployed on separate virtual machines. The application server hosted Apache, PHP and PrestaShop, while the database server hosted MariaDB.

The two servers communicated through a private Azure Virtual Network, with MariaDB accessed using the database server’s private IP address.

The final deployment successfully provided a publicly accessible PrestaShop storefront while maintaining separation between the application and database layers.

The architecture demonstrates fundamental cloud and cybersecurity principles including network segmentation, service separation, restricted database access and controlled public exposure.

||
Author 
Ismail Victor Elias 