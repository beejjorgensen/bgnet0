# Project: Using Select

In this project we're going to write a server that uses `select()` to
handle multiple simultaneous connections.

The client is already provided. You fill in the server.

We'll do this project in class as a pair-programming exercise.

## Demo Code

[Grab the ZIP file containing the files here]().

The `select_client.py` file is already complete.

You have to fill in the `select_server.py` file to get it going.

## Features to Add

Your server should do the following:

* When a client connects, the server prints out the client connection
  info in this form (it's the client IP and port number in front):

  ```
  ('127.0.0.1', 61457): connected
  ```

* When a client disconnects, the server prints out the late client's
  connection info in this form:

  ```
  ('127.0.0.1', 61457): disconnected
  ```

  **Hint**: You can use the `.getpeername()` method on a socket to get
  the address of the remote side even after it has disconnected. It'll
  come back as a tuple containing `("host", port)`, just like what you
  pass to `connect()`.

* When a client sends data, the server should print out the length of
  the data as well as the raw bytestring object received:

  ```
  ('127.0.0.1', 61457) 22 bytes: b'test1: xajrxttphhlwmjf'
  ```

## Example Run

Running the server:

```
python select_server.py 3490
```

Running the clients:

```
python select_client.py alice localhost 3490
python select_client.py bob localhost 3490
python select_client.py chris localhost 3490
```

The first argument to the client can be any string--the server prints it
out with the data to help you identify which client it came from.

Example output:

```
waiting for connections
('127.0.0.1', 61457): connected
('127.0.0.1', 61457) 22 bytes: b'test1: xajrxttphhlwmjf'
('127.0.0.1', 61457) 22 bytes: b'test1: geqtgopbayogenz'
('127.0.0.1', 61457) 23 bytes: b'test1: jquijcatyhvfpydn'
('127.0.0.1', 61457) 23 bytes: b'test1: qbavdzfihualuxzu'
('127.0.0.1', 61457) 24 bytes: b'test1: dyqmzawthxjpkgpcg'
('127.0.0.1', 61457) 23 bytes: b'test1: mhxebjpmsmjsycmj'
('127.0.0.1', 61458): connected
('127.0.0.1', 61458) 23 bytes: b'test2: bejnrwxftgzcgdyg'
('127.0.0.1', 61457) 24 bytes: b'test1: ptcavvhroihmgdfyw'
('127.0.0.1', 61458) 24 bytes: b'test2: qrumcrmqxauwtcuaj'
('127.0.0.1', 61457) 26 bytes: b'test1: tzoitpusjaxljkfxfvw'
('127.0.0.1', 61457) 17 bytes: b'test1: mtcwokwquc'
('127.0.0.1', 61458) 18 bytes: b'test2: whvqnzgtaem'
('127.0.0.1', 61457): disconnected
('127.0.0.1', 61458) 21 bytes: b'test2: raqlvexhimxfgl'
('127.0.0.1', 61458): disconnected
```

<!-- Rubric:

5
Server prints proper client connection message

5
Server prints proper client disconnection message

5
Server prints proper client data received message

5
Server uses select() to wait for incoming connections

5
Server uses select() to wait for incoming client data

-->
