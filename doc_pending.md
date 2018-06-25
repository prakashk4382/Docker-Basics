## Docker - @docker


# Outline

- Extremely short intro to Docker

- Short intro to copy-on-write

- History of Docker storage drivers

- AUFS, BTRFS, Device Mapper, Overlay.gray[fs], VFS, ZFS

- Conclusions


---


# What's Docker?

- A *platform* made of the *Docker Engine* and the *Docker Hub*
- The *Docker Engine* is a runtime for containers
- It's Open Source, and written in Go 
- It's a daemon, controlled by a REST API



```
testo@tarrasque:~$ docker run -ti python bash
root@75d4bf28c8a5:/# pip install IPython
Downloading/unpacking IPython
  Downloading ipython-2.3.1-py3-none-any.whl (2.8MB): 2.8MB downloaded
Installing collected packages: IPython
Successfully installed IPython
Cleaning up...
root@75d4bf28c8a5:/# ipython
Python 3.4.2 (default, Jan 22 2015, 07:33:45) 
Type "copyright", "credits" or "license" for more information.

IPython 2.3.1 -- An enhanced Interactive Python.
?         -> Introduction and overview of IPython's features.
%quickref -> Quick reference.
help      -> Python's own help system.
object?   -> Details about 'object', use 'object??' for extra details.

In [1]:
```
]

---

# What happened here?

- We created a *container* (~lightweight virtual machine),
 with its own:

  - filesystem (based initially on a `python` image)
  - network stack
  - process space

- We started with a `bash` process
 (no `init`, no `systemd`, no problem)

- We installed IPython with pip, and ran it

---

# What did  happen here?

- We did not make a full copy of the `python` image

The installation was done in the *container*, not the *image*:

- We did not modify the `python` image itself

- We did not affect any other container
 (currently using this image or any other image)

---

# How is this important?

- We used a *copy-on-write* mechanism
 (Well, Docker took care of it for us)

- Instead of making a full copy of the `python` image,
  keep track of changes between this image and our container

- Huge disk space savings (1 container = less than 1 MB)

- Huge time savings (1 container = less than 0.1s to start)

---



# Short intro to copy-on-write

---

# History

Warning: I'm not a computer historian.

Those random bits are not exhaustive.

---

# Copy-on-write for memory (RAM)

- `fork()` (process creation)

  - Create a new process quickly

  - ... even if it's using many GBs of RAM

  - Actively used by e.g. Redis `SAVE`,
   to obtain consistent snapshots

- `mmap()` (mapped files) with `MAP_PRIVATE`

  - Changes are visible only to current process

  - Private maps are fast, even on huge files

Granularity: 1 page at a time (generally 4 KB)

---

# Copy-on-write for memory (RAM)

How does it work?

- Thanks to the MMU! (Memory Management Unit)

- Each memory access goes through it

- Translates memory accesses (location + operation) into:

  - actual physical location

  - or, alternatively, a *page fault*


Location = address = pointer

Operation = read, write, or exec

---

# Page faults

When a page faults occurs, the MMU notifies the OS.

Then what?

- Access to non-existent memory area = `SIGSEGV`
  (a.k.a. "Segmentation fault" a.k.a. "Go and learn to use pointers")

- Access to swapped-out memory area = load it from disk
  (a.k.a. "My program is now 1000x slower")

- Write attempt to code area = seg fault (sometimes)

- Write attempt to copy area = deduplication operation
  Then resume the initial operation as if nothing happened

- Can also catch execution attempt in no-exec area
  (e.g. stack, to protect against some exploits)

---

# Copy-on-write for storage (disk)

- Initially used (I think) for *snapshots*

  (E.g. to take a consistent backup of a busy database,
  making sure that nothing was modified between the beginning
  and the end of the backup)

- Initially available (I think) on external storage (NAS, SAN)

  (Because It's Complicated)

--

- Suddenly,
  A Wild CLOUD appears!

---

# Thin provisioning for VMs.red[¹]

- Put system image on copy-on-write storage

- For each machine.red[¹], create copy-on-write instance

- If the system image contains a lot of useful software,
  people will almost never need to install extra stuff

- Each extra machine will only need disk space for data!

WIN $$$ (And performance, too, because of caching)

.footnote[.red[¹] Not only VMs, but also physical machines with netboot, and containers!]]

