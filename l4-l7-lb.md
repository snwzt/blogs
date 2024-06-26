---
image:
    path: assets/images/og_image.png
    width: 300
    height: 169
---

# Why can't Classic Load Balancer provide functionality of Application Load Balancer?
If you have ever used any sort of Load Balancer in AWS, Azure or maybe you used Kubernetes CCM to create objects like Ingress or LoadBalancer service, you might've realized there is not one type of load balancer, there are mainly two type of Load Balancers, one being Application Load Balancer (known as Ingress resource in Kubernetes, implemented by Traefik or Nginx), and other being the Classic Load Balancer (the loadbalancer service).

Comparision in brief:

| Classic Load Balancers                | Application Load Balancers          |
|---------------------------------------|------------------------------------|
| Layer 4 (Transport Layer) | Layer 7 (Application Layer) |
| Basic load balancing functionality | Host based and Path based routing |
| Basic algorithms (round-robin, least connections) | Advanced algorithms (content-based routing, SSL termination) |
| Minimal, Hence highly performant | Comparatively higher resource consumption |
| Limited security features | Enhanced security features (content-based firewalling, DDoS protection) |

But even after looking at this table, it bugged me for days, why ingress object allows me to do path based and host based routing, but not the load balancer serivce in kubernetes? 

**Remember the OSI Model?**

So, at Layer 4 the **packet** (from Layer 3) which is now known as a **segment** looks something like this:

| Field             | Size (bytes) | Description                                                   |
|-------------------|--------------|---------------------------------------------------------------|
| Source Port       | 2            | Identifies the sender's port number                           |
| Destination Port  | 2            | Identifies the recipient's port number                        |
| Sequence Number   | 4            | Specifies the sequence number of the first data byte in this segment |
| Acknowledgment Number | 4        | Indicates the next sequence number the sender of the segment is expecting to receive |
| Data Offset       | 4 bits       | Specifies the number of 32-bit words in the TCP header         |
| Reserved          | 6 bits       | Reserved for future use                                        |
| Flags             | 6 bits       | Various control flags such as SYN, ACK, FIN, RST, etc.        |
| Window Size       | 2            | Indicates the size of the sender's receive window             |
| Checksum          | 2            | Provides error-checking for the header and data fields         |
| Urgent Pointer    | 2            | Indicates the end of the urgent data (if present)             |
| Options           | Variable     | Optional field for additional TCP options                      |
| Padding           | Variable     | Padding to ensure the header ends on a 32-bit boundary        |
| Data              | Variable     | Actual data being transmitted                                  |

But at Layer 7, the data is called a message, which looks something like this: (e.g. a Request Message)
                                    
| **Request Line**| Description                                                 |
|-----------------|-------------------------------------------------------------|
| Method          | GET, POST, PUT, DELETE, etc.                                |
| URI             | Uniform Resource Identifier                                  |
| HTTP Version    | Version of the HTTP protocol being used                      |

| **Headers**     | Description                                                 |
|-----------------|-------------------------------------------------------------|
| Host            | Specifies the host and port number of the server             |
| User-Agent      | Identifies the client making the request                     |
| Accept          | Specifies the media types that are acceptable for the response |
| Content-Type    | Specifies the media type of the request body (for POST and PUT requests) |
| Others          | Additional headers as required                               |

| **Body**        | Description                                                 |
|-----------------|-------------------------------------------------------------|
| Content         | Contains the data being sent with the request (if applicable) |
| Example         | Form data, JSON payload, or file content                    |

*The three tables above are part of a single message.*

Did you see that there is no mention of Host, Path or Method at Layer 4? Well, as you can see there isn't, so how is it supposed to even do the fancy load balacing that Application Load Balancer does? Exactly. While the Classic Load Balancers excel at efficiently distributing traffic based on IP addresses and port numbers, Application Load Balancers route requests based on application-level data.

Probably in the realm of Classic Load Balancer, story goes like this:

1. The packet gets rid of its IP address at Layer 3, becomes a segment and is sent to Layer 4.
2. The Load Balancer at Layer 4 determines the target server.
3. The packet is reassembled with the target server IP (DNAT) and sent to the destination.

I realized my stupidity and went on with my life with this new **obvious** knowledge in my head, which probably should've been there already.

Thankyou for reading my blog.