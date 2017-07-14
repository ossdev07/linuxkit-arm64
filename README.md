ARM64
=====

This repository is forked from https://github.com/linuxkit/linuxkit.git.
A new branch 'aarch64' is created in order to support ARM64 architecture,
which is checked out from commit '3f89a607362be0cee5d003e9316c1f3fc823bf9c'
in 'master' branch.

Based on 'aarch64' branch, we've built and boot a linuxkit image successfully
on our ARM64 server with below command sequence:

- `make bin/moby`
- `make bin/linuxkit`
- `bin/moby build linuxkit.yml`
- `bin/linuxkit run linuxkit`

We compile all the codes natively within an ARM64 server, so doesn't touch cross-compile.
As a PoC, now we only support 4.11.x kernel series (config file), we use below command
when building the kernel image:

- `make ORG=arm64b build_4.11.x`

Finally we get something like this:

...

[    5.745443] Freeing unused kernel memory: 576K

Starting containerd

Welcome to LinuxKit

                        ##         .
                  ## ## ##        ==
               ## ## ## ## ##    ===
           /"""""""""""""""""\___/ ===
          {                       /  ===-
           \______ O           __/
             \    \         __/
              \____\_______/


INFO[0000] starting containerd boot...			module=containerd

...

INFO[0000] containerd successfully booted in 0.046545s	 module=containerd

linuxkit-525400123456 login: root (automatic login)

Welcome to LinuxKit!

NOTE: This system is namespaced.

The namespace you are currently in may not be the root.

login[991]: root login on 'ttyAMA0'

(ns: getty) linuxkit-525400123456:/# uname -a

Linux linuxkit-525400123456 4.11.8 #1 SMP PREEMPT Thu Jul 6 05:38:19 UTC 2017 aarch64 Linux

(ns: getty) linuxkit-525400123456:/#

Status Update
=============
We use this branch as a start point, base on that we've submitted some PRs order to make ARM64
support for LinuxKit with only one code base.
- Refactor the kernel Dockerfile to support both amd64 and arm64.
- Remove the hardcode of virtual machine type, so now the upstream can run the LinuxKit image
on ARM64 with:
 `bin/linuxkit run -arch "aarch64"
- Remove the fixed built-in x86 bios firmware, thus we can support ARM64 UEFI with:
 `bin/linuxkit run qemu -containerized -arch "aarch64" -fw "/path/to/arm64_bios.bin" -uefi linuxkit`
- Kernel configuration for ARM64
- ...

Major Issue (Closed)
====================
No Ethernet interface generated(only have one 'lo' device), so dhcpcd service can not be launched.
This issue has been resolved after rebase to the latest LinuxKit codebase, but I think this issue
should be related with the kernel configuration file.

Next Plan
=========
- Multi-Arch support. Need to work with LinuxKit community to support both amd64 and arm64 with
one code base only. Now we have some hardcodes in it, especially the docker image.
- Resolve the networking issue and fix all arm64-specific bugs.
- Add UEFI/ACPI support.
- ARM64 support for qemu Dockerfile
- ...

# LinuxKit

LinuxKit, a toolkit for building custom minimal, immutable Linux distributions.

- Secure defaults without compromising usability
- Everything is replaceable and customisable
- Immutable infrastructure applied to building Linux distributions
- Completely stateless, but persistent storage can be attached
- Easy tooling, with easy iteration
- Built with containers, for running containers
- Designed for building and running clustered applications, including but not limited to container orchestration such as Docker or Kubernetes
- Designed from the experience of building Docker Editions, but redesigned as a general-purpose toolkit
- Designed to be managed by external tooling, such as [Infrakit](https://github.com/docker/infrakit) or similar tools
- Includes a set of longer-term collaborative projects in various stages of development to innovate on kernel and userspace changes, particularly around security

## Getting Started

### Build the `moby` and `linuxkit` tools

LinuxKit uses the `moby` tool for image builds, and the `linuxkit` tool for pushing and running VM images.

Simple build instructions: use `make` to build. This will build the tools in `bin/`. Add this
to your `PATH` or copy it to somewhere in your `PATH` eg `sudo cp bin/* /usr/local/bin/`. Or you can use `sudo make install`.

If you already have `go` installed you can use `go get -u github.com/moby/tool/cmd/moby` to install
the `moby` build tool, and `go get -u github.com/linuxkit/linuxkit/src/cmd/linuxkit` to install the `linuxkit` tool.

On MacOS there is a `brew tap` available. Detailed instructions are at [linuxkit/homebrew-linuxkit](https://github.com/linuxkit/homebrew-linuxkit),
the short summary is
```
brew tap linuxkit/linuxkit
brew install --HEAD moby
brew install --HEAD linuxkit
```

Build requirements from source:
- GNU `make`
- Docker
- optionally `qemu`

### Building images

Once you have built the tool, use

```
moby build linuxkit.yml
```
to build the example configuration. You can also specify different output formats, eg `moby build -output raw linuxkit.yml` to
output a raw bootable disk image. See `moby build -help` for more information.

### Booting and Testing

You can use `linuxkit run <name>` or `linuxkit run <name>.<format> to execute the image you created with `moby build <name>.yml`.
This will use a suitable backend for your platform or you can choose one, for example VMWare.
See `linuxkit run --help`.

Currently supported platforms are:
- Local hypervisors
  - [HyperKit (macOS)](docs/platform-hyperkit.md)
  - [Hyper-V (Windows)](docs/platform-hyperv.md)
  - [qemu (macOS, Linux, Windows)](docs/platform-qemu.md)
  - [VMware (macOS, Windows)](docs/platform-vmware.md)
- Cloud based platforms:
  - [Amazon Web Services](docs/platform-aws.md)
  - [Google Cloud](docs/platform-gcp.md)
  - [Microsoft Azure](docs/platform-azure.md)
  - [packet.net](docs/platform-packet.md)


#### Running the Tests

The test suite uses [`rtf`](https://github.com/linuxkit/rtf)
To install this you should use `make bin/rtf && make install`.

To run the test suite:

```
cd test
rtf -x run
```

This will run the tests and put the results in a the `_results` directory!

Run control is handled using labels and with pattern matching.
To run add a label you may use:

```
rtf -x -l slow run
```

To run tests that match the pattern `linuxkit.examples` you would use the following command:

```
rtf -x run linuxkit.examples
```

## Building your own customised image

To customise, copy or modify the [`linuxkit.yml`](linuxkit.yml) to your own `file.yml` or use one of the [examples](examples/) and then run `moby build file.yml` to
generate its specified output. You can run the output with `linuxkit run file`.

The yaml file specifies a kernel and base init system, a set of containers that are built into the generated image and started at boot time. You can specify the type
of artifact to build with the `moby` tool eg `moby build -output vhd linuxkit.yml`.

### Yaml Specification

The yaml format specifies the image to be built:

- `kernel` specifies a kernel Docker image, containing a kernel and a filesystem tarball, eg containing modules. The example kernels are built from `kernel/`
- `init` is the base `init` process Docker image, which is unpacked as the base system, containing `init`, `containerd`, `runc` and a few tools. Built from `pkg/init/`
- `onboot` are the system containers, executed sequentially in order. They should terminate quickly when done.
- `services` is the system services, which normally run for the whole time the system is up
- `files` are additional files to add to the image

For a more detailed overview of the options see [yaml documentation](https://github.com/moby/tool/blob/master/docs/yaml.md)

## Architecture and security

There is an [overview of the architecture](docs/architecture.md) covering how the system works.

There is an [overview of the security considerations and direction](docs/security.md) covering the security design of the system.

## Roadmap

This project was extensively reworked from the code we are shipping in Docker Editions, and the result is not yet production quality. The plan is to return to production
quality during Q3 2017, and rebase the Docker Editions on this open source project during this quarter. We plan to start making stable releases on this timescale.

This is an open project without fixed judgements, open to the community to set the direction. The guiding principles are:
- Security informs design
- Infrastructure as code: immutable, manageable with code
- Sensible, secure, and well-tested defaults
- An open, pluggable platform for diverse use cases
- Easy to use and participate in the project
- Built with containers, for portability and reproducibility
- Run with system containers, for isolation and extensibility
- A base for robust products

## Development reports

There are weekly [development reports](reports/) summarizing work carried out in the week.

## FAQ

See [FAQ](docs/faq.md).

Released under the [Apache 2.0 license](LICENSE).
