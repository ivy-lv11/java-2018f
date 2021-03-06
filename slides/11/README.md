
## Networking

<small>Java的最强能力之一</small>

<small>https://docs.oracle.com/javase/tutorial/networking/index.html</small>

🚺

---

## java.net

- 标准Java包
  + java.net.URL
  + java.net.URLConnection
  + java.net.InetAddress
  + java.net.ServerSocket
  + java.net.Socket
  + java.net.DatagramSocket
  + java.net.DatagramPacket

---

## Networking Basics

- TCP (Transmission Control Protocol) 
  + TCP is a connection-based protocol that provides a reliable flow of data between two computers.

- UDP (User Datagram Protocol) 
  + UDP is a protocol that sends independent packets of data, called datagrams, from one computer to another with no guarantees about arrival. UDP is not connection-based like TCP.

---

## Networking Basics

- Port
  + The TCP and UDP protocols use ports to map incoming data to a particular process running on a computer.
  + 0-65535

- Networking Classes in the JDK
  + URL, URLConnection, Socket, and ServerSocket
  + DatagramPacket, DatagramSocket, and MulticastSocket

---

## URL

- URL is the acronym for Uniform Resource Locator. It is a reference (an address) to a resource on the Internet. 

- A URL takes the form of a string that describes how to find a resource on the Internet. URLs have two main components: the protocol needed to access the resource and the location of the resource.
  + Protocol: HTTP, FTP, Gopher, File, News
  + Resource name: Host Name; File Name; Port Number; Reference

---

## An Example

- Read from an URLConnection

```java
import java.net.*;
import java.io.*;

public class URLConnectionReader {
    public static void main(String[] args) throws Exception {
        URL oracle = new URL("http://www.oracle.com/");
        URLConnection yc = oracle.openConnection();
        BufferedReader in = new BufferedReader(new InputStreamReader(
                                    yc.getInputStream()));
        String inputLine;
        while ((inputLine = in.readLine()) != null) 
            System.out.println(inputLine);
        in.close();
    }
}
```

---

## Socket

- Socket
  + A socket is one endpoint of a two-way communication link between two programs running on the network. A socket is bound to a port number so that the TCP layer can identify the application that data is destined to be sent to.
  + Usage: java.net.Socket; java.net.ServerSocket

- URLs are relatively high-level connection to the Web.

---

## Example

- Server

```java
import java.net.*;
import java.io.*;

public class EchoServer {
    public static void main(String[] args) throws IOException {
        
        if (args.length != 1) {
            System.err.println("Usage: java EchoServer <port number>");
            System.exit(1);
        }
        
        int portNumber = Integer.parseInt(args[0]);
        
        try (
            ServerSocket serverSocket =
                new ServerSocket(Integer.parseInt(args[0]));
            Socket clientSocket = serverSocket.accept();     
            PrintWriter out =
                new PrintWriter(clientSocket.getOutputStream(), true);                   
            BufferedReader in = new BufferedReader(
                new InputStreamReader(clientSocket.getInputStream()));
        ) {
            String inputLine;
            while ((inputLine = in.readLine()) != null) {
                out.println(inputLine);
            }
        } catch (IOException e) {
            System.out.println("Exception caught when trying to listen on port "
                + portNumber + " or listening for a connection");
            System.out.println(e.getMessage());
        }
    }
}

```

---

## Example

- Server-side programming

 +  Create a server socket and a common socket
 +  Enable the server socket listen
 +  Create an input stream and output steam for this common socket
 +  Read from the input stream or write to the output stream.
 +  Close all objects

---

## Example

- Client

```java

import java.io.*;
import java.net.*;

public class EchoClient {
    public static void main(String[] args) throws IOException {
        
        if (args.length != 2) {
            System.err.println(
                "Usage: java EchoClient <host name> <port number>");
            System.exit(1);
        }

        String hostName = args[0];
        int portNumber = Integer.parseInt(args[1]);

        try (
            Socket echoSocket = new Socket(hostName, portNumber);
            PrintWriter out =
                new PrintWriter(echoSocket.getOutputStream(), true);
            BufferedReader in =
                new BufferedReader(
                    new InputStreamReader(echoSocket.getInputStream()));
            BufferedReader stdIn =
                new BufferedReader(
                    new InputStreamReader(System.in))
        ) {
            String userInput;
            while ((userInput = stdIn.readLine()) != null) {
                out.println(userInput);
                System.out.println("echo: " + in.readLine());
            }
        } catch (UnknownHostException e) {
            System.err.println("Don't know about host " + hostName);
            System.exit(1);
        } catch (IOException e) {
            System.err.println("Couldn't get I/O for the connection to " +
                hostName);
            System.exit(1);
        } 
    }
}

```

---

##  Example

- Client-side programming

 + Open a socket.
 + Open an input stream and output stream to the socket.
 + Read from and write to the stream according to the server's protocol.
 + Close the streams.
 + Close the socket.

---

## Connection-oriented programming

![](images/SocketProgramming.png)<!-- .element height="60%" width="60%" -->


---

## Support multiple clients

```java
while (true) {
    //accept a connection;
    //create a thread to deal with the client;
}
```

<span style="color:#0099ff">Multi-thread!</span><!-- .element: class="fragment" -->


---

## Datagram

- Datagram
  + A datagram is an independent, self-contained message sent over the network whose arrival, arrival time, and content are not guaranteed.
  + The delivery of datagrams to their destinations is not guaranteed. Nor is the order of their arrival.

