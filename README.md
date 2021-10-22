# KIO-FUSE XFSTESTS analysis

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

