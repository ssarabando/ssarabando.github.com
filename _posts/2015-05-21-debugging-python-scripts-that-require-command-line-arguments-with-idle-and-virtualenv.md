---
layout: post
title: Debugging Python 3.4 scripts that require command line arguments with IDLE from a VirtualEnv
comments: False
---

# {{ page.title }}

_21 May 2015_

To my future self: the detour I mentioned in the [last post]({{ site.baseurl }}/2015/02/01/pyrx-and-windows-service.html) is bigger than anticipated and I'm still in Python lands.

Today I had to debug a Python script _in production_.

The script was running in a `virtualenv` and it required command line arguments.

I also needed **IDLE** to follow up the source and locals.

Setting that up was harder then I thought so I took the time to save it here for future reference.

First, open a command prompt with administrative privileges and `cd` into your virtualenv.
Now, and since IDLE requires **Tcl** and **Tk**, you'll have to symlink the required folders into the virtualenv's `Lib` folder. In my case, that's Tcl/Tk 8.6:

<pre>
mklink /d tcl8.6 C:\Python34\tcl\tcl8.6
mklink /d tk8.6 C:\Python34\tcl\tk8.6
</pre>

Next, `cd` into your project's folder and activate your virtualenv. Run the following command:

<pre>
python -c "from idlelib.PyShell import main; main()"
</pre>

Now that IDLE is open, let's prepare the environment. In the Python console, write:

``` python
import sys
filename = 'yourfilename.py'
# replace arg1 and arg2 with your arguments (add/remove as needed)
sys.argv = [filename, arg1, arg2]
g = globals()
g['__file__'] = filename
```

Time to turn the debugger on and write the next command:

``` python
exec(compile(open(filename, 'rb').read(), filename, 'exec'), g)
```

At this stage, the debugger will simply stop at the 1st line. Select the `Source` checkbox and `Step` and IDLE will open the file for you. You can now start debugging your code.

This solution was heavily influenced by the following posts:

* [When running a python script in IDLE, is there a way to pass in command line arguments (args)?](http://stackoverflow.com/q/2148994/215576)
* [Alternative to execfile in Python 3.2+?](http://stackoverflow.com/q/6357361/215576) (namely Sven Marnach's [answer](http://stackoverflow.com/a/6357418/215576))
* [How to launch python Idle from a virtual environment (virtualenv)](http://stackoverflow.com/q/4924068/215576) (mainly _srock_ 's [answer](http://stackoverflow.com/a/25773601/215576))
