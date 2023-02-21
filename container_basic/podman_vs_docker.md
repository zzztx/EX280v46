

# Podman Comparison to Docker

## 1. CLI
Podman offers the same set of commands exposed by the Docker client. In other words, there is a one-to-one mapping between the commands of these two utilities.

## 2. Container Model
Docker uses a client-server architecture for the containers, whereas Podman uses the traditional fork-exec model common across Linux processes. The containers created using Podman, are the child process of the parent Podman process. 
```
$ docker info
Client:
 Context:    default
 Debug Mode: false

Server:
 Containers: 0
  Running: 0
  Paused: 0
  Stopped: 0
 Images: 0
 Server Version: 20.10.12
 Storage Driver: overlay2
  Backing Filesystem: extfs
  Supports d_type: true
  Native Overlay Diff: true
  userxattr: false
 Logging Driver: json-file
 Cgroup Driver: systemd
 Cgroup Version: 1
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog
 Swarm: inactive
 Runtimes: io.containerd.runc.v2 io.containerd.runtime.v1.linux runc
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: 
 runc version: 
 init version: 
 Security Options:
  apparmor
  seccomp
   Profile: default
 Kernel Version: 5.4.0-131-generic
 Operating System: Ubuntu 20.04.5 LTS
 OSType: linux
 Architecture: x86_64
 CPUs: 1
 Total Memory: 1.937GiB
 Name: controlplane
 ID: J57V:ILJI:DWZP:DXAR:Z4ZY:G43G:FKDQ:IZWF:XJ6U:NW5Z:PL3V:4CNV
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
 Registry: https://index.docker.io/v1/
 Labels:
 Experimental: false
 Insecure Registries:
  127.0.0.0/8
 Registry Mirrors:
  https://mirror.gcr.io/
  https://docker-mirror.killercoda.com/
  https://docker-mirror.killer.sh/
 Live Restore Enabled: false

WARNING: No swap limit support

$ podman info
host:
  arch: amd64
  buildahVersion: 1.23.1
  cgroupControllers:
  - cpuset
  - cpu
  - cpuacct
  - blkio
  - memory
  - devices
  - freezer
  - net_cls
  - perf_event
  - net_prio
  - hugetlb
  - pids
  - rdma
  cgroupManager: systemd
  cgroupVersion: v1
  conmon:
    package: 'conmon: /usr/libexec/podman/conmon'
    path: /usr/libexec/podman/conmon
    version: 'conmon version 2.0.30, commit: '
  cpus: 1
  distribution:
    codename: focal
    distribution: ubuntu
    version: "20.04"
  eventLogger: journald
  hostname: controlplane
  idMappings:
    gidmap: null
    uidmap: null
  kernel: 5.4.0-131-generic
  linkmode: dynamic
  logDriver: journald
  memFree: 92626944
  memTotal: 2079682560
  ociRuntime:
    name: crun
    package: 'crun: /usr/bin/crun'
    path: /usr/bin/crun
    version: |-
      crun version 1.3.7-506ba
      commit: 7b4a7042370eea7fb00d3a4da34332b26f080acd
      spec: 1.0.0
      +SYSTEMD +SELINUX +APPARMOR +CAP +SECCOMP +EBPF +CRIU +YAJL
  os: linux
  remoteSocket:
    path: /run/podman/podman.sock
  security:
    apparmorEnabled: true
    capabilities: CAP_CHOWN,CAP_DAC_OVERRIDE,CAP_FOWNER,CAP_FSETID,CAP_KILL,CAP_NET_BIND_SERVICE,CAP_SETFCAP,CAP_SETGID,CAP_SETPCAP,CAP_SETUID,CAP_SYS_CHROOT
    rootless: false
    seccompEnabled: true
    seccompProfilePath: /usr/share/containers/seccomp.json
    selinuxEnabled: false
  serviceIsRemote: false
  slirp4netns:
    executable: /usr/bin/slirp4netns
    package: 'slirp4netns: /usr/bin/slirp4netns'
    version: |-
      slirp4netns version 1.1.8
      commit: unknown
      libslirp: 4.3.1-git
      SLIRP_CONFIG_VERSION_MAX: 3
      libseccomp: 2.5.1
  swapFree: 0
  swapTotal: 0
  uptime: 38m 9.44s
plugins:
  log:
  - k8s-file
  - none
  - journald
  network:
  - bridge
  - macvlan
  volume:
  - local
registries:
  search:
  - docker.io
  - quay.io
store:
  configFile: /etc/containers/storage.conf
  containerStore:
    number: 0
    paused: 0
    running: 0
    stopped: 0
  graphDriverName: overlay
  graphOptions:
    overlay.mountopt: nodev,metacopy=on
  graphRoot: /var/lib/containers/storage
  graphStatus:
    Backing Filesystem: extfs
    Native Overlay Diff: "false"
    Supports d_type: "true"
    Using metacopy: "true"
  imageStore:
    number: 0
  runRoot: /run/containers/storage
  volumePath: /var/lib/containers/storage/volumes
version:
  APIVersion: 3.4.2
  Built: 0
  BuiltTime: Thu Jan  1 00:00:00 1970
  GitCommit: ""
  GoVersion: go1.15.2
  OsArch: linux/amd64
  Version: 3.4.2

```

## 3. Rootless Mode
Podman doesn't require root access to run its commands. Docker, on the other hand, being dependent on the daemon process, requires root privileges or requires the user to be part of the docker group to be able to run the Docker commands without root privilege.
```
ubuntu@controlplane:~$ podman ps
CONTAINER ID  IMAGE       COMMAND     CREATED     STATUS      PORTS       NAMES
ubuntu@controlplane:~$ docker ps
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/containers/json": dial unix /var/run/docker.sock: connect: permission denied

$ sudo usermod -aG docker ubuntu

```



