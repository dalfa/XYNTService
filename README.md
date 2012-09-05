# XYNTService

This is a copy of the XYNTSerivce source code that used to be on
codeproject.com, created by Xiangyang Liu. It was taken down in 14 Jul
2012. This is the last known good copy I could find. What follows is the
documentation taken from the original website.

## Introduction

Typically, an NT service is a console application, which does not have a message
pump.  An NT Service can be started without the user having to login to the
computer and it won't die after user logoff.  However, it is hard, sometimes
impossible, to use many existing ActiveX controls within a console application.

On the other hand, MFC and VB applications are windows applications so using
ActiveX controls in MFC or VB programs is extremely easy. It would be nice to
make your MFC and VB programs run like an NT service so that:

* They will be started before the user logs into to the computer.
* They will keep running after the user has logged off. 

It is possible to write an NT service as a Windows program but I am proposing a
much easier solution.  I have included with this article the source code for a
simple NT Service program that can start and shutdown other programs.  All you
need to do is install this service and modify an .ini file.  Here are the
advantages of using this simple NT service:

* It can start as many programs as you want.  The started programs behave like NT
services (i.e. they will be running in the background without user having to
login to the machine).

* A user cannot kill the programs started by this service without proper privilege
(unless the machine is shutdown, of course).

* You can test and debug your programs outside of the NT Service.  For example,
you can run your programs in the DevStudio debugger, step into the source code
to find the bugs, etc.  When it is "bug free", you deploy it in production,
starting it from the NT Service.

## XYNTService

XYNTService.exe is the name of the executable for this NT service program.  It
is part of a client-server development tool I invented.  You can freely use and
modify the source code included with this article.  I am now aware that there
are other utility programs that provide almost the same functionality as
XYNTService.  However, as you will see, XYNTService has more features and it is
a lot easier to use (no editing of the registry is required, for example).  Here
is how to use the program.

To install the service, run the following at the command prompt: 

    XYNTService -i

To un-install the service, run the following at the command prompt:

    XYNTService -u 

By default, the installed service will be started automatically when you reboot
the computer.  You can also start and shutdown the service from the Control
Panel using the Services icon.  When the service is started, it will create all
the processes you defined in the `XYNTService.ini` file one by one.  When the
service is shutdown, it will terminate each of the processes it created (in
reverse order).  The `XYNTService.ini` file should be placed in the same directory
as the executable.  Here is a sample of the file:

    [Settings]
    ServiceName = XYNTService
    ProcCount = 3
    CheckProcess = 30
    [Process0]
    CommandLine = c:\MyDir\XYRoot.exe
    WorkingDir = c:\MyDir
    PauseStart = 1000
    PauseEnd = 1000
    UserInterface = Yes
    Restart = Yes
    [Process1]
    CommandLine = c:\MyDir\XYDataManager.exe
    WorkingDir = c:\MyDir
    PauseStart = 1000
    PauseEnd = 1000
    UserInterface = Yes
    Restart = Yes
    [Process2]
    CommandLine= java XYRoot.XYRoot XYRootJava.ini
    UserInterface = No
    Restart = No

The `ServiceName` property specifies the name you want to use for this NT service,
the default name is XYNTService. If you copy the executable and the .ini file
into a different directory, and modify the `ServiceName` property in the .ini
file, then you can install and configure a different service!

The `ProcCount` property specifies how many processes you want this service to
create.  The sections [Process0], [Process1] , ..., etc., define properties
related to each of these processes. As you can see, there are 3 processes to
create in this example, `XYRoot.exe` , `XYDataManager`, and java are the names of
the programs, and you can specify parameters for each of these processes in the
`CommandLine` property.  You must specify the full path of the executable file for
the corresponding process in the `CommandLine` property unless the executable is
already in the system path.

