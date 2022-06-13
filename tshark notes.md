# Tshark

## output file format

pcap: most common 
pcapng: upgraded pcap, stores more info.

---
---

## Options

---

### For captures 

`-w <filename>` : set the output to a specific file

`-F <file_foramt>` : set output format. Use pcapng when ever possible

`-i <interface>` : the interface to capture on, such as `lo` , `eno1` 

`-b <ringbuffer opt>` : Set option for ring buffered files (rotating log), The possible options for this are:
* duration:NUM - switch to next file after NUM secs
* interval:NUM - create time intervals of NUM secs
* filesize:NUM - switch to next file after NUM KB
* files:NUM - ringbuffer: replace after NUM files

Each -b option can only specify one of the options. To set multiple options, specify -b multiple times. 
example: capture on interface lo, output pcapng and output file prefix of tmpcap.pcapng 10 files, each of 99KB:

```
tshark -i lo -F pcapng -w /tmp/tmpcap.pcapng -b filesize:99 -b files:10
```

`-f <capture filter>` :      packet filter in libpcap filter syntax

Capture filters are applied before they packets are captured. Help reduce capture size. 
Capture filters are based on [BPF syntax](https://biot.com/capstats/bpf.html). **tcpdump** also use it. (required by libcap)

`-C <config profile>`      start with specified configuration profile. However I can't understand this 

` --time-stamp-type <type>` :Change the interface’s timestamp method.

use `tshark -i eno2 --list-time-stamp-types` to show. `adapter_unsynced` is what we usually want. 


**conclusion**

For capturing 
* everything on eno2 `-i eno2` 
* use hardware timestamp ` --time-stamp-type adapter_unsynced `
* filter only ip address (from or to) `-f "host 192.168.1.21"` **Make sure type the correct ip for this interface**
* Save as pcapng file format `-F pcapng` 
* Rotating log file, 10G pre file, 5 files `-w ~/tshark-capture/capture.pcapng -b filesize:10485760 -b files:10` **Doesn't auto create folders**

```
tshark -i eno2 --time-stamp-type adapter_unsynced -f "host 192.168.1.21" -F pcapng -w ~/tshark-capture/capture.pcapng -b filesize:10485760 -b files:10
```

---------------------------------------------------------------------------------------------------

### For processing (analysis)

`-2` : two pass (not usable in many conditions)

`-r|--read-file <infile>` : Read packet data from infile

`-Y|--display-filter <displaY filter>`

Cause the specified filter (which uses the syntax of read/display filters, rather than that of capture filters) to be applied before printing a decoded form of packets or writing packets to a file. Packets matching the filter are printed or written to file; packets that the matching packets depend upon (e.g., fragments), are not printed but are written to file; packets not matching the filter nor depended upon are discarded rather than being printed or written.

Use this instead of -R for filtering using single-pass analysis. If doing two-pass analysis (see -2) then only packets matching the read filter (if there is one) will be checked against this filter.


`-R|--read-filter <Read filter>`

Cause the specified filter (which uses the syntax of read/display filters, rather than that of capture filters) to be applied during the first pass of analysis. Packets not matching the filter are not considered for future passes. Only makes sense with multiple passes, see -2. For regular filtering on single-pass dissect see -Y instead.

Note that forward-looking fields such as 'response in frame #' cannot be used with this filter, since they will not have been calculate when this filter is applied.

-----------------------------------------------------------------------------------------------------------

### For printing (display)

`-T ek|fields|json|jsonraw|pdml|ps|psml|tabs|text` Set the format of the output when viewing decoded packet data. The options are one of:

fields The values of fields specified with the -e option, in a form specified by the -E option. For example,

```
tshark -T fields -E separator=, -E quote=d
```

`-E <field print option>`

Set an option controlling the printing of fields when -T fields is selected.

Options are:

bom=y|n If y, prepend output with the UTF-8 byte order mark (hexadecimal ef, bb, bf). Defaults to n.

header=y|n If y, print a list of the field names given using -e as the first line of the output; the field name will be separated using the same character as the field values. Defaults to n.

separator=/t|/s|<character> Set the separator character to use for fields. If /t tab will be used (this is the default), if /s, a single space will be used. Otherwise any character that can be accepted by the command line as part of the option may be used.

occurrence=f|l|a Select which occurrence to use for fields that have multiple occurrences. If f the first occurrence will be used, if l the last occurrence will be used and if a all occurrences will be used (this is the default).

aggregator=,|/s|<character> Set the aggregator character to use for fields that have multiple occurrences. If , a comma will be used (this is the default), if /s, a single space will be used. Otherwise any character that can be accepted by the command line as part of the option may be used.

quote=d|s|n Set the quote character to use to surround fields. d uses double-quotes, s single-quotes, n no quotes (the default).


`-e <field>`

Add a field to the list of fields to display if -T ek|fields|json|pdml is selected. This option can be used multiple times on the command line. At least one field must be provided if the -T fields option is selected. Column names may be used prefixed with "_ws.col."

Example: tshark -e frame.number -e ip.addr -e udp -e _ws.col.Info -e frame.time_epoch

Giving a protocol rather than a single field will print multiple items of data about the protocol as a single field. Fields are separated by tab characters by default. -E controls the format of the printed fields.

``` 
tshark -i eno2 -Tfields -e ip.src -e udp.port 
```

**conclusion**

 (in stdout)
```
tshark -r file -Tfields -e frame.time_epoch
```

# editcap


https://www.wireshark.org/docs/man-pages/editcap.html

editcap is a software to edit/alter the content in a capture file. 

Could use it to chop out a section of capture (time based, packet number based), change the format, reduce duplication, etc. 

## Usages:

### Limit the time range:

Generate a new pcapng file within the given time. 

```
editcap -A ${START_TIME} -B ${END_TIME} ${IN_FILE} ${OUT_FILE}
```

The time format suppose to be ISO 8601. (however it doesn't seems to understand sub-second stuff)

**The manual said it supports the epoch time stamp. However on ubutnu20.04, epoch doesn't seed to be reconized.** A workaround could be using `date` command to help conver it. 

```
editcap -A $( date -d @${START_EPOCH} +"%Y-%m-%d%H:%M:%S.%N") -B $( date -d @${END_EPOCH} +"%Y-%m-%d%H:%M:%S.%N")  ${IN_FILE} ${OUT_FILE}
```


--------------
--------------

--------------
--------------


# tcpdump
tcpdump rotating files 
https://stackoverflow.com/questions/21567963/how-to-save-a-new-file-when-tcpdum-file-size-reaches-10mb 


# Get address

make all interfaces into a list in bash

```
interfaces=( $(ls /sys/class/net/) )
```

Use IP can cut to list all interface and their IP 

```
ip -4 -o addr | cut -d " " -f2,7
```
output 
```
lo 127.0.0.1/8
eno2 192.168.1.21/24
wlo1 192.168.1.20/24
```

```
python3 << EOF
import io
text_block= """$(ip -4 -o addr | cut -d " " -f2,7)"""

for line in text_block.split("\n"):
	print (f"line is : [{line}]")
	[interface, ip] =line.split(" ")[0:2]
	if "192.168." in ip:
		print(f"Interface {interface} is usable!, with ip {ip}")
	else:
		print(f"Interface {interface} is not usable!, with ip {ip}")
EOF
```