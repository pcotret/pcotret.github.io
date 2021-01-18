# Aircrack 101 - How to get the password of a WEP/WPA2 session captured in Wireshark

## Prerequisites
- `sudo apt install aircrack-ng `
- Some wordlists/dictionaries. [Kali Linux](https://www.kali.org/) has already some of them.

## WiFi handshakes 101

Introduction from [@evilsocket](https://github.com/evilsocket)'s Pwnagotchi: [https://pwnagotchi.ai/intro/#wifi-handshakes-101](https://pwnagotchi.ai/intro/#wifi-handshakes-101)

## Dictionary-based attack

In order to test a PCAP with a given dictionary:

```bash
aircrack-ng -w <wordlist> <wireshark_file>
```

You have several dictionaries and sample PCAPs extracted from a Kali distro here: https://github.com/pcotret/aircrack-101-material

One of them should work with a short dictionary. Find it :wink:

### Wireshark captures
Lots of sample captures are available of Wireshark Gitlab: https://gitlab.com/wireshark/wireshark/-/wikis/SampleCaptures

