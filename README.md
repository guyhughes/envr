# envr

Manipulate and transform .env files.

# Use it

```python
from envr import Envr

# load a file
e = Envr("/app/.env")

# load a stream
o = Envr(None, stream=sys.stdin)

# set keys, get values
e['NEW_KEY'] = e['OLD_KEY'] + e['OTHER_KEY']

# delete key
del e['OLD_KEY']

# save the changes
e.save()

```

# Transforms
```python
# returns an  OrderedDict
e_as_dict = e.dict()

# returns indented json
e_as_json = e.json()

# returns the in memory state, that is as close as possible to the
# original file format, perserving comments and incompatible lines
state = e.env()
state = str(e)

# returns an envr syntax-compatible, POSIX-compliant shell script as a
# string, containing only the variable assignments
e_as_strict_env = e.env(strict=True)

# turns off quoting of variable values
e_unquoted = e.env(quoted=False)
e_unquoted = e.env(strict=True, quoted=False)
```

```sh
# reads standard input, transforms it with Envr.env(strict=True), and writes
# the result to standard output
cat script.sh | python -m envr
```

# `.env` envr syntax

`.env` syntax is a subset of POSIX shell scripts.

Each parseable line of the file contains a variable assignment of the form:
```
VARIABLE_NAME=VALUE COMMENT
```

`NAME` must match `[a-zA-Z0-9_]+`.

`VALUE` must match `[^"']` if enclosed by one pair of any character in the set `["']`; otherwise it must match `[^\s"']`.

`COMMENT` may be omitted, and if present must begin with a `#` after the space separating it from `VALUE`.

# compatible with

`.env` envr syntax is compatible as a subset of `.env` files for

- honcho
- django-environ
- python-dotenv (except for a bug with single quotes)

`.env` envr syntax generated by `Envr(...).env(strict=True, quoted=False)` is compatible as a subset of `.env` files for the above-listed projects as well as the following useful outlaying project, which does not parse quote pairs around values as of 2017-01-02. 
- [docker-compose](https://docs.docker.com/compose/env-file/)
  (This is a strange design choice for docker-compose because it parses assignments with a value beginning with space, i.e. `NAME= FAIL`, as if the space is the first byte of the value, which would not be compatible with POSIX-compliant shells, and all the things.)

# known issue

If you do `Envr(...).env(strict=True, quoted=False)` and you have value that begins with a space, the outputted result, since it is not quoted, will not be compatible with POSIX-compliant shells. `quoted=False` was introduced in version 0.3.3 to support docker-compose, and should not be used unless necessary. 