---

# Modern copy-on-write on your desktop

(In no specific order; non-exhaustive list)

- LVM (Logical Volume Manager) on Linux

- ZFS on Solaris, then FreeBSD, Linux ...

- BTRFS on Linux

- AUFS, UnionMount, overlay.gray[fs] ...

- Virtual disks in VM hypervisors

---

# Copy-on-write and Docker: a love story

- Without copy-on-write...

  - it would take forever to start a container

  - containers would use up a lot of space

- Without copy-on-write "on your desktop"...

  - Docker would not be usable on your Linux machine

  - There would be no Docker at all.
    And no meet-up here tonight.
    And we would all be shaving yaks instead.
    ☹
---



# Thank you:

Junjiro R. Okajima (and other AUFS contributors)

Chris Mason (and other BTRFS contributors)

Jeff Bonwick, Matt Ahrens (and other ZFS contributors)

Miklos Szeredi (and other overlay.gray[fs] contributors)

The many contributors to Linux device mapper, thinp target, etc.

... And all the other giants whose shoulders we're sitting on top of, basically

---



# History of Docker storage drivers

---

# First came AUFS

- Docker used to be dotCloud
  (PaaS, like Heroku, Cloud Foundry, OpenShift...)

- dotCloud started using AUFS in 2008
  (with vserver, then OpenVZ, then LXC)

- Great fit for high density, PaaS applications
  (More on this later!)

---

# AUFS is not perfect

- Not in mainline kernel

- Applying the patches used to be *exciting*

- ... especially in combination with GRSEC

- ... and other custom fancery like `setns()`

---

# But some people believe in AUFS!

- dotCloud, obviously

- Debian and Ubuntu use it in their default kernels,
  for Live CD and similar use cases:

  - Your root filesystem is a copy-on-write between
    - the read-only media (CD, DVD...) 
    - and a read-write media (disk, USB stick...)

- As it happens, we also .red[♥] Debian and Ubuntu very much

- First version of Docker is targeted at Ubuntu (and Debian)

---

# Then, some people started to believe in Docker

- Red Hat users *demanded* Docker on their favorite distro

- Red Hat Inc. wanted to make it happen

- ... and contributed support for the Device Mapper driver

- ... then the BTRFS driver

- ... then the overlay.gray[fs] driver


.footnote[Note: other contributors also helped tremendously!]

---



# Special thanks:

Alexander Larsson

Vincent Batts

\+ all the other contributors and maintainers, of course

(But those two guys have played an important role in the
initial support, then maintenance, of the BTRFS, Device Mapper, and overlay drivers.
Thanks again!)]

---



# Let's see each

# storage driver

# in action

---



# AUFS

---

# In Theory

- Combine multiple *branches* in a specific order

- Each branch is just a normal directory

- You generally have:

  - at least one read-only branch (at the bottom)

  - exactly one read-write branch (at the top)

  (But other fun combinations are possible too!)

---

# When opening a file...

- With `O_RDONLY` - read-only access:

  - look it up in each branch, starting from the top

  - open the first one we find

- With `O_WRONLY` or `O_RDWR` - write access:

  - look it up in the top branch;
    if it's found here, open it

  - otherwise, look it up in the other branches;
    if we find it, copy it to the read-write (top) branch,
    then open the copy

.footnote[That "copy-up" operation can take a while if the file is big!]

---

# When deleting a file...

- A *whiteout* file is created
  (if you know the concept of "tombstones", this is similar)


```
# docker run ubuntu rm /etc/shadow

# ls -la /var/lib/docker/aufs/diff/$(docker ps --no-trunc -lq)/etc
total 8
drwxr-xr-x 2 root root 4096 Jan 27 15:36 .
drwxr-xr-x 5 root root 4096 Jan 27 15:36 ..
-r--r--r-- 2 root root    0 Jan 27 15:36 .wh.shadow
```
]

---

# In Practice

- The AUFS mountpoint for a container is
  `/var/lib/docker/aufs/mnt/$CONTAINER_ID/`

- It is only mounted when the container is *running*

