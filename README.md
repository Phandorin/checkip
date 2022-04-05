# checkip

Checkip is a CLI tool and library that provides generic and security information
about an IP address in quick way. It uses various free public services to do so.

```
$ checkip 91.228.166.47
abuseipdb.com  --> domain: eset.sk, usage type: Data Center/Web Hosting/Transit
dns mx         --> eset.sk: a.mx.eset.com, b.mx.eset.com
dns name       --> skh1-webredir01-v.eset.com
iptoasn.com    --> AS description: ESET-AS
maxmind.com    --> city: Bratislava, country: Slovakia (SK)
shodan.io      --> OS: n/a, 2 open ports: tcp/80 (nginx), tcp/443 (nginx)
urlscan.io     --> 0 related URLs
virustotal.com --> network: 91.228.164.0/22, SAN: www.eset.com, eset.com
Malicious      --> 0% ✅

$ checkip 218.92.0.158 2> /dev/null
abuseipdb.com  --> domain: chinatelecom.com.cn, usage type: n/a
dns mx         --> chinatelecom.com.cn: testmail.chinatelecom.com.cn
iptoasn.com    --> AS description: CHINANET-BACKBONE No.31,Jin-rong Street
maxmind.com    --> city: n/a, country: China (CN)
urlscan.io     --> 0 related URLs
virustotal.com --> network: 218.92.0.0/16, SAN: n/a
Malicious      --> 50% 🚫
```

The CLI tool also supports JSON output.

```
$ checkip -j 218.92.0.158 2> /dev/null | \
# select Sec (1) and InfoSec (2) check types and 
# show if they consider the IP address malicious
jq -r '.checks[] | select(.type == 1 or .type == 2) | "\(.malicious)\t\(.name)"'
false	abuseipdb.com
true	blocklist.de
false	cinsscore.com
true	github.com/stamparm/ipsum
true	otx.alienvault.com
false	threatcrowd.org
false	urlscan.io
true	virustotal.com
```

NOTE: Some of the [checks][1] are active, i.e. they interact with the target IP
address. You should run active checks (`-a`) only against your hosts or hosts
you have permission to scan.

```
$ checkip -a 45.33.32.156 # scanme.nmap.org
Open TCP ports --> 22 (ssh), 80 (http), 9929 (nping-echo), 31337 (Elite)
Ping           --> 0% packet loss, sent 5, recv 5, avg round-trip 192 ms
abuseipdb.com  --> domain: linode.com, usage type: Data Center/Web Hosting/Transit
dns mx         --> linode.com: inbound-mail1.linode.com, inbound-mail3.linode.com
dns name       --> scanme.nmap.org
iptoasn.com    --> AS description: LINODE-AP Linode, LLC
maxmind.com    --> city: Fremont, country: United States (US)
shodan.io      --> OS: n/a, 3 open ports: tcp/22 (OpenSSH, 6.6.1p1 Ubuntu-2ubuntu2.13), tcp/80 (Apache httpd, 2.4.7), udp/123
urlscan.io     --> 0 related URLs
virustotal.com --> network: 45.33.0.0/17, SAN: n/a
Malicious      --> 0% ✅
```

## Installation

To install the CLI tool

```
go install github.com/jreisinger/checkip@latest
```

or download a [release](https://github.com/jreisinger/checkip/releases)
binary (from under "Assets") for your system and architecture.

## Configuration

For some checks to work you need to register and get an API (LICENSE) key. See
the service web site for how to do that.

Store the keys in `$HOME/.checkip.yaml` file.

```
ABUSEIPDB_API_KEY: aaaaaaaabbbbbbbbccccccccddddddddeeeeeeeeffffffff11111111222222223333333344444444
MAXMIND_LICENSE_KEY: abcdef1234567890
SHODAN_API_KEY: aaaabbbbccccddddeeeeffff11112222
URLSCAN_API_KEY: abcd1234-a123-4567-678z-a2b3c4b5d6e7
VIRUSTOTAL_API_KEY: aaaaaaaabbbbbbbbccccccccddddddddeeeeeeeeffffffff1111111122222222
```

You can also use environment variables with the same names.

## Development

Checkip is easy to extend. If you want to add a new way to check an IP address,
just write a function of type [Check][2]. Add the function to `checks.Passive`
or `checks.Active` [variable][3].

```
make run

git tag | sort -V
git tag -a v0.16.2 -m "improve docs"
git push -u origin --tags
```

[1]: https://pkg.go.dev/github.com/jreisinger/checkip/checks
[2]: https://pkg.go.dev/github.com/jreisinger/checkip/check#Check
[3]: https://pkg.go.dev/github.com/jreisinger/checkip/checks#pkg-variables