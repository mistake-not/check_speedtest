# check_speedtest
A Nagios check script for Ookla Speedtest (www.speedtest.net)

## Origins
This work is entirely based on https://github.com/jonwitts/nagios-speedtest by https://github.com/jonwitts

Jon's work is excellent and functional but writen for BASH. I wanted to run the test on my pfSense firewall to position it as close to the WAN as possible. pfSense however is based on FreeBSD which does not have BASH, only ASH.

I loath shell scripting and I've taken a spoon to the original source rather than a scalpel; it is in a "works for me" state and has not been rigorously debugged. That said, I'm publishing it openly so if you do find a bug I'm open to fixing it and if my code can be improved I'm open to improving myself. Moving away from BASH hopefully makes it more transportable in general, any further suggestions to improve on portability would be graciously received. You're free to fork it of course.

## New Features
Behaves almost exactly as the Jon Witts original. Minor and internal changes are:
* removed bashisms to make it work with /bin/sh on FreeBSD (Almquist shell) - probably increases portability generally but not tested for that
* some changes to debug messages
* an extra option to allow speedtest-cli to automatically identify a server to use rather than specifying a particular server by url or id
* internally uses the csv output of speedtest-cli rather than the simple output
* the bc maths has been tweaked to add a leading 0 to results where < 1 ie .8 becomes 0.8 (surprising that's not an option in bc but I'm sure there's a reason)

## Usage
### Example
In this example the download warning and critical thresholds are 12 and 9 Mbps and the upload 0.8 and 0.6 Mbps. The "-l p" option allows speedtest-cli to choose the server automatically by ping time. Perf data is requested so the ideal download and upload speeds are also defined as 18 and 1 Mbps.
```
check_speedtest-cli -w 12 -c 9 -W 0.8 -C 0.6 -l p -p -m 18 -M 1"
```
### Options
```
    -h    Show usage
    -w    Download Warning Level - *Required* - integer or decimal Mbit/s
    -c    Download Critical Level - *Required* - integer or decimal Mbit/s
    -W    Upload Warning Level - *Required* - integer or decimal Mbit/s
    -C    Upload Critical Level - *Required* - integer or decimal Mbit/s
    -l    Location of speedtest server - *Required* - takes one of:
              "i" internal Mini Server, set url with -s
              "e" external server, set id with -s, list ids with
                  speedtest-cli --list
              "p" automatically select a server
    -s    Server integer, URL - *Required with -l i and -l e*
    -p    Output Performance Data
    -m    Download Maximum Level - *Required with -p*
          - integer or decimal Mbit/s
          Provide the maximum possible download level for your connection
    -M    Upload Maximum Level - *Required if you request perfdata*
          - integer or decimal Mbit/s
          Provide the maximum possible upload level for your connection
    -v    Output plugin version
    -V    Output debug info for testing
```
## Nagios Configuration
Using this check throughout the day while the WAN is being shared is silly. I've found the check takes about 20s for me so I haven't found it necessary to change the default timeout in Nagios. I run it in the morning between 0600 and 0630 as a check of the WAN speed before everyone starts hammering it. I use a time period to make Nagios schedule a single test in the morning. I use the check_by_ssh wrapper on the Nagios server and put check_speedtest-cli on the router itself.
```
define timeperiod{
    timeperiod_name morning_speedtest
    alias           Half hour in the morning to running an unimpeded speed test
    sunday          06:00-06:30
    monday          06:00-06:30
    tuesday         06:00-06:30
    wednesday       06:00-06:30
    thursday        06:00-06:30
    friday          06:00-06:30
    saturday        06:00-06:30
}

define service{
    use                   generic-service
    host_name             firewall
    service_description   Internet Speed Test Check
    check_period          morning_speedtest
    check_interval        1440
    notification_interval 1440
    max_check_attempts    1
    check_command         check_by_ssh!-H $HOSTADDRESS$ -t 60 -C "/usr/local/libexec/nagios/check_speedtest-cli -w 12 -c 9 -W 0.8 -C 0.6 -l p -p -m 18 -M 1"
    notifications_enabled 1
    servicegroups         firewall
}
```
