# Dæmon git tools

This is a set of git tools purposed to ease the management of repositories of the [Dæmon game engine](https://github.com/DaemonEngine/Daemon) or Dæmon-based games like the [Unvanquished game](https://github.com/Unvanquished/Unvanquished) and [Unvanquished game data](https://github.com/UnvanquishedAssets/UnvanquishedAssets) repositories with their sets of submodules.


## Distribution

Author: Thomas _“illwieckz”_ Debesse <hidden email="dev [@] illwieckz.net"/>

License: Those tools are distributed under the highly permissive [ISC](COPYING.md) license.


## setenv

This is a simple helper to add all those scripts to your current `PATH`, just do that:

```sh
eval "$(setenv)"
```


## git-checkout-web

The [`git-checkout-web`](git-checkout-web) helper is a git wrapper to help people checking out pull or merge requests using their web url.


### Real life examples

Checkout the current `HEAD` of the `#1597` submodule of the Dæmon engine, without having to add any upstream, and without having to care about it having-been force-pushed since the last time it was checkout out.

```sh
git-checkout-web https://github.com/DaemonEngine/Daemon/pull/1597
```


## git-commit-modules

The [`git-commit-modules`](git-commit-modules) helper is a git wrapper to help people committing submodules.

Its first purpose is to frequently commit the submodules references of repositories with a large and recursive modules tree.


### Real life examples

Checkout and pull the master branch of every modules of the Unvanquished repository, then commit them, then push them:

```sh
git-commit-modules --yes --branch=master Unvanquished/
```

If there is a merge conflict, the execution will stop and open a shell for the user to fix the merge conflict. The merge conflicts must be solved by the user on a per-case basis. Once the user exits the shell after fixing conflicts, the execution continue.

Checkout and pull the for-0.56.0/sync branch of every modules of the Unvanquished repository, merge the master branch, then commit them, then push them:

```sh
git-commit-modules --yes --branch=master Unvanquished/
```

This will generate submodule merge conflicts most of the time, when this happen the execution will stop and open a shell for the user to fix it, the tool will provide instructions on how to fix the submodule merge conflicts. The non-submodules merge conflicts must be solved by the user on a per-case basis. Once the user exits the shell after fixing conflicts, the execution continue.

The `git-commit-modules` tool makes use of the `git-checkout-modules` tool.


## git-checkout-modules

The [`git-checkout-modules`](git-checkout-modules) helper is a git wrapper to help people to checkout the same branch name across submodules when such branch name exists.

Its first purpose is integration in CI tools for testing branches across multiple repositories.


### What it is for

You can add this command to your CI build script:

```sh
git-checkout-modules --detect-branch --branch-has='/sync$' --revert --print
```

And when the submitted CI job is working on detached head from a branch named like `name/sync`, the branch will be autodetected and all submodules having a branch named like this would be automatically checked out using this branch name.

This enables projects to sync submodules easily without commiting references when it's not desirable, like work-in-progress branches.

Since the data is stored in branch name,

- you can update submodules without having to commit their references to the parent module;
- you don't have to worry about to-be-deleted temporary references to be accidentally merged.

Since the keyword is a branch suffix, it integrates well with the `contributor/topic` branch name convention, so you would just name your branch `contributor/topic/sync`.


### Limitations

It cannot work across forks so contributors must push their work-in-progress branches to your upstream for every module shipping changes for the same branch name.


### What can be done with it

The tool can also be used for day-to-day usage, here are examples of things you can do with it:

```sh
# list all modules recursively:
```

```sh
# print current modules references
git-checkout-modules --print
```

```sh
# revert all submodules to registered references
git-checkout-modules --revert
```

```sh
# detect branch name for current commit and checkout current module with that branch name
git-checkout-modules --detect-branch
```

```sh
# checkout all submodules with the current branch name when possible
git-checkout-modules --current-branch
```

```sh
# if current branch name has pattern
# checkout all submodules with this branch name when possible
git-checkout-modules --current-branch:has='pattern'
```

```sh
# detect tag name for current commit and checkout current module with that tag name
git-checkout-modules --detect-branch
```

```sh
# checkout all submodules with the current tag name when possible
git-checkout-modules --current-tag
```

```sh
# if current tag name has pattern
# checkout all submodules with this branch name when possible
git-checkout-modules --current-tag:has='pattern'
```


```sh
# checkout all submodules with the given reference name when possible
# even if given reference name does not exist in current module
git-checkout-modules --ref='name'
```

```sh
# checkout all submodules with the given reference name when possible
# if this reference name has pattern
# even if given reference name does not exist in current module
git-checkout-modules --ref='name':has='pattern'
```

```sh
# checkout all submodules with the given branch name when possible
git-checkout-modules --sub-ref='name'
```

```sh
# checkout all modules with the given branch name when possible
# and given branch name exists in current module
git-checkout-modules 'name'
```

```
# print builtin help
git-checkout-modules --help
```

Actions can be done sequentially:

```sh
# checkout this branch name in all modules when possible
# then revert all references in all submodules
# then checkout this branch name in all modules when possible
git-checkout-modules master --revert feature
```

This means submodules would be checked out to `feature` branches or, when such branch does not exist in given module, keep the references that were registered in `master` and that were checked out first..

And of course, you can do what it is made for:

```sh
# if current branch name ends with /sync
# checkout all submodules with this branch name when possible
# or checkout other submodules with their registered references
# then print submodules references
git-checkout-modules --current-branch:has='/sync$' --revert --print
```


### Real life examples

List all modules recursively:

```sh
git-checkout-modules --list
```

```
Unvanquished
Unvanquished/daemon
Unvanquished/daemon/libs/breakpad
Unvanquished/daemon/libs/crunch
Unvanquished/daemon/libs/freetype
Unvanquished/daemon/libs/freetype/subprojects/dlg
Unvanquished/daemon/libs/googletest
Unvanquished/daemon/libs/pdcursesmod
Unvanquished/libs/glm
Unvanquished/libs/lua
Unvanquished/libs/recastnavigation
Unvanquished/libs/RmlUi

```

Checkout all modules using `unvanquished/0.51.1` tag when possible and print them:


```sh
git-checkout-modules 'unvanquished/0.51.1' --print
```

```
Checkout modules references with 'unvanquished/0.51.1'

Print modules references
MODULE                                     BRANCH  TAG                  REFERENCE
Unvanquished                               HEAD    unvanquished/0.51.1  2b70fb35ddfd7c6807fbdf2f0f968e3e52e69784
Unvanquished/src/utils/cbse                HEAD    unvanquished/0.51.1  80d36f22c16ed7a7a321a9f48d1e8e994b85d841
Unvanquished/libs/libRocket                HEAD    unvanquished/0.51.1  7f7c34e67dee17bf2c1cb3e61fe6d847698267a3
Unvanquished/daemon                        HEAD    unvanquished/0.51.1  a1731c3239850add83590ea1d04007b55135fb31
Unvanquished/daemon/libs/breakpad          HEAD    unvanquished/0.51.1  15fbc760aa1e4db2a3b36493ff3b4cf49e3df282
Unvanquished/daemon/libs/crunch            HEAD    unvanquished/0.51.1  559a1b045b50b5f716294b47325c0170c8236dbc
Unvanquished/daemon/libs/recastnavigation  HEAD    unvanquished/0.51.1  6b68934d6d2715501e01b1e115413cefaa0aa7d3
Unvanquished/pkg/unvanquished_src.dpkdir   HEAD    unvanquished/0.51.1  99a8ec6197c0d4c07368b552b35f8e5e004a9420
```

Print modules references

```sh
git-checkout-modules --print 2>/dev/null
```

```
MODULE                                     BRANCH  TAG                  REFERENCE
Unvanquished                               HEAD    unvanquished/0.51.1  2b70fb35ddfd7c6807fbdf2f0f968e3e52e69784
Unvanquished/src/utils/cbse                HEAD    unvanquished/0.51.1  80d36f22c16ed7a7a321a9f48d1e8e994b85d841
Unvanquished/libs/libRocket                HEAD    unvanquished/0.51.1  7f7c34e67dee17bf2c1cb3e61fe6d847698267a3
Unvanquished/daemon                        HEAD    unvanquished/0.51.1  a1731c3239850add83590ea1d04007b55135fb31
Unvanquished/daemon/libs/breakpad          HEAD    unvanquished/0.51.1  15fbc760aa1e4db2a3b36493ff3b4cf49e3df282
Unvanquished/daemon/libs/crunch            HEAD    unvanquished/0.51.1  559a1b045b50b5f716294b47325c0170c8236dbc
Unvanquished/daemon/libs/recastnavigation  HEAD    unvanquished/0.51.1  6b68934d6d2715501e01b1e115413cefaa0aa7d3
Unvanquished/pkg/unvanquished_src.dpkdir   HEAD    unvanquished/0.51.1  99a8ec6197c0d4c07368b552b35f8e5e004a9420
```

Checkout all modules using `responsive` branch when possible:

```sh
git-checkout-modules 'responsive' --print
```

```
Checkout modules references with 'responsive'

Print modules references
MODULE                                     BRANCH      TAG                  REFERENCE
Unvanquished                               responsive  undefined            81ef85396ecdf8b1fd97683869ee75d8f4cae983
Unvanquished/src/utils/cbse                HEAD        unvanquished/0.51.1  80d36f22c16ed7a7a321a9f48d1e8e994b85d841
Unvanquished/libs/libRocket                HEAD        unvanquished/0.51.1  7f7c34e67dee17bf2c1cb3e61fe6d847698267a3
Unvanquished/daemon                        responsive  undefined            ef4565c364f3b56e046560c05c7184fb3e954bef
Unvanquished/daemon/libs/breakpad          HEAD        unvanquished/0.51.1  15fbc760aa1e4db2a3b36493ff3b4cf49e3df282
Unvanquished/daemon/libs/crunch            HEAD        unvanquished/0.51.1  559a1b045b50b5f716294b47325c0170c8236dbc
Unvanquished/daemon/libs/recastnavigation  HEAD        unvanquished/0.51.1  6b68934d6d2715501e01b1e115413cefaa0aa7d3
Unvanquished/pkg/unvanquished_src.dpkdir   responsive  undefined            04cc39d9c67f3fbcaf9ba95464d2c206d2ff65d7
```

While current module is on branch `illwieckz/test/sync`, if that branch name ends with `/sync`, checkout all modules using this branch name when possible:

```sh
git checkout 'illwieckz/test/sync'
git checkout "$(git rev-parse HEAD)"
git-checkout-modules --detect-branch --current-branch:has='/sync$' --print
```

```
Checkout detected branch 'illwieckz/test/sync'
M	daemon
M	pkg/unvanquished_src.dpkdir
M	src/utils/cbse
Switched to branch 'illwieckz/test/sync'

Checkout modules with branch 'illwieckz/test/sync' because of pattern '/sync$'
Submodule path 'daemon': checked out '60af69e85f7a23726eb92f9ca89fbd111f1f700d'
Submodule path 'daemon/libs/crunch': checked out '85bab3d798a54abe32a22d5275e625ec06df6917'
Submodule path 'pkg/unvanquished_src.dpkdir': checked out '40459674e4c7daa03045a8a3b763925dd8d26294'
Submodule path 'src/utils/cbse': checked out 'e6a9e8d4805d8f5188c430d0449524d23796c573'

Print modules references
MODULE                                     BRANCH               TAG                  REFERENCE
Unvanquished                               illwieckz/test/sync  undefined            d2f75b56504fb13746d1f3c325cfe1c687626eea
Unvanquished/src/utils/cbse                HEAD                 undefined            e6a9e8d4805d8f5188c430d0449524d23796c573
Unvanquished/libs/libRocket                HEAD                 unvanquished/0.51.1  7f7c34e67dee17bf2c1cb3e61fe6d847698267a3
Unvanquished/daemon                        illwieckz/test/sync  undefined            218d88c03697d1a650e90de583e09b3e0a58e381
Unvanquished/daemon/libs/breakpad          HEAD                 unvanquished/0.51.1  15fbc760aa1e4db2a3b36493ff3b4cf49e3df282
Unvanquished/daemon/libs/crunch            illwieckz/test/sync  undefined            935295b78adb1ec8b1a05fae4164127dce8a1c5a
Unvanquished/daemon/libs/recastnavigation  HEAD                 unvanquished/0.51.1  6b68934d6d2715501e01b1e115413cefaa0aa7d3
Unvanquished/pkg/unvanquished_src.dpkdir   illwieckz/test/sync  undefined            6b9d10bdf2e8801802487c84349ba154b8ed30ff
```

Checkout current module to `master` then revert all submodules to registered references:

```sh
git-checkout-modules 'master' --revert --print
```

```
Checkout modules references with 'master'
Submodule path 'daemon': checked out '60af69e85f7a23726eb92f9ca89fbd111f1f700d'
Submodule path 'daemon/libs/crunch': checked out '85bab3d798a54abe32a22d5275e625ec06df6917'
Submodule path 'pkg/unvanquished_src.dpkdir': checked out '40459674e4c7daa03045a8a3b763925dd8d26294'

Revert modules references
Submodule path 'daemon': checked out '145ff5fe1aaa043b7366ea3860967f3a9add62f1'

Print modules references
MODULE                                     BRANCH  TAG                  REFERENCE
Unvanquished                               master  undefined            18803c9d3e43eab1c5fc502be54eb4268106ebab
Unvanquished/src/utils/cbse                master  unvanquished/0.51.1  80d36f22c16ed7a7a321a9f48d1e8e994b85d841
Unvanquished/libs/libRocket                master  unvanquished/0.51.1  7f7c34e67dee17bf2c1cb3e61fe6d847698267a3
Unvanquished/daemon                        HEAD    undefined            145ff5fe1aaa043b7366ea3860967f3a9add62f1
Unvanquished/daemon/libs/breakpad          master  unvanquished/0.51.1  15fbc760aa1e4db2a3b36493ff3b4cf49e3df282
Unvanquished/daemon/libs/crunch            master  unvanquished/0.51.1  559a1b045b50b5f716294b47325c0170c8236dbc
Unvanquished/daemon/libs/recastnavigation  master  unvanquished/0.51.1  6b68934d6d2715501e01b1e115413cefaa0aa7d3
Unvanquished/pkg/unvanquished_src.dpkdir   master  undefined            8f2e40b31c182f5e10d67f80252746eb3391ea8e
```

Attempt to checkout all modules with `parallax` branch, but this branch does not exist on current module:

```sh
git-checkout-modules 'parallax'
```
```
ERROR: Reference does not exist in current module: 'parallax'
```

Checkout all sub modules with `parallax` branch, even if this branch does not exist on current module:

```sh
git-checkout-modules --ref='parallax'
```
```
Checkout modules references with 'parallax'
```

Checkout all sub modules with `parallax` branch:

```sh
git-checkout-modules --sub-ref='parallax' --print
```
```
Checkout modules references with 'parallax'

Print modules references
MODULE                                     BRANCH    TAG                  REFERENCE
Unvanquished                               master    undefined            18803c9d3e43eab1c5fc502be54eb4268106ebab
Unvanquished/src/utils/cbse                master    unvanquished/0.51.1  80d36f22c16ed7a7a321a9f48d1e8e994b85d841
Unvanquished/libs/libRocket                master    unvanquished/0.51.1  7f7c34e67dee17bf2c1cb3e61fe6d847698267a3
Unvanquished/daemon                        parallax  undefined            218d88c03697d1a650e90de583e09b3e0a58e381
Unvanquished/daemon/libs/breakpad          master    unvanquished/0.51.1  15fbc760aa1e4db2a3b36493ff3b4cf49e3df282
Unvanquished/daemon/libs/crunch            master    unvanquished/0.51.1  559a1b045b50b5f716294b47325c0170c8236dbc
Unvanquished/daemon/libs/recastnavigation  master    unvanquished/0.51.1  6b68934d6d2715501e01b1e115413cefaa0aa7d3
Unvanquished/pkg/unvanquished_src.dpkdir   master    undefined            8f2e40b31c182f5e10d67f80252746eb3391ea8e
```
