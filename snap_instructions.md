## Steps to build and install locally as Snap

1. At repo top level: `snapcraft`
2. `sudo snap install --classic --dangerous *.snap`
    Note `--dangerous` here lets you install the snap without it being signed.
3. To iterate, do `snapcraft clean oldfashioned -s build`, then repeat steps
    1 & 2.

## To test:

- Leave the directory and pull down your desired livecd-rootfs branch.
- `cd` into the livecd-rootfs dir
- `oldfashioned --series <series>`

## To publish:

Directions summarized from [here](https://snapcraft.io/first-snap/python/linux/push)

Note these directions are untested (as of this commit) on this app.

1. Make sure you have an Ubuntu One account
2. `cd` to this repo top level
3. `snapcraft login`
4. `snapcraft register oldfashioned`
5. `snapcraft push --release <channel> *.snap`
