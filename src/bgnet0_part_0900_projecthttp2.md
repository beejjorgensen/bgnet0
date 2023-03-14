# Project: A Better Web Server

Time to improve the web server so that it serves actual files!

We're going to make it so that when a web client (in this case we'll use
a browser) requests a specific file, the webserver will return that
file.

There are some interesting details to be found along the way.

## Restrictions

In order to better understand the sockets API at a lower level, this
project may **not** use any of the following helper functions:

* The `socket.create_connection()` function.
* The `socket.create_server()` function.
* Anything in the `urllib` modules.

After coding up the project, it should be more obvious how these helper
functions are implemented.

## The Process

If you go to your browser and enter a URL like this (substituting the
port number of your running server):

```
http://localhost:33490/file1.txt
```

The client will send a request to your server that looks like this:

```
GET /file1.txt HTTP/1.1
Host: localhost
Connection: close

```

Notice the file name is right there in the `GET` request on the first
line!

Your server will:

1. Parse that request header to get the file name.
2. Strip the path off for security reasons.
3. Read the data from the named file.
4. Determine the type of data in the file, HTML or text.
5. Build an HTTP response packet with the file data in the payload.
6. Send that HTTP response back to the client.

The response will look like this example file:

```
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 357
Connection: close

<!DOCtype html>

<html>
<head>
...
```

[The rest of the HTML file has been truncated in this example.]

At this point, the browser should display the file.

Notice a couple things in the header that need to be computed: the
`Content-Type` will be set according to the type of data in the file
being served, and the `Content-Length` will be set to the length in
bytes of that data.

We're going to want to be able to display at least two different types
of files: HTML and text files.

## Parsing the Request Header

