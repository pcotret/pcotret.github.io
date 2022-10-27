# Building Wireshark from source


## Download and build
```bash
git clone https://gitlab.com/wireshark/wireshark
sudo wireshark/tools/./debian-setup.sh
cd wireshark
mkdir build
cd build
cmake .. -DUSE_qt6=OFF
make -j$(nproc)
sudo make install
```

