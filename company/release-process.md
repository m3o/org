# Release Process

As we mature as an organisation we need to refine the release process to ensure that we can maintain a high velocity while also keeping code quality and stability high. This applies to both the OSS go-micro as much as any work on m3o or any other product.

The overarching approach should be to ensure that all changes undergo some form of extended testing period. Remember, the blast radius of errors in our code goes far beyond our own organisation so care must be taken.

## Background
We attempted using a [git-flow](https://nvie.com/posts/a-successful-git-branching-model/) model of branching with a `develop` and `master` branch for both `micro` and `go-micro`. However, Github does not support this model very well due to the way it tracks commits and diffs. See https://github.community/t/github-pull-requests-showing-invalid-diff-for-already-merged-branch/559. TLDR if you try to maintain two parallel branches you'll quickly end up in a position where diffs show, incorrectly, that there are missing commits. e.g. after a couple of releases if you do a compare between `develop` and `master` github will show you that there are differences between the two where in fact there isn't. Another downside of this approach was when accepting PRs from the community, outside contributors don't know to branch and PR against `develop`, rather, they do it against `master` and things very quickly get messy. 

The above points to the fact that it is easiest to maintain a single branch, `master`, and create feature branches off that i.e. using the [Github Flow](https://guides.github.com/introduction/flow/). This is what projects like kubernetes and terraform do so we're in good company. 

## micro

The `micro` project can be consumed as both tagged releases *and* directly from `master`. 

### Process
We will have 1 long lived branch; 
- `master` - reflects the latest and greatest development. All code is reviewed before being merged in to this branch

All feature branches will be branched from and merged back to `master` via the usual PR process. 

New release candidates will be cut from the `master` branch roughly every 2 weeks. New minor releases are cut from the `master` branch as `release-X.X`. From this point, only bug fixes should be added to the release branch. 

If all clear after 7 days we will promote the release candidate to full version and cut a new release which means
1. tagging the `release-X.X` branch with the new release version `vX.X.X`
2. merging the release branch to `master`
3. the `release-X.X` branch is *not* deleted. It will be retained for any future patch fixes that may be required.

#### Patching
Bug fixes should be made on the `release-X.X` branch, tagged, and then merged back to `master`

### Testing
Testing will include 
- building all of the example projects in https://github.com/micro/services
- running all integration tests in CI 
- running in non prod environment of m3o to shake out any bugs that require longer running

### Versioning
We follow the [semantic versioning](https://semver.org/) approach to release names. 

Just before we cut a new major release (v3, v4, etc) we should also create a branch of master to be maintained as the ongoing branch for the current release. This will allow us to provide bugfix support more easily for the release while the master branch moves on to the next major release. For example, before we start work on `v3` we should create a new branch from `master` as `v2`
