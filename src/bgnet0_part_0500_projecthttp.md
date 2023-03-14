# Project: HTTP Client and Server

We're going to write a sockets program that can download files from a
web server! This is going to be our "web client". This will work with
almost any web server out there, if we code it right.

And as if that's not enough, we're going to follow it up by writing a
simple web server! This program will be able to handle requests from the
web client we write... or indeed any other web client such as Chrome or
Firefox!

These programs are going to speak a protocol you have probably heard of:
HTTP, the HyperText Transport Protocol.

And because they speak HTTP, and web browsers like Chrome speak HTTP,
they should be able to communicate!

## Restrictions

In order to better understand the sockets API at a lower level, this
project may **not** use any of the following helper functions:

* The `socket.create_connection()` function.
* The `socket.create_server()` function.
* Anything in the `urllib` modules.

After coding up the project, it should be more obvious how these helper
functions are implemented.

## Python Character Encoding

The sockets in Python send and receive sequences of bytes, which are
different than Python strings. You'll have to convert back and forth
when you want to send a string, or when you want to print a byte
sequence as a string.

The sequences of bytes depend on the _character encoding_ used by the
string. The character encoding defines which bytes correspond to which
characters. Some encodings you might have heard of are ASCII and UTF-8.
There are hundreds.

The default character encoding of the web is "ISO-8859-1".

This is important because you have to encode your Python strings into a
sequence of bytes and you can tell it the encoding when you do that. (It
defaults to UTF-8.)

To convert from a Python string to an ISO-8859-1 sequence of bytes:

```
s = "Hello, world!"          # String
b = s.encode("ISO-8859-1")   # Sequence of bytes
```

That sequence of bytes is ready to send over the socket.

To convert from a byte sequence you received from a socket in ISO-8859-1
format to a string:

```
s = b.decode("ISO-8859-1")
```

And then it's ready to print.

Of course, if the data is not encoded with ISO-8859-1, you'll get weird
characters in your string or an error.

The encodings ASCII, UTF-8, and ISO-8859-1 are all the same for your
basic latin letters, numbers, and punctuation, so your strings will all
work as expected unless you start getting into some weird Unicode
characters.

If you're writing this is C, it's probably best just not to worry about
it and print the bytes out as you get them. A few might be garbage, but
it'll work for the most part.

## HTTP Summary

HTTP operates on the concept of _requests_ and _responses_. The client
requests a web page, the server responds by sending it back.

A simple HTTP request from a client looks like this:

```
GET / HTTP/1.1
Host: example.com
Connection: close

```

That shows the request _header_ which consists of the request method,
path, and protocol on the first line, followed by any number of header
fields.  There is a blank line at the end of the header.

This request is saying "Get the root web page from the server
example.com and I'm going to close the connection as soon as I get your
response."

Ends-of-line are delimited by a Carriage Return/Linefeed combination. In
Python or C, you write a CRLF like this:

```
"\r\n"
```

If you were requesting a specific file, it would be on that first line,
for example:

```
GET /path/to/file.html HTTP/1.1
```

(And if there were a payload to go with this header, it would go just
afgter the blank line. There would also be a `Content-Length` header
giving the length of the payload in bytes. We don't have to worry about
this for this project.)

A simple HTTP response from a server looks like:

```
HTTP/1.1 200 OK
Content-Type: text/plain
Content-Length: 6
Connection: close

Hello!
```

This response says, "Your request succeeded and here's a response that's
6 bytes of plain text. Also, I'm going to close the connection right
after I send this to you. And the response payload is 'Hello!'."

Notice that the `Content-Length` is set to the size of the payload: 6
bytes for `Hello!`.

Another common `Content-Type` is `text/html` when the payload has HTML
data in it.

## The Client

The client should be named `webclient.py`.

You can write the client before the server first and then test it on a
real, existing webserver.  No need to write both the client and server
before you test this.

The goal with the client is that you can run it from the command line,
like so:

```
$ python webclient.py example.com
```

for output like this:

