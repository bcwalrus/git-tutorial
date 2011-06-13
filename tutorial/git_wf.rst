===============
My Git Workflow
===============

:Author: bc Wong
:Date: Jun 12, 2011

.. sectnum::
    :depth: 2
.. contents::

Preamble
========
This is intended as a concise sing-along guide for task-specific git commands.
I have strong feelings about how a beginner should not be exposed to the full
spectrum of the git's power, or dangerous commands like "``git pull``". So this
is an attempt to cover a subset of the concepts and commands in greater depth,
which hopefully help readers learn more about git on their own. I am using
git version 1.7.1 as I write this.


Basic Concepts for Beginners
============================
repository
    A repository (or repo) is a stand-alone revision controlled tree. Your
    repository may be connected to other repositories, and know about the
    states of these other repositories. These are called "remote"s.

    Repositories are identified by human-friendly names.

commit object
    In git, commits work at the repository level, not at the individual files
    level. A commit object describe a set of changes across many files. A commit
    object also includes any addition or removal of files. **Each commit object
    points to its previous commit and represents the entire lineage**, like a
    linked list. So given
    a single commit object, you can see the entire history of the repo up to
    that commit. But there is no way to identify later commits in general.

    Commit objects are identified by hashes (long non-sensible strings).
    Commits may have "nicknames" like branch names and tag names.

    Once a commit object is created, it stays. Even if nothing is pointing to
    the commit, you can still get to it if you know its hash.

branch
    You should think of a branch as a name that points to a commit. A
    repository can have many branches. Most of the time, you will be working on
    a branch. We call that the currently checked-out branch. As you create a
    new commit on the current branch, the branch "pointer" auto advances to
    point to the new commit.

    Branches are identified by human-friendly names. The ``master`` branch is
    typically trunk, by convention.

``HEAD``
    A special name for the currently checked out branch. Since a branch is a
    pointer to a commit, ``HEAD`` also refers to the most recent commit in the
    current branch.

    There is only one ``HEAD`` in a repository, at most.

working tree
    The working tree is your view of repository tree. Git stores its commits
    and branches in its own private area (the ``.git`` directory under the repo
    root). You can modify the working tree all you want; git can always reset
    the working tree to the state described by any commit.

    If you have changed any file that git is tracking, your working tree is
    *dirty*. Git will warn you when you are about to overwrite any dirty file.

.. note::
    This guide is written as a sing-along. You will learn git better if you
    follow each step. Run this first to set things up::

        $ git clone --bare git://github.com/bcwalrus/git-tutorial.git /tmp/git-tutorial

    And I want you to immediately forget what just happened. Right now. Forget.


Setup and Introspection
=======================

Initialize a Local Repository
-----------------------------
Our project already exists elsewhere on some central server. Let's
initialize our local repository at ``~/ws/git-tutorial``::

    $ mkdir -p ~/ws
    $ cd ~/ws
    $ git clone git://github.com/bcwalrus/git-tutorial.git
    Initialized empty Git repository in /home/bcwalrus/ws/git-tutorial/.git/
    ...
    Resolving deltas: 100% (38/38), done.

    $ cd git-tutorial/
    $ git remote -v
    origin      git://github.com/bcwalrus/git-tutorial.git (fetch)
    origin      git://github.com/bcwalrus/git-tutorial.git (push)

* ``git clone git://github.com/bcwalrus/git-tutorial.git``
    This creates a local repo from the remote repo, which is specified by the
    ``git://`` URL here. This remote repo is automatically linked, and is
    called ``origin``. You can rename ``origin`` to something else. It is
    just a name.

* ``git remote -v``
    This shows you the remote repositories that we know about. This is useful
    if you collaborate with other people who have their own repositories, and
    need to link to their repos directly.