You'll want to read in the full request header, so your probably doing
something like accumulating data from all your `recv()`s in a single
variable and searching it (with something like string's `.find()` method
to find the `"\r\n\r\n"` that marks the end of the header.

At that point, you can `.split()` the header data on `"\r\n"` to get
individual lines.

The first line is the `GET` line.

You can `.split()` that single line into its three parts: the request
method (`GET`), the path (e.g. `/file1.txt`), and the protocol
(`HTTP/1.1`).

We only really need the path.

## Stripping the Path down to the Filename

**SECURITY RISK!** If we don't strip the path off, a malicious attacker
could use it to accesss arbitrary files on your system. Can you think of
how they might build a URL that reads `/etc/password`?

Real web servers just check to make sure the path is restricted to a
certain directory hierarchy, but we'll take the easy way and just strip
all the path information off and only serve files from the directory the
webserver is running in.

In the `os.path` module, you'll find a function called
[`.split()`](https://docs.python.org/3/library/os.path.html#os.path.split)
that will pull a file name off a path.

```
os.path.split("/foo/bar/baz.txt")
```

returns a tuple with two elements, the second of which is the file name:

```
('/foo/bar', 'baz.txt')
```

Use that to just get the file name you want to serve.

## MIME and Getting the `Content-Type`

In HTTP, the payload can be anything--any collection of bytes. So how
does the web browser know how to display it?

The answer is in the `Content-Type` header, which gives the
[MIME](https://en.wikipedia.org/wiki/MIME) type of the data. This is
enough for the client to know how to display it.

Some example MIME types:

<!-- CAPTION: MIME Types -->
|MIME Type|Description|
|-|-|
|`text/plain`|Plain text file|
|`text/html`|HTML file|
|`application/pdf`|PDF file|
|`image/jpeg`|JPEG image|
|`image/gif`|GIF image|
|`application/octet-stream`|Generic unclassified data|

There are [a lot of MIME
types](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types/Common_types)
to identify any kind of data.

You put these right in the HTTP response in the `Content-Type` header:

```
Content-Type: application/pdf
```

But how do you know what type of data a file holds?

The classic way to do this is by looking at the file extension,
everything after the last period in the file name.

Luckily,
[`os.path.spltext()`](https://docs.python.org/3/library/os.path.html#os.path.splitext)
gives us an easy way to pull the extension off a file name:

```
os.path.splitext('keyboardcat.gif')
```

returns a tuple containing:

```
('keyboardcat', '.gif')
```

You can just map the following extensions for this assignment:

<!-- CAPTION: Extension Mapping -->
|Extension|MIME Type|
|-|-|
|`.txt`|`text/plain`|
|`.html`|`text/html`|

So if the file has a `.txt` extension, be sure to send back:

```
Content-Type: text/plain
```

in your response.

If you really want to be correct, add `charset` to your header to
specify the character encoding:

```
Content-Type: text/plain; charset=iso-8859-1
```

but that's not necessary, since browsers typically default to that
encoding.

## Reading the File, `Content-Length`, and Handling Not Found

Here's some code to read an entire file and check for errors:

```
try:
    with open(filename) as fp:
        data = fp.read()   # Read entire file
        return data

except:
    # File not found or other error
    # TODO send a 404
```

The data you get back from `.read()` is what will be the payload. Be
sure to encode it as `ISO-8859-1`, and then use `len()` to compute the
number of bytes.

The number of bytes will be send back in the `Content-Length` header,
like so:

```
Content-Length: 357
```

(with the number of bytes of your file).

What about this `404 Not Found` thing? It's common enough that you've
probably seen it in normal web usage from time to time.

This just means you've requested a file or other resource that doesn't
exist.

In our case, we'll detect some kind of file open error (with the
`except` block, above) and return a `404` response.

The `404` response is an HTTP response, except instead of

```
HTTP/1.1 200 OK
```

our response will start with

```
HTTP/1.1 404 Not Found
```

So when you try to open the file and it fails, you're going to just
return the following (verbatim) and close the connection:

```
HTTP/1.1 404 Not Found
Content-Type: text/plain
Content-Length: 13
Connection: close

404 not found
```

(Both the content length and the payload can just be hardcoded in this
case.)

## Extensions

These are here if you have time to give yourself the additional
challenge for greater understanding of the material. Push yourself!

* Add MIME support for other file types so you can serve JPEGs and other
  files.

* Add support for showing a directory listing. If the user doesn't
  specify a file in the URL, show a directory listing where each file
  name is a link to that file.

  Hint:
  [`os.listdir`](https://docs.python.org/3/library/os.html#os.listdir)
  and
  [`os.path.join()`](https://docs.python.org/3/library/os.path.html#os.path.join)

* Instead of just dropping the entire path, allow serving out of
  subdirectories from a root directory your specify on the server.

  **SECURITY RISK!** Make sure the user can't break out of the root
  directory by using a bunch of `..`s in the path!

  Use
  [`os.path.realpath()`](https://docs.python.org/3/library/os.path.html#os.path.realpath)
  and compare the first part with the absolute path your server's root
  directory to make sure they match. If they don't, `404`.


## Example Files

You can copy and paste these into files for testing purposes:

### `file1.txt`

```
This is a sample text file that has all kinds of words in it that
seemingly go on for a long time but really don't say much at all.

And as that weren't enough, here is a second paragraph that continues
the tradition. And then goes for a further sentence, besides.
```

### `file2.html`

```
<!DOCTYPE html>

<html>
<head>
<title>Test HTML File</title>
</head>

<body>
<h1>Test HTML</h1>

<p>This is my test file that has <i>some</i> HTML in in that the browser
should render as HTML.

<p>If you're seeing HTML tags that look like this <tt>&lt;p&gt;</tt>,
you're sending it out as the wrong MIME type! It should be
<tt>text/html</tt>!

<hr>
</body>
```

The idea is that these URLs would retrieve the above files (with the
appropriate port given):

```
http://localhost:33490/file1.txt
http://localhost:33490/file2.html
```

<!--
Rubric

100 points

10 File name parsed from request header.
20 File path is either stripped off or sandboxed properly.
10 Data is read from the file named in the URL
15 Proper Content-Type set in header
15 Proper Content-Length set in header
15 HTTP response header is constructed correctly and ISO-8859-1 encoded
15 HTTP response payload is constructed correctly and ISO-8859-1 encoded
-->
