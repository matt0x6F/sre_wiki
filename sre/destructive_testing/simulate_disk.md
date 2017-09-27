Simulating Disk
=====

The idea here is to cause the file system to fail and see how your monitoring and application reacts.

## Fill up a particular disk

`dd if=/dev/zero of=<dest> bs=4k iflag=fullblock,count_bytes count=<size>G`
