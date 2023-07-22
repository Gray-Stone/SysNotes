# Printer 

## CUPS

CUPS is the most common printer system. 

For arch, CUPS needs to be installed `pacman -S cups`
https://wiki.archlinux.org/title/CUPS

Cups comes with some systemd unit file. A socket activated style, or simly enable and start the service. 

CUPS have a web management protal, at localhost:631

### CUPS admin 

User need certain previlidge to be alled to manage / control printers 

Cups have a config file at `/etc/cups/cups-files.conf` 

In the file, there is a line of `SystemGroup xxx xxx`. This determent which group of users can be admin of CUPS 
(On some system group lp or lp-admin would be in this)

For example with arch, this list only contains `sys root wheel`. Which means the actual admin user need to be added to one of these groups.


## Brother printer 

brother printer would need some additional drivers. [brlaser](https://github.com/pdewacht/brlaser) is a good one to go with, available on many pcakges. 