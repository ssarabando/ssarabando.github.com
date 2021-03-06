---
layout: post
title: RxPY (Python Reactive Extensions) and a Python Windows service
comments: False
---

# {{ page.title }}

_2 Feb 2015_

To my future self: I had to take a small detour from the announced program. Instead of the ASP.NET MVC 4 globalization hurdles post, I'll be taking notes on how to run a Rx timer subscription from a Python script running as a Windows service.

First things first: I am aware that I could be using `Observable.interval` and that I could even do this without using Rx at all. The thing is, I'll be using Rx in the near future (if all goes according to plan) and in Python and as a _daemonized_ process so I took this as a low-risk opportunity to test all of those out.

Now, you'll need to install both `PyWin32` and `RxPy` (I used `pip` to install both) for this to work (and Python of course -- v3.4 32 bit in my case).

The Rx part is straightforward: just declare an observable timer and subscribe to it. Example:

``` python
import rx
# Start a timer in 5s and then have it 'tick' every 1s
timer = rx.Observable.timer(5000, 1000)
# Just print
subscription = timer.subscribe(lambda t: print(t))
```

Bolting in the Windows service bits is, again, straightforward:

``` python
import rx
import servicemanager
import win32event
import win32service
import win32serviceutil

class MyService(win32serviceutil.ServiceFramework):
    _svc_name_ = "MyService"
    _svc_display_name_ = "My Service"

    def __init__(self, args):
        win32serviceutil.ServiceFramework.__init__(self, args)

    def SvcStop(self):
        self.ReportServiceStatus(win32service.SERVICE_STOP_PENDING)
        servicemanager.LogMsg(servicemanager.EVENTLOG_INFORMATION_TYPE,
                              servicemanager.PYS_SERVICE_STOPPED,
                              (self._svc_name_, ''))
    
    def SvcDoRun(self):
        servicemanager.LogMsg(servicemanager.EVENTLOG_INFORMATION_TYPE,
                              servicemanager.PYS_SERVICE_STARTED,
                              (self._svc_name_, ''))
        self.main()

    def write(self, msg):
        # Write to a file just to know the service is ticking
        with open('C:\\temp\\myservice.log', 'a') as f:
            print('{}'.format(msg), file=f)

    def main(self):
        # Just hard code duetime and period here
        timer = rx.Observable.timer(5000, 1000)
        subscription = timer.subscribe(self.write)

if __name__ == '__main__':
    win32serviceutil.HandleCommandLine(MyService)
```

Of course, the 1st time I run it, it just started and stopped without writing anything.

Only then I understood why the recipe was using an event: to stop the script from ending until it received a stop event.

Since my use case is simpler than the one in the recipe (the script must be always running until it receives a stop event), that led me to this:

``` python
import rx
import servicemanager
import win32event
import win32service
import win32serviceutil

class MyService(win32serviceutil.ServiceFramework):
    _svc_name_ = "MyService"
    _svc_display_name_ = "My Service"

    def __init__(self, args):
        win32serviceutil.ServiceFramework.__init__(self, args)
        self.hWaitStop = win32event.CreateEvent(None, 0, 0, None)

    def SvcStop(self):
        self.ReportServiceStatus(win32service.SERVICE_STOP_PENDING)
        # Signal the stop event
        win32event.SetEvent(self.hWaitStop)
        servicemanager.LogMsg(servicemanager.EVENTLOG_INFORMATION_TYPE,
                              servicemanager.PYS_SERVICE_STOPPED,
                              (self._svc_name_, ''))
    
    def SvcDoRun(self):
        servicemanager.LogMsg(servicemanager.EVENTLOG_INFORMATION_TYPE,
                              servicemanager.PYS_SERVICE_STARTED,
                              (self._svc_name_, ''))
        self.main()

    def write(self, msg):
        # Write to a file just to know the service is ticking
        with open('C:\\temp\\myservice.log', 'a') as f:
            print('{}'.format(msg), file=f)

    def main(self):
        # Just hard code duetime and period here
        timer = rx.Observable.timer(5000, 1000)
        subscription = timer.subscribe(self.write)
        # Wait on the stop event
        win32event.WaitForSingleObject(self.hWaitStop, win32event.INFINITE)
        # Clean up the subscription
        subscription.dispose()

if __name__ == '__main__':
    win32serviceutil.HandleCommandLine(MyService)
```

This will now serve as a basis for more prototyping: adding in 0MQ!

#### References

This solution was heavily influenced by the following post:

* [How to create a Windows service in Python (Python recipe)](http://code.activestate.com/recipes/576451-how-to-create-a-windows-service-in-python/)
