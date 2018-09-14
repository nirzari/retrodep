Backvendor
==========

This command inspects a Go source tree with vendored packages and attempts to work out the versions of the packages which are vendored, as well as the version of top-level package itself.

It does this by comparing file hashes of the packages with those from the upstream repositories.

If no semantic version tag matches but a commit is found that matches, a pseudo-version is generated.

Installation
------------

```
go get github.com/release-engineering/backvendor
```

Running
-------

```
$ backvendor -h
backvendor: help requested
usage: backvendor [OPTION]... PATH
  -debug
        show debugging output
  -deps
        show vendored dependencies (default true)
  -exclude-from string
        ignore glob patterns listed in provided file
  -help
        print help
  -importpath string
        top-level import path
```

In many cases backvendor can work out the import path for the top-level project. In those cases, simply supply the directory name to examine:
```
$ backvendor src
```

If it cannot determine the import path, provide it with -importpath:
```
$ backvendor -importpath github.com/example/name src
```

By default both the top-level project and its vendored dependencies are examined. To ignore vendored dependencies supply -deps=false:
```
$ backvendor -deps=false -importpath github.com/example/name src
```

If there are additional local files not expected to be part of the upstream version they can be excluded:
```
$ cat exclusions
.git
Dockerfile
$ ls -d src/Dockerfile src/.git
src/Dockerfile
src/.git
$ backvendor -exclude-from=exclusions src
```

Example output
--------------

```
$ backvendor -importpath github.com/docker/distribution go/src/github.com/docker/distribution
*github.com/docker/distribution@90705d2fb81dda1466be49bd958ed8a0dd9a6145 ~v2.6.0rc.1-1.20180831002537-90705d2fb81d
github.com/opencontainers/image-spec@87998cd070d9e7a2c79f8b153a26bea0425582e5 =v1.0.0 ~v1.0.0
github.com/ncw/swift@b964f2ca856aac39885e258ad25aec08d5f64ee6 ~v1.0.25-0.20160617142549-b964f2ca856a
golang.org/x/oauth2@2897dcade18a126645f1368de827f1e613a60049 ~v0.0.0-0.20160323192119-2897dcade18a
rsc.io/letsencrypt ?
...
```

In this example,

* github.com/docker/distribution is the top-level repository (indicated by \*)
* github.com/opencontainers/image-spec had a matching semantic version tag (v1.0.0)
* github.com/ncw/swift's matching commit had semantic version tag v1.0.24 reachable (so the next patch version would be v1.0.25)
* golang.org/x/oauth2 had a matching commit but no reachable semantic version tag
* rsc.io/letsencrypt had no matching commit (because the vendored source was from a fork)
* for github.com/docker/distribution, tag v2.6.0rc.1 was reachable from the matching commit (note, 1.timestamp instead of 0.timestamp)


Pseudo-versions
---------------

The pseudo-versions generated by this tool are:

* v0.0.0-0.yyyyddmmhhmmss-abcdefabcdef (commit with no relative tag)
* vX.Y.Z-pre.0.yyyyddmmhhmmss-abcdefabcdef (commit after semver vX.Y.Z-pre)
* vX.Y.(Z+1)-0.yyyyddmmhhmmss-abcdefabcdef (commit after semver vX.Y.Z)
* tag-1.yyyyddmmhhmmss-abcdefabcdef (commit after tag)

Limitations
-----------

The vendor directory is assumed to be complete.

Original source code is assumed to be available.

Only git repositories are currently supported, and a working 'git' is assumed to be available.

Non-Go code is not considered, e.g. binary-only packages, or CGo.

Commits with additional files (e.g. \*\_linux.go) are identified as matching when they should not.

Packages vendored from forks will not have matching commits.

Files marked as "export-subst" in .gitattributes files in the vendored copy are ignored.
