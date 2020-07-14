```powershell
Microsoft Windows [版本 10.0.18363.900]
(c) 2019 Microsoft Corporation。保留所有权利。

C:\Users\cpucode>pip install sphinx sphinx-autobuild
Collecting sphinx
  Downloading Sphinx-3.1.2-py3-none-any.whl (2.9 MB)
     |████████████████████████████████| 2.9 MB 91 kB/s
Collecting sphinx-autobuild
  Downloading sphinx-autobuild-0.7.1.tar.gz (14 kB)
Collecting babel>=1.3
  Downloading Babel-2.8.0-py2.py3-none-any.whl (8.6 MB)
     |███████████████████████         | 6.3 MB 5.6 kB/s eta 0:06:55ERROR: Exception:
Traceback (most recent call last):
  File "d:\program files\python\install\lib\site-packages\pip\_vendor\urllib3\response.py", line 425, in _error_catcher
    yield
  File "d:\program files\python\install\lib\site-packages\pip\_vendor\urllib3\response.py", line 507, in read
    data = self._fp.read(amt) if not fp_closed else b""
  File "d:\program files\python\install\lib\site-packages\pip\_vendor\cachecontrol\filewrapper.py", line 62, in read
    data = self.__fp.read(amt)
  File "d:\program files\python\install\lib\http\client.py", line 454, in read
    n = self.readinto(b)
  File "d:\program files\python\install\lib\http\client.py", line 498, in readinto
    n = self.fp.readinto(b)
  File "d:\program files\python\install\lib\socket.py", line 669, in readinto
    return self._sock.recv_into(b)
  File "d:\program files\python\install\lib\ssl.py", line 1241, in recv_into
    return self.read(nbytes, buffer)
  File "d:\program files\python\install\lib\ssl.py", line 1099, in read
    return self._sslobj.read(len, buffer)
socket.timeout: The read operation timed out

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "d:\program files\python\install\lib\site-packages\pip\_internal\cli\base_command.py", line 188, in _main
    status = self.run(options, args)
  File "d:\program files\python\install\lib\site-packages\pip\_internal\cli\req_command.py", line 185, in wrapper
    return func(self, options, args)
  File "d:\program files\python\install\lib\site-packages\pip\_internal\commands\install.py", line 332, in run
    requirement_set = resolver.resolve(
  File "d:\program files\python\install\lib\site-packages\pip\_internal\resolution\legacy\resolver.py", line 179, in resolve
    discovered_reqs.extend(self._resolve_one(requirement_set, req))
  File "d:\program files\python\install\lib\site-packages\pip\_internal\resolution\legacy\resolver.py", line 362, in _resolve_one
    abstract_dist = self._get_abstract_dist_for(req_to_install)
  File "d:\program files\python\install\lib\site-packages\pip\_internal\resolution\legacy\resolver.py", line 314, in _get_abstract_dist_for
    abstract_dist = self.preparer.prepare_linked_requirement(req)
  File "d:\program files\python\install\lib\site-packages\pip\_internal\operations\prepare.py", line 467, in prepare_linked_requirement
    local_file = unpack_url(
  File "d:\program files\python\install\lib\site-packages\pip\_internal\operations\prepare.py", line 255, in unpack_url
    file = get_http_url(
  File "d:\program files\python\install\lib\site-packages\pip\_internal\operations\prepare.py", line 129, in get_http_url
    from_path, content_type = _download_http_url(
  File "d:\program files\python\install\lib\site-packages\pip\_internal\operations\prepare.py", line 281, in _download_http_url
    for chunk in download.chunks:
  File "d:\program files\python\install\lib\site-packages\pip\_internal\cli\progress_bars.py", line 166, in iter
    for x in it:
  File "d:\program files\python\install\lib\site-packages\pip\_internal\network\utils.py", line 15, in response_chunks
    for chunk in response.raw.stream(
  File "d:\program files\python\install\lib\site-packages\pip\_vendor\urllib3\response.py", line 564, in stream
    data = self.read(amt=amt, decode_content=decode_content)
  File "d:\program files\python\install\lib\site-packages\pip\_vendor\urllib3\response.py", line 529, in read
    raise IncompleteRead(self._fp_bytes_read, self.length_remaining)
  File "d:\program files\python\install\lib\contextlib.py", line 131, in __exit__
    self.gen.throw(type, value, traceback)
  File "d:\program files\python\install\lib\site-packages\pip\_vendor\urllib3\response.py", line 430, in _error_catcher
    raise ReadTimeoutError(self._pool, None, "Read timed out.")
pip._vendor.urllib3.exceptions.ReadTimeoutError: HTTPSConnectionPool(host='files.pythonhosted.org', port=443): Read timed out.

C:\Users\cpucode>pip install sphinx sphinx-autobuild
Collecting sphinx
  Using cached Sphinx-3.1.2-py3-none-any.whl (2.9 MB)
Collecting sphinx-autobuild
  Using cached sphinx-autobuild-0.7.1.tar.gz (14 kB)
Collecting sphinxcontrib-applehelp
  Downloading sphinxcontrib_applehelp-1.0.2-py2.py3-none-any.whl (121 kB)
     |████████████████████████████████| 121 kB 159 kB/s
Collecting sphinxcontrib-jsmath
  Downloading sphinxcontrib_jsmath-1.0.1-py2.py3-none-any.whl (5.1 kB)
Collecting sphinxcontrib-qthelp
  Downloading sphinxcontrib_qthelp-1.0.3-py2.py3-none-any.whl (90 kB)
     |████████████████████████████████| 90 kB 248 kB/s
Collecting docutils>=0.12
  Downloading docutils-0.16-py2.py3-none-any.whl (548 kB)
     |████████████████████████████████| 548 kB 234 kB/s
Collecting requests>=2.5.0
  Downloading requests-2.24.0-py2.py3-none-any.whl (61 kB)
     |████████████████████████████████| 61 kB 106 kB/s
Collecting alabaster<0.8,>=0.7
  Downloading alabaster-0.7.12-py2.py3-none-any.whl (14 kB)
Collecting Pygments>=2.0
  Downloading Pygments-2.6.1-py3-none-any.whl (914 kB)
     |████████████████████████████████| 914 kB 148 kB/s
Requirement already satisfied: setuptools in d:\program files\python\install\lib\site-packages (from sphinx) (47.1.0)
Collecting Jinja2>=2.3
  Downloading Jinja2-2.11.2-py2.py3-none-any.whl (125 kB)
     |████████████████████████████████| 125 kB 133 kB/s
Collecting imagesize
  Downloading imagesize-1.2.0-py2.py3-none-any.whl (4.8 kB)
Collecting packaging
  Downloading packaging-20.4-py2.py3-none-any.whl (37 kB)
Collecting snowballstemmer>=1.1
  Downloading snowballstemmer-2.0.0-py2.py3-none-any.whl (97 kB)
     |████████████████████████████████| 97 kB 181 kB/s
Collecting babel>=1.3
  Downloading Babel-2.8.0-py2.py3-none-any.whl (8.6 MB)
     |████████████████████████████████| 8.6 MB 99 kB/s
Collecting colorama>=0.3.5; sys_platform == "win32"
  Downloading colorama-0.4.3-py2.py3-none-any.whl (15 kB)
Collecting sphinxcontrib-htmlhelp
  Downloading sphinxcontrib_htmlhelp-1.0.3-py2.py3-none-any.whl (96 kB)
     |████████████████████████████████| 96 kB 120 kB/s
Collecting sphinxcontrib-devhelp
  Downloading sphinxcontrib_devhelp-1.0.2-py2.py3-none-any.whl (84 kB)
     |████████████████████████████████| 84 kB 110 kB/s
Collecting sphinxcontrib-serializinghtml
  Downloading sphinxcontrib_serializinghtml-1.1.4-py2.py3-none-any.whl (89 kB)
     |████████████████████████████████| 89 kB 93 kB/s
Collecting watchdog>=0.7.1
  Downloading watchdog-0.10.3.tar.gz (94 kB)
     |████████████████████████████████| 94 kB 167 kB/s
Collecting argh>=0.24.1
  Downloading argh-0.26.2-py2.py3-none-any.whl (30 kB)
Collecting pathtools>=0.1.2
  Downloading pathtools-0.1.2.tar.gz (11 kB)
Collecting PyYAML>=3.10
  Downloading PyYAML-5.3.1-cp38-cp38-win_amd64.whl (219 kB)
     |████████████████████████████████| 219 kB 121 kB/s
Collecting tornado>=3.2
  Downloading tornado-6.0.4-cp38-cp38-win_amd64.whl (417 kB)
     |████████████████████████████████| 417 kB 148 kB/s
Collecting port_for==0.3.1
  Downloading port-for-0.3.1.tar.gz (18 kB)
Collecting livereload>=2.3.0
  Downloading livereload-2.6.2.tar.gz (25 kB)
Collecting idna<3,>=2.5
  Downloading idna-2.10-py2.py3-none-any.whl (58 kB)
     |████████████████████████████████| 58 kB 302 kB/s
Collecting certifi>=2017.4.17
  Downloading certifi-2020.6.20-py2.py3-none-any.whl (156 kB)
     |████████████████████████████████| 156 kB 187 kB/s
Collecting chardet<4,>=3.0.2
  Downloading chardet-3.0.4-py2.py3-none-any.whl (133 kB)
     |████████████████████████████████| 133 kB 159 kB/s
Collecting urllib3!=1.25.0,!=1.25.1,<1.26,>=1.21.1
  Downloading urllib3-1.25.9-py2.py3-none-any.whl (126 kB)
     |████████████████████████████████| 126 kB 211 kB/s
Collecting MarkupSafe>=0.23
  Downloading MarkupSafe-1.1.1-cp38-cp38-win_amd64.whl (16 kB)
Collecting six
  Downloading six-1.15.0-py2.py3-none-any.whl (10 kB)
Collecting pyparsing>=2.0.2
  Downloading pyparsing-2.4.7-py2.py3-none-any.whl (67 kB)
     |████████████████████████████████| 67 kB 241 kB/s
Collecting pytz>=2015.7
  Downloading pytz-2020.1-py2.py3-none-any.whl (510 kB)
     |████████████████████████████████| 510 kB 145 kB/s
Using legacy setup.py install for sphinx-autobuild, since package 'wheel' is not installed.
Using legacy setup.py install for watchdog, since package 'wheel' is not installed.
Using legacy setup.py install for pathtools, since package 'wheel' is not installed.
Using legacy setup.py install for port-for, since package 'wheel' is not installed.
Using legacy setup.py install for livereload, since package 'wheel' is not installed.
Installing collected packages: sphinxcontrib-applehelp, sphinxcontrib-jsmath, sphinxcontrib-qthelp, docutils, idna, certifi, chardet, urllib3, requests, alabaster, Pygments, MarkupSafe, Jinja2, imagesize, six, pyparsing, packaging, snowballstemmer, pytz, babel, colorama, sphinxcontrib-htmlhelp, sphinxcontrib-devhelp, sphinxcontrib-serializinghtml, sphinx, pathtools, watchdog, argh, PyYAML, tornado, port-for, livereload, sphinx-autobuild
    Running setup.py install for pathtools ... done
    Running setup.py install for watchdog ... done
    Running setup.py install for port-for ... done
    Running setup.py install for livereload ... done
    Running setup.py install for sphinx-autobuild ... done
Successfully installed Jinja2-2.11.2 MarkupSafe-1.1.1 PyYAML-5.3.1 Pygments-2.6.1 alabaster-0.7.12 argh-0.26.2 babel-2.8.0 certifi-2020.6.20 chardet-3.0.4 colorama-0.4.3 docutils-0.16 idna-2.10 imagesize-1.2.0 livereload-2.6.2 packaging-20.4 pathtools-0.1.2 port-for-0.3.1 pyparsing-2.4.7 pytz-2020.1 requests-2.24.0 six-1.15.0 snowballstemmer-2.0.0 sphinx-3.1.2 sphinx-autobuild-0.7.1 sphinxcontrib-applehelp-1.0.2 sphinxcontrib-devhelp-1.0.2 sphinxcontrib-htmlhelp-1.0.3 sphinxcontrib-jsmath-1.0.1 sphinxcontrib-qthelp-1.0.3 sphinxcontrib-serializinghtml-1.1.4 tornado-6.0.4 urllib3-1.25.9 watchdog-0.10.3

C:\Users\cpucode>pip install restructuredtext-lint
Collecting restructuredtext-lint
  Downloading restructuredtext_lint-1.3.1.tar.gz (13 kB)
Requirement already satisfied: docutils<1.0,>=0.11 in d:\program files\python\install\lib\site-packages (from restructuredtext-lint) (0.16)
Using legacy setup.py install for restructuredtext-lint, since package 'wheel' is not installed.
Installing collected packages: restructuredtext-lint
    Running setup.py install for restructuredtext-lint ... done
Successfully installed restructuredtext-lint-1.3.1

C:\Users\cpucode>
```



