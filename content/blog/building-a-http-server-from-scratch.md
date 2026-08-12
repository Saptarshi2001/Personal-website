---
title: "Building a HTTP Server from Scratch"
subtitle: "Understanding HTTP Under the Hood"
description: "Build a small HTTP server on top of TCP in Python and explore request parsing, methods, status codes, headers, and logging."
summary: "Build a small HTTP server on top of TCP in Python and explore request parsing, methods, status codes, headers, and logging."
tags: ["Python", "HTTP", "networking"]
weight: 2
---

Hi!! In this post, we will discuss the intricacies of HTTP and how it works under the hood. In the end, we will build a basic implementation of an HTTP server. You can read more about the HTTP protocol in [RFC 9110](https://datatracker.ietf.org/doc/html/rfc9110).

## The TCP

Before we build our HTTP server, we need to build a TCP server. After all, HTTP is built on top of TCP.

```python
class TcpServer:
    host='127.0.0.1'
    port=80
    conn=0
    addr=0
    allconn=[]
    def start(self):
        sock=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
        sock.bind((self.host,self.port))
        print("listening at ",sock.getsockname())
        sock.listen(1)
        try:
            while True:
                self.conn,self.addr=sock.accept()
                t=threading.Thread(target=self.handle_client,args=(self.conn,self.addr))
                t.start()
                self.allconn.append(t)
        except socket.error:
            print("coudnt connect with the socket")
            SystemExit
        finally:
            if sock:
                sock.close()
            for t in self.allconn:
                t.join()

    def handle_client(self,conn,addr):
        data=conn.recv(1024)
        response=self.handle_request(data)
        conn.sendall(response)
        conn.close()

    def handle_request(self,data):
        return data
```

In this server, first, we create a socket using the socket module in Python. `AF_INET` shows that the socket belongs to the IPv4 address family and `SOCK_STREAM` shows that the socket is a TCP socket type. It then binds itself to host 127.0.0.1 and port 80. The socket then listens for connections. When it connects, it gets the data as well as the address of the client. What it next does is create a separate thread from the main thread with `handle_client` and `conn, addr` as its arguments. The `handle_client` receives the data i.e., the request from the client, and passes it to `handle_request` which is implemented by HTTP (as I will explain later). Now this threading helps in two things:

- It allows multiple clients to connect at the same time
- It prevents the main thread from blocking

## The HTTP Server

Now we build the HTTP server on top of the TCP server. The HTTP server inherits from the TCP server and overrides the `handle_request` method.

```python
def handle_request(self,data):
    blank_line = b'\r\n'
    res = self.parse(data)
    cntlngth=len(res)
    dict2={'Content-length':cntlngth}
    self.headers.update(dict2)
    if  res==b'':
        head=self.get_headers(self.headers).encode()
        stscds=self.get_codes(404)
        body = b'<h1>Error 404 Page not found</h1>'
        return b"".join([stscds,head,blank_line,body])

    elif res==b'Not operatable':
        head=self.get_headers(self.headers).encode()
        stscds=self.get_codes(501)
        body=b'<h1>Error 501 Not Implemented</h1>'
        return b"".join([stscds,head,blank_line,body])

    elif res==b'Deleted':
        head=self.get_headers(self.headers).encode()
        stscds=self.get_codes(200)
        body=b'<h1>File Deleted</h1>'
        return b"".join([stscds,head,blank_line,body])

    elif res==b'return head':
        head=self.get_headers(self.headers)
        stscds=self.get_codes(200)
        return b"".join([stscds,head.encode(),blank_line])

    else:
        self.logger.addHandler(self.evnt_logger)
        self.logger.info("Connected to"+" "+f"{self.addr}"+"and page "+" "+f"{self.uri}"+"requested")
        head=self.get_headers(self.headers)
        stscds=self.get_codes(200)
        return b"".join([stscds,head.encode(),blank_line,res])
```

#### Parsing the Requests

```python
def parse(self,data):
    lines=data.split(b"\r\n")
    reqline=lines[0]
    self.method=reqline.decode().split(' ')[0]
    if self.method!='GET' and self.method!='DELETE' and self.method!='POST'  and self.method!='HEAD':
        return b'Not operatable'
    elif self.method=='DELETE':
        self.handle_DELETE(reqline)
        return b'Deleted'
    elif self.method=='HEAD':
        return b'return head'

    response=self.handle_GET(reqline)
    return response
```

In the parse section, we split the data by pulling a delimiter of '\r\n'. '\r' means carriage return and '\n' means newline, the split data is basically:

```
[GET /index.html HTTP/1.1],
[Host: example.com],
[Connection: keep-alive],
[User-Agent: Mozilla/5.0]
```

From here we only use the first line. Now if we have a request other than GET or DELETE or POST or HEAD, then it means that function can't be executed. So we return 'Not operatable'. We go back once again to the `handle_request` function.

Now since we get `res` as not operatable, we make the body 'Error 501 Not implemented'. First up, we need to put all our headers into the variable `head`.

```python
def get_headers(self,headers):
    header_lines = [f'{key}: {value}' for key, value in headers.items()]
    return '\r\n'.join(header_lines)+'\r\n\r\n'
```

Nothing much here to be honest except that you traverse the dictionary and create a key:value list. Now depending on the operation, we use the `get_codes` function by passing the code (in this case, it's 501 for Not operatable) as an argument. Now, in the `get_codes` function, we find out the value of 501 present in the dictionary `status_codes`. We then join it with HTTP/1.1, encode and return it.

We then join the headers, status codes, blank line, and body into a byte addressable string and send it to the response present in `handle_client` which sends it to the client. This header and status thing is present in every operation.

```python
def get_codes(self,code):
    msg=self.status_codes[code]
    msgresponse='HTTP/1.1 %s %s \r \n'%(code,msg)
    return msgresponse.encode()
```

## GET Operation

For the GET operation, what we did was find out the requested page by finding its start and end. We put it inside `uri`. Now we see whether it exists in the system and if it does, then we read that body and send it to `res` in the form of bytes.

```python
def handle_GET(self,reqline):
    body=b''
    start=reqline.decode().find('/')+1
    end=reqline.decode().find(' ',start)
    self.uri=reqline[start:end]
    if os.path.exists(self.uri) and not os.path.isdir(self.uri):
        with open(self.uri,'rb')as r:
            body=r.read()
    return body
```

## DELETE Operation

For the DELETE operation, we do the same thing except that we remove the resource.

```python
def handle_DELETE(self,reqline):
    body=b''
    start=reqline.decode().find('/')+1
    end=reqline.decode().find(' ',start)
    self.uri=reqline[start:end]
    if os.path.exists(self.uri):
        os.remove(self.uri)
```

## HEAD Method

For the HEAD method, since there's no body to be read, we didn't create any function. Instead, we packaged headers, status, and blank line into bytes and sent them to be sent to the `handle_request` function.

```python
elif res==b'return head':
    head=self.get_headers(self.headers)
    stscds=self.get_codes(200)
    return b"".join([stscds,head.encode(),blank_line])
```

## Understanding the Logger

```python
def __init__(self):
    self.uri=None
    self.method=None
    self.logger=logging.getLogger()
    self.logger.setLevel(logging.INFO)
    self.evnt_logger=logging.FileHandler("access.log",mode='w')
    self.evnt_logger.setFormatter(logging.Formatter('%(asctime)s  %(message)s'))
```

We have already seen this piece of code before, but now its time to understand it. What the logger does is create a log file when you fire up the server.

```python
self.logger.addHandler(self.evnt_logger)
self.logger.info("Connected to"+" "+f"{self.addr}"+"and page "+" "+f"{self.uri}"+"requested")
head=self.get_headers(self.headers)
stscds=self.get_codes(200)
return b"".join([stscds,head.encode(),blank_line,res])
```

What it does is every time you access a page it writes the url, address with the date and time of its creation into the access log file. This acts as a log file that helps show the history of all the pages accessed by your server.

## Putting it all together

```python
import socket
import threading
import os
import signal
import mimetypes
import logging
import json
import cgi
from datetime import date

class TcpServer:
    host='127.0.0.1'
    port=80
    conn=0
    addr=0
    allconn=[]
    def start(self):
        sock=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
        sock.bind((self.host,self.port))
        print("listening at ",sock.getsockname())
        sock.listen(1)
        try:
            while True:
                self.conn,self.addr=sock.accept()
                t=threading.Thread(target=self.handle_client,args=(self.conn,self.addr))
                t.start()
                self.allconn.append(t)
        except socket.error:
            print("coudnt connect with the socket")
            SystemExit
        finally:
            if sock:
                sock.close()
            for t in self.allconn:
                t.join()
    def handle_client(self,conn,addr):
        data=conn.recv(1024)
        response=self.handle_request(data)
        conn.sendall(response)
        conn.close()
    def handle_request(self,data):
        return data

class HttpServer(TcpServer):
    cntlngth=''
    headers={
        'Content-Type':'text/html ',
        'Server':'First server ',
        'Date'  :date.today(),
        'Content-length':''
    }
    status_codes={
        200:'Ok \n',
        404:'Not found \n',
        501:'Not implemented \n'
    }
    def __init__(self):
        self.uri=None
        self.method=None
        self.logger=logging.getLogger()
        self.logger.setLevel(logging.INFO)
        self.evnt_logger=logging.FileHandler("access.log",mode='w')
        self.evnt_logger.setFormatter(logging.Formatter('%(asctime)s  %(message)s'))
    # ... rest of the methods

if __name__ == '__main__':
    server=HttpServer()
    server.start()
```

And that's the end of this blog post. Here's the source [code](https://github.com/Saptarshi2001/PyHttp) and the [roadmap](https://github.com/Saptarshi2001/PyHttp).

## Conclusion

Thanks and see you next time!