Look Around
-----------
Let's explore a bit::

    $ git branch
    * master

    $ git branch -r
      origin/HEAD -> origin/master
      origin/adam
      origin/branch-1.0
      origin/master
      origin/tutorial

    $ git log HEAD --graph --pretty=format:'%h -%d %s'
    * 5c09211 - (HEAD, origin/master, origin/HEAD, master) Updated README.rst with usage and examples
    * 406e897 - Fix python paths in scripts
    * ecd00d3 - Update import statements to work with python 2.4
    * 3b67880 - import * from a relative path is not allowed in python2.5
    * 8563952 - Initial commit

Before we go further, run ``gitk --all`` (Linux) or ``gitx`` (Mac) if you have
them installed. If you don't, install them now.

* ``git branch``
    This shows you all local branches. The current branch has a
    star in front of it. When you give it the ``-r`` flag, it shows the
    branches in the remote repos. When you want to refer to a remote branch,
    you need to prefix it with the repo name. e.g
    ``<remote_repo>/<branch_name>``.
    Remote branches, which you do not directly manipulate, are totally separate
    from your local branches. They are different pointers.

* ``git log HEAD ...``
    This shows you the commit log starting from ``HEAD``. You can replace
    ``HEAD`` with another commit object. The flags makes the
    output more concise and readable. I use a ``gitlog``
    alias that is even more elaborate. See `My Git Setup`_.

    The first line of the log output tells us that we at are commit 5c09211.
    This part: ``(HEAD, origin/master, origin/HEAD, master)`` lists the
    pointers pointing to this commit.


Make More Branches
------------------
For a beginner of git, your branching strategy should be:

* Try to only work on one thing at a time, and do that work on your local
  ``master`` branch.
* If you want to work on code that lives on a different remote branch, create a
  local branch with the same name.

Remember that our current local branch is ``master``. Suppose we want to work
on (or look at) the code in ``origin/branch-1.0``. We do::

    $ git branch branch-1.0 origin/branch-1.0
    Branch branch-1.0 set up to track remote branch branch-1.0 from origin.

    $ git branch
      branch-1.0
    * master

    $ git checkout branch-1.0
    Switched to branch 'branch-1.0'

    $ git branch
    * branch-1.0
      master

    $ git log HEAD --graph --pretty=format:'%h -%d %s'
    * f3bcac9 - (HEAD, origin/branch-1.0, branch-1.0) Show command output in error message
    * 406e897 - Fix python paths in scripts
    * ecd00d3 - Update import statements to work with python 2.4
    ...

    $ git checkout master
    Switched to branch 'master'

* ``git branch <new_branch> <starting_point>``
    We created a new local branch called ``branch-1.0``, which points to
    whatever commit that ``origin/branch-1.0`` is currently pointing to.
    The new branch shows up in the next ``git branch`` command.

* ``git checkout <branch_name>``
    Checking out a branch means replacing your working tree with the content
    from the specified branch (i.e. the commit object that the branch points
    to). Git will only replace objects that it is tracking. It will not
    overwrite your dirty changes. This command is always safe.


Edit and Commit
===============

Make Changes
------------
Let's make a change on the current branch, ``master``. Edit
``src/gitreview/cli/exceptions.py``, and change the end of the file to::

    class CommandArgumentsError(CLIError):
        def __init__(self, msg):
            msg = 'Command argument error: %s' % (msg,)
            CLIError.__init__(self, message)

Next we add a TODO file::

    $ cd ~/ws/git-tutorial
    $ echo "Allow 3-way diff against common ancestor" > TODO

To look at the state of your working tree::

    $ git status
    # On branch master
    # Changed but not updated:
    #   (use "git add <file>..." to update what will be committed)
    #   (use "git checkout -- <file>..." to discard changes in working directory)
    #
    #       modified:   src/gitreview/cli/exceptions.py
    #
    # Untracked files:
    #   (use "git add <file>..." to include in what will be committed)
    #
    #       TODO
    no changes added to commit (use "git add" and/or "git commit -a")