```powershell

尝试新的跨平台 PowerShell https://aka.ms/pscore6

PS D:\Date\github\ReadTheDocs>  sphinx-quickstart
Welcome to the Sphinx 3.1.2 quickstart utility.

Please enter values for the following settings (just press Enter to
accept a default value, if one is given in brackets).

Selected root path: .

You have two options for placing the build directory for Sphinx output.
Either, you use a directory "_build" within the root path, or you separate
"source" and "build" directories within the root path.
> Separate source and build directories (y/n) [n]: 

The project name will occur in several places in the built documentation.
> Project name: ReadTheDocs 
> Author name(s): cpucode
> Project release []: 0.0.1

If the documents are to be written in a language other than English,
you can select a language here by its language code. Sphinx will then
translate text that it generates into that language.

For a list of supported codes, see
https://www.sphinx-doc.org/en/master/usage/configuration.html#confval-language.
> Project language [en]: 

Creating file D:\Date\github\ReadTheDocs\conf.py.
Creating file D:\Date\github\ReadTheDocs\index.rst.
Creating file D:\Date\github\ReadTheDocs\Makefile.
Creating file D:\Date\github\ReadTheDocs\make.bat.

Finished: An initial directory structure has been created.

You should now populate your master file D:\Date\github\ReadTheDocs\index.rst and create other documentation
source files. Use the Makefile to build the docs, like so:
   make builder
where "builder" is one of the supported builders, e.g. html, latex or linkcheck.

PS D:\Date\github\ReadTheDocs> 
```



![image-20200714160533085](https://gitee.com/cpu_code/picture_bed/raw/master//20200714160540.png)

![image-20200714200850303](https://gitee.com/cpu_code/picture_bed/raw/master//20200714200850.png)