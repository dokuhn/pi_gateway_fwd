	 / _____)             _              | |    
	( (____  _____ ____ _| |_ _____  ____| |__  
	 \____ \| ___ |    (_   _) ___ |/ ___)  _ \ 
	 _____) ) ____| | | || |_| ____( (___| | | |
	(______/|_____)_|_|_| \__)_____)\____)_| |_|
	  (C)2013 Semtech-Cycleo

Lora network packet forwarder project
======================================

Changes in this repository compared to [Dragino's repository](https://github.com/dragino/pi_gateway_fwd):
- Modify default device name for GPS (`/dev/ttyS0` instead of `/dev/ttyACM0` already used by bluetooth)
- Add .gitignore for generated files
- Fix output of tmms value

1. Core program: lora_pkt_fwd
-------------------------------

The packet forwarder is a program running on the host of a Lora gateway that
forwards RF packets receive by the concentrator to a server through a IP/UDP
link, and emits RF packets that are sent by the server. It can also emit a
network-wide GPS-synchronous beacon signal used for coordinating all nodes of
the network.

	((( Y )))
	    |
	    |
	+- -|- - - - - - - - - - - - -+        xxxxxxxxxxxx          +--------+
	|+--+-----------+     +------+|       xx x  x     xxx        |        |
	||              |     |      ||      xx  Internet  xx        |        |
	|| Concentrator |<----+ Host |<------xx     or    xx-------->|        |
	||              | SPI |      ||      xx  Intranet  xx        | Server |
	|+--------------+     +------+|       xxxx   x   xxxx        |        |
	|   ^                    ^    |           xxxxxxxx           |        |
	|   | PPS  +-----+  NMEA |    |                              |        |
	|   +------| GPS |-------+    |                              +--------+
	|          +-----+            |
	|                             |
	|            Gateway          |
	+- - - - - - - - - - - - - - -+

Uplink: radio packets received by the gateway, with metadata added by the
gateway, forwarded to the server. Might also include gateway status.

Downlink: packets generated by the server, with additional metadata, to be
transmitted by the gateway on the radio channel. Might also include
configuration data for the gateway.

2. Helper programs
-------------------

Those programs are included in the project to provide examples on how to 
communicate with the packet forwarder, and to help the system builder use it 
without having to implement a full Lora network server.

### 3.1. util_sink ###

The packet sink is a simple helper program listening on a single port for UDP 
datagrams, and displaying a message each time one is received. The content of 
the datagram itself is ignored.

### 3.2. util_ack ###

The packet acknowledger is a simple helper program listening on a single UDP 
port and responding to PUSH_DATA datagrams with PUSH_ACK, and to PULL_DATA 
datagrams with PULL_ACK.

### 3.3. util_tx_test ###

The network packet sender is a simple helper program used to send packets 
through the gateway-to-server downlink route.

4. Helper scripts
-----------------

### 4.1. lora_gateway/reset_lgw.sh

This script, provided with the HAL (lora_gateway), must be launched on IoT Start
Kit platform to reset concentrator chip through GPIO, before starting any
application using the concentrator, like the packet forwarder.

### 4.2. packet_forwarder/lora_pkt_fwd/update_gwid.sh

This script allows automatic update of Gateway_ID with unique MAC address, in
packet forwarder JSON configuration file.
Please refer to the script header for more details.

5. Changelog
-------------

### v4.0.1 - 2017-03-16 ###

* Class-B: Added xtal error correction to beacon frequency
* Class-B: Added support for all regions to beacon frame format (various
datarates imply different frame sizes), as defined by LoRaWAN v1.1.

### v4.0.0 - 2017-01-10 ###

* Added Class-B support, as defined in LoRaWAN v1.1
* Downlink only support "tmst" or "tmms" timestamp. "time" is not supported
anymore ("time" field is kept in Uplink as an informative field).
* Reworked thread_gps to handle GPS UBX messages for native GPS time.
* Updated Gateway <-> NetworkServer protocol to describe the new "tmms" field.
* Updated global_conf.PCB286*.json to remove indexes of the TX gain LUT above
20dBm. Use PCB336 (aka GW v1.5) to comply with ETSI TX mask between 20dBm and
27dBm.

### v3.1.0 - 2016-09-07 ###

* Updated "Listen-Before-Talk" JSON configuration to match with LBT rework.
* Added TX Notch Filter JSON configuration.
* Updated Parson library to latest version
* Fixed Class-B beacon CRC-16 calculation
* Removed JiT time_on_air local function, and use lgw_time_on_air() function

### v3.0.0 - 2016-05-19 ###

* Merged all different flavours of packet forwarder into one unique lora_pkt_fwd
  Note: Various flavours can still be achieved using the corresponding
        global_conf.json.XXX file provided in lora_pkt_fwd/cfg.
* Added downlink "just-in-time" scheduling to optimize downlink capacity.
* Updated Gateway <-> NetworkServer protocol to describe the new format of
"tx_ack" message.
* Added "Listen-Before-Talk" JSON configuration.
* Splitted reset_pkt_fwd.sh script in 2 different scripts:
    - reset_lgw.sh, provided with the HAL (lora_gateway)
    - update_gwid.sh, provided with lora_pkt_fwd

WARNING: Gateway <-> Network Server protocol version has changed. Please refer
         to PROTOCOL.txt file.

### v2.2.1 - 2016-04-12 ###

* util_tx_test: added FSK support and specific payload for easier PER testing.
* base64: fixed padding check.
* Updated all makefiles to handle the creation of obj directory when necessary.
* [gps/beacon]_pkt_fwd: fixed crash on exit when GPS not enabled.
* [*]_pkt_fwd: added a cfg/ directory containing different flavours or the
global_conf.json file for different boards: Ref Design PCB_E336 (GW1.5-27dBm),
Ref Design PCB_E286 (GW1.0), Ref Design with US902 frequency plan.

### v2.2.0 - 2015-10-08 ###

* Removed FTDI support in makefiles to align with HAL v3.2.0.
* Force IPv4 mode usage on UDP socket, instead of auto. The auto mode was
causing an issue to properly resolve LoRa server hostname given in JSON
configuration file (MariaDB issue: https://mariadb.atlassian.net/browse/MDEV-4356,
https://mariadb.atlassian.net/browse/MDEV-4379).

### v2.1.0 - 2015-06-29 ###

* Added helper script for concentrator reset through GPIO, needed on IoT
Starter Kit (reset_pkt_fwd.sh).
* The same reset_pkt_fwd.sh script also allows to automatically update the
Gateway_ID field in JSON configuration file, with board MAC address.
* Updated JSON configuration file with proper default value for IoT Starter
Kit: server address set to local server, GPS device path set to proper value
(/dev/ttyAMA0).

### v2.0.0 - 2015-04-30 ###

* Changed: Several configuration parameters are now set dynamically from the
JSON configuration file: RSSI offset, concentrator clock source, radio type,
TX gain table, network type. The HAL does not need to be recompiled any more to
update those parameters. An example for IoT Starter Kit platform is provided in
global_conf.json for basic, gps and beacon packet_forwarder.
* Removed: Band frequency JSON configuration file has been removed. An example
for EU 868MHz is provided in global_conf.json for basic, gps and beacon packet
forwarder.
* Changed: Updated makefiles to allow cross compilation from environment
variable (ARCH, CROSS_COMPILE).

** WARNING: **
** Update your JSON configuration file with new dynamic parameters. **

### v1.4.1 - 2015-01-23 ###

* Bugfix: fixed LP-116, fdev parameter parsed incorrectly, making FSK TX fail.
* Bugfix: fixed a platform-dependant minor rounding issue.
* Beta: updated beacon format, partially aligned with latest class B proposal.

### v1.4.0 - 2014-10-16 ###

* Feature: Adding TX FSK support.
* Feature: optional auto-quit if a certain number of PULL_ACK is missed.
* Feature: upstream and downstream ping time is displayed on console.
* Bugfix: some beacons were missed at high beaconing frequency.
* Bugfix: critical snprintf error caused a crash for long payloads.
* FSK bitrate now appears in the upstream JSON.

### v1.3.0 - 2014-03-28 ###

* Feature: adding preliminary beacon support for class B development.
* Solved warnings with 64b integer printf when compiling on x86_64.
* Updated build system for easier deployment on various hardware.
* Changed threads organization in the forwarder programs.
* Removed duplicate protocol documentation.

### v1.2.0 - 2014-02-03 ###

* Feature: added a GPS-enabled packet forwarder, used to timestamp received
packet with a globally-synchronous microsecond-accurate timestamp.
* Feature: GPS packet forwarder sends status report on the uplink, protocol
specification updated accordingly (report include gateway geolocation).
* Feature: packets can be sent without CRC at radio layer.
* Bugfix: no more crash with base64 padded input.
* Bugfix: no more rounding errors on the 'freq' value sent to server.
* A minimum preamble of 6 Lora symbol is enforced for optimum sensitivity.
* Padded Base64 is sent on uplink, downlink accepts padded and unpadded Base64.
* Updated the Parson JSON library to a version that supports comments.
* Added .md (Markdown) extension to readme files for better Github viewing.

### v1.1.0 - 2013-12-09 ###

* Feature: added packet filtering parameters to the JSON configuration files.
* Bugfix: will not send a datagram if all the packets returned by the receive()
function have been filtered out.
* Bugfix: removed leading zeros for the timestamp in the upstream JSON because
it is not compliant with JSON standard (might be interpreted as octal number).
* Removed TXT extension for README files for better Github integration.
* Cleaned-up documentation, moving change log to top README.
* Modified Makefiles to ease cross-compilation.

### v1.0.0 - 2013-11-22 ###

* Initial release of the packet forwarder, protocol specifications and helper
programs.

6. Legal notice
----------------

The information presented in this project documentation does not form part of 
any quotation or contract, is believed to be accurate and reliable and may be 
changed without notice. No liability will be accepted by the publisher for any 
consequence of its use. Publication thereof does not convey nor imply any 
license under patent or other industrial or intellectual property rights. 
Semtech assumes no responsibility or liability whatsoever for any failure or 
unexpected operation resulting from misuse, neglect improper installation, 
repair or improper handling or unusual physical or electrical stress 
including, but not limited to, exposure to parameters beyond the specified 
maximum ratings or operation outside the specified range. 

SEMTECH PRODUCTS ARE NOT DESIGNED, INTENDED, AUTHORIZED OR WARRANTED TO BE 
SUITABLE FOR USE IN LIFE-SUPPORT APPLICATIONS, DEVICES OR SYSTEMS OR OTHER 
CRITICAL APPLICATIONS. INCLUSION OF SEMTECH PRODUCTS IN SUCH APPLICATIONS IS 
UNDERSTOOD TO BE UNDERTAKEN SOLELY AT THE CUSTOMER�S OWN RISK. Should a 
customer purchase or use Semtech products for any such unauthorized 
application, the customer shall indemnify and hold Semtech and its officers, 
employees, subsidiaries, affiliates, and distributors harmless against all 
claims, costs damages and attorney fees which could arise.

*EOF*