You should read the instructions in that output. Since git is already tracking
``exceptions.py``, it automatically notices that it has changed. But we have
to tell git to track the new file manually::

    $ git add TODO


Commit
------
You should commit when you have made a consistent change. Commit frequently.
And always review your changes before you commit. (More on that later in
section `Code Review`_.)
Here, I will show you 2 options.

1. You can ``git add`` all files you would like to
   commit, and then proceed to commit::

    $ git add src/gitreview/cli/exceptions.py
    $ git status
    ...
    $ git commit

2. Or, if you only have a small number of modified files, you can pass the
   files as arguments::

    $ git commit src/gitreview/cli/exceptions.py TODO

After you entered ``git commit ...``, you will see the commit screen.
You will now enter the commit message. This is a reasonable guideline for
`good commit messages
<https://github.com/erlang/otp/wiki/Writing-good-commit-messages>`_. For this
commit message, simply enter "Test commit", save the file and quit.

Let's look at our history again::

    $ git log HEAD --graph --pretty=format:'%h -%d %s'
    * caca1f5 - (HEAD, master) Test commit
    * 5c09211 - (origin/master, origin/HEAD) Updated README.rst with usage and examples
    * 406e897 - Fix python paths in scripts
    ...

The ``HEAD`` and ``master`` branch pointers automatically advance to point to
the new commit. Note that this is a local commit. You have **not** pushed your
work back to the ``origin`` repo yet -- ``origin/master`` **remains unchanged**
for now.


Amend a Commit
--------------
We messed up. There is an error in our code. The last line below should
say ``msg`` instead of ``message``::

    class CommandArgumentsError(CLIError):
        def __init__(self, msg):
            msg = 'Command argument error: %s' % (msg,)
            CLIError.__init__(self, message)              ## ERROR !!!

Edit the file ``src/gitreview/cli/exceptions.py`` and fix that line. If you do
``git status``, you will see that the file is dirty again::

    $ git status
    ...
    $ git add src/gitreview/cli/exceptions.py
    $ git commit --amend

The ``--amend`` flag amends ``HEAD``. You also have a chance to amend the
commit message. Save the commit message. Done. The new commit object will have
a different hash, and is a different object from the old commit. The old commit
object is still hanging around. If there are branches or tags pointing to the
old commit object, they do **not** see your amendment. But don't worry about it
if nothing is pointing to the old commit. (Look at it in ``gitk --all``.)


Fetch, Push and Conflicts
=========================

.. note::
    To simulate a conflict, let's have another user change ``origin`` behind our
    back. Run the following and immediately forget about it::

        $ git remote set-url origin /tmp/git-tutorial
        $ pushd /tmp/git-tutorial
        $ git symbolic-ref refs/heads/master refs/heads/adam ; popd


Push to Remote
--------------
To share the work we have done, we need to push this commit back to the
``master`` branch (i.e. trunk) on ``origin``::

    $ git push origin HEAD:master
    To /tmp/git-tutorial.git
     ! [rejected]        HEAD -> master (non-fast-forward)
    error: failed to push some refs to '/tmp/git-tutorial.git'
    To prevent you from losing history, non-fast-forward updates were rejected
    Merge the remote changes before pushing again.  See the 'Note about
    fast-forwards' section of 'git push --help' for details.

Obviously it has failed. The ``master`` branch in the remote repo has changed.
We will deal with that in the next section.

* non-fast-forward
    A fancy word for conflict. "Fast forwarding" a branch to a commit is only
    possible when that commit strictly builds on top of the branch, and
    includes all commits from that branch.
    Our push failed because ``origin/master`` has been updated.