The `CheckProcess` property specifies whether and how often you want to check
processes started by XYNTService .  If the property has value 0 , then no
checking is done.  If the property value is 30, for example, then every 30
minutes XYNTService will query the operating system to see if the processes it
started are still running and the dead ones will be restarted if the Restart
property value (explained later) is defined to be Yes for that process.  The
default value of this property (if you don't specify it) is 60 .

The `WorkingDir` property is the working directory of the current process.  If you
don't specify this property, then the working directory of the current process
will be `c:\winnt\system32`.  The `PauseStart` property is the number of
milliseconds the service will wait after starting the current process (and
before starting the next process).  This is useful in the case where the next
process depends on the previous process.  For example, the second process has to
"connect" to the first process so that it should not be run until the first
process is finished with initialization.  If you don't specify the `PauseStart`
property, the default value will be 100 milliseconds.

When XYNTService is shutdown, it will post `WM_QUIT` messages to the processes it
created first and then call the WIN32 function `TerminateProcess`. The `PauseEnd`
property is the number of milliseconds the service will wait before
`TerminateProcess` is called.  This property can be used to give a process
(started by XYNTService) a chance to clean up and shutdown itself.  If you don't
specify the `PauseEnd` property, the default value will be 100 milliseconds.

The `UserInterface` property controls whether a logged on user can see the
processes created by XYNTService. However, this only works when XYNTService is
running under the local system account, which is the default. In this case,
processes created by XYNTService will not be able to access a specific user's
settings (e-mail profiles, etc.).  You can configure XYNTService to run under a
user account, which is done easily from the Control Panel (double click the
Services icon and then double click XYNTService in the installed services list
to bring up a dialog box).

The `Restart` property is used to decided whether you want XYNTService to restart
a dead process.  If this property is No (which is the default if you don't
specify it), then the corresponding process will not be restarted.  If this
property is Yes, then the dead process will be restarted by XYNTService.  See
the `CheckProcess` property above on how often dead processes are restarted.

You can bounce (stop and restart) any process defined in the .ini file from the
command line. For example, the following command

    XYNTService -b 2

will stop and restart the process defined in the `[Process2]` section of the .ini file.

XYNTService can also be used to start and stop other services from the command
line.  Here are the commands to start (run) and stop (kill) other services

    XYNTService -r NameOfServiceToRun

    XYNTService -k NameOfServiceToKill

In particular, you can use the above commands to start and stop XYNTService
itself from command line!  Please note that you cannot start XYNTService by
running it from the command prompt without any argument.

All errors while running XYNTService are written into a log file in the same
directory as the executable.  The error code in the log file is a decimal number
returned by the `GetLastError` API, you can look it up in MSDN.

## Latest Updates

A new feature is added so that XYNTService can check the processes it started
periodically.  A dead process will be restarted by XYNTService if you specify
the Restart property for this process in the `XYNTService.ini` file.

The author would like to thank user `WolfSupernova` for finding a bug in the code
that will prevent XYNTService from terminating the programs defined in
`XYNTService.ini` when the machine is rebooted.

## Frequently Asked Questions

* Why can't XYNTService start my program?  There could be many reasons.  The
likely ones are, you did not give the correct path of the executable in the
`XYNTService.ini` file or your program is located on a mapped network drive (see
the following question).

* My program works fine outside of XYNTService, why does it fail when started by
XYNTService?  XYNTService is running under the "local system" account by
default, any program started by it will also use this account.  Your program may
need some resource that is not available to this account.  For example, "local
system" cannot access the current user's registry settings, nor can it access a
mapped network drive or any other resource on the LAN.  However, you can change
the account used by XYNTService, this topic is covered by the question below.

* How do I change the account XYNTService uses?  On Windows 2000, use the
"Adminstrative Tools" menu, select "Services" and then double click XYNTService
from the displayed list to change the logon information (domain name, user name,
and password).  On Windows NT 4.0, use the Control Panel, select/open the
Services icon, then double click XYNTService from the displayed list to change
the logon information.  Please note that you cannot see your program started by
XYNTService if XYNTService is not using the "local system" account.

* How to run my program as a service?  You can't, unless you rewrite your program
from scratch.  A service is a special program which requires some special
knowledge to write.  If you don't want to learn how to write a service, then use
XYNTService to start your program as described in the article so that your
program behaves like a service.

* How do I debug a service?  First, build your service program in debug mode from
Visual Studio, setting break points if necessary.  Then install and start the
service.  Finally, attach the debugger to the service executable (that is
already running).  Note that a service has to be started before the process can
be attached to the debugger, so some parts of it can never be stepped into (from
the debugger).  In fact, most useful code in XYNTService cannot be debugged
because the code is executed while XYNTService is starting.

* Why my java program won't run under XYNTService or dies when user logs off the
machine?  This could be caused by a bug in java itself, I think.  I had some
problems with recent versions of JDK, but not with JDK 1.2.2.  If you cannot use
JDK 1.2.2, then try the Xrs option when using java and also change the account
used by XYNTService to a domain user instead of the default "local system"
account.  This may help you to get around the problem.

