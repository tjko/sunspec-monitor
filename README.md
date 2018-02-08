# sunspec-monitor
Monitoring Sunspec compatible Solar Inverters over Modbus TCP.
This is collection of Perl scripts to monitor Inverter status and to log
production data.

Included tools:

* sunspec-status


Tested to work with following Inverters:

Manufacturer|Model|Firmware Version|Notes
------------|-----|----------------|-----
SolarEdge|SE11400|3.1968|often first connection fails, but subsequent connections work
SolarEdge|SE11400|3.2180|works ok
SolarEdge|SE3500|3.2173|often first connection fails, but subseuqent connetions work


This script should work with any inverter that supports Modbus TCP and Sunspec standard...


## Requirements

- [Device::Modbus::TCP](https://github.com/jfraire/Device-Modbus-TCP) module


## sunspec-status

This is simple script to query Inverter (and production/consumption meter) status.
Script can be used interactively to display inverter status and details. Or alternatively
it can be used to log production data into a CSV formatted file. 

### Syntax

```
syntax: sunspec-status [options] <host>

Options:
 --port=<port>, -p <port>      Use port (default 502)
 --meter=<meter>, -m <meter>   Query meter (default 1) 
                               (meter = 1..3  or 0 = no meter)
 --numeric, -n                 Numeric output mode (time, status)
 --timeout=<sec>, -t <sec>     Timeout (default 10)
 --verbose, -v                 Verbose mode
```


### Examples

To query Inverter information and current status *-v* (or *--verbose*) option can be used:

```

# sunspec-status -v myinverter

INVERTER:
             Model: SolarEdge  SE11400
  Firmware version: 3.1968
     Serial Number: 7Dxxxxxx

            Status: ON (MPPT)

 Power Output (AC):         8014 W
  Power Input (DC):         8136 W
        Efficiency:        98.50 %
  Total Production:      148.122 kWh
      Voltage (AC):       239.50 V (59.95 Hz)
      Current (AC):        33.64 A
      Voltage (DC):       360.60 V
      Current (DC):        22.56 A
       Temperature:        53.97 C (heatsink)

METER (#1):
             Model: WattNode RWND-3D-240-MB
            Option: Production
  Firmware version: 25
     Serial Number: 40xxxxx

   Exported Energy:      148.457 kWh
   Imported Energy:        0.012 kWh
        Real Power:         8026 W
    Apparent Power:         8031 VA
      Power Factor:         1.00
      Voltage (AC):       239.90 V (60.00 Hz)
      Current (AC):        33.39 A

```

To get inverter status information in CSV format simply invoke script withouth *--verbose* (or *--debug*) options.

CSV output is in following format:

Columns|1|2|3|4|5|6|7|8|9|10|11|12
-------|-|-|-|-|-|-|-|-|-|--|--|---
Values|timestamp|status|ac_power|dc_power|total_production|ac_voltage|ac_current|dc_voltage|dc_current|temperature|exported_energy|imporoted_energy

Example output:

```
# sunspec-status myinverter
2018-01-30 13:42:01,ON,8007,8128,148275,237.90,33.71,359.90,22.58,54.21,148609,12
```

```
# sunspec-status -n myinverter
1517348521,4,8007,8128,148275,237.90,33.71,359.90,22.58,54.21,148609,12
```


To log data every five minutes following cronjob could be used:
```
*/5 * * * * /usr/local/bin/sunspec-status myinverter >> /var/log/myinverter.csv &
```



