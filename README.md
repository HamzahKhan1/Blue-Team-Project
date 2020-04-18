# **Blue-Team-Project**

## **Background**
In this scenario, you're analyzing a snort log in Wireshark to determine details surrounding a security breach.

## **Objectives**
As provided by instructors:

### **How long did the attack last?**
- Roughly 9 minutes, looks like a Brute Force password attack based on this pic:

### **How many password attempts were made?**
- First, comb through the packets. There’s quite a high amount of HTTP GET requests going on, all from the same source IP to the same destination IP. Looking at one, we see the request URI is `/company_folders/secret_folder`, and then a username and password credential that has been attempted - definitely a red flag.
- In the User-Agent field, we see `Mozilla/4.0 (Hydra) \r\n`. Hydra sounds like the brute force tool, and we don’t like that, but now we have a term we can filter by.
- We want to see the total amount of requests of this kind made, so first, we need to narrow down the results to just these requests. In the dynamic filter bar, since we know the User-Agent involves the word Hydra, we can use the expression `http.user_agent matches “(Hydra)”` to pull up all of the requests. 
- 1. Note that using `==` messes with the syntax. For this kind of request, you’ll need to use `matches` and put the `(Hydra)` in double quotes.
- Now that we have all of these in one window, we can visit `Statistics>HTTP>Requests` and we see a window like this (picture) that displays **10143** HTTP Requests by the HTTP Host.

### **In which packet was the correct password found?**
- From a previous Red Team project directly linked to this project, we know the password that was cracked was “leopoldo”. Operating on this knowledge, we want to narrow down our search results to packets that contain this word. We can do so by using the `http.authbasic` filter and setting it equal to “Leopoldo”. As you can see, this gets us 3 packets, all with the same credentials. We’ll go with packet 60789, since it was the earliest packet in which the correct password can be found.

### **In which packet was the shell placed onto the server?**
- We’re narrowing our searches down to the HTTP protocol, since the brute force attack involved a lot of GET requests in which different credentials were tested against the target login. 
- This time, we can filter by `http contains shell.php`, 
- We see 10 results. The differences between them are `PROPFIND`, `GET`, `DELETE`, and `PUT`.
- Web Distributed Authoring and Versioning (WebDAV) is an extension of the Hypertext Transfer Protocol (HTTP) that facilitates collaboration between users in editing and managing documents and files stored on World Wide Web servers.
- PROPFIND — used to retrieve properties, stored as XML, from a web resource. It is also overloaded to allow one to retrieve the collection structure (a.k.a. directory hierarchy) of a remote system.
- GET, DELETE, and PUT are all HTTP methods. GET requests data from the server, DELETE deletes a specified resource, and PUT replaces a target resource with a request payload. We see only one request has a PUT in the info column, packet **60982**.

### **In which packet was the shell activated?**
- In the same results page, we see that the GET request happens after the PUT request, in packet **61010**. Therein lies our answer. 

### **Remediation Strategies**
- Select packet. `Tools>Firewall ACL Rules`. Create rules for Netfilter (iptables), copy, and paste to your firewall to run the rule.
