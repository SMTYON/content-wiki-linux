# unleash the beauty of Arch

As much good as Arch gets we always have a fear of Arch going down, this fear makes us afraid of trying and learning. but not to day, we will unleash the beauty of Arch and make you use it without this fear.

#### Note:

<aside>
⚠️

This blog works on Arch Based Distros with a BTRFS file system, other distros and file systems have a similar ways to do this but they will not be included in this blog.

</aside>

## Agenda:

1. Making snapshots of the system
2. Adding snapshots to Grub.
3. automating everything
4. finalize everything

## 1. Making snapshots of the system:

for making a snapshot the best tool for this is “Time Shift”, you can install it using this command:

```cpp
yay -S timeshift
```

after this open Time Shift like any other program, you have to give it you password as it will snapshot every file in the system.

it will look like something like this:

![Screenshot From 2026-04-13 14-47-05.png](Screenshot_From_2026-04-13_14-47-05.png)

now we are going to click on wizard to configure it, then choose BTRFS, choose where you want your snapshots to be, choose how regularly you want time shift to make a snapshot and how many snapshots to keep (recommended: daily keep 4), then you click finish.

## 2. Adding snapshots to Grub:

when your system goes down you will not be able to load your snapshots, this is where “grub-btrfs’ comes in handy, it makes you be able to load all your snapshots from the grub boot menu loader.

to install it:

```cpp
yay -S grub-btrfs
```

now this tool will not add the snapshots to grub automatically but you have to run this command:

```cpp
sudo /etc/grub.d/41_snapshots-btrfs
// then we need to make the grub config file
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

 but we do not need to do this as we will automate everything later on. 

## 3. automating everything:

now we want to automate everything, first let us understand how to do this:

firstly, we need a tool to automate the previous two commands, hopefully it is already exit in grub_btrfs and it is called grub-btrfsd.

secondly, we need a tool that listen to our snapshots and when a new snapshot is taken it calles grub-btrfsd.

this tool is called inotify-tools, this is the installation command:

```cpp
yay -S inotify-tools
```

now there is a small problem, grub-btrfsd configuerd to work on a tool called snaper not Time Shift, but we can edit this, by using this command:

```cpp
sudo systemctl edit --full grub-btrfsd
```

scroll down to “ExectStart” section and make it like this:

```cpp
ExecStart=/usr/bin/grub-btrfsd --syslog --timeshift-auto
```

then save it (ctrl+o), and exit (ctrl+x).

now we want to enable grub-btrfsd, we can do this by running this command:

```cpp
sudo systemctl enable grub-btrfsd
sudo systemctl start grub-btrfsd
```

## 4. finalize everything:

NOTE: this is an additional step but it is recommended.

let us make not just making a snapshot every day, no but also every time you are trying to upgrade your system or install things through pacman.

the tool that do this is called “timeshift-autosnap”, you can istall it by runing this command:

```cpp
yay -S timeshift-autosnap
```

now if you look carefully when you upgrading things through pacman you will see every thing wark:

#### creating a snapshot before upgrading and saving it:

![Screenshot From 2026-04-13 15-27-12.png](Screenshot_From_2026-04-13_15-27-12.png)

#### Grub-btrfsd making the grub config file:

![Screenshot From 2026-04-13 15-28-23.png](Screenshot_From_2026-04-13_15-28-23.png)

now we can use Arch without any fear of missing everything up, and if we did so we can just load to the snapshot from grub boot menu loader.
