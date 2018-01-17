A machine-global temporary filesystem (tmpfs) on the L4Re microkernel OS (https://l4re.org). 
===========================================


L4Re originally only supported task-local tmpfs, which means that one process does not see the modifications to the filesystem (creation of new files, deletion or modification of existing files) that another process has created.
This limitation is mitigated with the creation of fs_srv. fs_srv depends on libfs by Florian Pester.

It was also planned to extend dom0, the network task-manager (https://gitlab.lrz.de/josef.stark/l4re_taskmgr/), to take advantage of this tmpfs, but it turned out that this is not possible without changing the operation system fundamentally. The reason is that the global tmpfs is based on a central fs-server, with which the tasks communicate via IPC calls. But at the time of its early initialization, a newly created process cannot yet make use of IPC and thus is not able to access files inside the tmpfs (like a program transferred over network). Everything up to that point, however, is working and can be tested with the example scenario. The folders dom0-l4re_kernel, dom0-main and dom0-ned contain the respective changes of the original dom0 code.

The repository contains a snapshot of the used L4Re version (2014092821).


Building:
----------------------------------------
0. Install dependencies. On debian:
$ sudo apt-get install make gawk g++ binutils pkg-config g++-multilib subversion dialog qemu vde2
1. Configure the snapshot:
$ make -C l4re-snapshot-2014092821/ setup
Select x86-32 in the dialog.
2. Delete the ankh-examples because their build is broken in the current snapshot:
$ rm -r l4re-snapshot-2014092821/src/l4/pkg/ankh/examples
3. Some files need to be copied into the l4re-tree (some of them replacing others) before building:
$ cp -r required_files/l4re-snapshot-2014092821/* l4re-snapshot-2014092821/
4. Build Fiasco.OC and L4Re (optionally parallelized with -jX):
$ make -C l4re-snapshot-2014092821/obj/fiasco/ia32/
$ make -C l4re-snapshot-2014092821/obj/l4/x86/
5. The automatic ankh build is also broken in the current snapshot. Do it manually:
$ make -C l4re-snapshot-2014092821/src/l4/pkg/ankh O=../../../../obj/l4/x86/
6. Build the external components:
$ make O=l4re-snapshot-2014092821/obj/l4/x86/
The makefile then builds all (sub-)projects automatically.
Important: Do not parallelize this build with -jX,
so that dependencies can be built in the correct order.
(Reason: dependency handling is not available for external packages)

Testing:
----------------------------------------
If no error occured, the testscripts can now be launched:
1. Start a vde_switch:
$ sudo vde_switch -daemon -mod 660 -group users
(If this fails, try adding your user to the "users" group, then try again.)
2. Change into the L4 build directory:
$ cd l4re-snapshot-2014092821/obj/l4/x86/
3. Execute one of the testscripts:
- For the filesystem-test (tmpfs/ libfs/ machine-global tmpfs):
$ ./run-fs.sh
- For the simple network-test (1 echo-server and three clients):
$ ./run-network.sh
- For the full-blown dom0-example (4 instances):
$ ./run-dom0.sh
