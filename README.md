# Dataset used for training IoT C&C classifier

This dataset was used for training IoT C&C classifier for the [BOTA (Botnet Analysis)](https://github.com/danieluhricek/bota) system. It is provided in the form of extended bidirectional flow data. The flow data were generated by [ipfixprobe](https://github.com/CESNET/ipfixprobe) flow exporter and converted into CSV files. Apart from traditional flow information (IP addresses, ports, amount of transferred data), ipfixprobe was set with default timeouts (5 minutes active, 30 s inactive) to generate per-packet information for the first 30 packets. The flow records were then aggregated into 5-minute intervals - when the flow was split due to inactivity, the aggregator then stitched the flow back into a single one.

The column headers in provided CSV files stand for:

|  Column Name             | Description                                                            |
|--------------------------|------------------------------------------------------------------------|
| ipaddr DST_IP            | Source IP address                                                      |
| ipaddr SRC_IP            | Destination IP address                                                 |
| uint64 BYTES             | The number of transmitted bytes from SRC->DST                          |
| uint64 BYTES_REV         | The number of transmitted bytes from DST->SRC                          |
| time TIME_FIRST          | Timestamp of the first packet in the flow in format YYYY-MM-DDTHH-MM-SS|
| time TIME_LAST           | Timestamp of the last packet in the flow in format YYYY-MM-DDTHH-MM-SS |
| macaddr DST_MAC          | Destination MAC address                                                |
| macaddr SRC_MAC          | Source MAC address                                                     |
| uint32 COUNT             | Number of aggregated flow records                                      |
| uint32 PACKETS           | The number of packets transmitted from Source to Destination           |
| uint32 PACKETS_REV       | The number of packets transmitted from Destination to Source           |
| uint16 DST_PORT          | Destination port                                                       |
| uint16 SRC_PORT          | Source port                                                            |
| uint8 DIR_BIT_FIELD      | Flag for distinguishin WAN(1)/LAN(0)                                   |
| uint8 PROTOCOL           | The number of transport protocol                                       |
| uint8 TCP_FLAGS          | Logic OR across all TCP flags in the packets transmitted SRC->DST      |
| uint8 TCP_FLAGS_REV      | Logic OR across all TCP flags in the packets transmitted DST->SRC      |
| int8* PPI_PKT_DIRECTIONS | Array with packets' direction (1)- SRC->DST, (-1)-DST->SRC             |
| uint8* PPI_PKT_FLAGS     | Array with packets' TCP flags                                          |
| uint16* PPI_PKT_LENGTHS  | Array with packets' payload lengths                                    |
| time* PPI_PKT_TIMES      | Array with packets' timestamps                                         |


## Directory tree of provided dataset

```
.
├── README.md
├── cesnet
├── custom-cnc
│   ├── kaiten
│   ├── mirai
│   └── qbot
├── iot-23
│   ├── benign
│   └── cnc
└── iottraces
```

This repository contains four parts of the dataset:

1. `cesnet` - This part was created by packet capturing on the metering points located at the
perimeter of the CESNET2 network. The CESNET training capture was used as benign traffic in the C&C model training
and testing pipeline to cover potential nuances and variability of benign data
seen in the ISP-level network. Since we deal with data from the production network,
we cannot guarantee a benign nature of all captured communication. However, we
verified every IP address according to the internal blocklist of the CESNET
association and external ones. We used [AbuseIPDB](https://www.abuseipdb.com)
and [URLhaus](https://urlhaus.abuse.ch/) blocklists.

2. `custom-cnc` - From leaked source codes, we picked one variant from each of the most prevalent
client-server IoT botnet families: (1) Tsunami, (2) Gafgyt, (3) Mirai. Each implements
a distinct communication protocol; Tsunami is an example of an IRC bot; Gafgyt
uses a simple text-based protocol; Mirai implements a custom binary protocol.
Afterward, we prepared virtualized testing environment. We deployed the malware in a controlled manner,
filtering out its scanning and exploiting activities. The dataset covers the most notable C&C behavior.
As previously recognized, the C&C communication consists of C&C heartbeat and
bot commands. Thus, for each of the three prepared malware variants, we first
imagine the malware running with no received commands. That includes the
initiation of the TCP connection to the C&C server, which continues
for one hour. And then, we imagine the malware receiving commands from its
C&C server. The position of the command packets is chosen arbitrarily relative
to the background heartbeat packets because, in the real-world scenario, the
timing of the commands is tied to a random human action.

3. `iot-23` - Aposemat IoT-23 [1] is a dataset prepared by researchers at Stratosphere Laboratory at CTU. This research group already published another better-known dataset, called CTU-13 [2], which covered several Windows botnets. On the other hand, IoT-23 focuses solely on IoT malware families. The datasets are divided into captures, called scenarios (13 and 23 of them, respectively). Each scenario consists of a pcap capture and other higher-level data. This part of the repository contains flows extracted by ipfixprobe from the said captures.

4. `iottraces` - In [3], the authors studied the possibilities of machine learning methods concerning IoT device classification tasks. As part of the research, they released a dataset containing traffic (named by the authors as traffic traces) of 28 unique devices. The devices are divided into six categories: (1) cameras, (2) controllers and hubs, (3) energy management devices, (4) appliances, (5) healthcare devices, and (6) non-IoT devices such as laptops and smartphones. The traffic was captured using tcpdump, and the data is published in the form of pcaps. Each pcap is 24 hours long, and at the time of writing, the dataset contained 61 separably accessible pcaps. Flows extracted by ipfixprobe served as an another benign part of the dataset.


## Acknowledgement

This research was funded by the Ministry of Interior of the Czech Republic,
grant No. VJ02010024: Flow-Based Encrypted Traffic Analysis and also by the
Grant Agency of the CTU in Prague, grant No. SGS20/210/OHK3/3T/18 funded by
the MEYS of the Czech Republic.

## References

[1] Sebastian Garcia, Agustin Parmisano, & Maria Jose Erquiaga. (2020). IoT-23: A labeled dataset with malicious and benign IoT network traffic (Version 1.0.0) [Data set]. Zenodo. doi:10.5281/zenodo.4743746.

[2] Garcia, S.; Grill, M.; et al. An empirical comparison of botnet detection methods. Computers & Security, volume 45, 2014: pp. 100–123, ISSN 0167-4048, doi:10.1016/j.cose.2014.05.011.

[3] Sivanathan, A.; Gharakheili, H. H.; et al. Classifying IoT Devices in Smart Environments Using Network Traffic Characteristics. IEEE Trans-actions on Mobile Computing, volume 18, no. 8, 2019: pp. 1745–1759, doi:10.1109/TMC.2018.2866249.
