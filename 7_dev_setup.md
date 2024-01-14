# Dev Setup

## Setup Go

Download the latest **go** version from `https://go.dev/dl/` and install it.

```sh
# Download the latest version
wget https://golang.org/dl/go1.21.6.linux-amd64.tar.gz -P ~/Downloads
# Check the checksum
sha256sum ~/Downloads/go1.21.6.linux-amd64.tar.gz
3f934f40ac360b9c01f616a9aa1796d227d8b0328bf64cb045c7b8c4ee9caea4  go1.21.6.linux-amd64.tar.gz
# Delete any previous version
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf ~/Downloads/go1.21.6.linux-amd64.tar.gz
```

## Setup VSCode

Setup Sync Settings extension to sync settings across machines.

Wait for the sync to complete.