- The AUFS branches (read-only and read-write) are in
  `/var/lib/docker/aufs/diff/$CONTAINER_OR_IMAGE_ID/`

- All writes go to `/var/lib/docker`

```
dockerhost# df -h /var/lib/docker
Filesystem      Size  Used Avail Use% Mounted on
/dev/xvdb        15G  4.8G  9.5G  34% /mnt
```

---

# Under the hood

- To see details about an AUFS mount:

  - look for its internal ID in `/proc/mounts`

  - look in `/sys/fs/aufs/si_.../br*`

  - each branch (except the two top ones)
  translates to an image

---

# Example


```
dockerhost# grep c7af /proc/mounts
none /mnt/.../c7af...a63d aufs rw,relatime,si=2344a8ac4c6c6e55 0 0

dockerhost# grep . /sys/fs/aufs/si_2344a8ac4c6c6e55/br[0-9]*
/sys/fs/aufs/si_2344a8ac4c6c6e55/br0:/mnt/c7af...a63d=rw
/sys/fs/aufs/si_2344a8ac4c6c6e55/br1:/mnt/c7af...a63d-init=ro+wh
/sys/fs/aufs/si_2344a8ac4c6c6e55/br2:/mnt/b39b...a462=ro+wh
/sys/fs/aufs/si_2344a8ac4c6c6e55/br3:/mnt/615c...520e=ro+wh
/sys/fs/aufs/si_2344a8ac4c6c6e55/br4:/mnt/8373...cea2=ro+wh
/sys/fs/aufs/si_2344a8ac4c6c6e55/br5:/mnt/53f8...076f=ro+wh
/sys/fs/aufs/si_2344a8ac4c6c6e55/br6:/mnt/5111...c158=ro+wh

dockerhost# docker inspect --format {{.Image}} c7af
b39b81afc8cae27d6fc7ea89584bad5e0ba792127597d02425eaee9f3aaaa462

dockerhost# docker history -q b39b 
b39b81afc8ca
615c102e2290
837339b91538
53f858aaaf03
511136ea3c5a
```
]

---

# Performance, tuning

- AUFS `mount()` is fast, so creation of containers is quick

- Read/write access has native speeds

- But initial `open()` is expensive in two scenarios:

  - when writing big files (log files, databases ...)

  - with many layers + many directories in `PATH`
    (dynamic loading, anyone?)

- Protip: when we built dotCloud, we ended up putting
  all important data on *volumes*

- When starting the same container 1000x, the data is loaded
  only once from disk, and cached only once in memory
  (but `dentries` will be duplicated)

---



# Device Mapper

---

# Preamble

- Device Mapper is a complex subsystem; it can do:

  - RAID

  - encrypted devices

  - snapshots (i.e. copy-on-write)

  - and some other niceties

- In the context of Docker, "Device Mapper" means
  "the Device Mapper system + its *thin provisioning target*"
  (sometimes noted "thinp")

---

# In theory

- Copy-on-write happens on the *block* level
  (instead of the *file* level)

- Each container and each image gets its own block device

- At any given time, it is possible to take a snapshot:

  - of an existing container (to create a frozen image)

  - of an existing image (to create a container from it)

- If a block has never been written to:

  - it's assumed to be all zeros

  - it's not allocated on disk
  (hence "thin" provisioning)

---

# In practice

- The mountpoint for a container is
  `/var/lib/docker/devicemapper/mnt/$CONTAINER_ID/`

- It is only mounted when the container is *running*

- The data is stored in two files, "data" and "metadata"
  (More on this later)

- Since we are working on the block level, there is not much
  visibility on the diffs between images and containers

---

# Under the hood

- `docker info` will tell you about the state of the pool
  (used/available space)

- List devices with `dmsetup ls`

- Device names are prefixed with `docker-MAJ:MIN-INO`

  MAJ, MIN, and INO are derived from the block major,
  block minor, and inode number where the Docker data is
  located (to avoid conflict when running multiple Docker
  instances, e.g. with Docker-in-Docker)]

