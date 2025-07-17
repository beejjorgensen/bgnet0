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

``` {.default}
http://localhost:33490/file2.html
```

The client will send a request to your server that looks like this:

``` {.default}
GET /file2.html HTTP/1.1
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

``` {.default}
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 373
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

You'll want to read in the full request header, so you're probably doing
something like accumulating data from all your `recv()`s in a single
variable and searching it (with something like string's `.find()` method
to find the `"\r\n\r\n"` that marks the end of the header.

At that point, you can `.split()` the header data on `"\r\n"` to get
individual lines.

The first line is the `GET` line.

You can `.split()` that single line into its three parts: the request
method (`GET`), the path (e.g. `/file1.txt`), and the protocol
(`HTTP/1.1`).

Don't forget to `.decode("ISO-8859-1")` the first line of the request so
that you can use it as a string.

We only really need the path.

## Stripping the Path down to the Filename

**SECURITY RISK!** If we don't strip the path off, a malicious attacker
could use it to accesss arbitrary files on your system. Can you think of
how they might build a URL that reads `/etc/password`?

Real web servers just check to make sure the path is restricted to a
certain directory hierarchy, but we'll take the easy way and just strip
all the path information off and only serve files from the directory the
webserver is running in.

The path is going to be made up of directory names separated by a slash
(`/`), so the easiest thing to do at this point is to use `.split('/')`
on your path and filename, and then look at the last element.

``` {.py}
fullpath = "/foo/bar/baz.txt"

file_name = fullpath.split("/")[-1]
```

A more portable way is to use the standard library function
`os.path.split`. The value returned by `os.path.split` is will be a
tuple with two elements, the second of which is the file name:

``` {.py}
fullpath = "/foo/bar/baz.txt"

os.path.split(fullpath)
=> ('/foo/bar', 'baz.txt')
```

Select the last element:

``` {.py}
fullpath = "/foo/bar/baz.txt"

file_name = os.path.split(fullpath)[-1]
```

Use that to get the file name you want to serve.

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

``` {.default}
Content-Type: application/pdf
```

But how do you know what type of data a file holds?

The classic way to do this is by looking at the file extension,
everything after the last period in the file name.

Luckily,
[`os.path.splitext()`](https://docs.python.org/3/library/os.path.html#os.path.splitext)
gives us an easy way to pull the extension off a file name:

``` {.py}
os.path.splitext('keyboardcat.gif')
```

returns a tuple containing:

``` {.py}
('keyboardcat', '.gif')
```

You can just map the following extensions for this assignment:

<!-- CAPTION: Extension Mapping -->
|Extension|MIME Type|
|-|-|
|`.txt`|`text/plain`|
|`.html`|`text/html`|

So if the file has a `.txt` extension, be sure to send back:

``` {.default}
Content-Type: text/plain
```

in your response.

If you really want to be correct, add `charset` to your header to
specify the character encoding:

``` {.default}
Content-Type: text/plain; charset=iso-8859-1
```

but that's not necessary, since browsers typically default to that
encoding.

## Reading the File, `Content-Length`, and Handling Not Found

Here's some code to read an entire file and check for errors:

``` {.py}
try:
    with open(filename, "rb") as fp:
        data = fp.read()   # Read entire file
        return data

except:
    # File not found or other error
    # TODO send a 404
```

The data you get back from `.read()` is what will be the payload.
Use `len()` to compute the number of bytes.

The number of bytes will be send back in the `Content-Length` header,
like so:

``` {.default}
Content-Length: 357
```

(with the number of bytes of your file).

> You might be wondering what the `"rb"` thing is in the `open()` call.
> This causes the file to open for reading in binary mode. In Python, a
> file open for reading in binary mode will return a bytestring
> representing the file that you can send straight out on the socket.

What about this `404 Not Found` thing? It's common enough that you've
probably seen it in normal web usage from time to time.

This just means you've requested a file or other resource that doesn't
exist.

In our case, we'll detect some kind of file open error (with the
`except` block, above) and return a `404` response.

The `404` response is an HTTP response, except instead of

``` {.default}
HTTP/1.1 200 OK
```

our response will start with

``` {.default}
HTTP/1.1 404 Not Found
```

So when you try to open the file and it fails, you're going to just
return the following (verbatim) and close the connection:

``` {.default}
HTTP/1.1 404 Not Found
Content-Type: text/plain
Content-Length: 13
Connection: close

404 not found
```

(Both the content length and the payload can just be hardcoded in this
case, but of course have to be `.encode()`'d to bytes.)

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

  Normally you'd have some kind of configuration variable that specified
  the server root directory as an absolute path. But if you're in one of
  my classes, that would make my life miserable when I went to grade
  projects. So if that's the case, please use a relative path for your
  server root directory and create a full path with the
  [`os.path.abspath()`](https://docs.python.org/3/library/os.path.html#os.path.abspath)
  function.

  ``` {.py}
  server_root = os.path.abspath('.')        # This...
  server_root = os.path.abspath('./root')   # or something like this
  ```

  This would set `server_root` to a full path to where you ran your
  server. For example, on my machine, I might get:

  ``` {.default}
  /home/beej/src/webserver                  # This...
  /home/beej/src/webserver/root             # or something like this
  ```

  Then when the user tries to `GET` some path, you can just append it to
  server root to get the path to the file.

  ``` {.py}
  file_path = os.path.sep.join((server_root, get_path))
  ```

  So if they tried to `GET /foo/bar/index.html`, then `file_path` would
  get set to:

  ```
  /home/beej/src/webserver/foo/bar/index.html
  ```

  **And now the security crux!** You have to make sure that `file_path`
  is within the server root directory. See, a villain might try to:

  ``` {.http}
  GET /../../../../../etc/passwd HTTP/1.1
  ```
  
  And if they did that, we'd unknowingly serve out this file:

  ```
  /home/beej/src/webserver/../../../../../etc/passwd
  ```

  which would get them to my password file in `/etc/passwd`. I don't
  want that.

  So I need to make sure that wherever they end up is still within my
  `server_root` hierarchy. How? We can use `abspath()` again.

  If I run the crazy `..` path above through `abspath()`, it just
  returns `/etc/passwd` to me. It resolves all the `..`s and other
  things and returns the "real" path.

  But I know my server root in this example is
  `/home/beej/src/webserver`, so I can just verify that the absolute
  file path begins with that. And 404 if it doesn't.

  ``` {.py}
  # Convert to absolute path
  file_path = os.path.abspath(file_path)

  # See if the user is trying to break out of the server root
  if not file_path.startswith(server_root):
      send_404()
  ```

## Example Files

You can copy and paste these into files for testing purposes:

### `file1.txt`

``` {.default}
This is a sample text file that has all kinds of words in it that
seemingly go on for a long time but really don't say much at all.

And as that weren't enough, here is a second paragraph that continues
the tradition. And then goes for a further sentence, besides.
```

### `file2.html`

``` {.html}
<!DOCTYPE html>

<html>
<head>
<title>Test HTML File</title>
</head>

<body>
<h1>Test HTML</h1>

<p>This is my test file that has <i>some</i> HTML in in that the browser
should render as HTML.</p>

<p>If you're seeing HTML tags that look like this <tt>&lt;p&gt;</tt>,
you're sending it out as the wrong MIME type! It should be
<tt>text/html</tt>!</p>

<hr>
</body>
</html>
```

The idea is that these URLs would retrieve the above files (with the
appropriate port given):

``` {.default}
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
15 HTTP response payload is constructed correctly
-->
