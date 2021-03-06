# Roll-Your-Own-CDN-using-Amazon-AWS
A prototype of Content Delivery Network using Amazon Servers

INTRODUCTION
============

In this project, we created the building blocks of a CDN.

A content delivery network consist of 

1) a large number of servers geographically distributed worldwide; 
2) a system that maps clients to "good" replica servers and 
3) a system that determines those mappings. 
In this project, we implemented the basic functionality in each of these areas.

HIGH LEVEL APPROACH
====================

A) DNS Server:
------------------
We implemented the dns server which takes in dns query requests from the client and 
reponds with the IP address of the closest replica to the client.

1) We first created a UDP socket and the socket was bound to the port (a value greater than 40000 and less than 65535) 
specified in the command line argument. 
2) The socket listens for a DNS query from the client. After receiving the query, 
the code parses through the DNS query and extracts the Transaction ID, flags from the header and domain name, 
type and class from the question part. 
3) The parsing logic extracts the domain name queried in the question. This domain name is compared with the 
domain name specified in the command line argument after the -n switch.
4) If the domain name queried is same as the one specified, then then DNS response packet is constructed 
with the IP address of the domain name. This is done using the pack function.
5) The DNS response packet consists of Headers, Query, and Answers. 
The headers contain transaction ID, flags, number of questions, number of answers, number of authoritative answers, 
and number of additional answers. 
6) The Query consists of domain name, type and class. The answer section contains the Name, type, class, ttl, length and IP address. 
After packing all this according to the network byte order, the packet is sent from the UDP socket to the client. 
7) In order to provide the best service to the client, we have used the geo-location, keeping into consideration that the the replica server
that is close to the client will provide quick service as compared to the other replica server.  

IP Geolocation:
--------------------
We use IP location service of IPInfoDB.com. After registering an account, we get an api key.
We can do a query for the location of a given IP address using the url:
http://api.ipinfodb.com/v3/ip-city/?key=<api_key>&ip=<ip address>

The query result contains the latitude and longitude of the location of the given IP address. 
It is easy to calculate the distance between two location. So we can find the closest replica server to the client.


B) HTTP Server:
-----------------------------
We implemented the web server which takes in http requests from the client and reponds with requested web object to the client.

1) We imported various libraries required like socket, sys, urlparse, urllib, os, sqlite3, zlib, and threading
2) We first created an UDP socket and connected to a server to get the IP address of the HTTP server 
that is the IP of the local machine on which the code runs.
3) We defined the port number ( a value greater than 40000 and less than 65535) and the name of the origin server 
from command line arguments specified.
4) We created a TCP socket for HTTP server (that is the local machine) and bind it to the port number specified in the command line argument.
5) We extracted the path from the client HTTP query and forwarded it to the Origin server 
using GET HTTP request to the fetch the data stored on the origin server.
6) The HTTP server will forward the fetched data to the client using HTTP response packet.

Cache Management:
----------------------
We improved the Http server performance by Caching the web content. 

1) For every fresh request, the server requests the object from the origin server and creates a copy of the object in the cache database.
2) It then sends a copy to the client. For every subsequent request, the server checks for the copy in the cache database. 
If found, returns it from the cache, else returns the object from the origin server.
3) To maintain the cache database, we used sqlite3 library to store the content.
4) Cache limit was set to 10MB. Therefore we used zlib to compress the data and store it into the cache file
5) We managed to maintain the record of the hit counts i.e. to check the popularity of the page 
by counting the number of times the page was requested
6) If the cache memory becomes full, the content with the least number of hits was discarded from the cache 
and the newly requested content was added to the cache database


C) Deploy, Run and Stop Scripts for CDN
----------------------------------------
We wrote scripts to automate the process of deployment, starting the servers at the replicas and shutting down the servers.
 
1) Each script follows the following input
	./[deploy|run|stop]CDN -p <port> -o <origin> -n <name> -u <username> -i <keyfile>
2) Each of the scripts have a list of replica servers defined, which is assumed to be a constant
3) Each of the scripts have dns server defined as cs5700cdnproject.ccs.neu.edu
4) For deploying our code into the replicas we have used scp and ssh and gave the execution permission to each replicas.  


TESTING
=======
1) The DNS server is tested using dig command of the format - 
	dig @dns_server_ip -p port_no -n cs5700cdn.example.com
2) The HTTP server is tested using the wget command of the format -
	wget http://(HTTP server ip):(port number)/ path of content

Future scope
=============
1) Server load at all replicas and RTT could be calculated to find the best replica which would improve client percieved performance.
2) Caching could be improved by using ranking and other algorithms.
3) DHT's could be used to store the contents as key value store to check content availability at the http servers.


 PERFORMANCE ENHANCING TECHNIQUES & DECISIONS
=============================================
1)  Used the technique of Geolocation to find the closest replica to the client 
and avoided unnecessary bandwidth and time usage caused by calculation of RTT.   
1)  Used sqlite for faster retrieval of files instead of a python dictionary
2)  Used zlib for compressing the file and storing it to occupy less space
3)  Used threads to tackle simultaneous requests at the application level

 CHALLENGES FACED
===================
 - Contructing the DNS responses with precision was tricky since a good understanding of DNS reponse packet was necessary
 - Calculation of the closest replica by maintaining a dictionary was challenging 
 - Keeping track of the disk usage and setting a limit on the same while inserting the page into cache was tricky
 - Deciding on what persistent mechanism to use for the cache was a challenge in the beginning since we had to make sure that the db
 file gets stored in the current directory.
 - Mapping the links with their hit count and deleting the pages with lowest hit count was tricky
 - ssh for remotely executing commands required a good understanding of how background processes are executed in linux

 FURTHER ENHANCEMENTS POSSIBLE:
===============================
 - Related to cache management:
        - Along with page count, frequency at which the pages are requested can be considered as the criteria for the most popular page.
	- If two pages are of similar popularity, the page with higher size stays if there is an issue with the max memory
	- If a page is requested, subsequently the hyperlinks present in the page could also be cached.
	- Based on the location, the current affairs and the popularity of certain pages, 
	  the pages can be pre-loaded in the cache before the request comes in.

 - Related to DNS redirection:
	- the amount of load on a server could have been used to rank the best available server
	- the TTL can be used judiciously to prevent client from sending frequent requests
	- Usage of Database for storing precalulated best available servers for clients


INDIVIDUAL CONTRIBUTION:
==========================

Riteshkumar Gupta: NU ID:001280361

1) Implemented the handling of query and response DNS in the DNS server script.
2) Implemented the cache management, compression of the content, and the popularity in HTTP server script
3) Wrote the runCDNscript.
4) Wrote the stopCDN script.

Prajwal Patil: NU ID: 001280390

1) Implemented the computation of best replica in DNS server script. 
2) Implemented the HTTP server script and handled request, parsing and sending response.
3) Implemented multi threading to DNS Server and Http Server
3) Wrote the DeployCDN script.