- Get more info about them with `dmsetup info`, `dmsetup status`
  (you shouldn't need this, unless the system is badly borked)

- Snapshots have an internal numeric ID

- `/var/lib/docker/devicemapper/metadata/$CONTAINER_OR_IMAGE_ID`
  is a small JSON file tracking the snapshot ID and its size

---

<!-- # Example -->

# Extra details

- Two storage areas are needed:
  one for *data*, another for *metadata*

- "data" is also called the "pool"; it's just a big pool of blocks
  (Docker uses the smallest possible block size, 64 KB)

- "metadata" contains the mappings between virtual offsets (in the
  snapshots) and physical offsets (in the pool)

- Each time a new block (or a copy-on-write block) is written,
  a block is allocated from the pool

- When there are no more blocks in the pool, attempts to write
  will stall until the pool is increased (or the write operation
  aborted)

---

# Performance

- By default, Docker puts data and metadata on a loop device
  backed by a sparse file

- This is great from a usability point of view
  (zero configuration needed)

- But terrible from a performance point of view:

  - each time a container writes to a new block,
  - a block has to be allocated from the pool,
  - and when it's written to,
  - a block has to be allocated from the sparse file,
  - and sparse file performance isn't great anyway

---

# Tuning

- Do yourself a favor: if you use Device Mapper,
  put data (and metadata) on real devices!

  - stop Docker

  - change parameters

  - wipe out `/var/lib/docker` (important!)

  - restart Docker


```
docker -d --storage-opt dm.datadev=/dev/sdb1 --storage-opt dm.metadatadev=/dev/sdc1
```
]

---

# More tuning

- Each container gets its own block device

  - with a real FS on it

- So you can also adjust (with `--storage-opt`):

  - filesystem type

  - filesystem size

  - `discard` (more on this later)

- Caveat: when you start 1000x containers,
  the files will be loaded 1000x from disk!

---

# See also



- https://www.kernel.org/doc/Documentation/device-mapper/thin-provisioning.txt

- https://github.com/docker/docker/tree/master/daemon/graphdriver/devmapper

- http://en.wikipedia.org/wiki/Sparse_file

- http://en.wikipedia.org/wiki/Trim_%28computing%29

]

---



# BTRFS

---

# In theory

- Do the whole "copy-on-write" thing at the filesystem level

- Create.red[¹] a "subvolume" (imagine `mkdir` with Super Powers)

- Snapshot.red[¹] any subvolume at any given time

- BTRFS integrates the snapshot and block pool management features
  at the filesystem level, instead of the block device level

 This can be done with the `btrfs` tool.]  

---

# In practice

- `/var/lib/docker` has to be on a BTRFS filesystem!

- The BTRFS mountpoint for a container or an image is
  `/var/lib/docker/btrfs/subvolumes/$CONTAINER_OR_IMAGE_ID/`

- It should be present even if the container is not running

- Data is not written directly, it goes to the journal first
  (in some circumstances.red[¹], this will affect performance)

 E.g. uninterrupted streams of writes.
The performance will be half of the "native" performance.]

---

# Under the hood

- BTRFS works by dividing its storage in *chunks*

- A chunk can contain meta or metadata

- You can run out of chunks (and get `No space left on device`)
  even though `df` shows space available
  (because the chunks are not full)

- Quick fix:

```
# btrfs filesys balance start -dusage=1 /var/lib/docker
```

---

<!-- # Example -->

# Performance, tuning

- Not much to tune

- Keep an eye on the output of `btrfs filesys show`!

This filesystem is doing fine:


```
# btrfs filesys show
Label: none  uuid: 80b37641-4f4a-4694-968b-39b85c67b934
        Total devices 1 FS bytes used 4.20GiB
        devid    1 size 15.25GiB used 6.04GiB path /dev/xvdc
```
]

This one, however, is full (no free chunk) even though there
is not that much data on it:


```
# btrfs filesys show
Label: none  uuid: de060d4c-99b6-4da0-90fa-fb47166db38b
        Total devices 1 FS bytes used 2.51GiB
        devid    1 size 87.50GiB used 87.50GiB path /dev/xvdc
```
]

---



# Overlay.gray[fs]

---

# Preamble

What with the grayed .gray[fs]?

- It used to be called (and have filesystem type) `overlayfs`

- When it was merged in 3.18, this was changed to `overlay`

---

# In theory