---

## Example

- Server

```java

import java.io.*;

public class QuoteServer {
    public static void main(String[] args) throws IOException {
        new QuoteServerThread().start();
    }
}

```

---

## Example

- Server

```java

import java.io.*;
import java.net.*;
import java.util.*;

public class QuoteServerThread extends Thread {

    protected DatagramSocket socket = null;
    protected BufferedReader in = null;
    protected boolean moreQuotes = true;

    public QuoteServerThread() throws IOException {
	      this("QuoteServerThread");
    }

    public QuoteServerThread(String name) throws IOException {
        super(name);
        socket = new DatagramSocket(4445);

        try {
            in = new BufferedReader(new FileReader("one-liners.txt"));
        } catch (FileNotFoundException e) {
            System.err.println("Could not open quote file. Serving time instead.");
        }
    }

    public void run() {

        while (moreQuotes) {
            try {
                byte[] buf = new byte[256];

                // receive request
                DatagramPacket packet = new DatagramPacket(buf, buf.length);
                socket.receive(packet);

                // figure out response
                String dString = null;
                if (in == null)
                    dString = new Date().toString();
                else
                    dString = getNextQuote();

                buf = dString.getBytes();

		// send the response to the client at "address" and "port"
                InetAddress address = packet.getAddress();
                int port = packet.getPort();
                packet = new DatagramPacket(buf, buf.length, address, port);
                socket.send(packet);
            } catch (IOException e) {
                e.printStackTrace();
		moreQuotes = false;
            }
        }
        socket.close();
    }

    protected String getNextQuote() {
        String returnValue = null;
        try {
            if ((returnValue = in.readLine()) == null) {
                in.close();
		moreQuotes = false;
                returnValue = "No more quotes. Goodbye.";
            }
        } catch (IOException e) {
            returnValue = "IOException occurred in server.";
        }
        return returnValue;
    }
}

```

---

## Example

- Client 

```java

import java.io.*;
import java.net.*;

public class QuoteClient {
    public static void main(String[] args) throws IOException {

        if (args.length != 1) {
             System.out.println("Usage: java QuoteClient <hostname>");
             return;
        }

            // get a datagram socket
        DatagramSocket socket = new DatagramSocket();

            // send request
        byte[] buf = new byte[256];
        InetAddress address = InetAddress.getByName(args[0]);
        DatagramPacket packet = new DatagramPacket(buf, buf.length, address, 4445);
        socket.send(packet);
    
            // get response
        packet = new DatagramPacket(buf, buf.length);
        socket.receive(packet);

	    // display response
        String received = new String(packet.getData(), 0, packet.getLength());
        System.out.println("Quote of the Moment: " + received);
    
        socket.close();
    }
}

```

---

## Broadcasting

```java
MulticastSocket socket = new MulticastSocket(4446);
InetAddress group = InetAddress.getByName("203.0.113.0");
socket.joinGroup(group);

DatagramPacket packet;
for (int i = 0; i < 5; i++) {
    byte[] buf = new byte[256];
    packet = new DatagramPacket(buf, buf.length);
    socket.receive(packet);

    String received = new String(packet.getData());
    System.out.println("Quote of the Moment: " + received);
}

socket.leaveGroup(group);
socket.close();

```

---

## HTTP Client APIs

- Package: com.oracle.httpclient

- Goal: a Java based framework for communication with Web Services.

---

## HTTP CLient APIs

- easy way of creating and configuring of HTTP requests
- synchronous and asynchronous executing of request
- performing request pre-processing and response post-processing.

---

## Creating and configuring HTTP request

```java

HttpClient client = HttpClientBuilder.getInstance()
                .addRequestFilter(new UserRequestFilter())
                .addResponseFilter(new UserResponseFilter())
                .addConnectionOptions(new ConnectionOption<>("Timeout", 2000))
                .build();

HttpRequest clientRequest = client.build("http://example.org")
                .setHeader(HttpHeader.ACCEPT, "text/plain")
                .setMethod(HttpMethod.GET)
                .build();

```

---

## Request invocation

```java

HttpResponse response = clientRequest.invoke();
        if (HttpResponse.OK == response.getResponseCode()) {
            // handle the response
        }
        

clientRequest.invoke(
                new HttpResponseListener() {
                    @Override
                    public void handle(HttpResponse response) {
                        // handle the response
                    }
                    @Override
                    public void failed(HttpClientException cause) {
                        // handle the exception
                    }
        });

```

---

## Filters

- Filters are used in order to provide capabilities as logging, authentication, etc.

```java
public class UserAgentFilter implements HttpRequestFilter {
            private final String userAgent;

            public UserAgentFilter() {
                userAgent = "Profile/" + System.getProperty("microedition.profiles") +
                            " Configuration/" + System.getProperty("microedition.configuration");
            }

            @Override
            public void filter(ProcessedHttpRequest request, HttpRequestProcessingContext processingContext) {
                if (!request.getHeaders().contains(HttpHeader.USER_AGENT)) {
                    request.getHeaders().setHeaderValue(HttpHeader.USER_AGENT, userAgent);
                }
            }
        }

        HttpClientBuilder clientBuilder = HttpClientBuilder.getInstance()
                .addRequestFilter(new UserAgentFilter());


```

---

让小伙伴们的葫芦娃联网战斗吧！

---

# END