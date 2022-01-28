# git-workflow

Example repository of a fully automated [`git-flow` workflow](https://nvie.com/posts/a-successful-git-branching-model/). Using a set of tools and flows to follow. 

# What kind of project is a good fit for this workflow? 

This project is modeled after the [`git-flow` workflow](https://nvie.com/posts/a-successful-git-branching-model/) of releasing a software project. 

I am a fan of this type of workflow and I use it in all of my projects that I create. 

Quoting the `git-flow` doc...

Client-side apps are...
> apps that are typically continuously delivered, not rolled back, and you don't have to support multiple versions of the software running in the wild. 

> If, however, you are building software that is explicitly versioned, or if you need to support multiple versions of your software in the wild, then git-flow may still be as good of a fit to your team as it has been to people in the last 10 years. In that case, please read on.

# What do I need to run this flow? 

1. Git repository (this project assumes you're using GitHub). 
2. CI server (this project assumes you're using GitHub Actions). 

That's it! Well, I do recommend that you learn about each of the tools used in this repository and how to use them but I wanted to show that the list above is all of the tools that you need to be able to even consider this workflow. 

# Features

Following the `git-flow`, this project has...

### 2 permanent branches

...`main` and `develop`. 

> We consider `main` to be the main branch where the source code of HEAD always reflects a production-ready state.

> We consider `develop` to be the main branch where the source code of HEAD always reflects a state with the latest delivered development changes for the next release. Some would call this the “integration branch”. This is where any automatic nightly builds are built from.

### Lint pull requests to avoid mistakes hurting deployment workflow 

When you make a pull request, what branch you merge into matters. 

* Building a new feature? That should go in the `develop` branch to go into the next release. 
* Found a bug in the latest `beta` branch? Go ahead and make the PR merge into `beta`. 
* Found a critical bug in production (`main` branch)? Go ahead and make PR merge into `main` as a hotfix release. 

This project uses [a tool](https://github.com/levibostian/action-semantic-pr) (see `./.github/workflows/lint-helper.yml`) that checks to make sure that the change that you want to make in the pull request is appropriate. You can say that only bug fixes and docs updates are allowed to be merged directly into `beta`, for example. 

### Automated strict semantic versioning and updating metadata files 

When you need to make a new deployment of your software, usually it requires you to manually update some metadata about your project. Update the changelog, update the semantic version of some files, etc. It is tedious and prone to human error. 

This project uses a couple of tools to make this better for you. 

1. Every pull request that you make must have a title that conforms to [the conventional commits](https://www.conventionalcommits.org/en/v1.0.0/) format. For example, if the pull request is adding a new feature to the project, a pull request title of `feat: edit profile picture of profile` describes this change in a human and machine readable format. 
  * Pull requests are linted and merged by [a tool](https://github.com/levibostian/action-semantic-pr). This helps avoid merging in commits that are not in correct conventional commits format. 
2. These merged conventional commits are then read by [semantic-release](https://github.com/semantic-release/semantic-release/) tool (see `.github/workflows/make-git-tag.yml`) that automatically determines the next strict semantic version, updates changelog, updates any other metadata files that you need, makes a git tag release and pushes deployment commits to your git repository. 

### Pre-release deployments to `alpha`, `beta`, to production. 

Want to make releases of your software before they hit production? A common pattern is to release software in the flow:

```
development --> alpha --> beta --> production
```

When you want to make a deployment, all that you need to do is [manually run the "Promote a release" workflow](https://docs.github.com/en/actions/managing-workflow-runs/manually-running-a-workflow) in your GitHub repository. 


Let's say that you want to deploy `develop` to `alpha`. Open the URL `https://github.com/<username>/<name of repository>/actions/workflows/promote.yml` > Run workflow > Select `develop` branch > Run. 

[This tool](https://github.com/levibostian/action-promote-semantic-release) is configured (see `.github/workflows/promote.yml`) with the branch sequence `develop -> alpha -> beta -> main`. When you run the promote workflow from branch `beta` for example, see that the next branch in the sequence is `main`. So, the tool will merge `beta` into `main`, push up `main`, then delete the `beta` branch.  

### Merge in pre-release branches into `develop` after production deployments 

After your project finally merges code into the `main` branch for a production deployment, you need to merge those deployment commits into `develop`. [This tool](https://github.com/levibostian/action-sync-branches) will merge in commits from `main` into `develop` so that the two branches are identical (see `.github/workflows/make-git-tag.yml`). 

# List of tools used in this project 
 
1. https://github.com/levibostian/action-sync-branches - Copy commits from `main` into `develop` after a production release. 
2. https://github.com/levibostian/action-promote-semantic-release - Manages release branches (`main`) and pre-release branches (`alpha`, `beta`) allowing you to deploy releases of your software. 
3. https://github.com/levibostian/action-semantic-pr - Lints and merges pull requests to assert that they follow the [conventional commits](https://www.conventionalcommits.org/en/v1.0.0/) format. 
4. https://github.com/semantic-release/semantic-release/ - Automate updating of semantic versioning and metadata for a deployment. 

