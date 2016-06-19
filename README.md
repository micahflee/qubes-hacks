# Qubes Hacks

A small collection of hacks that make weird Qubes things work better for me.

## Git signing

If you have your secret keys in a different security domain ([Qubes Split GPG](https://www.qubes-os.org/doc/split-gpg/)), you can still generate signatures using git (e.g. `git tag -s sometag`), it just takes a little bit of setup work.

First, specify your signing key and gpg program in `~/.gitconfig`. Notice that the the program is `qubes-gpg-client-wrapper-wrapper`, not `qubes-gpg-client-wrapper`. This uses the wrapper that I wrote in this repo instead, because it adds `--no-tty` to the args before calling `qubes-gpg-client-wrapper`.

Here's mine:

```
[user]
  name = Micah Lee
  email = micah@micahflee.com
  signingkey = 927F419D7EC82C2F149C1BD1403C2657CD994F73

[color]
  branch = auto
  diff = auto
  interactive = auto
  status = auto

[gpg]
  program = qubes-gpg-client-wrapper-wrapper
```

And make sure to copy `qubes-gpg-client-wrapper-wrapper` to the right place:

```sh
sudo cp qubes-gpg-client-wrapper-wrapper /rw/usrlocal/bin
sudo chmod 755 /rw/usrlocal/bin/qubes-gpg-client-wrapper-wrapper
```

Now you can use git like normal, and when you sign something it will use your secret key in the domain specified in `/rw/config/gpg-split-domain`.

## Publishing to Launchpad PPAs

Similar to git, but a bit more annoying. When I release a python program to my PPA, first I build a source package, and then I use `dpkg-buildpackage -S` to sign it before [uploading it](https://help.launchpad.net/Packaging/PPA/Uploading).

Something like this:

```sh
# Build source package
python setup.py --command-packages=stdeb.command sdist_dsc
cd deb_dist/${PROJECT_NAME}-${VERSION}

# Make a signed Debian source package
dpkg-buildpackage -S
cd ..

# Push to PPA
dput ppa:micahflee/ppa ${PROJECT_NAME}_${VERSION}-1_source.changes
cd ..
```

In order to make `dpkg-buildpackage` use `qubes-gpg-client-wrapper-wrapper` instead of `gpg` or `gpg2`, you have to specify the `-p` flag, and you should also specify your signing key with `-k`.

This does it for me:

```sh
dpkg-buildpackage -S -pqubes-gpg-client-wrapper-wrapper -k927F419D7EC82C2F149C1BD1403C2657CD994F73
```

Note that I had to hack `qubes-gpg-client-wrapper-wrapper` to deal with `dpkg-buildpackage`'s use of GPG's `--output` flag. `qubes-gpg-client-wrapper` only lets you use `--output` if you set it to `-` (which outputs to stdout), so my wrapper checks for `--output` and if it exists, it changes it to `-` and then writes everything from the subprocess's stdout into the file it was intended to go into. And it works!

# Open links in separate AppVM

You might want to open links that you click on in one AppVM (in software like pidgin, thunderbird, keepassx, etc.) in a separate AppVM. You can do this in Qubes using `qvm-open-in-vm`. I've created `browser_vm.desktop`, and if you look inside you'll see the line:

```
Exec=qvm-open-in-vm browser %u
```

Edit this line to change "browser" to the name of the AppVM you'd like to open links in.

Then install it in an AppVM like this:

```sh
$ mkdir ~/.local/share/applications
$ cp browser_vm.desktop ~/.local/share/applications
$ xdg-settings set default-web-browser browser_vm.desktop
```

Now when you open links in that VM, they'll open in the browser AppVM instead.

