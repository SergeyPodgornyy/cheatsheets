# Python — Dependencies

To enable `ensurepip`, on Debian/Ubuntu systems, you need to install the `python3-venv` package using the following command:

```console
$ sudo apt install python3.10-venv
```

Create a virtual environment using `venv` in each of your projects:

```console
$ python3 -m venv <DIR>
```

Once the virtual environment is created, you need to activate it:

```console
$ source <DIR>/bin/activate
```

`pip-tools` must be installed in each of your project's virtual environments:

```console
$ pip install pip-tools
```

More: https://pypi.org/project/pip-tools/

## Install packages

```console
$ pip install <package_name>==<version>
$ echo "<package>" >> requirements.in
```

## Freeze current dependencies

```console
$ pip freeze > requirements.in
```

## Adding Hashes from the `requirements.in` file

```console
$ pip-compile --generate-hashes
```

## Adding Hashes from `pyproject.toml` file

```console
$ pip-compile -o requirements.txt pyproject.toml
```

## Install dependencies from the file

```console
$ pip install -r requirements.txt
```

## Useful packages

Static type checker:

```console
$ mypy main.py
```

<!-- nav -->
---

← [Python — Exceptions](11-exceptions.md) · [Index](../README.md) · [Dart — Basics](../dart/01-basics.md) →
<!-- nav -->
