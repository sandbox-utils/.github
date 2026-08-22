sandbox-utils
=============
Lightweight and portable app utilities for
sandboxing, security and observation
of untrusted programs.


Problem statement
-----------------
**Running other people's programs is inherently insecure.**
[Rogue dependencies]([https://www.google.com/search?q=malicious+python+packages&tbm=nws](https://altpower.app/?q=malicious+python+packages#gsc.tab=0&gsc.q=malicious%20python%20packages&gsc.sort=date%3Ad&gsc.ref=more%3Anews))\*
🎯 or [hacked library code]([https://www.google.com/search?q=(hacked+OR+hijacked+OR+backdoored+OR+"supply+chain+attack")+(npm+OR+pypi)&tbm=nws&num=100](https://altpower.app/?q=%28hacked+OR+hijacked+OR+backdoored+OR+%22supply+chain+attack%22%29+%28npm+OR+pypi%29#gsc.tab=0&gsc.q=(hacked%20OR%20hijacked%20OR%20backdoored%20OR%20%22supply%20chain%20attack%22)%20(npm%20OR%20pypi)&gsc.sort=date%3Ad&gsc.ref=more%3Anews))
:pirate_flag: ([et cetera](https://slsa.dev/spec/draft/threats-overview) :warning:)
**can wreak havoc, including access all your private parts** :bangbang:—think
all current user's credentials and more personal bits like:
* `~/.ssh`,
* `~/.gnupg`,
* `~/.pki/nssdb/`,
* `~/.mozilla/firefox/<profile>/key4.db` ...

> [!CAUTION]
> Running any
> [Electron app](https://www.electronjs.org/apps)
> relies on impeccability of hundreds or thousands of dependencies,
> Node JS and Google Chromium to say the least!
> Don't get pwned—use a sandbox!


Solutions
---------
The alternatives are listed by popularity, descending.
The recommended solution is listed last.


### File ownership and permissions, ACLs

A simple method is to run untrusted programs as a different user.
This way, the untrusted program can't access anything outside its `$HOME` dir.
In reality, most Linux distributions set
[default umask to `0022`](https://pubs.opengroup.org/onlinepubs/9799919799/utilities/umask.html),
which means that **the untrusted program will be able to _read_
files of other users**, unless specifically prevented: secret key directories like
`~/.gnupg` or `~/.ssh` set `chmod o-rwx`, but other common directories may not.


### Docker / podman

Popular solution: run untrusted software in separate Docker containers:
```shell
# Run in a debian-slim container with shared network and bind-mounted current dir
podman run --rm -it -v "$PWD:$PWD" --workdir="$PWD" \
           --net=host debian:stable-slim ./scary-binary
```
While fairly accessible (containerization comes free when the project
spans multiple stacks and already deals in
[Containerfiles](https://manpages.debian.org/unstable/Containerfile)),
this solutions is not safe: Docker/podman ecosystem is vast
and **prone to its own set of risks and supply chain attacks**:
using others' prebuilt images,
downloading images from third-party registries,
exposed docker socket,
stewardship by a major US for-profit corporation,
fast-paced development (even after >10 years 🙄),
countless high-impact
[security](https://docs.docker.com/security/security-announcements/)
[vulnerabilities](https://www.tenable.com/cve/search?q=podman)
in the last few years alone,
[monoculture risk](https://en.wikipedia.org/wiki/Monoculture_(computer_science))
etc.


### AppArmor

On GNU/Linux, [AppArmor](https://apparmor.net), even with
[apparmor.d](https://github.com/roddhjav/apparmor.d) applied,
doesn't ship profiles for many popular programs,
let alone for major execution environments like `python3` and `npm`, and
for custom invocation one need go
through explicit `aa-exec --profile my-custom-env`,
but writing custom AppArmor profiles is arcane and
much less common than simply using [containers](#docker--podman) ...


### Firejail

[Firejail](https://github.com/netblue30/firejail/) is
an indie C project that sets up its own sandbox with virtually no dependencies.
[<del>Red Hat</del><ins>IBM</ins> has a reasonable position on it](https://github.com/containers/bubblewrap?tab=readme-ov-file#related-project-comparison-firejail)
([second opinion](https://madaidans-insecurities.github.io/linux.html#firejail)).
Even after 10 years, the development is still very active and fast-moving.
It's a matter of trust.
Similarly to AppArmor, it requires writing custom profiles.


### Bubblewrap

Intimidated by [Canonical](https://canonical.com)'s technologies like
[AppArmor](#apparmor) and [Snap](https://en.wikipedia.org/wiki/Snap_(software)) (app package format),
Red Hat needed something to sandbox their own
packaged app image format [Flatpak](https://en.wikipedia.org/wiki/Flatpak).
[They didn't like the existing Firejail](https://github.com/containers/bubblewrap?tab=readme-ov-file#related-project-comparison-firejail),
so they rolled their own
[Bubblewrap](https://github.com/containers/bubblewrap).
It's a reasnobaly simple piece of software
([5k SLOC of C, some 1000 lines of Shell](https://ghloc.dev/containers/bubblewrap))
that takes many command-line arguments
that can be used to adapt the sanbox environment to the needs of almost any program.
It has its bugs and issues,
but if you trust International Business Machines Inc. (est. 1911),
Bubblewrap is a reasonable choice.


### MacOS

On macOS, [`sandbox-exec`](https://igorstechnoclub.com/sandbox-exec/) utility exists
but is considered deprecated, with most fingers pointing towards
Apple Containerization®, which has limitations and risks akin to
[Docker as above](#docker--podman).


### Sandbox-utils ✔️

A collection of portable Shell scripts that can be used to sandbox
programs and applications on modern Linuces:

* [`sandbox-run`](https://github.com/sandbox-utils/sandbox-run):
  run command in a secure OS sandbox.
  Provides container-like security (execution in separate Linux namespaces;
  seccomp filtering)
  using Linux-native commands
  [`unshare(1)`](https://manpages.debian.org/unstable/unshare),
  [`setpriv(1)`](https://manpages.debian.org/unstable/setpriv) and
  [`enosys(1)`](https://manpages.debian.org/unstable/enosys),
  with only some 150 ms of runtime overhead.
  No dependencies and just a few lines of Shell.
* [`sandbox-virt`](https://github.com/sandbox-utils/sandbox-virt):
  run command in a virtual machine sandbox.
  Uses QEMU/KVM and libvirt to quickly spin ad-hoc Linux VMs
  to run your untrusted programs in.
  Takes under 30 seconds to build a fresh VM and, once cached,
  only about two seconds to run/resume it.
  Easily extended / managed with
  [`virsh`](https://manpages.debian.org/unstable/virsh) (CLI) or
  [`virt-manager`](https://manpages.debian.org/unstable/virt-manager) (GUI).
* [`sandbox-venv`](https://github.com/sandbox-utils/sandbox-venv):
  secure container virtualenv wrapper. A simple wrapper for
  Python virtual environments that wraps
  `.venv/bin/python` and other binaries inside `.venv/bin` with
  `sandbox-run`.


Contributing
------------
This namespace is open to contributions.
If you'd like to contrib + maintain useful open-source app
sandboxing / security / observability scripts
for a particular, maybe hitherto non-covered platform, the simple rule is:
start with POSIX Shell and continue from there. Get in touch. :+1:
