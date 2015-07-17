> :warning: This page is a guide for folks working directly on the YCSB project. It is unlikely to be of use if you are a downstream user of YCSB. That said, we love it when downstream users decide to pitch in on the project. Let us know if part of this could be clearer.

This document is meant to provide guidance for collaborator who have taken on the responsibility of generating a release.

In order, we're going to cover

* [Assumptions](#assumptions) about the state of your permissions and local setup
* [Set Release Gates](#set-release-gates) to establish at the start what bar of completeness the release will meet
* [Create Release Issue](#create-release-issue) so there's a single focal point for vetting the release
* [Create Release Staging Branch](#create-release-staging-branch) to isolate polishing the release from mainline development
* [Generate a Release Candidate](#generate-a-release-candidate) explains how to make artifacts folks can use for testing
* [Collect Test Feedback](#collect-test-feedback) to make sure that fixes to problems that show up in testing the release make it into future release candidates and ultimately the release
* [Stage Release Notes](#stage-release-notes) to make sure that we give our downstream users a useful guide to the work we've done

# Assumptions
This document has several assumptions built into it. We'll attempt to cover them briefly to reduce stumbling blocks.

* Release managers have 'collaborator' status on the [YCSB repo](https://github.com/brianfrankcooper/YCSB/)
* Git commands all start at the root of a local working copy of the YCSB repo
* The remote YCSB repo is named 'origin' in your local working copy
* The in-progress release you are managing is referred to as _version_; you should always replace this with an actual version string i.e. 0.3.0. In bash examples, this version will be referred to with the placeholder ```$VERSION```
* Your release version was determined prior to this process inline with [our versioning policy](https://github.com/brianfrankcooper/YCSB/wiki/Version-Numbers-and-Compatibility)
* Your local development environment is correct
  * We presume Linux for release builds
  * We require JDK 7 for release builds
  * We require Maven 3.2+ for release builds

# Set Release Gates

Before starting the release process, you should have a set of goals for the release to meet. At a minimum you should maintain some minimum verified code quality:

* the generated binary artifacts should run on both Linux and Windows ([presuming a valid environment](https://github.com/brianfrankcooper/YCSB/wiki/Prerequisites-for-Windows))
* at least three of the datastore bindings should be verified as working
* someone needs to run ```mvn apache-rat:check``` to verify licensing

Additionally, you should review issues/PRs that have been closed since the last release. If any of them include major restructuring or new functionality you should consider requiring that they are tested before a release can happen. Drawing the line between required testing and simply labeling a feature as experimental is a judgement call. Changes to how we measure throughput or latency, especially by default, should certainly require testing. An additional export format for measurements has relatively little risk for downstream users if it is labeled experimental.

# Create Release Issue

Once you know what you'd like covered in the release, [create a new issue](https://github.com/brianfrankcooper/YCSB/issues/new) with the title "Release version _version_". The conversation on this issue will serve as the central hub of activity related to the release: you'll post links here to release candidates as they're created and testers will post their results.

It's important to stay responsive on this thread. If an issue is discovered that you believe needs to be in the final release, provide a link to it and note that it is a blocker. If you think an issue can be sufficiently mitigated try to get consensus on having it listed as a known issue instead.

On the initial issue description, you should include your expected release gating criteria and a brief timeline. Essentially, let folks know when you will create the release staging branch and when you'd like to see all of the testing completed.

# Create Release Staging Branch

Releases are driven out of a dedicated branch so that unrelated development can continue in the master branch. The release staging branch marks _feature freeze_ for the release; no additional improvements or enhancements should go into the branch. One important exception to this rule is the addition of a README.md for datastore bindings. While documentation changes would not normally be considered an improvement, this particular document causes our build to generate a convenience binary specific to that datastore. You should accept (and encourage!) these changes in a release branch, especially if they are paired with test results.

Creating a release staging branch involves a couple of git commands and updating version numbers. The next version number will generally be the next _minor_ release number (for more info see [our versioning policy](https://github.com/brianfrankcooper/YCSB/wiki/Version-Numbers-and-Compatibility)).


```bash
  $ git checkout master
  $ # We presume a clean working directory
  $ git status
# On branch master
nothing to commit (working directory clean)
  $ git checkout -b "${VERSION}-staging"
Switched to a new branch '${VERSION}-staging'
  $ git checkout master
  $ # you'll need to update the version number across master
  $ # list of files
  $ grep -rl "${VERSION}-SNAPSHOT" *
  $ # e.g. edit in vim and replace with the next -SNAPSHOT version
  $ vim `grep -rl "${VERSION}-SNAPSHOT" *`
23 files to edit
  $ git add -p
  $ git commit -m "[version] update master to ${NEXT_VERSION}-SNAPSHOT."
  $ git push origin ${VERSION}-staging master
```

If any changes are merged into master before you attempt to push the update version numbers you'll receive an error like this:

```bash
  $ git push origin _version_-staging master
To git@github.com:brianfrankcooper/YCSB.git
 ! [rejected]        master -> master (non-fast-forward)
error: failed to push some refs to 'git@github.com:brianfrankcooper/YCSB.git'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. Merge the remote changes (e.g. 'git pull')
hint: before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```

To fix this, you should fetch new changes and rebase your master branch before pushing.

```bash
  $ git fetch origin
remote: Counting objects: 126, done.
remote: Compressing objects: 100% (39/39), done.
remote: Total 126 (delta 33), reused 19 (delta 19), pack-reused 42
Receiving objects: 100% (126/126), 20.27 KiB, done.
Resolving deltas: 100% (33/33), completed with 5 local objects.
From github.com:brianfrankcooper/YCSB
   35720bb..2c83327  master     -> origin/master
  $ git rebase origin/master
First, rewinding head to replay your work on top of it...
Applying: [version] update master to ${NEXT_VERSION}-SNAPSHOT.
  $ git push origin master
```
Note that these additional changes will not be present in your release staging branch. If you wish to include them, delete the branch and start over.

Once you create the release staging branch it will be up to you to ensure any changes you want included in the release are cherry-picked back to this branch. We'll cover the mechanics of that later in [Collect Test Feedback](#collect-test-feedback).

# Generate a Release Candidate

Testing in the release conversation should always be done on a release candidate so that everyone is operating on the same version of the code. An accessible release candidate reduces the burden on the volunteers we ask to do our release vetting. At the same time, we don't want downstream users relying on these candidates since they have not, by definition, been tested yet. To achieve a balance between these two opposing goals we make use of GitHub's pre-release markings. Additionally, we don't create any convenience binaries for release candidates.

Before tagging a release candidate, you will need to update the version in the repo. In this example, we'll use the placeholder ```${RC_NUMBER}``` for the current release candidate, i.e. ```RC_NUMBER="RC1"``` for the first release candidate.

```bash
  $ git checkout "${VERSION}-staging"
  $ # We presume a clean working directory
  $ git status
# On branch ${VERSION}-staging
nothing to commit (working directory clean)
  $ grep -rl "-SNAPSHOT" *
  $ # e.g. edit in vim and replace -SNAPSHOT versions with one for the RC
  $ # on the first RC, this will be replacing ${VERSION}-SNAPSHOT with ${VERSION}-RC1
  $ # on subsequent RCs it will be ${VERSION}-${RC_NUMBER}-SNAPSHOT with ${VERSION}-${RC_NUMBER}
  $ vim `grep -rl "-SNAPSHOT" *`
23 files to edit
  $ git add -p
  $ git commit -m "[release] mark ${VERSION} ${RC_NUMBER}"
  $ # Now create a tag that we can reference on github
  $ git tag "${VERSION}-${RC_NUMBER}"
  $ # Now update the versions so that the repo will be -SNAPSHOT for the next candidate.
  $ # that way fixes that come in during testing can be picked back here now
  $ # files that need updating
  $ grep -rl "${VERSION}-${RC_NUMBER}"
  $ # e.g edit in vim and replace ${VERSION}-RC1 with ${VERSION}-RC2-SNAPSHOT
  $ vim `grep -rl "${VERSION}-${RC_NUMBER}"`
23 files to edit
  $ git add -p
  $ git commit -m "[release] update version for ${NEXT_RC_NUMBER}"
  $ # finally push all of this up to GitHub
  $ git push origin ${VERSION}-staging ${VERSION}-${RC_NUMBER}
```

Now the YCSB repo should have all of your changes available on GitHub.

1. Visit the ["Draft new release" page](https://github.com/brianfrankcooper/YCSB/releases/new)
2. In the "Tag version" field, enter the tag you created above, ```${VERSION}-${RC_NUMBER}```
3. Ensure the @ Target branch is the staging branch from above, ```${VERSION}-staging```
4. Fill in a title of "_version_ Release Candidate _rc-number_", e.g. "0.3.0 Release Candidate 1"
5. Fill in the description. Give folks a pointer to the release issue. Be sure to include a warning that this is an RC and not for downstream use. If this is the first RC, it will help your later release notes if you can describe what's different since the last release. If this is a follow on RC, you should include what has changed since the last RC.
6. Be sure to check the "this is a pre-release" box.

Here's an example template for a Release Candidate description

>  UPPERCASE WARNING: THIS IS A RELEASE CANDIDATE AND IS NOT INTENDED FOR DOWNSTREAM USE.
> 
> This source is a candidate for release _version_. Please see #_release issue_ for testing instructions and where to leave your feedback.
>
> Compared to the previous release, this candidate includes
> * some list
> * of changes
>

When you've published a particular release candidate, you should add a brief note to the release issue pointing to it and asking folks who are testing to use it from then on.

# Collect Testing Feedback

As folks work to verify a release candidate, they'll open PRs for problems they encounter and hopefully those PRs will result in fixes on the master branch. As the Release Manager, you'll need to ensure that any of these fixes needed for the release make it into the staging branch. You accomplish this by using the ```git cherry-pick``` command to apply individual patches from the master branch on the release staging branch.

As a simple example, consider the fix for #319 in the 0.2.0 release line. This was a problem with the scan operation in the Tarantool datastore binding that was discovered between RC2 and RC3.

Here's the fix landing in master:

```
*   ecadf1f - (HEAD, master) Merge pull request #323 from bigbes/master (3 weeks ago)
|\  
| * 25e0ec8 - Issue #319: [tarantool] Error on scan operation and replace problems (3 weeks ago)
|/  
*   45631e8 - Merge pull request #304 from gkamat/issue-278 (3 weeks ago)
```

And here's how master compared to the 0.2.0-staging branch at the time:

```
* f2e333b - (0.2.0-staging) [hbase] Update to commit 80b74a94359179a60032289e5a115dec1bf6ee8b. Pass original exception back when issues discovered during initialization. (3 weeks ago)
* 1e437eb - [hbase] Verify that the table exists during DB binding initialization (3 weeks ago)
* 31b86d4 - [mongodb] update parameter for connection url to match previous behavior and docs. (3 weeks ago)
* f3aac41 - [release] update version for RC3 (4 weeks ago)
* dde1fb8 - (0.2.0-RC2) [release] mark 0.2.0 RC2 (4 weeks ago)
* 643f007 - [scripts] Include only the binding-specific jars and conf directory in the classpath, rather than all of them. (4 weeks ago)
* dc794ca - [scripts] Add only the conf directory for the relevant binding to the classpath, rather than all of them. (4 weeks ago)
* fb9c41a - [release] update version for RC2 (4 weeks ago)
* 7bc13d8 - (0.2.0-RC1) [release] mark 0.2.0 RC1 (4 weeks ago)
| *   ecadf1f - (HEAD, master) Merge pull request #323 from bigbes/master (3 weeks ago)
| |\  
| | * 25e0ec8 - Issue #319: [tarantool] Error on scan operation and replace problems (3 weeks ago)
| |/  
| *   45631e8 - Merge pull request #304 from gkamat/issue-278 (3 weeks ago)
| |\  
| | * 1c3db41 - [hbase] Update to commit 80b74a94359179a60032289e5a115dec1bf6ee8b. Pass original exception back when issues discovered during initialization. (4 weeks ago)
| | *   6d85b13 - Merge remote-tracking branch 'upstream/master' into issue-278 (4 weeks ago)
| | |\  
| | * \   2517e26 - Merge remote-tracking branch 'upstream/master' into issue-278 (4 weeks ago)
| | |\ \  
| |_|/ /  
|/| | |   
| | * |   36194ed - Merge branch 'issue-278' of https://github.com/gkamat/YCSB into issue-278 (4 weeks ago)
| | |\ \  
| | | * | d59ce9b - [hbase] Verify that the table exists during DB binding initialization and bail out if not found.  If this is not checked here, the workload will continue running and generating errors 
| | * | | 80b74a9 - [hbase] Verify that the table exists during DB binding initialization and bail out if not found.  If this is not checked here, the workload will continue running and generating errors 
| | |/ /  
| * | |   9bc02d2 - Merge pull request #301 from saggarsunil/master (3 weeks ago)
| |\ \ \  
| | * \ \   ba66be6 - Merge branch 'master' of https://github.com/saggarsunil/YCSB (4 weeks ago)
| | |\ \ \  
| | | * \ \   c4e3943 - Merge branch 'master' of https://github.com/saggarsunil/YCSB (4 weeks ago)
| | | |\ \ \  
| | * | \ \ \   ea22a7b - Merge branch 'master' of https://github.com/saggarsunil/YCSB (4 weeks ago)
| | |\ \ \ \ \  
| | | |/ / / /  
| | |/| / / /   
| | | |/ / /    
| | | * | | 42b39f5 - Changes: 1. mongodb configuration parameter bug (mongodb.url) 2. README file change 3. Some cleanup. Default DB is 'ycsb' (4 weeks ago)
| | * | | | 4cfd40f - [mongodb] update parameter for connection url to match previous behavior and docs. Changes: 1. mongodb configuration parameter bug (mongodb.url) 2. README file change 3. Some cleanup
| | |/ / /  
| * | | |   7dcbb32 - Merge pull request #305 from gkamat/issue_302 (4 weeks ago)
| |\ \ \ \  
| | |_|_|/  
| |/| | |   
| | * | | 2559ce6 - [scripts] Include only the binding-specific jars and conf directory in the classpath, rather than all of them. (4 weeks ago)
| | * | |   be6c4d9 - Merge branch 'master' of https://github.com/brianfrankcooper/YCSB into issue_302 (4 weeks ago)
| | |\ \ \  
| | * | | | ced4306 - [scripts] Add only the conf directory for the relevant binding to the classpath, rather than all of them. (4 weeks ago)
| | | |_|/  
| | |/| |   
| * | | | 9c80044 - [version] update master to 0.3.0-SNAPSHOT. (4 weeks ago)
|/ / / /  
* | | |   700b71c - Merge pull request #307 from busbey/issue-297 (4 weeks ago)
```

To get this same fix in our release staging branch, we can ignore the merge and just use cherry-pick on the fix itself

```bash
  $ git cherry-pick 25e0ec8
[0.2.0-staging 7de528c] Issue #319: [tarantool] Error on scan operation and replace problems
 Author: bigbes <bigbes@gmail.com>
 1 file changed, 7 insertions(+), 5 deletions(-)
```

Now you can see the comparison of the branches after:

```
* 44330fb - (HEAD, 0.2.0-staging) Issue #319: [tarantool] Error on scan operation and replace problems (47 seconds ago)
* f2e333b - [hbase] Update to commit 80b74a94359179a60032289e5a115dec1bf6ee8b. Pass original exception back when issues discovered during initialization. (3 weeks ago)
* 1e437eb - [hbase] Verify that the table exists during DB binding initialization (3 weeks ago)
* 31b86d4 - [mongodb] update parameter for connection url to match previous behavior and docs. (3 weeks ago)
* f3aac41 - [release] update version for RC3 (4 weeks ago)
* dde1fb8 - (0.2.0-RC2) [release] mark 0.2.0 RC2 (4 weeks ago)
* 643f007 - [scripts] Include only the binding-specific jars and conf directory in the classpath, rather than all of them. (4 weeks ago)
* dc794ca - [scripts] Add only the conf directory for the relevant binding to the classpath, rather than all of them. (4 weeks ago)
* fb9c41a - [release] update version for RC2 (4 weeks ago)
* 7bc13d8 - (0.2.0-RC1) [release] mark 0.2.0 RC1 (4 weeks ago)
| *   ecadf1f - (master) Merge pull request #323 from bigbes/master (3 weeks ago)
| |\  
| | * 25e0ec8 - Issue #319: [tarantool] Error on scan operation and replace problems (3 weeks ago)
| |/  
| *   45631e8 - Merge pull request #304 from gkamat/issue-278 (3 weeks ago)
| |\  
| | * 1c3db41 - [hbase] Update to commit 80b74a94359179a60032289e5a115dec1bf6ee8b. Pass original exception back when issues discovered during initialization. (4 weeks ago)
| | *   6d85b13 - Merge remote-tracking branch 'upstream/master' into issue-278 (4 weeks ago)
| | |\  
| | * \   2517e26 - Merge remote-tracking branch 'upstream/master' into issue-278 (4 weeks ago)
| | |\ \  
| |_|/ /  
|/| | |   
| | * |   36194ed - Merge branch 'issue-278' of https://github.com/gkamat/YCSB into issue-278 (4 weeks ago)
| | |\ \  
| | | * | d59ce9b - [hbase] Verify that the table exists during DB binding initialization and bail out if not found.  If this is not checked here, the workload will continue running and generating errors 
| | * | | 80b74a9 - [hbase] Verify that the table exists during DB binding initialization and bail out if not found.  If this is not checked here, the workload will continue running and generating errors 
| | |/ /  
| * | |   9bc02d2 - Merge pull request #301 from saggarsunil/master (3 weeks ago)
| |\ \ \  
| | * \ \   ba66be6 - Merge branch 'master' of https://github.com/saggarsunil/YCSB (4 weeks ago)
| | |\ \ \  
| | | * \ \   c4e3943 - Merge branch 'master' of https://github.com/saggarsunil/YCSB (4 weeks ago)
| | | |\ \ \  
| | * | \ \ \   ea22a7b - Merge branch 'master' of https://github.com/saggarsunil/YCSB (4 weeks ago)
| | |\ \ \ \ \  
| | | |/ / / /  
| | |/| / / /   
| | | |/ / /    
| | | * | | 42b39f5 - Changes: 1. mongodb configuration parameter bug (mongodb.url) 2. README file change 3. Some cleanup. Default DB is 'ycsb' (4 weeks ago)
| | * | | | 4cfd40f - [mongodb] update parameter for connection url to match previous behavior and docs. Changes: 1. mongodb configuration parameter bug (mongodb.url) 2. README file change 3. Some cleanup
| | |/ / /  
| * | | |   7dcbb32 - Merge pull request #305 from gkamat/issue_302 (4 weeks ago)
| |\ \ \ \  
| | |_|_|/  
| |/| | |   
| | * | | 2559ce6 - [scripts] Include only the binding-specific jars and conf directory in the classpath, rather than all of them. (4 weeks ago)
| | * | |   be6c4d9 - Merge branch 'master' of https://github.com/brianfrankcooper/YCSB into issue_302 (4 weeks ago)
| | |\ \ \  
| | * | | | ced4306 - [scripts] Add only the conf directory for the relevant binding to the classpath, rather than all of them. (4 weeks ago)
| | | |_|/  
| | |/| |   
| * | | | 9c80044 - [version] update master to 0.3.0-SNAPSHOT. (4 weeks ago)
|/ / / /  
* | | |   700b71c - Merge pull request #307 from busbey/issue-297 (4 weeks ago)
```

Since the release staging branch doesn't maintain the git history for the original fix, it's important to make sure the commit message is good. If the above fix hadn't included an issue number, for example, then we can edit the commit message while cherry-picking to improve it.

```bash
  $ git cherry-pick --edit 25e0ec8
```

An example of a good commit message follows.

TODO

In some cases, you'll need to cherry pick multiple commits. The main caveat in that case is to make sure you pick them in the same order they were originally applied (bottom up if you're looking at the git log).

If the fix you need came from a feature branch with multiple merges, or if any of those merges involved substantive changes you'll need to use some more advanced git features. Ask for help on the release issue and someone will help out. (and hopefully update this document!)

For more information on git cherry-pick, check out [the cherry-pick section of Think Like a Git](http://think-like-a-git.net/sections/rebase-from-the-ground-up/cherry-picking-explained.html).

# Stage Release Notes

Before ultimately publishing the release you will need to write up a short document that lets downstream users know what they ought to watch out for when upgrading. This is an important chance to minimize the headache (and our future support!) by warning them of incompatibilities or known problems.

Things to make sure you include:

* Incompatible changes, especially if a datastore binding was removed, we changed how the tool is run, or comparative analysis with prior versions is invalid.
* Known limitations of the current datastore bindings with particular datastore versions, particular workloads, or on particular operating systems.
* Datastore bindings. Call out new ones, but be sure to list _every_ datastore binding that is present in the binary distribution. Those that were tested in the release thread should be called out at such, and should have links to their project pages. Those that weren't tested should still be listed, but the lack of testing should be noted and no links made.

Things to leave out:

* Changes that are only internal to the project, e.g. build tooling that doesn't impact the final binary artifacts

For examples of this kind of writing, check out some of our prior releases:

* [Version 0.2.0](https://github.com/brianfrankcooper/YCSB/releases/tag/0.2.0)

Release notes are ultimately posted in Markdown. To create a draft, you can use [GitHub Gists](https://gist.github.com/) or any other pastebin. It's best if you can mark them "private" or "secret"—essentially anything that makes the draft only visible to those with the link. This should help cut down on searchers accidentally ending up at the draft prior to the final release.

Once you have a draft, post it to the release thread and wait for someone to give it a :thumbsup:.

# Publish Release

Once you have consensus that the release is ready and an appropriate set of release notes, it's time to publish the release. This process will involve one last round of version updates, building the convenience artifacts, and using GitHub's release publishing feature.

## Finalize Version

## Build Convenience Artifacts

## Draft Release

TODO link to page, explanation of tag preference, what artifacts to include

# Announce Release

You should let interested communities know that a new version is available. In particular, we try to make sure any of the communities with datastore bindings in the release know.

TODO add template

It's usually helpful to tailor the template to the particular community, especially if you're notifying a datastore community and theirs is in the 'experimental' camp. For example:

TODO add template w/call out

Here's a list of communities we have previously announced our releases in (TODO links):

* user@hbase
* user@cassandra
* infinispan forum
* tarantool group
* user@accumulo
* mongodb group
* couchbase group
* elasticsearch forum
* dev@geode
* hypertable group
* orientdb group
* redis group
* user@phoenix

# Clean up

Once the release is out, there are a few final steps to take care of. 

* Remove staging branch in git``` $ git push origin :_version_-staging```
* Close your release issue, include a message that points to the published release
* Create a PR to update the top-level README.md to reference downloading the new version
* Update the [top-level wiki page](https://github.com/brianfrankcooper/YCSB/wiki) to reference downloading the new version
* Add your release issue to the [examples for future release managers](#Examples)

# Examples

Below are prior release issues, so you can get a better idea of timing and the kinds of status updates that have been useful before.

* Version 0.2.0 - RM Sean Busbey - #266
