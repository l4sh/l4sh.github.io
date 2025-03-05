---
layout: post
current: post
navigation: True
title: Some pointers on installing mod_wsgi on Windows
author: l4sh
date: 2018-10-08
category: tech
class: post-template
subclass: post
tags:
  - sysadmin
comments: true
---

Recently I needed to install mod_wsgi in a Windows box and this proved to be a little bit tricky. Granted this was a different base installation from the one recommended by the mod_wsgi developer and I'm not really a Windows user, but I thought this might help someone with the same issues.

If you can choose the Apache httpd distribution to install, just use the one from Apache Lounge (<https://apachelounge.com>). Is the one recommended in the mod_wsgi readme and the easiest one to build the package for. In my case I needed to install mod_wsgi onto an already running instance of Apache httpd.

The installation reflected in this post is a 32 bit one. Use the appropriate packages accordingly. Don't mix different architechture packages.

## Install python

Download the python installer from <https://www.python.org/downloads/windows/> and execute it. Make sure that it's being installed for all users (it will install to `C:\Program Files\Python`), you might need to use the "Customize installation" option.

Once the process is done open a prompt and try running the following commands.

```terminal
python -V
pip -V
```

It should print both the version of the python interpreter and the python installer.

## Install the Visual C++ compiler

Download and install "Build tools for Visual Studio" from <https://visualstudio.microsoft.com/downloads/>

Note that the "Visual C++ redistributable package" installs just the runtime components. For this we need the build tools package.

For more information check <https://wiki.python.org/moin/WindowsCompilers>

## Upgrade setuptools

Open a command prompt and run:

```terminal
pip install --upgrade setuptools
```

## Install mod_wsgi

If the Apache httpd instance is installed in a path other than `C:\Apache\Apache24` (Apache Lounge's default installation path) the `MOD_WSGI_APACHE_ROOTDIR` environment variable will need to be set. In this case I had to set it to the following path `C:\Program Files (x86)\Apache Software Foundation\Apache`

From within a command prompt run:

```terminal
pip install mod_wsgi
```

If everything is alright it should just display a message announcing that the package has been installed.

In my case it threw an error complaining about missing files in the `include` directory. This directory contains the headers needed for compiling the `mod_wsgi` module. This Apache httpd installation lacked some files and contained a few extra ones when compared with an Apache Lounge release installed in a test box. This issue was solved by copying the files from the Apache Lounge release and running again the `mod_wsgi` install command. If you need to do this, make sure that the major and minor versions of Apache httpd match.

## Configure mod_wsgi

After installing the `mod_wsgi` package some configuration is needed.

In order to obtain the proper paths for we need to run the following command:

```terminal
mod_wsgi-express module-config
```

It should return an output like:

```apache
LoadFile "c:/program files (x86)/python37-32/python37.dll"
LoadModule wsgi_module "c:/program files (x86)/python37-32/lib/site-packages/mod_wsgi/server/mod_wsgi.cp37-win32.pyd"
WSGIPythonHome "c:/program files (x86)/python37-32"
```

This output will then need to be copied in the `conf/httpd.conf` file or other relevant configuration file in the Apache httpd installation folder.

Upon restarting Apache httpd the server control window should now display a new item in the status bar that reads "mod_wsgi".

This should be enough to run a WSGI app with the module.

## Testing it all

Create a new python file (`C:\WSGIAppDir\wsgi_app.py`) with the following content.

```python
def wsgi_app(environ, start_response):
    output = b'It Works!'
    status = '200 OK'
    headers = [('Content-type', 'text/plain'),
               ('Content-Length', str(len(output)))]
    start_response(status, headers)
    return output
```

Add a new section to the apache configuration, either in a new vhost configuration file or at the end of `httpd.conf` with the following content:

```apache
<IfModule wsgi_module>
    WSGIScriptAlias /test-wsgi/ "C:/WSGIAppDir/wsgi_app.py"
    <Directory "C:/WSGIAppDir">
        AllowOverride None
        Options None
        Order deny,allow
        Allow from all
    </Directory>
</IfModule>
```

Notice the forward slashes in the directories instead of the MS Windows traditional backward slashes.
