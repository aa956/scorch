# scorch (Silent CORruption CHecker)

A tool to help discover silent corruption on a filesystem. If using ZFS, BTRFS, SnapRaid, or similar this tool isn't needed. While tools like {md5,sha1}sum and other tools can hash files and check files against that hash **scorch** provides a full workflow for doing so.

### Usage
```
usage: scorch [-h] -d DB [-v] [-r {sticky,readonly}] [-f FNFILTER]
              [-s {none,radix,reverse-radix,natural,reverse-natural,random}]
              [-m MAX] [-b]
              {add,append,check,check+update,delete,cleanup,list,list-unhashed,list-dups}
              dir [dir ...]

a tool to help discover file corruption

positional arguments:
  {add,append,check,check+update,delete,cleanup,list,list-unhashed,list-dups}
                        actions
  dir                   directories to work on

optional arguments:
  -h, --help            show this help message and exit
  -d DB, --db DB        database which stores hashes
  -v, --verbose         print details of files
  -r {sticky,readonly}, --restrict {sticky,readonly}
                        restrict action to certain types of files
  -f FNFILTER, --fnfilter FNFILTER
                        restrict action to files which match regex
  -s {none,radix,reverse-radix,natural,reverse-natural,random}, --sort {none,radix,reverse-radix,natural,reverse-natural,random}
                        when adding/appending/checking sort files before
                        acting on them
  -m MAX, --max MAX     max number of actions to take
  -b, --break-on-error  break on first failure / error

```

### Instructions

* add: for all regular files found, compute and store it's hash and metadata.
* append: for all regular files not found in the hash database, compute and store it's hash and metadata.
* check: for all files in the database, recompute hash and report if mismatched. If mtime or size of file differ from when originally  computed it will warn and list differences.
* check+update: same as **check** but when a file is found changed recompute the hash and metadata and overwrite existing entry.
* delete: remove files from hash database.
* cleanup: remove files no longer in filesystem from hash database.
* list: a **md5sum** compatible listing of files and their hashes.
* list-unhashed: list files which are not hashed in the database.
* list-dups: returns a listing of files which have the same hash value.

### Example

```
$ ls -lh /tmp/files
total 0
-rw-rw-r-- 1 bile bile 0 May  3 16:30 a
-rw-rw-r-- 1 bile bile 0 May  3 16:30 b
-rw-rw-r-- 1 bile bile 0 May  3 16:30 c

$ scorch -v -d /var/tmp/hash.db add /tmp/files
1/3 /tmp/files/c: d41d8cd98f00b204e9800998ecf8427e
2/3 /tmp/files/a: d41d8cd98f00b204e9800998ecf8427e
3/3 /tmp/files/b: d41d8cd98f00b204e9800998ecf8427e

$ scorch -v -d /var/tmp/hash.db check /tmp/files
1/3 /tmp/files/a: OK
2/3 /tmp/files/b: OK
3/3 /tmp/files/c: OK

$ echo asdf > /tmp/files/d

$ scorch -v -d /var/tmp/hash.db list-unhashed /tmp/files
/tmp/files/d

$ scorch -v -d /var/tmp/hash.db append /tmp/files
1/1 /tmp/files/d: 2b00042f7481c7b056c4b410d28f33cf

$ scorch -v -d /tmp/hash.db list-dups /tmp/files
d41d8cd98f00b204e9800998ecf8427e /tmp/files/a /tmp/files/b /tmp/files/c

$ echo foo > /tmp/files/a
$ scorch -v -d /tmp/hash.db check+update /tmp/files
1/4 /tmp/files/b: OK
2/4 /tmp/files/c: OK
3/4 /tmp/files/a: FILE CHANGED
 - size: 0 4
 - mtime: 1462307452 1462307725
 - updated hash: d3b07384d113edec49eaa6238ad5ff0
4/4 /tmp/files/d: OK

$ scorch -v -d /tmp/hash.db list /tmp/files | md5sum -c
/tmp/files/c: OK
/tmp/files/d: OK
/tmp/files/a: OK
/tmp/files/b: OK
```

### Automation

A typical setup would probably be initialized manually by using **add** or **append**. After it's finished creating the database a cron job can be created to check, update, and append to the database. By not placing **scorch** into verbose mode only failures will be printed and the output from the job running will be emailed to the user.

```
#!/bin/sh

scorch -d /var/tmp/hash.db check+update /tmp/files
scorch -d /var/tmp/hash.db append /tmp/files
scorch -d /var/tmp/hash.db cleanup /tmp/files
```

Since if a file's modify time or size change it is likely it was changed intentionally the **check+update** instruction will warn about the change and update the database rather than indicating it's a corruption ("FAILED"). Only if the mtime and size are the same and the hashes differ do we consider it corrupted.

# Support

#### Contact / Issue submission
* github.com: https://github.com/trapexit/scorch/issues
* email: trapexit@spawn.link
* twitter: https://twitter.com/_trapexit

#### Support development

This software is free to use and released under a very liberal license. That said if you like this software and would like to support its development donations are welcome.

* Bitcoin (BTC): 12CdMhEPQVmjz3SSynkAEuD5q9JmhTDCZA
* Bitcoin Cash (BCH): 1AjPqZZhu7GVEs6JFPjHmtsvmDL4euzMzp
* Ethereum (ETH): 0x09A166B11fCC127324C7fc5f1B572255b3046E94
* Litecoin (LTC): LXAsq6yc6zYU3EbcqyWtHBrH1Ypx4GjUjm
* Ripple (XRP): rNACR2hqGjpbHuCKwmJ4pDpd2zRfuRATcE
* PayPal: trapexit@spawn.link
* Patreon: https://www.patreon.com/trapexit
