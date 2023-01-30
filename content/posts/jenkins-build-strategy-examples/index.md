---
title: "Jenkins basic build strategy configuration examples"
date: 2023-01-30T10:00:00+01:00
draft: false
tags: 
  - ci
  - swe
  - jenkins
---

In this post I want to show you a configuration for the jenking plugin [basic-branch-build-strategies-plugin](https://github.com/jenkinsci/basic-branch-build-strategies-plugin). The main idea of this plugin is to configure, when a pipeline is triggered. 

## Requirements

For my configuration I wanted to have following setup:
1. Build pushes on any branch
2. Build pull requests
3. Build tags on update or creation
4. Don't build everything after a restart of jenkins

## Implementation

> Scroll down for the final example

In generell the plugin provides for each of the requirements above a strategy.

1. `buildNamedBranches`
2. `buildChangeRequests`
3. `buildTags`
4. `skipInitialBuildOnFirstBranchIndexing`

But the problem is that they need to be combined very carefully in order to make them work together.

In addition to the previously named strategies, the plugin provides container strategies that trigger only when all sub strategies match a specific result. Following container strategies are available:

- `buildAnyBranches`: Matches as soon as one of the sub strategies is matching
- `buildAllBranches`: Matches when all sub strategies are matching
- `buildNoneBranches`: Matches when no sub strategy is matching

The following strategy would match most of our requirements but not the last the one.

```
buildAnyBranches {
  stratagies {
    buildChangeRequests {
        ignoreTargetOnlyChanges(true)
        ignoreUntrustedChanges(false)
    }
    buildRegularBranches()
    buildTags {
        atLeastDays '-1'
        atMostDays '7'
    }
  }
}
```

In order to integrate `skipInitialBuildOnFirstBranchIndexing` with our existing configure we need to add a `buildAllBranches`. The whole block will then not match in the first build.

```
buildAllBranches {
  strategies {
    skipInitialBuildOnFirstBranchIndexing()
    buildAnyBranches {
      stratagies {
        buildChangeRequests {
            ignoreTargetOnlyChanges(true)
            ignoreUntrustedChanges(false)
        }
        buildRegularBranches()
        buildTags {
            atLeastDays '-1'
            atMostDays '7'
        }
      }
    }
  }
}
```

Now there is only one problem left. This configuration will not trigger when a new tag is created since `skipInitialBuildOnFirstBranchIndexing` will not match on the first creation. This can be solved by another `buildAnyBranches` block, which combines `skipInitialBuildOnFirstBranchIndexing` and `buildTags`

### Final example

```
buildAllBranches {
  strategies {
    buildAnyBranches {
        strategies {
            skipInitialBuildOnFirstBranchIndexing()
            buildTags {
                atLeastDays '-1'
                atMostDays '7'
            }
        }
    }
    buildAnyBranches {
      stratagies {
        buildChangeRequests {
            ignoreTargetOnlyChanges(true)
            ignoreUntrustedChanges(false)
        }
        buildRegularBranches()
        buildTags {
            atLeastDays '-1'
            atMostDays '7'
        }
      }
    }
  }
}
```

## Opinion on this jenkins solution

First of all I think the configuration is very unintuitive and gets very complex even in  simple scenarios. In order to fully understand the given plugin strategies, I had to look up the source code. I also think this configuration should not be part of the Job, but the pipeline configuration instead. The problem is, that the Jenkins declarative pipeline keyword `when` can not be used for a complete pipelines but only stages. A workaround where you would include all your stages in one parent stage would result in a configuration where all jobs will be triggered in the first place but then skip all stages.