# Dependency management

cloud-provider-azure uses [go modules] for Go dependency management.

## Usage

Run [`hack/update-dependencies.sh`] whenever vendored dependencies change.
This takes a minute to complete.

### Updating dependencies

New dependencies causes golang to recompute the minor version used for each major version of each dependency. And
golang automatically removes dependencies that nothing imports any more.

To upgrade to the latest version for all direct and indirect dependencies of the current module:

* run `go get -u <package>` to use the latest minor or patch releases
* run `go get -u=patch <package>` to use the latest patch releases
* run `go get <pacakge>@VERSION` to use the specified version

You can also manually editing `go.mod` and update the versions in `require` and `replace` parts.

Because of staging in Kubernetes, manually `go.mod` updating is required for Kubernetes and
its staging packages. In cloud-provider-azure, their versions are set in `replace` part, e.g.

```go
replace (
    ...
    k8s.io/kubernetes => k8s.io/kubernetes v0.0.0-20190815230911-4e7fd98763aa
    k8s.io/legacy-cloud-providers => k8s.io/kubernetes/staging/src/k8s.io/legacy-cloud-providers v0.0.0-20190815230911-4e7fd98763aa
)
```

To update their versions, you need switch to `$GOPATH/src/k8s.io/kubernetes`, checkout to
the version you want upgrade to, and finally run the following commands to get the go modules expected version:

```sh
commit=$(git log --date=format:'%Y%m%d%H%M%S' -1 | head -n 1 | awk '{print $2}' | cut -c1-12)
date=$(git log --date=format:'%Y%m%d%H%M%S' -1 | awk '/Date:/{print $2}')
echo "v0.0.0-$date-$commit"
```

After this, replace all kubernetes and staging versions (e.g. `v0.0.0-20190815230911-4e7fd98763aa` in above example) in `go.mod`.

Always run `hack/update-dependencies.sh` after changing `go.mod` by any of these methods (or adding new imports).

See golang's [go.mod], [Using Go Modules] and [Kubernetes Go modules] docs for more details.


[go.mod]: https://github.com/golang/go/wiki/Modules#gomod
[go modules]: https://github.com/golang/go/wiki/Modules
[`hack/update-dependencies.sh`]: hack/update-dependencies.sh
[Using Go Modules]: https://blog.golang.org/using-go-modules
[[Kubernetes Go modules]: https://github.com/kubernetes/enhancements/blob/master/keps/sig-architecture/2019-03-19-go-modules.md
