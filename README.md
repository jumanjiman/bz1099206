# Reproducer for BZ1099206

https://bugzilla.redhat.com/show_bug.cgi?id=1099206

On a fresh installation of Fedora 20 with `golang-1.2.2-7.fc20.x86_64`
or Fedora Rawhide with `golang-1.3rc1-1.fc21.x86_64`,
`go get` errors out with:

    go install runtime/cgo: open /usr/lib/golang/pkg/linux_amd64/runtime/cgo.a: permission denied

If you run `yum -y reinstall golang golang-pkg* golang-src`,
the `go get` command finishes successfully.
One should not have to reinstall the packages to get a working set.

To reproduce the behavior, clone this repo [1] and run
as a user with privileges to run `docker`:

    script/bootstrap
    script/test

`script/bootstrap` builds four docker images:

* f20
* f20_with_reinstall
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
