git-cache
=========

Context
-------

If you repeatedly clone a Git repository, for instance for a software you use to hack, you could bothered by always waiting during the clone operation, particularly long for big softwares. On a other side, if you have multiple copies of a Git repository on your computer, it takes place you could prefer not to use.

Git has a feature "--reference"/alternates, which gives the possibility to indirect commits to another directory on the same computer. The use case is:

    $ cd ~/test
    $ git clone https://example.org/git/repo.git normal-repo
      (you wait minutes or hours, and the resulting directory is big)
    $ git clone --reference ~/test/repo https://example.org/git/repo.git referenced-repo
      (you wait seconds, and the resulting directory is small - only the size of the files, no size used by the history)

Both clones contain the whole history of the Git repository. Internally, the second clone contains a file `.git/objects/info/alternates` containing the path of the first directory, where git searches the commits it doesn’t find the in the second clone. Obviously, if you delete the first clone, the second is useless and will display tons of errors.


git-cache
---------

git-cache creates a central cache directory on a computer, which contains most of the commits of regularly cloned Git repositories.

If you are a regular hacker of a software, say MediaWiki, you can cache most commits and then only download recent commits. Use case:

    $ sudo git cache init  # cache is located in /var/cache/git-cache
    $ git cache add mediawiki https://git.wikimedia.org/git/mediawiki/core.git
      
      You can now use the cache directory with:
      
          git clone --reference /var/cache/git-cache https://git.wikimedia.org/git/mediawiki/core.git
      
      (1607 seconds, the cache takes 314 Mio)
    $ git clone --reference /var/cache/git-cache https://git.wikimedia.org/git/mediawiki/core.git clone-1
      (46 seconds, the directory takes 98 Mio, the .git subdirectory takes 17 Mio because there were new commits)
    $ git clone --reference /var/cache/git-cache https://git.wikimedia.org/git/mediawiki/core.git clone-2
      (58 seconds, the directory takes 98 Mio, the .git subdirectory takes 17 Mio because there were new commits)


Usage
-----

__Installation:__ Copy git-cache inside your git commands directory (on Ubuntu: /usr/lib/git-core) and be sure it can be executed by the users (mode 755).

__Use:__ All commands have the format

    git cache ACTION PARAMETERS

__Commands:__

    # General maintenance commands
    git cache init [DIR]       initialise the cache directory
    git cache delete --force   delete the cache directory
    
    # Daily commands
    git cache add NAME URL     add a cached Git repository
    git cache show [NAME]      show cached Git repositoryies/repository
    git cache fetch            fetch all cached Git repositories
    git cache rm --force NAME  remove a cached Git repository
    
    (Any other command will be applied to the cache directory,
     e.g. `git cache gc` or `git cache remote show`.)

__Location of the cache directory:__ The default cache directory contains all cached repositories (each Git repository is a remote). If it is created by root, the cache directory is `/var/cache/git-cache`, else it is `~/.cache/git-cache`; it can also be another directory specified in the init command. This directory could become big, be sure you have enough place.

~~You can want to cron `git cache fetch` to automatically retrieve new commits.~~ (bug)


Development
-----------

Bug: git cache fetch does not retrieve new commits.

This is a first version, and feedback is needed to improve its daily usage. Some questions I wonder:
- Are the subcommands sufficiently explicit?
- Should we regularly run `git gc` or even `git gc --aggressive`?
- How git behaves when some commits are both local and in the cache? Does it remove local objects to gain place?
- etc.

Internally the cache directory is a big bare directory containing all remote repositories, even if they do not share commits (sort of orphans branches). Should the cache directory be splitted by repository: /var/cache/git-cache/mediawiki, /var/cache/git-cache/visualeditor, etc. Particularly in the second case, the name of the remote repository is useless, and I would prefer use unique names, e.g. md5(URL) and completely hide this to the user (possibly with soft links from the URL to the directory for usability inside the cache directory).


Licence
-------

- Original author: [Seb35](https://github.com/Seb35)
- Licence: [WTFPL 2.0](http://www.wpfpl.net)


References
----------

This program is a generalisation to arbitrary Git repositories of an idea and implementation by [Randy Fay](http://randyfay.com/content/reference-cache-repositories-speed-clones-git-clone-reference) specifically for Drupal. Hopefully this generalisation is sufficiently simple to stay useable and practical.

Similar program: [git-cached by dvessel](https://github.com/dvessel/git-cached)
