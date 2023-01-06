---
title: Use github fork as composer dependency
date: 2017-01-31T20:00:00+02:00
authors:
  - matks
weight: 10
TargetUrl: /posts/use-github-fork-as-composer-dependency
---

I struggled recently to use a GitHub branch on a fork as a Composer dependency. Here is
how I fixed it.

<!--more-->

Last month I created a Pull Request to add a feature to the wonderful
[deployer](https://deployer.org/) project. I am a big fan of Deployer and I built
a deployment tool out of it for my company, that is why I wanted to participate
to the project.
After coding the feature, I wanted to check that it would not break my tool.

My tool uses `deployer/deployer` as a composer dependency. To make sure it would
still work with my feature, I needed to build my tool dependency with my branch
from my fork `https://github.com/matks/deployer` instead of the official repository.

So basically my usecase was "how to use a GitHub branch as a Composer dependency"

Great news ! There is a blog article with this name:
["Use a GitHub Branch as a Composer Dependency"](https://lornajane.net/posts/2014/use-a-github-branch-as-a-composer-dependency)
and it is well written.
So I follow the steps:

This is my initial `composer.json`
```
{
    "name": "XXX",
    "license": "proprietary",
    "type": "project",
    "minimum-stability": "stable",
    "require": {
        "php": ">=5.4.0",
        "deployer/deployer": "~3.0.0",
        "deployphp/recipes": "~3.0"
    },
    "require-dev": {},
    "autoload": {},
    "config": {
      "bin-dir": "bin"
    }
}

```

<br/>
According to the blog post, I should update it to obtain this (my branch name
is 'enable-autocomplete-questions'):

```
{
    "name": "XXX",
    "license": "proprietary",
    "type": "project",
    "minimum-stability": "stable",
    "require": {
        "php": ">=5.4.0",
        "matks/deployer": "dev-enable-autocomplete-questions",
        "deployphp/recipes": "~3.0"
    },
    "repositories": [
        {
            "type": "git",
            "url": "https://github.com/matks/deployer"
        }
    ],
    "require-dev": {},
    "autoload": {},
    "config": {
      "bin-dir": "bin"
    }
}

```

I delete the `composer.lock` as I want a new one, and I run `composer install`.

BOUM !

```
Your requirements could not be resolved to an installable set of packages.

  Problem 1
    - The requested package matks/deployer could not be found in any version, there may be a typo in the package name.

Potential causes:
 - A typo in the package name
 - The package is not available in a stable-enough version according to your minimum-stability setting
   see <https://getcomposer.org/doc/04-schema.md#minimum-stability> for more details.

```

Composer cannot build my set of requirements and I don't understand why.

So I read again the tutorial and I double-check everything.
I try a few tricks (for example to change my 'minimum-stability' to 'stable') ... nothing. It seems
I cannot build my tool using my fork as a repository. Composer says it cannot
find my package `matks/deployer` although the Github page https://github.com/matks/deployer
is online.

Then I get it.

*Github* knows there is 2 projects: `matks/deployer` and `deployer/deployer`, each
of them is a git repository. But Composer does not look for Github repository names.
Composer looks for the *project name* in the `composer.json` file !
Adding the `repositories` element in my `composer.json` told Composer to look
*in* this Github repository for a `composer.json` file in which it would find
the package dependency.
And since I forked the main repository in order to create a Pull Request,
I did not change the project name. So my Github repository is `matks/deployer` but my Composer project name is still `deployer/deployer`.

Let's fix this !

So this is my final `composer.json`:

```
{
    "name": "XXX",
    "license": "proprietary",
    "type": "project",
    "minimum-stability": "stable",
    "require": {
        "php": ">=5.4.0",
        "deployer/deployer": "dev-enable-autocomplete-questions",
        "deployphp/recipes": "~3.0"
    },
    "repositories": [
        {
            "type": "git",
            "url": "https://github.com/matks/deployer"
        }
    ],
    "require-dev": {},
    "autoload": {},
    "config": {
      "bin-dir": "bin"
    }
}

```

And this one works, Composer performs the installation using, as expected, my
development branch.

#### Conclusion

Because of how we use them often together, it is easy to be confused between Github, Packagist,
Composer, repositories. This is what happened to me, I confused a Github property
for a Composer property, and I did not configure my `composer.json` properly.

I hope this blog post will help you not to do this mistake.

And thanks [@lornajane](https://twitter.com/lornajane) for the wonderful
["Use a GitHub Branch as a Composer Dependency"](https://lornajane.net/posts/2014/use-a-github-branch-as-a-composer-dependency)
blog post.