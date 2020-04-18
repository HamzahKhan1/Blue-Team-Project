# **Blue-Team-Project**

## **Background**
Analyzing a snort log in Wireshark to determine details surrounding a security breach.

## **Tools**
We'll be using **Wireshark** to analyze the snort log.

## **Objectives**

### **How long did the attack last?**
1. Open the file in Wireshark.
![1](https://user-images.githubusercontent.com/55573209/79670937-fccdbb80-818b-11ea-92ad-bcadf40eb264.png)

2. To find the length of the attack, we can trace the time that passed between the first and final packets in the PCAP file. Go to `View>Time Display Format>Time of Day` to display the time of each packet in a more understandable format.

3. Examine the first packet and last packet. Based on the time that lapsed between them, it looks like the attack lasted roughly 9 minutes.
![2](https://user-images.githubusercontent.com/55573209/79670940-03f4c980-818c-11ea-9a31-a11668dac083.png)
![3](https://user-images.githubusercontent.com/55573209/79670942-08b97d80-818c-11ea-9381-f6b6f36b39a3.png)

### **How many password attempts were made?**
1. Comb through the packets in the capture. There’s quite a high amount of HTTP GET requests going on, all from the same source IP to the same destination IP. Looking at any of them in the bottom pane, we see the request URI is `/company_folders/secret_folder`, and then we see a username and password credential that has been attempted - definitely a red flag that likely indicates a brute force attack.
![4](https://user-images.githubusercontent.com/55573209/79671189-2556b500-818e-11ea-8876-d45ea4589cb2.png)

2. In the User-Agent field in the picture above, we see `Mozilla/4.0 (Hydra) \r\n`. Hydra sounds like the brute force password cracking tool, so now we have a term we can filter by.
3. We want to see the total amount of requests of this kind made. In the display filter bar, since we know the User-Agent involves the word Hydra, we can use the expression `http.user_agent matches “(Hydra)”` to narrow down our search.
- 1. Note that using `==` messes with the syntax. For this kind of request, you’ll need to use `matches` and put the `(Hydra)` in double quotes.
![5](https://user-images.githubusercontent.com/55573209/79671097-5682b580-818d-11ea-8dd0-bd27a3464a9b.png)

4. Now that we have all of these in one window, we can visit `Statistics>HTTP>Requests` and we see a window like this that displays **10143** HTTP Requests by the HTTP Host.
![6](https://user-images.githubusercontent.com/55573209/79671159-d1e46700-818d-11ea-84de-8c89522b9103.png)

### **In which packet was the correct password found?**
1. From a previous Red Team assignment directly linked to this project, we know the password that was cracked was “leopoldo”. Operating on this knowledge, we want to narrow down our search results to packets that contain this word. We can do so by using the `http.authbasic` display filter and setting it equal to “Leopoldo”. As you can see, this gets us 3 packets, all with the same credentials. We’ll go with packet **60789**, since it was the earliest packet in which the correct password can be found.
![7](https://user-images.githubusercontent.com/55573209/79671176-09531380-818e-11ea-9fd8-e9ff3b82b437.png)


### **In which packet was the shell placed onto the server?**
1. We’re narrowing our searches down to the HTTP protocol, since the brute force attack involved a lot of GET requests in which different credentials were tested against the target login. This time, we can filter by `http contains shell.php`, and we see 10 results. The differences between them are `PROPFIND`, `GET`, `DELETE`, and `PUT`.
- 1. Web Distributed Authoring and Versioning (WebDAV) is an extension of the Hypertext Transfer Protocol (HTTP) that facilitates collaboration between users in editing and managing documents and files stored on World Wide Web servers.
- 2. PROPFIND — used to retrieve properties, stored as XML, from a web resource. It is also overloaded to allow one to retrieve the collection structure (a.k.a. directory hierarchy) of a remote system.
- 3. GET, DELETE, and PUT are all HTTP methods. GET requests data from the server, DELETE deletes a specified resource, and PUT replaces a target resource with a request payload. We see only one request that has a PUT in the info column, which is packet **60982**.
![8](https://user-images.githubusercontent.com/55573209/79671258-cfced800-818e-11ea-993f-f15f49e3db00.png)

### **In which packet was the shell activated?**
1. In the same picture above, we see that the GET request happens after the PUT request, in packet **61010**. 
![9](https://user-images.githubusercontent.com/55573209/79671285-fa209580-818e-11ea-989e-ba2ad22acb73.png)

### **Remediation Strategies**
1. Select packet of interest. Go to `Tools>Firewall ACL Rules`. Create rules for Netfilter (iptables), and then copy and paste the rule to your own firewall to run it.
![10](https://user-images.githubusercontent.com/55573209/79671319-21776280-818f-11ea-84eb-ff755b81af0a.png)
