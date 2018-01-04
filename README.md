# Ubuntu old fashioned

This script is meant to let you build ubuntu cloud images locally from a
livecd-rootfs checkout.

## Usage

- Add this script to your path, e.g. /usr/local/bin
- as root (sorry), cd to your livecd-rootfs checkout
- Invoke "build-from-tree"

The images will be built and copied to your $HOME in a "build.output/" folder.

## Speeding things up a bit

To speed up pacakge lookup you can install "squid-deb-proxy" on the host, then
point the host to its own proxy by adding a file like the following:

```bash
export LXD_ADDRESS=$(ifconfig lxdbr0 | grep 'inet addr:' | cut -d: -f2 | awk '{ print $1}')
cat > /etc/apt/apt.conf.d/30squid-proxy << EOF
Acquire::http::Proxy "http://$LXD_ADDRESS:8000";
EOF
```

## Running in the cloud
This was tried on GCE, launching an instance with the following command:

```bash
gcloud compute instances create --image-family=ubuntu-1604-lts --image-project=ubuntu-os-cloud "builder-$(date +%y%m%d-%H%M)" --machine-type n1-highcpu-2 --tags "test" --boot-disk-size 200GiB --zone us-central1-b
```
