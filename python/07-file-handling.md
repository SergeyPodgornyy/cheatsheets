# Python — File Handling

## Dynamic dialog

```python
from tkinter import filedialog as fd

# Open the directory to select the filename from it
path: str = fd.askopenfilename(title='Select a file',
                               initialdir='us',
                               filetypes=(('PDF', '*.pdf'), ('MP3', '*.mp3')))
```

## File modes

- **Read Mode (`'r'`)**: This is the default mode for opening files. It allows you to read the contents of a file.
- **Write Mode (`'w'`)**: This mode opens a file for writing. If the file exists, its contents are overwritten. If it doesn't exist, a new file is created.
- **Append Mode (`'a'`)**: This mode opens a file for appending new content. The new content is added to the end of the file. If the file doesn't exist, a new file is created. If we want to also read from the file, then `'a+'` should be used.
- **Binary Mode (`'b'`)**: This mode is used for working with binary files, such as images, audio files, etc. It's often combined with read (`'rb'`) or write (`'wb'`) modes.
- **Read and Write Mode (`'r+'`)**: This mode allows reading from and writing to a file. It doesn't truncate the file, preserving its existing contents.
- **Write and Read Mode (`'w+'`)**: This mode is similar to `'r+'`, but it also truncates the file, removing its existing contents.
- **Exclusive mode (`'x' or 'x+'`)**: Similar to normal read and write, but it calls the underlying operating system code with the two flags `O_CREAT` and `O_EXCL`, which attempts to open the file exclusively, creating a new one if it doesn't currently exist. If the file exists, raise `FileExistsError`.

## Listing folder content

```python
import os

file_names: list[str] = os.listdir(dir_path)
```

## Reading the file content

We can open the file using the `open` function in read (`r`) mode:

```python
from typing import TextIO

file_path: str = 'info.txt'
file: TextIO = open(file_path, 'r') # read mode is default anyway
content: str = file.read()
file.close()
```

But there is a huge problem with an example above. If anything goes wrong in between opening the file and the closing of the file, we are going to end up with a memory leak, because the file is going to remain open throught the lifetime of the program. We can wrap the code in `try/exept` block, but it can be easily forgotten. Better approach to work with files using `with` keyword, as `with` block will automatically close the file, as soon as we exit this block:

```python
with open(file_path, 'r') as file:
    content: str = file.read()
```

Important to add, the `with` block won't catch exception for us, it will just properly close the file, so it is still reasonable to add `try/exept` block to catch relevant errors, such as `FileNotFoundError`.

```python
start: str = file.read(5) # reads first 5 bytes
middle: str = file.read(5) # reads next 5 bytes
rest: str = file.read() # read operation is exhaustive, we can read only once
empty: str = file.read()

first: str = file.readline() # reads first line (with \n at the end)
start: str = file.readline(5) # reads first 5 bytes of the line
middle: str = file.readline(5) # reads next 5 bytes of the line
rest: str = file.readline() # reads the rest of the line

lines: list[str] = file.readlines() # reads all lines
```

## Writing/appending to the file

Similar to read, we need to open the file in append (`'a'`) mode:

```python
with open(file_path, 'a+') as file:
    now: datetime = datetime.now()
    file.write(f'Access date: {now:%y-%m-%d}\n')

    # line breaks should be added explicitly each time
    file.writelines([f'User: {user}\n', 'Timezone: UTC\n', '\n'])

    # we can select which position of file we want to navigate using the seek
    file.seek(0)
    content: str = file.read() # reads all file, as it seeks from 0 byte
```

## Deleting files

```python
import os

os.remove(file_path) # FileNotFoundError

if os.path.exists(file_path):
    os.remove(file_path)

os.rmdir(dir_path) # FileNotFoundError
```

## JSON

We can convert json file to Python data structures using json loader:

```python
import json

with open(file_path, 'r') as file:
    data: dict = json.load(file)
```

JSON can be also loaded directly from the string:

```python
input: str = '{"price": 19.99, "title": "Grokking Algorithms"}'

# loads stands for 'load from string'
data: dict = json.loads(input)
```

And vice versa, json module will take care of converting Python's dictionary or list to the universal standard of json file / string:

```python
data: dict = {'name': 'David Beckham', 'age': 49, 'job': None}

json.dump(data, file)
output: str = json.dumps(data)
```

## Pickling

Pickling is a process for converting Python objects into a byte stream. And then we can do the opposite by unpickling it.

```python
import pickle

with open('sample.pickle', 'wb') as file:
    pickle.dump(object, file)

with open('sample.pickle', 'rb') as file:
    object = pickle.load(file)
```

The `pickle` module is not secure, and you should unpickle only data you trust. It can contain malicious pickle data that can execute arbitrary code during unpickling.

## Glob

The glob module finds all the path names matching a specific pattern according to the rules used by the Unix shell, although the result is returned in an arbitrary order.

```python
import glob

# rules for the path is Unix path expansion rule, not regex

# * is a wildcard for any char
files: list[str] = glob.glob('*.py')
# ? is a wildcard for any single char
files: list[str] = glob.glob('m??n.py')

# [] is a wildcard for any char in a sequence
files: list[str] = glob.glob('[mi]*.py') # any file that starts from 'm' or 'i'
# [!] is a wildcard for any character not in a sequence
files: list[str] = glob.glob('[!mi]*.py') # any file that starts not from 'm' or 'i'

# ** is a wildcard for any folders before
files: list[str] = glob.glob('**/*.py',
                             root_dir='/home/user',
                             # recursive makes sure it continuously look into the folders until it finds
                             # the file we have specified
                             recursive=True,
                             include_hidden=True)
```

glob tries to load all files at once or find them all at once, which can lead to a huge delay in a program. The solution to not load all that data at once is to use a generator version.

```python
globs: itertools.chain = glob.iglob('**/*.py',
                                    root_dir='/home/user',
                                    recursive=True,
                                    include_hidden=True)
```

## Documentation (doc-string)

Any kind of string that appears at the top of a function, a class, or a module is considered a doc string. Python's docstrings allow the creation of documentation for classes and functions. Parameters and return values can be described in sphinx format.

```python
"""[Summary]

:param [ParamName]: [ParamDescription], defaults to [DefaultParamVal]
:type [ParamName]: [ParamType](, optional)
...
:raises [ErrorType]: [ErrorDescription]
...
:return: [ReturnDescription]
:rtype: [ReturnType]
"""
```

Documentation can be accessed programmatically using the dunder method:

```python
User.__doc__
func.__doc__
```

More: https://sphinx-rtd-tutorial.readthedocs.io/en/latest/docstrings.html

<!-- nav -->
---

← [Python — Modules / Packages / Libraries](06-modules.md) · [Index](../README.md) · [Python — Generators, Decorators, Memoization](08-gen-decorators.md) →
<!-- nav -->
