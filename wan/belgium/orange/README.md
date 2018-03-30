# MMCLI and Belgium Orange Mobile Network Operator aka MNO

## Documenting my use of mmcli on WAN (modem) enabled Lenovo x250 and Orange's mobile network.

## First what is mmcli, well you can read the official description here after:
[from the man of mmcli]
ModemManager  is  a  DBus-powered Linux daemon which provides a unified high level API
for communicating with (mobile broadband) modems. It acts as  a  standard  RIL  (Radio
Interface  Layer)  and  may be used by different connection managers, like NetworkMan‐
ager. Thanks to the built-in plugin architecture, ModemManager talks to very different
kinds  of  modems  with  very different kinds of ports. In addition to the standard AT
serial ports, Qualcomm-based QCDM and QMI ports are also supported.

## Second what is Orange as I said, Orange is one of the 3 MNO in Belgim. There are also about 20 VMNO or Virtual Mobile Network Operators, from witch I tested Lyca Mobile, but there was none difference with Orange, so I only documenting my Orange experience.

## Third I will show some commands in order to connect to the network, get and IP and test the connection. I simple documenting in order to be able to remind myself later if needed.

### list modems on my laptop
```colsole
$>sudo mmcli -L

Found 1 modems:
	/org/freedesktop/ModemManager1/Modem/1 [Sierra] MBIM [1199:A001]
```

### query the modem
```colsole
$>sudo mmcli -m 1

/org/freedesktop/ModemManager1/Modem/1 (device id 'b3fba7dccfa7e5e9d0be85ce8372e40b2aba5737')
  -------------------------
  Hardware |   manufacturer: 'Sierra'
           |          model: 'MBIM [1199:A001]'
           |       revision: 'FIH7160_V1.2_WW_01.1415.07'
           |      supported: 'gsm-umts, lte'
           |        current: 'gsm-umts, lte'
           |   equipment id: '01****************'
  -------------------------
  System   |         device: '/sys/devices/pci0000:00/0000:00:14.0/usb1/1-4'
           |        drivers: 'cdc_mbim, cdc_acm'
           |         plugin: 'Sierra'
           |   primary port: 'cdc-wdm0'
           |          ports: 'wwp0s20u4 (net), cdc-wdm0 (mbim), ttyACM0 (at)'
  -------------------------
  Numbers  |           own : 'unknown'
  -------------------------
  Status   |           lock: 'none'
           | unlock retries: 'sim-pin2 (3)'
           |          state: 'registered'
           |    power state: 'on'
           |    access tech: 'lte'
           | signal quality: '0' (cached)
  -------------------------
  Modes    |      supported: 'allowed: 2g, 3g, 4g; preferred: none'
           |        current: 'allowed: 2g, 3g, 4g; preferred: none'
  -------------------------
  Bands    |      supported: 'unknown'
           |        current: 'unknown'
  -------------------------
  IP       |      supported: 'ipv4, ipv6, ipv4v6'
  -------------------------
  3GPP     |           imei: '01*****************'
           |  enabled locks: 'sim, fixed-dialing'
           |    operator id: '20610'
           |  operator name: 'B Mobistar'
           |   subscription: 'unknown'
           |   registration: 'home'
  -------------------------
  SIM      |           path: '/org/freedesktop/ModemManager1/SIM/1'

  -------------------------
  Bearers  |          paths: 'none'

```

### unlock SIM "SURF" card with the factory PIN
```colsole
$>sudo mmcli -i 1 --pin=4994
successfully sent PIN code to the SIM
```

### disable PIN [you need to provide the current PIN too]
```colsole
$>sudo mmcli -i 1 --disable-pin --pin=4994
successfully disabled PIN code request in the SIM
```

### scan WAN networks [apparently scanning 3G is very long process always give a timeout :)]
```colsole
$>mmcli -m 1 --3gpp-scan --timeout=300

Found 9 networks:
20620 - BASE (unknown, available)
20601 - BEL PROXIMUS (unknown, available)
20601 - BEL PROXIMUS (unknown, available)
20620 - BASE (unknown, available)
20601 - BEL PROXIMUS (unknown, available)
20620 - BASE (unknown, available)
20610 - B Mobistar (unknown, available)
20610 - B Mobistar (unknown, available)
20610 - B Mobistar (unknown, current)
```

### start listening on WAN device [wwp0s20u4]
```colsole
$> ifconfig wwp0s20u4 up
$> tcpdump -nnni wwp0s20u4
```

### connect to mobile network [$ mmcli -m 1 --simple-connect="apn=mworld.be", but it turns out that no APN works like charm too]
```colsole
$>mmcli -m 1 --simple-connect="apn="
successfully connected the modem
```

### list modem and its bearers
```colsole
$>mmcli -m 1 --list-bearers

Found 1 bearers:

	/org/freedesktop/ModemManager1/Bearer/1
```

### query bearer see if connected and copy IP information
```colsole
$>mmcli -b 1
Bearer '/org/freedesktop/ModemManager1/Bearer/1'
  -------------------------
  Status             |   connected: 'yes'
                     |   suspended: 'no'
                     |   interface: 'wwp0s20u4'
                     |  IP timeout: '20'
  -------------------------
  Properties         |         apn: 'mworld.be'
                     |     roaming: 'allowed'
                     |     IP type: 'none'
                     |        user: 'none'
                     |    password: 'none'
                     |      number: 'none'
                     | Rm protocol: 'unknown'
  -------------------------
  IPv4 configuration |   method: 'static'
                     |  address: '10.226.10.47'
                     |   prefix: '24'
                     |  gateway: '10.226.10.1'
                     |      DNS: '212.224.255.252', '212.224.255.254'
  -------------------------
  IPv6 configuration |   method: 'unknown'
  -------------------------
  Stats              |          Duration: '0'
                     |    Bytes received: 'N/A'
                     | Bytes transmitted: 'N/A'
```

### assigne the IP Address to your modem TCP interface [in my case wwp0s20u4]
```colsole
$> sudo ifconfig wwp0s20u4 10.226.10.47 netmask 255.255.255.0
```

### I also needed to update default route and DNS servers
```colsole
$> sudo route add default gw 10.226.10.1
$> echo "nameserver 8.8.8.8" > /etc/resolv.conf
```

### test connection
```colsole
$> ping google.com
PING google.com (216.58.204.110) 56(84) bytes of data.
64 bytes from par10s28-in-f110.1e100.net (216.58.204.110): icmp_seq=1 ttl=52 time=25.7 ms
```

### close all connections
```colsole
$>mmcli -m 1 --simple-disconnect
successfully disconnected all bearers in the modem
```

#### SEND SMS [the surf only card is suppose to be only for surf right?], let me test SMSs from this card
```colsole
$>mmcli -m 1 --messaging-create-sms="text='test from orange',number='+32476******'"
Successfully created new SMS:
	/org/freedesktop/ModemManager1/SMS/1 (unknown)
```

### send the sms
```colsole
$>mmcli -s 1 --send
successfully sent the SMS
```

### and yes I got the SMS on my phone, in this case I also know the GMS number ##
