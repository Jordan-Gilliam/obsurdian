## What does cURL mean?

cURL (client URL) is an open source command line tool and a cross-platform library (_libcurl)_ used to transfer data between servers, distributed to nearly all new operating systems. cURL programming is used almost everywhere where sending or receiving data through internet protocols is required.

> cURL supports nearly every internet protocol (DICT, FILE, FTP, FTPS, GOPHER, HTTP, HTTPS, IMAP, IMAPS, LDAP, LDAPS, MQTT, POP3, POP3S, RTMP, RTMPS, RTSP, SCP, SFTP, SMB, SMBS, SMTP, SMTPS, TELNET and TFTP). 

## What does cURL do?

cURL is intended to transfer data through internet protocols. Everything else is outside its scope. It doesn’t even handle the data transferred, only performs the process itself.   

## Using cURL

### Sending requests

As cURL has initially been developed for HTTP, we can send all the usual requests (POST, GET, PUT, etc). In order to send a POST request to a URL, the -d (or –data) flag is used. 

>Most websites will deny such requests from unauthorized users, therefore we will use a [fake API for testing purposes](https://jsonplaceholder.typicode.com/). 

```bash
curl -d “name=something&value=somethingelse” https://jsonplaceholder.typicode.com/posts/
```

Sending such a request should return:

```json
{
  "name": "something",
  "value": "somethingelse",
  "id": 101
}
```

There are a few things at play here:  

-   curl begins our command
-   \-d is the “data” flag for POST requests
-   Quote marks (“”) begin our content statement. Note that some operating systems will only accept single quotes, others will accept double quotes.
-   Finally, the destination. URL syntax should always be exact as cURL doesn’t automatically follow redirects.

We can also send POST requests in the JSON format, although additional options will have to be provided in order to tell the server that we are sending a JSON. cURL does little interpretation on behalf of the user and would send the default _Content-Type_ header of _application/text_, so we have to add the header _Content-Type: application/json ourselves_. 

```bash
curl -H "Content-Type: application/json" --data "{\"data\":\"some data\"}"  https://jsonplaceholder.typicode.com/posts/
```

### Following redirects

cURL doesn’t automatically follow redirects. We should add an additional flag if we expect it to do so. Let’s take a look at an example:

```bash
curl https://google.com
```

Our browsers handle redirects by themselves, so we might not even notice an issue with such a request. Yet, if we send cURL in to do the work, we will receive a notice that the document has been moved as it tries to connect. To get cURL to follow redirects, we have to add a special flag “-L” (arguments are case sensitive).  

```bash
curl -L https://google.com
```

We should now have received the regular answer from Google as cURL follows the redirect from [https://google.com](https://google.com/) to [https://www.google.com](https://www.google.com/)[/](https://www.google.com/).

### Connecting through a proxy

cURL can be used to connect to any destination through a proxy. As with any other cURL statement, the URL, syntax and everything else stay the same except for the added flag and its attributes.

```bash
curl --proxy proxyaddress:port https://jsonplaceholder.typicode.com/
```

Entering the proxy and port after “–proxy” will route the connection through the inputted address. Proxies will often require credential details that can be sent through the -U flag.

```bash
curl --proxy proxy:port -U “username:password” https://jsonplaceholder.typicode.com/
```

Some websites will require authentication by themselves before they accept any connection request. A similar flag is used for server authentication: “-u”.

```bash
curl -u username:password https://jsonplaceholder.typicode.com/
```

#unix #bash