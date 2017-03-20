GENERATE DNSCRYPT COMMAND LINE
==============================

This script can be used to select and configure a public dnscrypt resolver to 
work with dnscrypt as described in 
http://nurdletech.com/linux-notes/dns/dnscrypt.html.

Before you can use it you must install the requests and docopt packages::

   pip install --user requests docopt


Finding a Suitable Resolver and Generating the Command
------------------------------------------------------

To find a suitable resolver, change to the directory that contains this script 
and run::

   ./generate-dnscrypt-cmdline

This accesses the list of resolvers from 
https://github.com/jedisct1/dnscrypt-proxy and then prints out a summary of the 
available resolvers. You should choose one. Say, for example, the OpenNIC server 
in Dallas, which has the name fvz-rec-us-dal-01:

   ./generate-dnscrypt-cmdline fvz-rec-us-dal-01

This produces a command line that can configures dnscrypt-proxy so that it talks 
to the chosen resolver.

This script contains some assumptions that you can change by editing the script.  
One is that the location of the dnscrypt-proxy executable. Another is that 
dnscrypt-proxy should listen on port 2053. A third is that you would like a PID 
file (a file that contains the process ID of the dnscrypt-proxy process) and 
that it should be placed in /tmp.

Configuring SystemD to run DNS Crypt
------------------------------------

Often one uses systemd to automatically start dnscrypt-proxy. To do so, add the 
generated command line to the ExecStart field in 
/etc/systemd/system/dnscrypt.service). The file should end up looking something 
like this::

   [Unit]
   Description = DNScrypt
   After = network.target

   [Service]
   ExecStart = /usr/local/sbin/dnscrypt-proxy 
       --provider-key=B00D:7AC0:1927:F4F7:519D:A0F1:CC8B:52B7:B331:815C:8D8E:6E30:49C4:FEDA:558A:A453 
       --provider-name=2.dnscrypt-cert.fvz-rec-us-sea-01.dnsrec.meo.ws 
       --resolver-address=23.226.230.72
       --local-address=127.0.0.1:2053
       --pidfile=/tmp/dnscrypt.pid
       --daemonize
   Restart = always
   Type = forking
   User = nobody
   PIDfile = /tmp/dnscrypt.pid

   [Install]
   WantedBy = default.target

.. important:

   For clarity the ExecStart command was given above using multiple lines, but 
   in the dnscrypt.service file the entire ExecStart entry should be on the same 
   line.

.. important:

   The PID file specified on the dnscrypt-proxy command line should match the 
   file specified for *PIDfile*.

After updating the dnscrypt.service file, you should run::

   systemctl daemon-reload

You can start (or restart) your dnscrypt-proxy service using::

   systemctl restart dnscrypt
   systemctl status dnscrypt

Be sure the status messages include the message: 'This certificate looks valid'.  
If not, there may be a problem with the resolver you chose. You might try 
choosing another.
