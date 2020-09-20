# Building Wireshark from source


## Prerequisites for Ubuntu 20
```bash
sudo apt install libgcrypt20-dev libglib2.0-dev libc-ares-dev libssh-dev libpcap-dev \
libsystemd-dev qtbase5-dev qttools5-dev qtmultimedia5-dev
```
## Download and build
```bash
git clone https://gitlab.com/wireshark/wireshark
cd wireshark
mkdir build
cd build
cmake ..
make
sudo make install
```