```
HTTP/1.1 200 OK
Age: 586480
Cache-Control: max-age=604800
Content-Type: text/html; charset=UTF-8
Date: Thu, 22 Sep 2022 22:20:41 GMT
Etag: "3147526947+ident"
Expires: Thu, 29 Sep 2022 22:20:41 GMT
Last-Modified: Thu, 17 Oct 2019 07:18:26 GMT
Server: ECS (sec/96EE)
Vary: Accept-Encoding
X-Cache: HIT
Content-Length: 1256
Connection: close

<!doctype html>
<html>
<head>
    <title>Example Domain</title>
    ...
```

(Output truncated, but it would show the rest of the HTML for the site.)

Notice how the first part of the output is the HTTP response with all
those fields from the server, and then there's a blank line, and
everything following the blank line is the response payload.

ALSO: you need to be able specify a port number to connect to on the
command line. This defaults to port 80 if not specified. So you could
connect to a webserver on a different port like so:

```
$ python webclient.py example.com 8088
```

Which would get you to port 8088.

First things first, you need the socket module in Python, so

```
import socket
```

at the top. Then you have access to the functionality.

See Exploration 2.1 for the overall approach, but here are some
Python-specifics:

* Use `socket.socket()` to make a new socket. You don't have to pass it
  anything--the default parameter values work for this project.

* Use `s.connect()` to connect the new socket to a destination. You can
  bypass the DNS step since `.connect()` does it for you.

  This function takes a tuple as an argument that contains the host and
  port to connect to, e.g.

  ```
  ("example.com", 80)
  ```

* Build and send the HTTP request. You can use the simple HTTP request
  shown above. **Don't forget the blank line at the end of the header,
  and don't forget to end all lines with `"\r\n"`!**

  I recommend using the `s.sendall()` method to do this. You could use
  `.send()` instead but it might only send part of the data.

  (C programmers will find an implementation of `sendall()` in Beej's
  Guide.)

* Receive the web response with the `s.recv()` method. It will return
  some bytes in response. You'll have to call it several times in a
  loop to get all the data from bigger sites.

  It will return a byte array of zero elements when the server closes
  the connection and there's no more data to read, e.g.:

  ```
  d = s.recv(4096)  # Receive up to 4096 bytes
  if len(d) == 0:
      # all done!
  ```

* Call `s.close()` on your socket when you're done.

Test the client by hitting some websites with it:

```
$ python webclient.py example.com
$ python webclient.py google.com
$ python webclient.py oregonstate.edu
```

## The Server

The server should be named `webserver.py`.

You'll launch the webserver from the command line like so:

```
$ python webserver.py
```

and that should start it listening on port 28333.

Code it so we could also specify an optional port number like this:

```
$ python webserver.py 12399
```

