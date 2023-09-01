# Ubuntu Bartender

Sometimes you just want an [Ubuntu Old Fashioned][1], but you don't want to
make it yourself.

Sometimes you just want someone else to gather the ingredients, make the
drink for you, and clean up afterwards.

So sit back, relax, and let the Ubuntu Bartender do the work for you.

## Really, what is this?

This script will collect all the required bits Ubuntu Old Fashioned
needs to build Ubuntu images, build those images for you, then clean up
afterwards.

## What is an Ubuntu Old Fashioned build?

It builds Ubuntu images. Like what you'd find [here][2]

See: [Ubuntu Old Fashioned][1]

## I'm still confused, can you go into more detail?

Absolutely. Here's an incomplete, high-level overview of the components
of an Ubuntu image build with Ubuntu Bartender:

#### [Ubuntu Bartender][3]
- Manages the lifecycle of a VM in which the image build is run
- Collects the dependencies of an Ubuntu Old Fashioned image build
- Runs an Ubuntu Old Fashioned image build
- Downloads any artifacts produced by the build

#### [Ubuntu Old Fashioned][1]
- Manages the lifecycle of an LXD container in which the image build is run
- Runs launchpad-buildd's build livefs command to create Ubuntu images
- Pulls any artifacts produced by the build out of the LXD container

#### [launchpad-build][4]
- Runs live-build to create Ubuntu images

#### [live-build][5]
- Runs hooks in livecd-rootfs to create Ubuntu images

#### [livecd-rootfs][6]
- Contains configuration and hooks that are used to create Ubuntu images

## Great, but how do I use this?

Just give it a run, like so:

```
ubuntu-bartender -- --series focal
```

## What's with those options?

With Ubuntu Bartender, you can configure all of the components of the build.

Where do you want the VM to run? You can specify:

```
ubuntu-bartender --build-provider aws # build on a remote AWS instance
ubuntu-bartender --build-provider azure # build on a remote Azure instance
ubuntu-bartender --build-provider gce # build on a remote GCE instance
ubuntu-bartender --build-provider multipass # build with a local multipass instance
```

Where should the livecd-rootfs hooks come from? Or the rootfs for Ubuntu Old Fashioned's LXD container? You can specify that, too:

```
ubuntu-bartender --livecd-rootfs-dir ~/local_livecd-rootfs # Use a local checkout of livecd-rootfs
ubuntu-bartender --chroot-archive local_rootfs.tar.gz # Use a different rootfs for the LXD container
```

If you want to include some extra hooks from a local checkout, that can also be done:

```
ubuntu-bartender --livecd-rootfs-dir ~/local_livecd-rootfs --hook-extras-dir ~/local_hooks_dir
```

## That's too many options, how can I keep track of them all?

Use `--help` to list all available options for Ubuntu Bartender. The help text will update to include any configuration already specified.

## You never explained the strange `--` in the first example. What is that?

When passing flags to Ubuntu Bartender, any flag specified after a `--` will be passed directly to Ubuntu Old Fashioned. So you can specify something like:

```
ubuntu-bartender -- \
  --series focal # Set the series to 20.04 LTS
  --project ubuntu-cpc # Build cloud images (CPC = Canonical Public Cloud)
```

__IMPORTANT NOTE:__ You _must_ always specify a series for Ubuntu Old Fashioned. Don't worry, the script will remind you to set this if you forget.

## I don't want to specify a series for Ubuntu Old Fashioned

You're in luck! There are a handful of helper scripts that set all of the relevant flags for you for a specified series. For example, compare the output of the following:

```
ubuntu-xenial-bartender --help
ubuntu-bionic-bartender --help
ubuntu-bartender --help
```

## You mentioned an example specifying different build providers. Why does that matter?

The Ubuntu image build happens in a LXD container within an Ubuntu VM. Ubuntu Bartender can manage the lifecycle of that VM, which can run in either [Multipass][7] or [AWS][8].

Multipass is more convenient - all of your VMs run locally and it's free.

AWS is faster - the cloud has fast in-cloud mirrors of the apt repositories, which dramatically reduces the time it takes to build an image.

## Configuration
You may find more yourself passing in the same arguments over and over again. To help out, Bartender will check a couple places for configuration files:

* `~/.ubuntu-bartender.rc`
* `/etc/ubuntu-bartender/ubuntu-bartender.rc`

