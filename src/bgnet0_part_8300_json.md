# Appendix: JSON {#appendix-json}

For the final project, we need to be able to encode and decode JSON
data.

If you're not familiar with that format, [get a quick introduction at
Wikipedia](https://en.wikipedia.org/wiki/JSON).

In this exploration, we'll take a look at what it means to encode and
decode JSON data.

## JSON versus Native

Here's a sample JSON object:

```
{
    "name": "Ada Lovelace",
    "country": "England",
    "years": [ 1815, 1852 ]
}
```

In Python, you can make a `dict` object that looks just like that:

```
d = {
    "name": "Ada Lovelace",
    "country": "England",
    "years": [ 1815, 1852 ]
}
```

But here's the key difference: _all JSON data are strings_. The JSON is
a string representation of the data in question.

## Converting Back and Forth

If you have a JSON string, you can turn it into Python native data with
the `json.loads()` function.

```
import json

data = json.loads('{ "name": "Ada" }')

print(data["name"])   # Prints Ada
```

Likewise, if you have Python data, you can convert it into JSON string
format with `json.dumps()`:

```
import json

data = { "name": "Ada" }

json_data = json.dumps(data)

print(json_data)  # Prints {"name": "Ada"}
```

## Pretty Printing

If you have a complex object, `json.dumps()` will just stick it all
together on one line.

This code:

```
d = {
    "name": "Ada Lovelace",
    "country": "England",
    "years": [ 1815, 1852 ]
}

json.dumps(d)
```

outputs:

```
'{"name": "Ada Lovelace", "country": "England", "years": [1815, 1852]}'
```

You can clean it up a bit by passing the `indent` argument to
`json.dumps()`, giving it an indentation level.

```
json.dumps(d, indent=4)
```

outputs:

```
{
    "name": "Ada Lovelace",
    "country": "England",
    "years": [
        1815,
        1852
    ]
}
```

Much cleaner.

## Double Quotes are Important

JSON requires strings and key names to be in double quotes. Single
quotes won't cut it. Missing quotes _definitely_ won't cut it.

## Review

* What's the difference between a JSON object and a Python dictionary?

* Looking at the Wikipedia article, what types of data can be
  represented in JSON?

