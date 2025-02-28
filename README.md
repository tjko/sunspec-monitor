# sunspec-monitor
Monitoring Sunspec compatible Solar Inverters over Modbus TCP.
This is collection of Perl scripts to monitor Inverter status and to log
production data.

Included tools:

* sunspec-status


Known to work with following Inverters:

Manufacturer|Model   |Firmware Version|Notes
------------|--------|----------------|-----
SolarEdge   |SE11400 |3.2251          |works ok
SolarEdge   |SE11400 |3.2180          |works ok, intermitted inverter error status reported if multiple meters connected
SolarEdge   |SE11400 |3.1968          |often first connection fails, but subsequent connections work
SolarEdge   |SE7600  |3.2305          |works ok
SolarEdge   |SE7K    |3.2251          |works ok, may need "-m 0"
SolarEdge   |SE6000  |3.2251          |works ok
SolarEdge   |SE3680  |3.2228          |works ok
SolarEdge   |SE3680  |3.2016          |works ok
SolarEdge   |SE3500  |3.2186          |works ok
SolarEdge   |SE3500  |3.2173          |often first connection fails, but subseuqent connetions work
SolarEdge   |SE10000 |4.0022          |works ok

Known to work with following Batteries:

Manufacturer|Model   |Firmware Version|Notes
------------|---------------|--------------------------------|-----
SolarEdge   |SE Energy Bank |SMCU 1.1.51 DCDC 1.1.33 BMS 1.0 |works ok

Known to work with following Meters:

Manufacturer|Model           |Firmware Version|Notes
------------|----------------|----------------|-----
SolarEdge   |SE-MTR-3Y-400V-A|79              |works ok
WattNode    |RWND-3D-240-MB  |25              |works ok
WattNode    |WNC-3D-240-MB   |24              |doesnt appear to work (as "Export+Import" meter) ?
WattNode    |WNC-3Y-400-MB   |24              |works ok (for help see #4)
WattNode    |WND-3Y-400-MB   |25              |doesnt appear to work (as "Export+Import" meter) ?

This script should work with any inverter that supports Modbus TCP and Sunspec standard...

## Requirements

- [Net::Server](https://metacpan.org/pod/Net::Server) module
- [Device::Modbus](https://metacpan.org/pod/Device::Modbus) module
- [Device::Modbus::TCP](https://github.com/jfraire/Device-Modbus-TCP) module


## sunspec-status

This is simple script to query Inverter (and production/consumption meter) status.
Script can be used interactively to display inverter status and details. Or alternatively
it can be used to log production data into a CSV formatted file. CSV format is the default,
but output in JSON format is also available.

If there is second (import/export) meter also connected to the inverter it is possible
to query it as well.  Default is to query meter in address 1. To query also second meter
option *-m 1,2* can be used.
If no meter is installed *-m 0* can be used to skip querying any meter.

StorEdge battery status can be also reported if installed. To enable display of battery
status option *-b* can be used. This support was contributed by Ilker Deligoz. Multiple battery support contributed by Alastair Bor.


### Syntax

```
syntax: sunspec-status [options] <host>

Options:
 --port=<port>, -p <port>             Use port (default 502)
 --address=<addr>, -a <addr>          Modbus Address (default 1)
 --meter=<meter>, -m <meter>          Query meter (default 1) 
                                      (meter = 1..3  or 0 = no meter)
 --phase=<phase>, -P <phase>          Report PF for single phase only
                                      (phase = A,B,C)
 -battery=<batteries>, -b <batteries> Query battery status (e.g., 1 or 1,2)
 --numeric, -n                        Numeric output mode (time, status)
 --json, -j                           Output in JSON (instead of CSV) format
 --timeout=<sec>, -t <sec>            Timeout (default 10)
 --output=<filename>, -f <filename>   Append results to a file
 --verbose, -v                        Verbose mode
 --debug, -d                          Debug mode (dump raw Sunspec register values)
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

Columns|1|2|3|4|5|6|7|8|9|10|11|12|13|14
-------|-|-|-|-|-|-|-|-|-|--|--|--|--|---
Values|timestamp|status|ac_power|dc_power|total_production|ac_voltage|ac_current|dc_voltage|dc_current|temperature|exported_energy_m1|imporoted_energy_m1|exported_energy_m2|imported_energy_m2

Example output:

```
# sunspec-status myinverter
2018-01-30 13:42:01,ON,8007,8128,148275,237.90,33.71,359.90,22.58,54.21,148609,12,0,0
```

```
# sunspec-status -n myinverter
1517348521,4,8007,8128,148275,237.90,33.71,359.90,22.58,54.21,148609,12,0,0
```


To log data every five minutes following cronjob could be used:
```
*/5 * * * * /usr/local/bin/sunspec-status myinverter >> /var/log/myinverter.csv &
```


If there are two meters in the system (typically a production meter and a second
import/export meter at grid connection point) both can be queried simultaneously:

```
# sunspec-status -m 1,2 myinverter
2018-01-30 13:42:01,ON,8007,8128,148275,237.90,33.71,359.90,22.58,54.21,148609,12,35603,8471
```

If there are two batteries in the system both can be queried simultaneously:

```
# sunspec-status -b 1,2 myinverter
2025-02-28 23:07:42,ON (MPPT),-15,-15,262360,247.80,0.49,403.90,0.62,41.13,39625093,29111250,0,0,17674,20830,0.00,16.46,95.00,8548,18544,0.00,16.85,97.00
```

