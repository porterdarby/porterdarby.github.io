---
layout: page
title: Docker and git: describe and tags
tags: Docker, git, CI/CD
---

It's really quite impressive what you can learn by digging through some documentation. A while back I found out about [`git describe`](https://git-scm.com/docs/git-describe). This command gives "an object a human readable named based on an available ref." What does that mean? It basically means you can run `git describe` on a commit and it'll give you information back about the commit. For example:

```sh
$ git tag -a v1.0.0 -m "Release 1.0.0"
$ git describe --always
v1.0.0
$ ...
$ git commit -am "New Commit"
$ git describe
v1.0.0-1-gdfa8390
```

The above final command shows that I just ran `describe` on a commit that is after the [annotated tag](https://git-scm.com/book/en/v2/Git-Basics-Tagging) `v1.0.0`. How do I know? Well, it's the first section of the description. And I know this commit is 1 commit away from that tag because of the second section, and I know that the short hash is `gdfa8390`, because that's the third section. For the most part, when you run `git describe` you will end up with two different results:

```sh
$ git describe
v1.0.0-1-gdfa8390

OR 

$ git describe
v1.0.0
```

Basically, you will either get the tag that is currently on the commit or you will get the full tag description:

```sh
<tag>-<commits after tag>-<git short hash>
```

I use this to make life easier when tagging Docker images. Instead of tagging the image with the Jenkins build number, the git hash, or the timestamp, this allows me to know exactly what commit I am building off while still being quite readable. Now if I see a docker image like `company/project:1.3.5`, I know that's the 1.3.5 release of the project that's deployed. If I see `company/project:1.4.1-3-abc123d4`, then I know that this is 3 commit in on version 1.4.1 and can easily go check out what the codebase looks like at this point in time. If I'm cleaning up a Docker registry I don't need to hunt down what commits are connected to what releases, I can just delete any that aren't tagged with a release version and go from there.

The key here is to make sure your tag is annotated -- use the `-a` and `-m` flags when creating the tag to annotate the commit with a full tag. This will allow `git describe` to find it.

Because `git describe` will give you information about the commit and will give you both the tag as well as additional information if needed, it's easy to switch out in something like a Jenkinsfile -- just do:

```diff
-    IMAGE_TAG = env.GIT_HASH
+    IMAGE_TAG = sh(script: 'git describe --always', returnStdout: true).trim()
```

Easy, clean, one-line change, and it'll give you a way more understandable information about what you are actually building.

Oh, and use `--always`, since it "Show[s] uniquely abbreviated commit object as a fallback" instead of erroring out. This is useful if you don't have any tags in your repository yet, but will eventually. I find myself reusing this section of a Jenkinsfile in a large majority of my projects.