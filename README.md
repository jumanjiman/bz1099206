# Reproducer for BZ1099206

Best workaround: Install `gcc` *before* golang in a separate yum transaction.

Related BZ's:

* original: https://bugzilla.redhat.com/show_bug.cgi?id=1099206
* primary:  https://bugzilla.redhat.com/show_bug.cgi?id=1105901
* related?: https://bugzilla.redhat.com/show_bug.cgi?id=1099732

See the results of this test suite at
https://app.wercker.com/#applications/5394ce3e85147b684f0c91a4

The best workaround is in the f20_with_gcc image.

On a fresh installation of Fedora 20 with `golang-1.2.2-7.fc20.x86_64`
or Fedora Rawhide with `golang-1.3rc1-1.fc21.x86_64`,
`go get` errors out with:

    go install runtime/cgo: open /usr/lib/golang/pkg/linux_amd64/runtime/cgo.a: permission denied

Also, the initial installation shows this error:

    Running transaction
      Installing : golang-src-1.2.2-7.fc20.noarch                              1/52
      Installing : golang-pkg-bin-linux-amd64-1.2.2-7.fc20.x86_64              2/52
      Installing : golang-1.2.2-7.fc20.x86_64                                  3/52
      Installing : golang-pkg-linux-amd64-1.2.2-7.fc20.noarch                  4/52
    # runtime/cgo
    exec: "gcc": executable file not found in $PATH
    warning: %post(golang-pkg-linux-amd64-1.2.2-7.fc20.noarch) scriptlet failed, exit status 2
    Non-fatal POSTIN scriptlet failure in rpm package golang-pkg-linux-amd64-1.2.2-7.fc20.noarch
      Installing : mpfr-3.1.2-4.fc20.x86_64                                    5/52

If you run `yum -y reinstall golang golang-pkg* golang-src`,
the `go get` command finishes successfully.
One should not have to reinstall the packages to get a working set.

A better workaround is to install `gcc` in a separate yum transaction
*before* `golang`. See the f20_with_gcc portion of this reproducer.

To reproduce the behavior, clone this repo [1] and run
as a user with privileges to run `docker`:

    script/bootstrap
    script/test

`script/bootstrap` builds five docker images:

* f20
* f20_with_reinstall
* f20_with_gcc
* rawhide
* rawhide_with_reinstall

`script/test` runs the `go get` command to demonstrate the error:

```
===> Running gotest in f20...
[INFO] docker run --rm -i -t -u user f20 /gotest
[INFO] rpm -qa golang*
golang-pkg-bin-linux-amd64-1.2.2-7.fc20.x86_64
golang-pkg-linux-amd64-1.2.2-7.fc20.noarch
golang-src-1.2.2-7.fc20.noarch
golang-1.2.2-7.fc20.x86_64
[INFO] go get github.com/golang/lint/golint
[INFO] go get github.com/rakyll/drive
go install runtime/cgo: open /usr/lib/golang/pkg/linux_amd64/runtime/cgo.a: permission denied
[INFO] go get github.com/epeli/hooktftp
go install runtime/cgo: open /usr/lib/golang/pkg/linux_amd64/runtime/cgo.a: permission denied

===> Running gotest in f20_with_reinstall...
[INFO] docker run --rm -i -t -u user f20_with_reinstall /gotest
[INFO] rpm -qa golang*
golang-pkg-bin-linux-amd64-1.2.2-7.fc20.x86_64
golang-pkg-linux-amd64-1.2.2-7.fc20.noarch
golang-src-1.2.2-7.fc20.noarch
golang-1.2.2-7.fc20.x86_64
[INFO] go get github.com/golang/lint/golint
[INFO] go get github.com/rakyll/drive
[INFO] go get github.com/epeli/hooktftp

===> Running gotest in f20_with_gcc...
[INFO] docker run --rm -i -t -u user f20_with_gcc /gotest
[INFO] rpm -qa golang*
golang-src-1.2.2-7.fc20.noarch
golang-1.2.2-7.fc20.x86_64
golang-pkg-bin-linux-amd64-1.2.2-7.fc20.x86_64
golang-pkg-linux-amd64-1.2.2-7.fc20.noarch
[INFO] go get github.com/golang/lint/golint
[INFO] go get github.com/rakyll/drive
[INFO] go get github.com/epeli/hooktftp

===> Running gotest in rawhide...
[INFO] docker run --rm -i -t -u user rawhide /gotest
[INFO] rpm -qa golang*
golang-1.3rc1-1.fc21.x86_64
golang-pkg-linux-amd64-1.3rc1-1.fc21.noarch
golang-src-1.3rc1-1.fc21.noarch
golang-pkg-bin-linux-amd64-1.3rc1-1.fc21.x86_64
[INFO] go get github.com/golang/lint/golint
[INFO] go get github.com/rakyll/drive
go install runtime/cgo: open /usr/lib/golang/pkg/linux_amd64/runtime/cgo.a: permission denied
[INFO] go get github.com/epeli/hooktftp
go install runtime/cgo: open /usr/lib/golang/pkg/linux_amd64/runtime/cgo.a: permission denied

===> Running gotest in rawhide_with_reinstall...
[INFO] docker run --rm -i -t -u user rawhide_with_reinstall /gotest
[INFO] rpm -qa golang*
golang-src-1.3rc1-1.fc21.noarch
golang-pkg-bin-linux-amd64-1.3rc1-1.fc21.x86_64
golang-1.3rc1-1.fc21.x86_64
golang-pkg-linux-amd64-1.3rc1-1.fc21.noarch
[INFO] go get github.com/golang/lint/golint
[INFO] go get github.com/rakyll/drive
[INFO] go get github.com/epeli/hooktftp
```

[1] https://github.com/jumanjiman/bz1099206
