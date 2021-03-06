# Building a Challenge Binary

This walk-through will guide you through building a challenge binary.  

All challenge binaries should derive from the service template found in on the CGC VM at /usr/share/cgc-cb/templates/service-template. Begin by making a copy of this directory.

	$ cp -r /usr/share/cgc-sample-challenges/templates/service-template/ /vagrant/my-cb
	$ cd /vagrant/my-cb

The Makefile included in the service-template is already configured to work properly within the CGC development virtual machine. The only changes that are required in the Makefile are to edit the first three lines to replace the AUTHOR_ID, SERVICE_ID, and TCP_PORT with values specific to your new binary. For example

    AUTHOR_ID  = LUNGE
    SERVICE_ID = 00001
    TCP_PORT   = 10000

might become

    AUTHOR_ID  = DDTEK
    SERVICE_ID = 00005
    TCP_PORT   = 31336

## Adding source files

The Makefile is pre-configured to compile all .c files found in both the src and lib sub directories. These directories are also added to the include file path to locate any .h header files that are part of the challenge binary. The recommended division of files is to place boiler plate code such as system call wrappers and standard library functions into the lib directory and files that implement service specific behaviors, including any vulnerabilities, into the src directory.

## PATCHED macro

The Make process will create both an unpatched, vulnerable binary and a patched, secure binary from a single set of source files.  When building the patched, secure CB, the PATCHED macro will be defined.  When building the vulnerable, for-release CB, the PATCHED macro will not be defined.  CB authors must test if the PATCHED macro is defined to determine which code should be included in a given build. Simple examples of the use of these macros are shown here

    #ifdef PATCHED
       fgets(buf, sizeof(buf), stdin);
    #endif

    #ifndef PATCHED
       gets(buf);
    #endif

    #ifdef PATCHED
       //perform some checks that ensure a vulnerability can't be triggered below
    #endif

## Service Pollers

All challenge binaries must include service poller specifications designed to exercise the normal functionality of the challenge binary. Pollers may be designated as "for-release" (poller/for-release directory) or "for-testing" (poller/for-testing directory. If a poller is designated as "for-release", then network traffic captured during the execution of that poller will be released to competitors along with the unpatched challenge binary when a competition event begins. Any poller designated as "for-testing" will be used only for testing the proper functionality of challenge binaries submitted by CB authors and replacement binaries submitted by competitors. No network traffic generated by "for-testing" binaries will be provided to competitors during competition events. It is entirely up to the challenge binary author which and how many (if any) pollers are to be designated as "for-release". Note that both the unpatched and patched challenge binary must pass all provided service polls (both for-release and for-testing). 

## Proofs of Vulnerability

All challenge binaries must include at least one proof of vulnerability (PoV) that demonstrates that a vulnerability may be triggered in a deterministic manner. A valid PoV must cause its associated CB to terminate on a SIGSEGV or SIGILL signal. If a CB contains more than one injected flaw, then PoVs that trigger each flaw must be provided. PoVs shall be placed in the CB's pov directory.

## Building the challenge binaries

The default (all) target in the Makefile will cause the "build", "test", and "package" targets to be built. The "build" target will compile and link both a patched and an unpatched version of the CB, placing the "bin" directory (which is created if missing). The "test" target will invoke the cb-replay utility once for each service poller and once for each pov. In order to successfully build the "test" target, the patched and unpatched binaries must successfully pass all service polls, the unpatched CB must generate a core file when replayed with each pov, and the patched binary must not generate a core file when replayed with each pov.  Additionally, the "test" target will cause network traffic to be captured for a replay of any poller in the for-release directory. All generated pcap files will be placed into a pcap directory (which is created if missing).

The provided Makefile is properly configured to utilize the CGC specific tool chain. CGC tool chain items intended to be specifically used with CBs can be found in standard cross-build locations for 'i386-linux-cgc'.  Specifically:

    /usr/i386-linux-cgc/bin/
    /usr/bin/i386-linux-cgc-*

When wishing to use these tools, you must specify the full path to the tool, use the 'long name' for the specific tool in /usr/bin, or prepend the /usr/i386-linux-cgc/bin directory to your path. For instance, objdump can be used in these ways.

The full path for either location of objdump:

    $ /usr/bin/i386-linux-cgc-objdump -f bin/DDTEK_00005
    $ /usr/i386-linux-cgc/bin/objdump -f bin/DDTEK_00005

Using the 'long name' which, because it is in /usr/bin, is already in PATH:

    $ i386-linux-cgc-objdump -f bin/DDTEK_00005

Prepending the location of the i386-linux-cgc tools to PATH:

    $ which objdump
    /usr/bin/objdump
    $ PATH=/usr/i386-linux-cgc/bin:$PATH
    $ which objdump
    /usr/i386-linux-cgc/bin/objdump
    $ objdump -f bin/DDTEK_00005

## SEE ALSO

For information regarding the testing a CB, see testing-a-cb.md

For information regarding the debugging a CB, see debugging-a-cb.md

For information regarding submitting a CB, see submitting-a-cb.md

For information regarding using the cb-server, see man cb-server(1)

For support please contact CyberGrandChallenge@darpa.mil
