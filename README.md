# check_speedtest
A Nagios check script for Ookla Speedtest (www.speedtest.net)

## Origins
This work is entirely based on https://github.com/jonwitts/nagios-speedtest by https://github.com/jonwitts

Jon's work is excellent and functional but writen for BASH. I wanted to run the test on my pfSense firewall to position it as close to the WAN as possible. pfSense however is based on FreeBSD and does not have BASH, only ASH.

I loath shell scripting and I've taken a spoon to the source rather than a scalpel; it is in a "works for me" state and has not been rigorously debugged. That said, I'm publishing it openly so if you do find a bug I'm open to fixing it and if my code can be improved I'm open to improving myself - about time I gave something back to the FOSS community I depend on.

## New Features
Behaves almost exactly as the Jon Witts original with some changes to the debug messages and an extra option to allow speedtest-cli to automatically identify a server to use rather than specifying a particular server by url or id.

