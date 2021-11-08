# KIO-FUSE XFSTESTS analysis

Results in tablular format can be found [here](https://docs.google.com/spreadsheets/d/1LkeBuplbXXJxoXBS-ApsfA1trRLy1CB6PA5a8E1LyyY/edit?usp=sharing).

## Failing tests

### generic/013

#### What does the test do? 

Tests using fstress. See [this link](https://www2.cs.duke.edu/ari/fstress/) for more information about fstress.

#### Analysis

- Passes fstress with the following set of parameters: 
  1. `-r 1000`
  2. `-p 20 -r 1000`
- KIO-FUSE fails fstress with following parameters: `-p 4 -z -f rmdir=10 -f link=10 -f creat=10 -f mkdir=10 -f rename=30 -f stat=30 -f unlink=30 -f truncate=20`

### generic/014

#### What does the test do? 

Performs the following loop a lot of times:
1. Open the file
2. Write random blocks to random places
3. Truncate the file
4. Close the file


#### Analysis

At some point, opening (using C’s `open()`) the file (step 1) fails with the message ‘invalid argument’. It is not clear whether this happens the first time the file is opened or in future in middle of the loop specified above.


### generic/020

#### What does the test do? 

Checks extended attributes of files by perforaming operations like listing a non-existent file, listing an empty file, query non-existant attribute, CRUD operations on attributes etc.

#### Analysis

KIO-FUSE fails the test when XFSTESTS tries to query and set time of a (potentially non-existent, not clear) file called `syscalltest` located in the `test` partition. *IF* the file does not exist, should this not be the ideal behaviour? If so, why do we fail the test?

### generic/023

#### What does the test do? 

Checks `renameat2` syscall without flags. `renameat2` has an additional flags argument and `renameat2` without flags is equivalent to `renameat` call.

#### Analysis

When XFSTESTS tries to create a directory in the `test` partition, it fails with an `invalid arguments` message. Subsequently, creating the file (using `touch`) fails in absence of the required directory.

### generic/024

#### What does the test do? 

Checks `renameat2` syscall with `RENAME_NOREPLACE` flag.

#### Analysis

Similar to `generic/023`.

### generic/025

#### What does the test do? 

Checks `renameat2` syscall with `RENAME_EXCHANGE` flag.

#### Analysis

Similar to `generic/023` except creation of two files fails because of the failure to create directory.

### generic/028

#### What does the test do? 

Tests `getcwd` call.

#### Analysis

Creation of the file named `t_getcwd_testfile` fails with message `invalid arguments`.


### generic/035

#### What does the test do? 

Checks overwriting rename system call.

#### Analysis

Similar to `generic/023` -- creation of directory required fails, and without the directory, the test cannot proceed and/or succeed.


### generic/070

#### What does the test do? 

'Improved' version of fstress (`generic/013`) test. Specifically tests for extended attribute writes. Passes the following arguments and flags to fstress:
```
_scale_fsstress_args \
	-d $TEST_DIR/fsstress \
	-f allocsp=0 \
	-f freesp=0 \
	-f bulkstat=0 \
	-f bulkstat1=0 \
	-f resvsp=0 \
	-f unresvsp=0 \
	-f attr_set=100 \
	-f attr_remove=100 \
  -p 1 -n 10000 -S c
```

#### Analysis

Setting times of a file called `syscalltest` fails with `no such file or directory`. This means creation of the file failed itself.

### generic/078

#### What does the test do? 

Checks `renameat2` syscall with RENAME_WHITEOUT flag

#### Analysis

XFSTests tries to find files named `none/symb`, `none/dire` & `none/tree` but fails to find them. Additionally, `samedir  regu/none` command fails with `Operation not supported` message.

### generic/080

#### What does the test do? 

Verifies that `mtime` is updated when writing to `mmap`-ed pages

#### Analysis

Test fails because `mtime` and `ctime` fails to update.

### generic/125

#### What does the test do? 

Tests `ftruncate` syscall.

#### Analysis

Truncation of the file seems to succeed but unlinking of the file fails. 

### generic/192

#### What does the test do? 

- Checks that `atime` is persistent after unmount
- Checks that `atime` updates correctly

#### Analysis

Stat-ing of the file used for testing fails with `file doesn't exist` message.

### generic/215

#### What does the test do? 

Checks that `ctime` and `mtime` updates are correct after mapped writes.

#### Analysis

`ctime` and `mtime` doesn't update after mapped writes.

### generic/221

#### What does the test do? 

Checks `ctime` updates when calling futimens without `UTIME_OMIT` for the `mtime` entry.

#### Analysis

XFSTESTS fails to update `ctime`.

### generic/248

#### What does the test do? 

Tests if `pwrite` hangs when writing from mmaped buffer of the same page.

#### Analysis

Opening the test file at `/mnt/test/test_file` fails with 'invalid argument' error.