The server is going to follow the procedure in Exploration 2.1. It's
going to run forever, handling incoming requests. (Forever means "until
you hit CTRL-C".)

And it's only going to send back one thing no matter what the request
is. Have it send back the simple server response, shown above.

So it's not a very full-featured webserver. But it's the start of one!

Here are some Python specifics:

* Get a socket just like you did for the client.

* Bind the socket to a port with `s.bind()`. This takes one argument, a
  tuple containing the address and port you want to bind to. The address
  can be left blank to have it choose a local address. For example, "Any
  local address, port 28333" would be passed like so:

  ```
  ('', 28333)
  ```

* Set the socket up to listen with `s.listen()`.

* Accept new connections with `s.accept()`. Note that this returns a
  tuple. The first item in the tuple is a new socket representing the
  new connection. (The old socket is still listening and you will call
  `s.accept()` on it again after you're done handling this request.)

  ```
  new_conn = s.accept()
  new_socket = new_conn[0]  # This is what we'll recv/send on
  ```

* Receive the request from the client. You should call
  `new_socket.recv()` in a loop similar to how you did it with the
  client.

  When you see a blank line (i.e. `"\r\n\r\n"`) in the request, you've
  read enough and can quit receiving.

  (We don't handle payloads in the request for this project. The _right_
  thing to do would be to look for a `Content-Length` header and then
  receive the header plus that many bytes. But that's a stretch goal for
  you.)

  **Beware**: you can't just loop until `recv()` returns an empty string
  this time! This would only happen if the client closed the connection,
  but the client isn't closing the connection and it's waiting for a
  response. So you have to call `recv()` repeatedly until you see that
  blank line delimiting the end of the header.

* Send the response. You should just send the "simple server reponse",
  from above.

* Close the new socket with `new_socket.close()`.

* Loop back to `s.accept()` to get the next request.

Now run the web server in one window and run the client in another, and
see if it connects!

Once it's working with `webclient.py`, try it with a web browser!

Run the server on an unused port (choose a big one at random):

```
$ python webserver.py 20123
```

Go to the URL [`http://localhost:20123/`](http://localhost:20123/) to
view the page.  (`localhost` is the name of "this computer".)

If it works, great!

Try printing out the value returned by `.accept()`. What's in there?

Did you notice that if you use a web browser to connect to your server,
the browser actually makes two connections? Dig into it and see if you
can figure out why!

## Hints and Help

### Address Already In Use

If your server crashes and then you start getting an "Address already in
use" error when you try to restart it, it means the system hasn't
finished cleaning up the port. (In this case "address" refers to the
port.) Either switch to a different port for the server, or wait a
minute or two for it to timeout and clean up.

### Receiving Partial Data

Even if you tell `recv()` that you want to get 4096 bytes, there's no
guarantee that you'll get all of those. Maybe the server sent fewer.
Maybe the data got split in transit and only part of it is here.

This can get tricky when processing an HTTP request or response because
you might call `recv()` and only get part of the data. Worse, the data
might get split in the middle of the blank line delimiter at the end of
the header!

Don't assume that a single `recv()` call gets you all the data. Always
call it in a loop, appending the data to a buffer, until you have the
data you want.

`recv()` will return an empty string (in Python) or `0` (in C) if you
try to read from a connection that the other side has closed. This is
how you can detect that closure.

### HTTP 301, HTTP 302

If you run the client and get a server response with code `301` or
`302`, probably along with a message that says `Moved Permanently` or
`Moved Temporarily`, this is the server indicating to you that the
particular resource you're trying to get at the URL has moved to a
different URL.

If you look at the headers below that, you'll find a `Location:` header
field.

For example, attempting to run `webclient.py google.com` results in:

```
HTTP/1.1 301 Moved Permanently
Location: http://www.google.com/
Content-Type: text/html; charset=UTF-8
Date: Wed, 28 Sep 2022 20:41:09 GMT
Expires: Fri, 28 Oct 2022 20:41:09 GMT
Cache-Control: public, max-age=2592000
Server: gws
Content-Length: 219
X-XSS-Protection: 0
X-Frame-Options: SAMEORIGIN
Connection: close

<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.com/">here</A>.
</BODY></HTML>
Connection closed by foreign host.
```

Notice the first line is telling us the resource we're looking for has
moved.

The second line with the `Location:` field tells us to where it has
moved.

When a web browser sees a `301` redirect, it automatically goes to the
other URL so you don't have to worry about it. 

Try it! Enter `google.com` in your browser and watch it update to
`www.google.com` after a moment.

### HTTP 400, HTTP 501 (or any 500s)

If you run the client and get a response from a server that has the code 
`400` or any of the `500`s, odds are you have made a bad request. That
is, the request data you sent was malformed in some way.

Make sure every field of the header ends in `\r\n` and that 

### HTTP 404 Not Found!

Make sure you have the `Host:` field set correctly to the same hostname
as you passed in on the command line. If this is wrong, it'll `404`.

## Extensions

These are here if you have time to give yourself the additional
challenge for greater understanding of the material. Push yourself!

* Modify the server to print out the IP address and port of the client
  that just connected to it. Hint: look at the value returned by
  `accept()` in Python.

* Modify the client to be able to send payloads. You'll need to be able
  to set the `Content-Type` and `Content-Length` based on the payload.

* Modify the server to extract and print the "request method" from the
  request.  This is most often `GET`, but it could also be `POST` or
  `DELETE` or many others.

* Modify the server to extract and print a payload sent by the client.

