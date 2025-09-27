# HTTP Analysis – CN Assignment no 1

## Q1. What is the name of website?
The website accessed is identified by the **Host header** in the GET request:  
`kurosesross.lms.php`  

---

## Q2. Find the packet that contains the first GET request.
The first GET request appears in **Packet No. 72**:  


---

## Q3. Describe all headers and their values in this GET request.
The headers in the GET request are:

- **Host:** kurosesross.lms.php  
- **User-Agent:** Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:143.0) Gecko/20100101 Firefox/143.0  
- **Accept:** text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8  
- **Accept-Language:** en-US,en;q=0.5  
- **Accept-Encoding:** gzip, deflate  
- **Connection:** keep-alive  
- **Upgrade-Insecure-Requests:** 1  

---

## Q4. Identify the status code in the first server response.
The first response from the server contains the status code:  
**200 OK**

---

## Q5. How many HTTP response messages are exchanged in total?
By filtering with `http.response` in Wireshark, we observe multiple response messages.  
- First response: `200 OK (text/html)`  
- Second response: `200 OK (text/css)`  
- (Additional responses may appear for other resources like images or scripts)  

**Total responses observed: 2 (minimum in this trace).**

---

## Q6. Determine whether the connection is persistent or not.
The connection is **persistent**, because:
- The request includes the header `Connection: keep-alive`.  
- Multiple HTTP responses (HTML + CSS) were exchanged over the same TCP connection without reconnecting.  

**Conclusion:** The HTTP connection is peristent.