* ``git push origin HEAD:master``
    The general syntax is "``git push <repository> <from_commit>:<to_branch>``".
    The branch in ``<to_branch>`` is always a remote branch in that
    ``<repository>``. For example, if you want to push to a feature branch, do
    "``git push origin master:bug-1234``". (Try it now.) This will create a new
    branch called ``bug-1234`` in ``origin``. This new branch will point to the
    same commit as your local ``master`` branch.

    When you push something to a remote branch, you are asking that the remote
    branch point to your commit. Git will transfer your commit object plus its
    lineage to the remote repo. The remote branch will have the same hash.


Fetch from Remote
-----------------
First we fetch the latest state of ``origin``::

    $ git fetch origin
    From /tmp/git-tutorial
       5c09211..f4aa278  master     -> origin/master

    $ gitk --all &

``git fetch`` is one of the 3 git commands that talk to a remote repo in this
guide. The other 2 are ``git clone`` and ``git push``. The rest of the time,
you are working on your local standalone repo, without affecting the rest of
the world.

``git fetch`` does **not** update your working tree. It does **not** ask you to
resolve conflicts (unlike ``svn update``). It simply downloads the state of
``origin`` (to the private ``.git`` directory) and updates the ``origin/xyz``
pointers. It is **always** safe to fetch, even if you are in the middle of
editing something. You can control when to merge with the remote branches and
handle conflicts.


Handle Conflicts
----------------
The diagram from ``gitk`` show the divergence::

    *   [HEAD, master] Test commit
    | * [origin/master] Enhance error logging
     \|
      * [HEAD^] Updated README.rst with usage and examples

What we want is a linear history with our "Test commit" applied directly on top
of "Enhance error logging" (``origin/master``)::

    $ git rebase origin/master
    ...
    CONFLICT (content): Merge conflict in src/gitreview/cli/exceptions.py
    ...
    When you have resolved this problem run "git rebase --continue".
    If you would prefer to skip this patch, instead run "git rebase --skip".
    To restore the original branch and stop rebasing run "git rebase --abort".

