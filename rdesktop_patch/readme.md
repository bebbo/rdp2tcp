https://sourceforge.net/p/rdesktop/patches/107/

This patch makes the virtual channel capability in
rdesktop visible to third parties, so that applications
which extend RDP using the virtual channels can "plug
in" to rdesktop by supplying a shared object.

A new variant of the "-r" option has been implemented,
namely:
-r soname.so:opt1:opt2:opt3...optn

where soname.so is the name (fully qualified if need
be) of the third party shared object. opt1 to optn are
optional arguments which will be passed to the shared
object if needed.

The API is implemented as follows:

* All third party shared object must implement two
named functions, rdesktop_init and rdesktop_cleanup.

* A new function called "init_external_vchannel" has
been added to channels.c. Whenever a shared object is
specified in the parameters, a handle is opened using
dlopen and this handle is added to an array for use
later. The rdesktop_init function is located and
called, passing in the addresses of the virtual channel
functions in rdesktop (channel_register, channel_init,
channel_send) and any arguments which were passed in on
the command line. The third party app can then use
these functions to send and receive data on virtual
channels, just like sound, cut and paste etc. The
rdesktop_cleanup address is then located and stored.

* At the end of the RDP session, rdesktop calls the
rdesktop_cleanup function for each third party shared
object.

