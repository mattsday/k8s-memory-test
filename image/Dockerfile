FROM ubuntu:focal
RUN apt-get update && apt-get -y upgrade && apt-get -y install stress-ng
ENTRYPOINT /usr/bin/stress-ng --vm-bytes "$VM_BYTES" --vm-keep -m 1
