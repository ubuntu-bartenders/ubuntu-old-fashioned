# Ubuntu old fashioned

This script is meant to let you build ubuntu cloud images locally from a
livecd-rootfs checkout.

## Usage

- Add this script to your path, e.g. /usr/local/bin
- as root (sorry), cd to your livecd-rootfs checkout
- Invoke "build-from-tree"

The images will be built and copied to your $HOME in a "build.output/" folder.

This was tried on GCE, launching an instance with the following command:

```bash
gcloud compute instances create --image-family=ubuntu-1604-lts --image-project=ubuntu-os-cloud "builder-$(date +%y%m%d-%H%M)" --machine-type n1-highcpu-2 --tags "test" --boot-disk-size 200GiB --zone us-central1-b
```
