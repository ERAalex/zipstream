
# python-zipstream

[![Build Status](https://travis-ci.org/allanlei/python-zipstream.png?branch=master)](https://travis-ci.org/allanlei/python-zipstream)
[![Coverage Status](https://coveralls.io/repos/allanlei/python-zipstream/badge.png)](https://coveralls.io/r/allanlei/python-zipstream)


## Cloudike - Time Control for ZIP Archives

Ticket: NRF-3930
When preparing a ZIP archive using ZipFile, there is no built-in way to control the creation time of files and folders.

### Solution
A new interface has been added to modify the timestamps of files and folders inside the archive.
In __init__.py, a new parameter time_set was introduced in the ZipFile class:
```python
self.time_set = time_set  # Added for time control when creating a ZIP archive
```

Later, in the __write method, a check is performed to see if this parameter was provided, and the timestamp is adjusted accordingly:

```python
if self.time_set:
    time_offset_minutes = self.time_set  # Example: 540 (9 hours - South Korea time)
    time_offset_seconds = time_offset_minutes * 60  # Convert to seconds

    # Convert date_time to a datetime object, apply the offset, and reformat it as a tuple
    dt = datetime(*date_time) + timedelta(seconds=time_offset_seconds)
    date_time = (dt.year, dt.month, dt.day, dt.hour, dt.minute, dt.second)
```

### ********** Base Info
zipstream.py is a zip archive generator based on python 3.3's zipfile.py. It was created to
generate a zip file generator for streaming (ie web apps). This is beneficial for when you
want to provide a downloadable archive of a large collection of regular files, which would be infeasible to
generate the archive prior to downloading or of a very large file that you do not want to store entirely on disk or on memory.

The archive is generated as an iterator of strings, which, when joined, form
the zip archive. For example, the following code snippet would write a zip
archive containing files from 'path' to a normal file:

```python
import zipstream

z = zipstream.ZipFile()
z.write('path/to/files')

with open('zipfile.zip', 'wb') as f:
    for data in z:
        f.write(data)
```

zipstream also allows to take as input a byte string iterable and to generate
the archive as an iterator.
This avoids storing large files on disk or in memory.
To do so you could use something like this snippet:

```python
def iterable():
    for _ in xrange(10):
        yield b'this is a byte string\x01\n'

z = zipstream.ZipFile()
z.write_iter('my_archive_iter', iterable())

with open('zipfile.zip', 'wb') as f:
    for data in z:
        f.write(data)
```

Of course both approach can be combined:

```python
def iterable():
    for _ in xrange(10):
        yield b'this is a byte string\x01\n'

z = zipstream.ZipFile()
z.write('path/to/files', 'my_archive_files')
z.write_iter('my_archive_iter', iterable())

with open('zipfile.zip', 'wb') as f:
    for data in z:
        f.write(data)
```

Since recent versions of web.py support returning iterators of strings to be
sent to the browser, to download a dynamically generated archive, you could
use something like this snippet:

```python
def GET(self):
    path = '/path/to/dir/of/files'
    zip_filename = 'files.zip'
    web.header('Content-type' , 'application/zip')
    web.header('Content-Disposition', 'attachment; filename="%s"' % (
        zip_filename,))
    return zipstream.ZipFile(path)
```

If the zlib module is available, zipstream.ZipFile can generate compressed zip
archives.

## Installation

```
pip install zipstream
```

## Requirements

  * Python 2.6, 2.7, 3.2, 3.3, pypy

## Examples

### flask

```python
from flask import Response

@app.route('/package.zip', methods=['GET'], endpoint='zipball')
def zipball():
    def generator():
    	z = zipstream.ZipFile(mode='w', compression=ZIP_DEFLATED)

    	z.write('/path/to/file')

    	for chunk in z:
    		yield chunk

    response = Response(generator(), mimetype='application/zip')
    response.headers['Content-Disposition'] = 'attachment; filename={}'.format('files.zip')
    return response

# or

@app.route('/package.zip', methods=['GET'], endpoint='zipball')
def zipball():
	z = zipstream.ZipFile(mode='w', compression=ZIP_DEFLATED)
	z.write('/path/to/file')

    response = Response(z, mimetype='application/zip')
    response.headers['Content-Disposition'] = 'attachment; filename={}'.format('files.zip')
    return response
```

### django 1.5+

```python
from django.http import StreamingHttpResponse

def zipball(request):
	z = zipstream.ZipFile(mode='w', compression=ZIP_DEFLATED)
	z.write('/path/to/file')

    response = StreamingHttpResponse(z, content_type='application/zip')
    response['Content-Disposition'] = 'attachment; filename={}'.format('files.zip')
    return response
```

### webpy

```python
def GET(self):
    path = '/path/to/dir/of/files'
    zip_filename = 'files.zip'
    web.header('Content-type' , 'application/zip')
    web.header('Content-Disposition', 'attachment; filename="%s"' % (
        zip_filename,))
    return zipstream.ZipFile(path)
```

## Running tests

With python version > 2.6, just run the following command: `python -m unittest discover`

Alternatively, you can use `nose`.