Configuration hierarchy is (listed in order of resolution)
1. DEFAULTS
2. Configuration File
3. CLI

### Configuring Providers
Provider options are prepended with the provider name. example `AWS_PROFILE`. This allows you to, for example, set your profile and keypair name for interacting with AWS. In a config file:

```
AWS_PROFILE=work_account
AWS_KEYPAIR_NAME=aws_testing_keypair
MULTIPASS_DISK_SIZE=multipass_disk_size
MULTIPASS_MEM_SIZE=multipass_memory_size
MULTIPASS_IMAGE=multipass_image
```

#### Azure

To build on Azure, specify `--build-provider azure`. You will also need to provide a [location](https://azure.microsoft.com/en-us/global-infrastructure/geographies/) (region) via `--azure-location` (default to `France Central`); a public SSH key for connecting to the VM (via `--azure-pubkey-path`, default to `$HOME/.ssh/$USER.pub`). Finally you can provide a custom image "URN" (via `--azure-urn`, just use the default if you don't know what you do) and the Azure instance [size](https://docs.microsoft.com/en-us/azure/virtual-machines/sizes) (type) with `--azure-instance-size` (default to `Standard_F8s_v2`).

Example:

```
ubuntu-bartender \
  --build-provider azure \
  --azure-pubkey-path ~/.ssh/ubuntu.pub \
  --azure-location "Germany West Central" \
  \
  --livecd-rootfs-branch ubuntu/focal \
  -- \
  --series focal \
  --project ubuntu-cpc \
  --subproject base \
  --image-target kvm
```

#### GCE

To build on GCE, please specify `--build-provider gce`. You will need `gcloud` installed locally and set up with
`gcloud init` (see https://cloud.google.com/sdk/gcloud/reference/init for instructions if unsure)

Example:
```
 ubuntu-bartender \
 --build-provider gce \
 --gce-zone europe-west2-a \
 --gce-machine-type n2-standard-8 \
 --hook-extras-dir "/path/to/extra/hooks/dir" \
 --livecd-rootfs-branch ubuntu/jammy \
 -- \
 --series jammy \
 --project ubuntu-cpc \
 --subproject base \
 --image-target gce
```

## ARM builds
ARM is growing in support, and there is demand for builds. However, most of us do not have an ARM based daily driver for development. Thankfully, AWS and GCE have come to the rescue!

There is a quick option for calling up ARM defaults for aws, `--aws-arm-build` (AWS_ARM_BUILD) and GCE `--gce-arm-build` (GCE_ARM_BUILD).
This is a boolean style flag, so via the cli, passing the flag is sufficient . 
For configuration in file or env_var, please use true or false (literal strings). The only truthy value is the literal string "true". 

### AWS ARM builds
This will run a build searching for an ARM based ami (`AWS_AMI_NAME_FILTER="ubuntu/images-testing/hvm-ssd/ubuntu-focal-daily-arm64-server-*"`) and 
use an arm instance type (`AWS_INSTANCE_TYPE="m6g.large"`). This option is meant as a quick flag so you don't need to look up filter name or instance types.

Know that setting `--aws-ami-name-filter` and `--aws-instance-type` **and** `--aws-arm-build` will result in the `--aws-arm-build` defaults being used, **not** your passed in values.
If you want to set specifics, please use `--aws-ami-name-filter` and `--aws-instance-type` directly.

### GCE ARM builds
This sets `GCE_IMAGE_FAMILY="ubuntu-2004-lts-arm64"` and `GCE_MACHINE_TYPE="t2a-standard-2"`, and will override **any**
passed in values for both. If you need to set specifics, please pass in `--gce-family` and `--gce-machine-type` directly and omit
`--gce-arm-build` instead.

## More questions?

Feel free to open an issue and we can add more to the README.

[1]: https://github.com/chrisglass/ubuntu-old-fashioned
[2]: https://cloud-images.ubuntu.com
[3]: https://github.com/chrisglass/ubuntu-old-fashioned/tree/master/scripts/ubuntu-bartender
[4]: https://launchpad.net/launchpad-buildd
[5]: https://manpages.debian.org/testing/live-build/live-build.7.en.html
[6]: https://launchpad.net/livecd-rootfs
[7]: https://multipass.run/
[8]: https://aws.amazon.com/
[9]: https://console.cloud.google.com/
