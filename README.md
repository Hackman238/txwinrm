txwinrm
=======

Asynchronous Python WinRM client


Current Feature Support
-----------------------

* HTTP
* Basic authentication
* WQL queries
* WinRS

Future Feature Support
----------------------

* Subscribe to the Windows Event Log
* Kerberos authentication (domain accounts)
* NTLM authentication (local accounts)
* HTTPS


Installation
------------

Install this application into your Python site libraries with:

    $ python setup.py install


Dependencies
------------

* Python 2.7
* Twisted 11.0 or later (utilizes HTTP connection pools with Twisted 12.1 or later)


Configuring the Target Windows Machines
---------------------------------------

You can enable the WinRM service on Windows Server 2003, 2008 and 2012. Run
Command Prompt as Administrator and execute the following commands

    winrm quickconfig
    winrm s winrm/config/service @{AllowUnencrypted="true";MaxConcurrentOperationsPerUser="4294967295"}
    winrm s winrm/config/service/auth @{Basic="true"}
    winrm s winrm/config/winrs @{MaxShellsPerUser="2147483647"}


WQL Queries
-----------

You can pass a single host and query via the command line...

    $ winrm -r host -u user -p passwd -f "select * from Win32_NetworkAdapter"


..., or create an ini-style config file and hit multiple targets with multiple
queries. Example config is at examples/config.ini

    $ winrm -c path/to/config.ini


This will send WinRM enumerate requests to the hosts listed in config.ini. It
will send a request for each WQL query listed in that file. The output will
look like

    <hostname> ==> <WQL query>
        <property-name> = <value>
        ...
        ---- (indicates start of next item)
        <property-name> = <value>
        ...
    ...


Here is an example...

    cupertino ==> Select name,caption,pathName,serviceType,startMode,startName,state From Win32_Service
      Caption = Application Experience
      Name = AeLookupSvc
      PathName = C:\Windows\system32\svchost.exe -k netsvcs
      ServiceType = Share Process
      StartMode = Manual
      StartName = localSystem
      State = Stopped
      ----
      Caption = Application Layer Gateway Service
      Name = ALG
    ...


A summary of the number of failures if any and number of XML elements processed
appears at the end. The summary and any errors are written to stderr, so
redirect stdin to /dev/null if you want terse output.

    $ winrm -c path/to/config.ini >/dev/null

    Summary:
      Connected to 3 of 3 hosts
      Processed 13975 elements
      Failed to process 0 responses
      Peak virtual memory useage: 529060 kB

      Remote CPU utilization:
        campbell
          0.00% of CPU time used by WmiPrvSE process with pid 1544
          4.00% of CPU time used by WmiPrvSE#1 process with pid 1684
          4.00% of CPU time used by WmiPrvSE#2 process with pid 3048
        cupertino
          0.00% of CPU time used by WmiPrvSE process with pid 1608
          3.12% of CPU time used by WmiPrvSE#1 process with pid 1764
          9.38% of CPU time used by WmiPrvSE#2 process with pid 2608
        gilroy
          1.08% of CPU time used by WmiPrvSE process with pid 1428
          5.38% of CPU time used by WmiPrvSE#1 process with pid 1760
          4.30% of CPU time used by WmiPrvSE#2 process with pid 1268


The '-d' option increases logging, printing out the XML for all requests and
responses, along with the HTTP status code.


WinRS
-----

The winrs program has four modes of operation:

* --single-shot or -s: Execute a single command and return its output
* --long-running or -l: Execute a single long-running command like 'typeperf -si 1' and check the output periodically
* --interactive or -i: Execute many commands in an interactive command prompt on the remote host
* --batch or -b: Opens a command prompt on the remote system and executes a list of commands (actually right now it executes one command twice as a proof-of-concept)


An example of single-shot

    $ winrs -u Administrator -p Z3n0ss -x 'typeperf "\Memory\Pages/sec" "\PhysicalDisk(_Total)\Avg. Disk Queue Length" "\Processor(_Total)\% Processor Time" -sc 1' -r oakland -s
    {'exit_code': 0,
     'stderr': [],
     'stdout': ['"(PDH-CSV 4.0)","\\\\AMAZONA-SDFU7B1\\Memory\\Pages/sec","\\\\AMAZONA-SDFU7B1\\PhysicalDisk(_Total)\\Avg. Disk Queue Length","\\\\AMAZONA-SDFU7B1\\Processor(_Total)\\% Processor Time"',
                '"04/19/2013 21:43:48.823","0.000000","0.000000","0.005660"',
                'Exiting, please wait...',
                'The command completed successfully.']}


