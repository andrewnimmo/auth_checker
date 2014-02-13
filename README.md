auth_checker
============

Detection of compromised or abused user accounts for multiple services

This script was inspired by the original Exim authentication detection script 
written by Todd Lyons in 2013. There are some details about it at 
[DetectSMTPAuthAbuse](https://github.com/Exim/exim/wiki/DetectSMTPAuthAbuse)

The purpose of the script is the detection of user accounts accessed in unusual 
patterns - either from countries not on the whitelist, or from not explicitly 
whitelisted IP addresses, or from a higher number of unknown IP addresses than 
specified. In other words, it can help detecting compromised accounts already 
being abused.

Currently, Exim SMTP, Dovecot POP3, IMAP, SSH, Apache, and OpenWebMail 
authentications are supported, but more can be added relatively simply.

This utility was **NOT** designed for detecting intrusion attempts. There are 
already plenty of intrusion detection programs. This script was designed to detect 
already compromised accounts, that (in contrary) intrusion detection usually does 
not detect.


Installation:
-------------
This is a Perl script, and except the basic Perl functionality you need also the 
Geo-IP-PurePerl module including current GeoIP databases. For installation hints,
please visit [Geo::IP](http://search.cpan.org/~borisz/Geo-IP-1.43/lib/Geo/IP.pm) or [Geo::IP::PurePerl](http://search.cpan.org/~borisz/Geo-IP-PurePerl-1.25/lib/Geo/IP/PurePerl.pm)


Configuration:
--------------

You can configure auth_checker by editing some of the variables directly in the 
auth\_checker.pl script. Namely you will want to adjust the variables ignored\_ips,
ignored\_users, ignored\_countries, and possibly also the list of ignore_ISPs. 
These lists are regular expressions. Look up the syntax for regular expressions 
in Perl, when in doubts. It is possible that in a later version, there will be
a separate configuration file with a simper syntax.

Other parameters can be changed directly on the command line when calling the script.
Type "./auth_checker.pl -h" on the command line to see the list of available
options.


Examples of use:
----------------

List all Exim SMTP accounts successfully accessed within one day by at least one 
IP addresses not whitelisted in the script:

./auth_checker.pl

List all user accounts successfully accessed by 10 or more different IP addresses 
in a single day. Also parsing the default Exim logs and looking for successful SMTP 
authentications:

./auth_checker.pl --limit 10

The same thing, but using the abbreviated option name 'l' (lowercase L):

./auth_checker.pl -l 10

Specify a different log file, and suppress grouping by date (makes sense only if the
file contains logs for more than a single day). We also use a negative --limit value 
for counting all IP addresses, including the whitelisted ones, and showing accounts
with more than 3 accessing IP addresses:

./auth_checker.pl --limit -3 --nodate /var/log/exim-main.log.02.bz2

Display all IP addresses (including the whitelisted ones) accessing POP3 or IMAP account 
of the user 'joedoe', that are recorded in the default Dovecot log file /var/log/maillog.
We also turn off the verbose GeoIP information by setting the brief --geoip mode 1
instead of the default 2 (0 suppresses the GeoIP information altogether):

./auth_checker.pl --mode dovecot --user joedoe --geoip 1

Short form: ./auth_checker.pl -m dovecot -u joedoe -g 1

Listing authorized accesses to password protected web pages, grouping all IP's together,
suppressing the grouping by user account (--nouser), suppressing the grouping by date
(--nodate), turning off the Geo IP information (--geoip 0), found in the log /logs/my.log.gz

./auth_checker.pl --mode apache --nouser --nodate --geoip 0 /logs/my.log.gz
./auth_checker.pl -m apache --nouser --nodate -g 0 /logs/my.log.gz

The following cronjob example every hour parses the default log files for all supported 
services (Exim SMTP, Dovecot POP3/IMAP, SSH, OpenWebMail, and Apache HTTP), and if it 
finds some accounts accessed by 20 or more IP addresses (whitelisted IP's not counting), 
it sends the resulting list by email to root. We also use the option --quiet (-q) to 
suppress the header in case there are no violations found. This is important to avoid 
sending empty emails every hour, and rather send the alert only when there is a reason.
This is achieved with the option -E at mail (suppression of sending empty messages).

55 * * * * /my/path/auth_checker.pl -l 20 -m all -q | /usr/bin/mail -Es "Auth Report" root


Typical otput of the script looks as follows. The first column shows the number of 
accesses of given IP address (second column), and the thrid column contains the 
country name, city (if known), and the name of the Internet Access Provider who
owns the IP address. The localizaton information can be optionally shortended with
the option --geoip 1, or suppressed altogether with --geoip 0

<pre>
Feb 13 -> account joedoe: 1 IPs (Last: 88.66.22.22 @ 00:13:34)
   1     88.66.22.22       France, Lyon - Free SAS
 
Feb 13 -> account smith: 1 IPs (Last: 65.55.41.140 @ 00:10:05)
   1     65.55.41.140      United States, Redmond - Microsoft Corporation

Feb 13 -> account hill: 2 IPs (Last: 37.188.232.65 @ 00:15:16)
   1     37.188.232.65     Czech Republic - Telefonica Czech Republic, a.s.
   1     37.188.236.46     Czech Republic - Telefonica Czech Republic, a.s.
</pre>


Contact:
--------

You can send your questions or kudos to [Ivo Truxa](mailto:truxa@truxoft.com)

You can also submit issues, or modifications through the [GitHub repository](https://github.com/truxoft/auth_checker/)


Thanks:
-------

**Todd Lyons** - for the original script detect_hacked_smtp_auth_conns.pl

**Ivo Truxa**  
   - the redesign, adding GeoIP support, gzip/bzip2 support
   - for uncompressing log file archives, adding the support
   - for Dovecot, SSH, OpenWebMail, Apache, and other modifications, 
   - and also for creating the public repository at [GitHub](https://github.com/truxoft/auth_checker)



License:
--------

auth_checker is free software; you can redistribute it and/or modify it under the
terms of the GNU General Public License as published by the Free Software
Foundation; either version 2 of the License, or (at your option) any later
version.

auth_checker is distributed in the hope that it will be useful, but WITHOUT ANY
WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A
PARTICULAR PURPOSE. See the GNU General Public License for more details.

You should have received a copy of the GNU General Public License along with
auth_checker; if not, write to the Free Software Foundation, Inc., 51 Franklin
Street, Fifth Floor, Boston, MA 02110, USA

