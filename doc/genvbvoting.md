Generalized version bits voting (bip-genvbvoting)
====================================================

1. What is this?
------------------------

bip-genvbvoting is a reworked version of BIP9 versionbits which allows each
versionbit to be configured with its own threshold etc.

The draft specification (under development as is the implementation)
can be found at:

https://github.com/sanch0panza/bips/blob/bip-genvbvoting/bip-genvbvoting.mediawiki


2. Design
------------------------

2.1. Config file
~~~~~~~~~~~~~~~~~~~~~~~~

Fork (deployment) information will be extracted to a configuration file
for ease of maintenance and regression testing.

This file, if present in a standard location (e.g. datadir), will override
the built-in client defaults.

The implementation will use a comma-separated value (CSV, RFC4180 [1])
configuration file which contains the versionbits configuration for each
chain known to the client (matched using the chains' strNetworkID defined
in chainparams.cpp).

File format:
Lines beginning with hashes or semicolons will be treated as comment lines and
ignored.
Data lines will consist of the following comma-separated fields:

    network,bit,name,starttime,timeout,windowsize,threshold,minlockedblocks,minlockedtime,gbtforce

The expected data types of these fields are:

    network         - ASCII string
    bit             - integer in range 0..28
    name            - ASCII string
    starttime       - integer representing UNIX (POSIX) timestamp
    timeout         - integer representing UNIX (POSIX) timestamp
    windowsize      - positive integer > 1
    threshold       - positive integer >= 1 and <= windowsize
    minlockedblocks - integer >= 0
    minlockedtime   - integer number of seconds >= 0
    gbtforce        - boolean (true/false)

Example of file content (with header comment):

    # forks.csv
    # This file defines the known consensus changes tracked by the software
    # MODIFY AT OWN RISK - EXERCISE EXTREME CARE
    # Line format:
    # network,bit,name,starttime,timeout,windowsize,threshold,minlockedblocks,minlockedtime,gbtforce
    # main network, 95% @ 2016 blocks:
    main,0,csv,1462060800,1493596800,2016,1916,2016,0,true
    main,1,segwit,1479168000,1510704000,2016,1916,2016,0,true
    main,28,testdummy,1199145601,1230767999,2016,1916,2016,0,true
    ; test network (testnet), 75% @ 2016 blocks:
    test,0,csv,1456790400,1493596800,2016,1512,2016,0,true
    test,1,segwit,1462060800,1493596800,2016,1512,2016,0,true
    test,28,testdummy,1199145601,1230767999,2016,1512,2016,0,true
    ; regtest chain 75% @ 144 blocks:
    regtest,0,csv,0,999999999999,144,108,108,0,true
    regtest,1,segwit,0,999999999999,144,108,108,0,true
    regtest,28,testdummy,0,999999999999,144,108,108,0,true

Bits that are not defined shall not be listed.
The 'testdummy' assignments on bit 28 are historically used by some tests,
but could fall away in the future if those tests no longer need them.


2.2. Adaptated files
~~~~~~~~~~~~~~~~~~~~~~~~

The new functionality will be added in versionbits.{h,cpp}.
Where possible existing interfaces will be preserved to keep changes to the
rest of the software minimal.

A class to read the forks.csv config could be added in its own forkcsv.{h,cpp} .
This is still to be determined.


3. Test plan
------------------------

3.1 Unit tests
~~~~~~~~~~~~~~~~~~~~~~~~

Unit tests shall be added for any adapted and new classes (e.g. for reading
and validating contents of the forks.csv file)

3.2 Regression tests
~~~~~~~~~~~~~~~~~~~~~~~~

Test that definitions on each bit are accepted for defined chains - they shall
override the default (compiled-in) settings.

Test that undefined config file settings lead to use of built-in defaults.

Test graceful handling of absent / empty / unreadable forks.csv file.

Test RPC information available for defined forks.

Test state transitions, also w.r.t. various threshold and windowsize boundary
conditions.

Test grace period parameters.

Test any notifications (if specified)


4. Recommendations received during spec review:
--------------------------------------------------------

Some observations on review comments received privately (outside of main review
list and forums) noted here with thoughts on implementation:

4.1 Be self-documenting
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This could be achieved by a command line argument which causes the client to
dump its built-in default deployment data in CSV format, ready to be used as
a template for the config file.

This should contain a standard commented file header similar to the above
example, so that a user can immediately know the meaning of the fields.

4.2 Be upgrade-ready
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The default mode would be to deliver built-in defaults corresponding to the
known deployments. An upgrade would work just like today - just install a
newer client which comes with updated deployment built-in.

For comfort, packaging scripts could generate the latest forks.csv matching
the built-in defaults, and place that in an informational folder where it
is available for reference.

Install scripts could, when upgrading, check for the presence of an active
forks.csv file (e.g. in the datadir) and warn the user in case there are
differences that might need merging.

4.3 Don't leave crappy confusing files on anyone's computer
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Uninstall scripts would remove any informational copies of the configuration
file, and leave the user's active file (if present) untouched unless the
user opts to remove them during de-installation.

No new files are generated automatically during runtime, this data only
needs to be read by the client.


References
------------------------

[1] https://tools.ietf.org/rfc/rfc4180.txt
