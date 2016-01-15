# Docker Storage Centos 7 Face-off
Docker 1.9.1 file IO testing raw vs loop lvm vs direct lvm 

# Spec
Intel(R) Xeon(R) CPU E5-2623 v3 @ 3.00GHz x2
16GB RAM
3 TB HDD
LVM 10GB
CentOS 7.1.1503
Docker version 1.9.1, build a34a1d5
iozone: 3.434 (Stable; go to http://www.iozone.org/ -> click "Stable tarball" under "Download Source") 

# How did I test
## Raw machine test: 

1. CentOS yum update
1. Yum install gcc, make
1. Follow this instruction (http://www.thegeekstuff.com/2011/05/iozone-examples/) download current stable and make the test (./iozone -a -b raw_data.xls)

## Docker loopback / thin-lvm test:

1. Install Docker as official document (https://docs.docker.com/engine/installation/centos/) instructed
1. Modify daemon (/usr/lib/systemd/system/docker.service) to work around build image issue (https://github.com/docker/docker/issues/17653; look at crosbymichael comment on Nov 11, 2015) and start docker daemon
Build c7perf image ("docker build -t c7perf --rm=true .") based on https://github.com/jeremyeder/docker-performance/blob/master/Dockerfiles/Dockerfile. Note that Dockerfile should be place next to iozone_3_434 directory (see Dockerfile enclosed)
1. Create /results directory, run c7perf docker image, and do test:

        # mkdir -p /results
        # chcon -Rt svirt_sandbox_file_t /results
        # docker run -it -v /results:/results c7perf bash
        # cd /root/src/current
        # ./iozone -a -b /results/loop_data.xls

## Docker direct LVM test:

1. Dump the image we just build (c7perf and centos) with Docker save (https://docs.docker.com/engine/reference/commandline/save/) to tar file, and save to somewhere
1. Stop docker service
1. Follow instruction on docker official site (https://docs.docker.com/engine/userguide/storagedriver/device-mapper-driver/) to make the change
1. For change daemon part, the line should look like this:
 
        ExecStart=/usr/bin/docker daemon --storage-driver=devicemapper --storage-opt dm.datadev=/dev/vg-docker/data --storage-opt dm.metadatadev=/dev/vg-docker/metadata -H fd://

1. Start Docker daemon and load images back (https://docs.docker.com/engine/reference/commandline/load/)
1. Repeat Docker loopback section to test:

        # docker run -it -v /results:/results c7perf bash
        # cd /root/src/current
        # ./iozone -a -b /results/directlvm_data.xls

# Test Result

See the excel file in this project