- This is just like AUFS, with minor differences:

  - only two branches (called "layers")

  - but branches can be overlays themselves

---

# In practice

- You need kernel 3.18

- On Ubuntu.red[¹]:

  - go to http://kernel.ubuntu.com/~kernel-ppa/mainline/

  - locate the most recent directory, e.g. `v3.18.4-vidi`

  - download the `linux-image-..._amd64.deb` file

  - `dpkg -i` that file, reboot, enjoy


Adapatation to other distros left as an exercise for the reader.
]

---

# Under the hood

- Images and containers are materialized under
  `/var/lib/docker/overlay/$ID_OF_CONTAINER_OR_IMAGE`

- Images just have a `root` subdirectory
  (containing the root FS)

- Containers have:

  - `lower-id` → file containing the ID of the image

  - `merged/` → mount point for the container (when running)]

  - `upper/` → read-write layer for the container

  - `work/` → temporary space used for atomic copy-up

---

<!-- # Example -->

# Performance, tuning

- Implementation detail:
  identical files are hardlinked between images
  (this avoids doing composed overlays)

- Not much to tune at this point

- Performance should be slightly better than AUFS:

  - no `stat()` explosion

  - good memory use

  - slow copy-up, still (nobody's perfect)

---



# VFS

---

# In theory

- No copy on write. Docker does a full copy each time!

- Doesn't rely on those fancy-pesky kernel features

- Good candidate when porting Docker to new platforms
  (think FreeBSD, Solaris...)

- Space .underline[in]efficient, slow

---

# In practice

- Might be useful for production setups

  (If you don't want / cannot use volumes, and don't want / cannot
  use any of the copy-on-write mechanisms!)

---



# ZFS

--

### (Coming Soon To A Docker Release Near You)

--

### (i.e. It Was Merged Into Master Less Than Ten Days Ago)

--

## WOOT!

---

# Sneak peek

- Obviously you need to setup ZFS yourself

- *Should* be compatible with all flavors
  (kernel-based and FUSE)

- Want to play with it?
  https://master.dockerproject.com/

---

# From a user POV...

- Similar to BTRFS

- Fast snapshot, fast creation, fast snapshot...

- Epic performance and reliability (YMMV)

- ZFS has the reputation to be quite Memory hungry
  (=you probably don't want that in a tiny VM)

---

# Implementation details

- Shells out to `zfs-utils`

- Why not `libzfs`?
  Because it's tied to a specific kernel version

---



# Conclusions

---



*The nice thing about Docker storage drivers,
is that there are so many of them to choose from.*

---

# What do, what do?

- If you do PaaS or other high-density environment:

  - AUFS (if available on your kernel)

  - overlayfs (otherwise)

- If you put big writable files on the CoW filesystem:

  - BTRFS or Device Mapper (pick the one you know best)

  - *Wait, really, you want me to pick one!?!*

---



# Bottom line

---



*The best storage driver to run your production
will be the one with which you and your team
have the most extensive operational experience.*

---



# Bonus track

## `discard` and `TRIM`

---

# `TRIM`

- Command sent to a SSD disk, to tell it:
  *"that block is not in use anymore"*

- Useful because on SSD, *erase* is very expensive (slow)

- Allows the SSD to pre-erase cells in advance
  (rather than on-the-fly, just before a *write*)

- Also meaningful on copy-on-write storage
  (if/when every snapshots as trimmed a block, it can be freed)

---

# `discard`

- Filesystem option meaning:
  *"can I has `TRIM` on this pls"*

- Can be enabled/disabled at any time

- Filesystem can also be trimmed manually with `fstrim`
  (even while mounted)

---

# The `discard` quandary

- `discard` works on Device Mapper + loopback devices

- ... but is particularly slow on loopback devices
  (the loopback file needs to be "re-sparsified"
  after container or image deletion, and this is a slow
  operation)

- You can turn it on or off depending on your preference

---



# That's all folks!

---

class: pic

![DockerCon is coming!](dockercon2015.png)

---



# Questions?

    </textarea>
    <script src="/assets/remark-0.5.9.min.js" type="text/javascript">
    </script>
    <script type="text/javascript">
      var slideshow = remark.create();
    </script>
  </body>
</html>