An example of long-running

    $ winrs -u Administrator -p Z3n0ss -x 'typeperf "\Memory\Pages/sec" "\PhysicalDisk(_Total)\Avg. Disk Queue Length" "\Processor(_Total)\% Processor Time" -si 1' -r oakland -l
      "(PDH-CSV 4.0)","\\AMAZONA-SDFU7B1\Memory\Pages/sec","\\AMAZONA-SDFU7B1\PhysicalDisk(_Total)\Avg. Disk Queue Length","\\AMAZONA-SDFU7B1\Processor(_Total)\% Processor Time"
      "04/19/2013 21:43:10.603","0.000000","0.000000","18.462005"
      "04/19/2013 21:43:11.617","0.000000","0.000000","0.000464"
      "04/19/2013 21:43:12.631","0.000000","0.000000","1.538423"
      "04/19/2013 21:43:13.645","0.000000","0.000000","0.000197"


An example of interactive

    $ winrs -u Administrator -p Z3n0ss -x 'typeperf "\Memory\Pages/sec" "\PhysicalDisk(_Total)\Avg. Disk Queue Length" "\Processor(_Total)\% Processor Time" -si 1' -r oakland -i
    Microsoft Windows [Version 6.2.9200]
    (c) 2012 Microsoft Corporation. All rights reserved.
    C:\Users\Default>dir
    Volume in drive C has no label.
    Volume Serial Number is 5E71-6BA3
    Directory of C:\Users\Default
    02/22/2013  03:42 AM    <DIR>          Contacts
    02/22/2013  03:42 AM    <DIR>          Desktop
    02/22/2013  03:42 AM    <DIR>          Documents
    02/22/2013  03:42 AM    <DIR>          Downloads
    02/22/2013  03:42 AM    <DIR>          Favorites
    02/22/2013  03:42 AM    <DIR>          Links
    02/22/2013  03:42 AM    <DIR>          Music
    02/22/2013  03:42 AM    <DIR>          Pictures
    02/22/2013  03:42 AM    <DIR>          Saved Games
    02/22/2013  03:42 AM    <DIR>          Searches
    02/22/2013  03:42 AM    <DIR>          Videos
    0 File(s)              0 bytes
    11 Dir(s)   7,905,038,336 bytes free

    C:\Users\Default>exit


An example of batch

    $ winrs -u Administrator -p Z3n0ss -x 'typeperf "\Memory\Pages/sec" "\PhysicalDisk(_Total)\Avg. Disk Queue Length" "\Processor(_Total)\% Processor Time" -sc 1' -r oakland -b
    Creating shell on oakland.

    Sending to oakland:
      typeperf "\Memory\Pages/sec" "\PhysicalDisk(_Total)\Avg. Disk Queue Length" "\Processor(_Total)\% Processor Time" -sc 1

    Received from oakland:
      "(PDH-CSV 4.0)","\\AMAZONA-SDFU7B1\Memory\Pages/sec","\\AMAZONA-SDFU7B1\PhysicalDisk(_Total)\Avg. Disk Queue Length","\\AMAZONA-SDFU7B1\Processor(_Total)\% Processor Time"
      "04/19/2013 21:43:39.198","0.000000","0.000000","0.000483"
      Exiting, please wait...
      The command completed successfully.

    Sending to oakland:
      typeperf "\Memory\Pages/sec" "\PhysicalDisk(_Total)\Avg. Disk Queue Length" "\Processor(_Total)\% Processor Time" -sc 1

    Received from oakland:
      "(PDH-CSV 4.0)","\\AMAZONA-SDFU7B1\Memory\Pages/sec","\\AMAZONA-SDFU7B1\PhysicalDisk(_Total)\Avg. Disk Queue Length","\\AMAZONA-SDFU7B1\Processor(_Total)\% Processor Time"
      "04/19/2013 21:43:41.054","0.000000","0.000000","0.000700"
      Exiting, please wait...
      The command completed successfully.

    Deleted shell on oakland.

    Exit code of shell on oakland: 0


Feedback
--------

To provide feedback on txwinrm start a discussion on the zenoss-windows forum on community.zenoss.org <http://community.zenoss.org/community/forums/zenoss-windows>.

Zenoss uses JIRA to track bugs. Create an account and file a bug, or browse reported bugs. <http://jira.zenoss.com/jira/secure/Dashboard.jspa>

Unit Test Coverage
------------------

As of Apr 16, 2013...

    $ txwinrm/test/cover
    ........................
    ----------------------------------------------------------------------
    Ran 24 tests in 7.910s

    OK
    Name                Stmts   Miss  Cover
    ---------------------------------------
    txwinrm/__init__        0      0   100%
    txwinrm/constants      18      0   100%
    txwinrm/enumerate     259     46    82%
    txwinrm/shell         114     34    70%
    txwinrm/util           89     24    73%
    ---------------------------------------
    TOTAL                 480    104    78%
