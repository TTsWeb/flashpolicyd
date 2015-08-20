#summary Introduction
#labels Featured

# Introduction #
A recent upgrade to Adobe Flash introduced an "improvement" to their security model that requires you to run a special service on port 843 serving up XML, more information about this insanity can be found [here](http://www.adobe.com/devnet/flashplayer/articles/fplayer9_security_04.html).

Unfortunately the servers supplied with Adobe is badly written with zero timeout or error handling.  In the event that clients do not send the correct format request the code supplied by Adobe simply hangs around indefinitely - presumably till the client closes their browser, leading to periods where there are large amounts of perl processes in my process table.

I set out writing a daemon of my own using Ruby and present the results here.

I have tested this daemon using 5 parallel running loops doing requests in parallel and have managed to serve about 100 000 requests in 5 minutes - could no doubt go faster my testing machine was a bit slow.

The code has been in production with uptimes of over a year and has proven to be stable, fast and without memory leaks or any undesired side effects.  I serve up millions of requests a year using this with no ill effects

## Status ##
As mentioned this code has been run in production and proven to be both stable and feature complete for my needs, I do not anticipate future releases unless someone file specific feature requests or Adobe changes their standards.

## Download ##
The latest version can be found [here](http://code.google.com/p/flashpolicyd/downloads/list), subscribe to my blog at http://www.devco.net/ to get updates of new releases.  Ruby RDoc documentation for the code can be read [here](http://www.devco.net/code/flashpolicyd-doc-2.0/).

## Installation ##
I wrote this on RedHat Enterprise 5.1, you need to install Ruby that is included in the base repositories now and it has no external requirements not met by Ruby.

On the download site is a RPM that will install the daemon, but I include below full manual install procedures that will help you get it going on other distros.

These instructions apply to RedHat and includes a service that will activate the daemon at startup.

Once Ruby installed grab the tarball and extract it:

```
# tar -xvzf flashpolicyd-0.1.tgz
# cd flashpolicyd-0.1
# mv flashpolicyd /usr/sbin
# mv flashpolicyd.init /etc/init.d/
```

You need to create a XML file to serve up, by default this should be placed in _/etc/flashpolicy.xml_.  Next you need to enable the service, this runs as _root_ since it has to listen on port 843.

```
# chkconfig --add flashpolicyd
# chkconfig flashpolicyd on
```

This assumes a lot of defaults, you can override these in _/etc/sysconfig/flashpolicyd_ a sample file can be seen below:

```
TIMEOUT=10
XML=/etc/flashpolicy.xml
LOGFREQ=1800
LOGFILE=/var/log/flashpolicyd.log
USER=nobody
```

|**RC Script Configuration**| |
|:--------------------------|:|
|TIMEOUT                    |If a request does not complete in this many seconds the socket will disconnect|
|XML                        |The file to serve up to clients|
|LOGFREQ                    |This is a frequency in seconds that the server will log general stats to log file|
|LOGFILE                    |The logfile to write, the file will auto rotate based on size, you should not be rotating it with your systems logrotation tool|
|USER                       |The user to run as after opening the port|

If you are a [Puppet](http://reductivelabs.com/projects/puppet/) user I've also included a module that will install this for you, locations etc are correct as for Red Hat Enterprise, see the puppet subdirectory.

## Usage ##
The server runs on port 843 as the root user, you can run it with _--verbose_ manually and you'll get a lot of debug in your log file.

```
I, [2008-09-13T08:05:48.443178 #2941]  INFO -- : -604375936: Had 1246803 clients and 37262 bogus clients. Uptime 58 days 14 hours 30 min. 0 connection(s) in use now.
```

A _bogus_ client is any client that did not end in a successful request, this may be due to timeouts or simply not receiving a valid request from the client.

The script includes a complete _--help_ output but you need to install _Ruby::RDoc_ - you can find this in the _ruby-rdoc_ package from RedHat - to use the _--help_ directly, you could just look at the top of the script the help is all there.

The daemon responds to several signals that can be sent using the kill command:

|**Signal**|**Description**|
|:---------|:--------------|
|USR1      |Prints a single line stat message, during normal running this stat will be printed every 30 minutes by default, settable using --logfreq|
|USR2      |Dumps the current threads and their statusses|
|HUP       |Toggles debug mode which will print more lines in the log file|
|TERM      |Exit the process closing all the sockets|

## Monitoring ##
The tarball includes a check script for this service.  The script will work with Nagios and other compatible monitoring systems, to get it going is pretty simple:

```
% ./check_flashpolicyd.rb --host your.server.com
OK: Got XML response in 0.043149 seconds

% ./check_flashpolicyd.rb --host your.server.com
CRITICAL: 5 seconds TIMEOUT exceeded
```

You can extend the timeout using the _--timeout_ option to the check script.  Making this work with your nagios installation is out of the scope of this doc.

## Known Issues ##
There is currently only 1 known issue, on older versions of Ruby such as 1.8.1 which you will find in Red Hat Enterprise 4 the logfile does not open in line buffered mode when it rotates, no patch from Red Hat is forthcoming.

## Changes ##
|2010/02/09|Add an option to run as a specific user - [issue #2](https://code.google.com/p/flashpolicyd/issues/detail?id=#2)|
|:---------|:---------------------------------------------------------------------------------------------------------------|
|2008/06/27|Version 2 released, a complete rewrite due to problems with GServer                                             |
|2008/06/22|Initial Release                                                                                                 |