* ``git rebase origin/master``
    Git will first save all the commits not in ``origin/master`` (i.e. all your
    local commits). Then git will reset to ``origin/master``, and play
    back those saved commits. Remember that your local commits are being played
    back. If there is a conflict, the "base" version is ``origin/master``.

    If you mess up during a rebase, you can always do "``git rebase --abort``"
    and start over again.

    Note that you can rebase a local branch on top of another local branch as
    well. You want to read the manpage carefully for this one ("``man git
    rebase``").

Our rebase runs into a genuine conflict. Edit
``src/gitreview/cli/exceptions.py``, fix it and save. Let's use the version
marked as ``HEAD``::

    class CommandArgumentsError(CLIError):
        def __init__(self, msg):
    <<<<<<< HEAD
            msg = 'bad arguments: %s' % (msg,)
    =======
            msg = 'Command argument error: %s' % (msg,)
    >>>>>>> Test commit
            CLIError.__init__(self, msg)

For fixing serious conflicts, I highly recommend ``p4merge``, which does
a real 3-way merge. You can invoke it with "``git mergetool``". Any way,
let's finish the rebase::

    $ git add src/gitreview/cli/exceptions.py
    $ git rebase --continue
    Applying: Test commit
    Recorded resolution for 'src/gitreview/cli/exceptions.py'.

    $ git log --graph --pretty=format:'%h -%d %s'
    * 6a91d20 - (HEAD, master) Test commit
    * f4aa278 - (origin/master, origin/adam, origin/HEAD) Enhance error logging
    * 5c09211 - Updated README.rst with usage and examples
    ...

Our commit is on top of ``origin/master`` again. Let's push::

    $ git push origin HEAD:master
    Counting objects: 4, done.
    ...
    To /tmp/git-tutorial
       f4aa278..6a91d20  HEAD -> master


Code Review
===========

Review Uncommitted Code
-----------------------
These are changes in the working tree that haven't been committed yet. Let's
make a change, and show the diff of **the working tree** against ``HEAD``::

    $ echo "Add a graphical interface" >> TODO
    $ git difftool TODO

* ``git difftool``
    You can configure the ``diff.tool`` option in your ``.gitconfig`` to avoid
    having to specify the tool every time.

    If you do not specify any files, i.e. simply ``git difftool``, it will loop
    and show you the diff of **all** modified files.

    If you have already ``git added`` your file, you need to add a ``--cached``
    argument.


Review Commits
--------------
Before you push your commit to ``origin``, make the habit of reviewing it again.
The instructions here are also applicable if you are the one reviewing someone
else's commit. Let's say that we are interested in our most recent commit::

    $ git difftool HEAD~..HEAD

* ``origin/master..HEAD``
    This "``<from_commit>..<to_commit>``" syntax is how you specify a commit
    range. As usual, you can use a hash or a pointer to refer to a commit. If
    you omit either end, git will use ``HEAD``.

    The "``HEAD~``" syntax means one commit before ``HEAD``. "``HEAD~2``" means
    2 commits before ``HEAD``, and so on. You can use the "``~``" on anything
    that refers to a commit (e.g. ``master~`` also works).

To use another example, suppose we are interested in the "Fix python paths"
commit::

    $ git log --graph --pretty=format:'%h -%d %s'
    * 6a91d20 - (HEAD, origin/master, origin/HEAD, master) Test commit
    * f4aa278 - (origin/adam) Enhance error logging
    * 5c09211 - Updated README.rst with usage and examples
    * 406e897 - Fix python paths in scripts
    ...

We can use the hash and do::

    $ git difftool 406e897~..406e897


Prepare a Patch
---------------
If you need to generate a patch for ReviewBoard or whatever reason, do::

    $ git diff HEAD~..HEAD > /tmp/bug.patch

The syntax is similar to ``git difftool``. ``git diff`` takes an optional
commit range, and an optional list of paths. If you don't specify any path,
it'll diff all modified files. If you omit the commit range, it'll diff your
working tree against ``HEAD``.

Tip: For ReviewBoard, I find `post-review
<http://www.reviewboard.org/docs/manual/dev/users/tools/post-review/>`_ more
convenient.


Self-study
==========
Spend at least a week with the git commands you've learned here. When you
get comfortable with them, I recommend these more advanced commands. Read the
manpages and experiment::

    git rebase -i
    git reset
    git show
    git cherry-pick

I also highly recommend the `git-review
<https://github.com/bcwalrus/git-review>`_ tool for doing code review.


My Git Setup
============
I have this alias::

    alias gitlog="git log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"

Here is my ``~/.gitconfig``. I highly recommend enhancing the
colour settings in yours. If you do not use ``p4merge`` or ``vimdiff``,
remove the sections on merge and diff.

::

    [user]
            name = bc Wong
            email = ***nospam***

    [color]
            diff = auto
            status = auto
            branch = auto
            interactive = auto
            ui = true
            pager = true
    [color "branch"]
            current = yellow reverse
            local = yellow
            remote = green
    [color "diff"]
            meta = blue bold
            frag = magenta bold
            old = red
            new = green
    [color "status"]
            added = yellow
            changed = green
            untracked = cyan

    [format]
            outputDirectory=/tmp/patches

    [rerere]
            enabled = true

    [merge]
            summary = true
            tool = "p4merge"

    [mergetool "p4merge"]
            path = /usr/share/p4v-2009.2.236331/bin/p4merge
            cmd = "/usr/share/p4v-2009.2.236331/bin/p4merge" \
                    "$PWD/$BASE" \
                        "$PWD/$LOCAL" \
                        "$PWD/$REMOTE" \
                        "$PWD/$MERGED"
            keepBackup = false
            trustExitCode = false

    [diff]
            renameLimit = 5000
            tool = vimdiff-ro
            guitool = gvimdiff

    [difftool "vimdiff-ro"]
            cmd = vimdiff -R $LOCAL $REMOTE

    [difftool]
            prompt = no
