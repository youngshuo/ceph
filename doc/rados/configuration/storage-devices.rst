=================
 Storage Devices
=================

There are two Ceph daemons that store data on devices:

* **Ceph OSDs** (or Object Storage Daemons) are where most of the
  data is stored in Ceph.  Generally speaking, each OSD is backed by
  a single storage device, like a traditional hard disk (HDD) or
  solid state disk (SSD).  OSDs can also be backed by a combination
  of devices, like a HDD for most data and an SSD (or partition of an
  SSD) for some metadata.  The number of OSDs in a cluster is
  generally a function of how much data will be stored, how big each
  storage device will be, and the level and type of redundancy
  (replication or erasure coding).
* **Ceph Monitor** daemons manage critical cluster state like cluster
  membership and authentication information.  For smaller clusters a
  few gigabytes is all that is needed, although for larger clusters
  the monitor database can reach tens or possibly hundreds of
  gigabytes.


OSD Backends
============

There are two ways that OSDs can manage the data they store.  Starting
with the Luminous 12.2.z release, the new default (and recommended) backend is
*BlueStore*.  Prior to Luminous, the default (and only option) was
*Filestore*.

BlueStore
---------

BlueStore is a special-purpose storage backend designed specifically for
managing data on disk for Ceph OSD workloads.  BlueStore is based on over a
decade of experience supporting and managing OSDs using FileStore.

Key BlueStore features include:

* Direct management of storage devices. BlueStore consumes raw block
  devices or partitions.  This avoids intervening layers of
  abstraction (such as local file systems like XFS) that can limit
  performance or add complexity.
* Metadata management with RocksDB. RocksDB's key/value database is embedded
  in order to manage internal metadata, including the mapping of object
  names to block locations on disk.
* Full data and metadata checksumming. By default, all data and
  metadata written to BlueStore is protected by one or more
  checksums. No data or metadata is read from disk or returned
  to the user without being verified.
* Inline compression.  Data can be optionally compressed before being written
  to disk.
* Multi-device metadata tiering. BlueStore allows its internal
  journal (write-ahead log) to be written to a separate, high-speed
  device (like an SSD, NVMe, or NVDIMM) for increased performance.  If
  a significant amount of faster storage is available, internal
  metadata can be stored on the faster device.
* Efficient copy-on-write. RBD and CephFS snapshots rely on a
  copy-on-write *clone* mechanism that is implemented efficiently in
  BlueStore. This results in efficient I/O both for regular snapshots
  and for erasure-coded pools (which rely on cloning to implement
  efficient two-phase commits).

For more information, see :doc:`bluestore-config-ref` and :doc:`/rados/operations/bluestore-migration`.

FileStore
---------

FileStore is the legacy approach to storing objects in Ceph. It
relies on a standard file system (normally XFS) in combination with a
key/value database (traditionally LevelDB, now RocksDB) for some
metadata.

FileStore is well-tested and widely used in production. However, it
suffers from many performance deficiencies due to its overall design
and its reliance on a traditional file system for storing object data.

Although FileStore is capable of functioning on most POSIX-compatible
file systems (including btrfs and ext4), we recommend that only the
XFS file system be used with Ceph. Both btrfs and ext4 have known bugs and
deficiencies and their use may lead to data loss. By default, all Ceph
provisioning tools use XFS.

For more information, see :doc:`filestore-config-ref`.
