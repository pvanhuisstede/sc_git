# Using Git for version control

## 0 Introduction

We all know/knew the situation when the file 'ultimate_final.doc' was
NOT that final document you thought it was, but where is that
'final_final.doc'?

Version control software is designed to keep track of changes between
versions of (a) document(s). They allow us to compare these versions
and, if necessary, to roll back to a previous version of a document.

Version control systems have been around for a long time (1980's), but
the modern ones (Git, Mercurial, Fossil) do not need a central server
to work. They are so-called 'distributed version control systems' that
can run on a single machine. Better still, Git repositories can be run
a bunch of single machines and kept in sync with a main repo on a
server elsewhere. In order to be able to do this, these systems have
powerful 'merge' options to harmonize the contents of these
repositories when repo's on single machines have diverged.

We will discuss the Git version control system in 3 stages:

I. Git on your local computer.

II. Git on your local computer and in sync with a remote (the repo on a
server elsewhere).

III. Given stage 2: Working together with others using the repo.

We have found that in order to better understand what Git is doing
under the hood, it helps to think of Git as a graph with the nodes
being content and the edges relations between the content. Tags can be
used to name (point to) nodes. This all sounds pretty abstract, but
later on it will help us to better grasp the Git commands we will be
using.

## I. Git on your local computer

Last week we saw a lot of our filesystems: directories and files
everywhere. It does not make sense to bring the whole of your harddisk
under version control. Git works on so-called plain text files. Files
that just consist of ASCII characters (then) or UTF-8 chars (now). Git
does not work on binary files, it can't get at their internals to do
its job, keeping track of changes, well. So, depending on what you are
working on, programs or text files for publications, Git supposes that
that project is somewhere on your computer, in a directory, so you can
bring that project/directory under version control.

We skip the the SC lesson '02-setup' for now. Let's concentrate on
getting Git working and understanding what it does.

Let's create a project/directory to explore Git in a, for you,
convenient place using the following command in your shell:

```shell
% mkdir alpha
% cd alpha
% ls -a # shows nothing there, just '.' here and '..' above.
```

Inside our alpha directory, we make another directory 'data' and we
add a file to that directory called 'letter.txt' that contains just
one character 'a'.

```shell
% mkdir data
% printf 'a' > data/letter.txt
```

That leaves us with:

alpha/data/letter.txt: 'a'

Now we put our project/directory, 'alpha', under version control:

% git init

The most important message is: 'Initialized empty Git repository in
/home/peter/Documents/gh/alpha/.git/' The hints are about the
connotations of the default name 'master'.

% ls -a

shows that there is a new 'hidden' directory '.git' that is all Git stuff:

```shell
% cd .git
% ls [dirs: branches, hooks, info, objects, refs; files: config, description, HEAD]
```

```shell
.git/branches [empty]
.git/hooks [contains 13 sample files]
.git/info [sample file of exclude patterns]
.git/objects/info [empty]
.git/objects/pack [empty]
.git/refs/heads [empty]
.git/refs/tags [empty]
```

This all is Git territory. All other files anywhere in the alpha
directory are called the 'working copy'. They are the user's files.

To get an overview of the housekeeping state of our Git repo, we can
ask for the status:

% git status

We see the following elements:
- On branch master
- No commits yet
- Untracked files (+ command suggestion: git add <file>)
- Nothing added to commit but untracked files present (+ command suggestion)

Now that we have brought our project 'alpha' under version control, Git noticed that we added something, the 'data' directory with the file 'letter.txt'.

Let's follow its suggestion and 'add' that new file:

% git add data/letter.txt

In the .git/objects directory we now find a subdir '2e' with a file named '65efe2a145dda7ee51d1741299f848e5bf752e'. So what happened here?

Git hashed the content of 'letter.txt', which was just one character 'a' to:
'2e65efe2a145dda7ee51d1741299f848e5bf752e'. The first 2 chars became the name of the directory, the rest the name of the file that holds the content of the added file.

Two important things:

1. The content of data/letter.txt is now in .git and it stays there even if we were to delete the file from our working copy;
2. Our new file is added to the Git index. Index contains all the files Git is keeping track of in the following manner:

% git ls-files --stage

So far, so good. Let's add a new file:

% printf '1234' > data/number.txt
% git status

Now we see the following status & suggestions:
- On branch master
- No commits yet
- Changes to be committed: new file: data/letter.txt (+ sug)
- Untracked files: data/number.txt (+ sug)

Let's proceed:

% git add data/numbers.txt
% git status # We now have 2 new files
% git ls-files --stage

But we made a small mistake: number.txt should just contain '1'. We open our editor and make the correction.

% git status
- On branch master
- No commits yet
- Changes to be committed:
    new file: data/letter.txt (+ sug)
    new file: data/number.txt
- Changes not staged for commit:
    modified: data/number.txt

This **commit** is the bread and butter of Git. Usually we do a commit when our project reached a stage that is important to save as-is. That could also mean committing after adding two stellar insights kept in our 2 files.

Note that at each stage we can choose to add a file. Not adding a file to the index keeps it out of the next commit. number.txt is as we like it, so we add it to the index:

% git add data/number.txt

Now, we still have 2 files in our index, but we have 3 files in our the .git/objects dir:
- 2e -> letter.txt
- d0 -> number.txt -> 27 -> number.txt

Let's commit our changes:

% git commit -m 'a1'

Which results in the following message:

[master (root-commit) 16fefc9] a1
 2 files changed, 2 insertions(+)
 create mode 100644 data/letter.txt
 create mode 100644 data/number.txt

 in .git/objects we now have the following file contents:

 % tree -a .git/objects

```git
 .git/objects
├── 16
│   └── fefc9cca79a9456ad90146b57ce14b614fabf3
├── 24
│   └── da0032963e5a04cd6c72c18833616c118b437d
├── 27
│   └── 4c0052dd5408f8ae2bc8440029ff67d79bc5c3
├── 2e
│   └── 65efe2a145dda7ee51d1741299f848e5bf752e
├── 8b
│   └── a079128273317ba8d54be41df728fbc75f904f
├── d0
│   └── 0491fd7e5bb6fa28c517a0bb32b8b506539d4d
├── info
└── pack
```

where:
- 16 -> commit
- 24 -> tree that points to our two files: letter.txt and number.txt
- 27 -> number.txt (previous version)
- 2e -> letter.txt
- 8b -> root tree (alpha) that points to data
- d0 -> number.txt

 ### Aside: snippet to get at the hashed content
 % python
 ```python
import zlib
filename = '24/da0032963e5a04cd6c72c18833616c118b437d'
compressed = open(filename, 'rb').read()
decompressed = zlib.decompress(compressed)
print(decompressed)
```

## Taking one step back

We now have to parallel worlds: working copy (project/directory with files) and a graph world keeping track of what happened when by whom in the working copy.

[drawing 1]

Next, we do an edit. We change the value of number.txt from '1' to '2' and we save the file. Now, the working copy is updated, but the INDEX and HEAD stay as they are.

% git add data/number.text

Now, a blob containing '2' is added to the objects directory and the INDEX points at the new blob.

[drawing 2A]

% git commit -m 'a2'

commit triggers the three actions we discussed:

1. generates the trees from the INDEX
2. commits a new commit object 'a2'
3. points the content of the master branch to the new commit

[drawing 2B]

Some remarks:

- Git stores the "things that changed in the objects (diffs)", so large parts of the graphs are being re-used.
- Each commit has a parent (a2 -> a1), so we keep history.
- Refs, like the commit message, point to commits and can be given meaningful content: Instead of 'a2', we can say: 'Changed content of number.txt'. To do this after you committed, you can use the command: 'git commit --amend'.
- We do not throw anything away from the objects/ directory, but the 'refs' can change.
- Although the working copy and the commits pointed at by refs, changing frequently, are readily available [short term memory], older commits are a bit more difficult to access [long term memory].

## Exploring history

In order to be able to better explore some history of our Git repo, we opne an editor and add a line to the file 'number.txt' containing '3'.

% git diff HEAD

The above command can be used to track changes we have in our working copy. Based on this information we can take a decision:

- this is what we meant number.txt to contain: add and commit the file as it is now
- go back to the HEAD before the current head and see the changes with:

% git diff HEAD~1

Let's add our changed file to the index: git add data/number.txt

After that commit the changes: git commit -m '[message]'

Using 'git log' we can see our three commits and HEAD pointing to the last commit.

If we want to roll back to 'number.txt' from our first commit, we can check that one out with the command:
 ```shell
% git checkout 16fefc9 data/number.txt
% cat data/number.txt [shows: '1']
```

Now we can do 2 things:

1. commit the modified file (number.txt of our first commit)
2. or keep the one we had (number.txt of our third commit) with:

```shell
% git checkout HEAD data/number.txt
% git status
```

### The detached HEAD

Happens when one checks out a commit without specifying a file, like so:

% git checkout 16fefc9

- Now head points to the a1 commit AND its tree graph (the one it is pointing at)
- Git writes the files from this tree graph to our working copy
- then Git updates the INDEX
- HEAD is now set to the hash 16fefc9...
- re-attach HEAD to master with: git checkout master

### Ignoring things

Git is meant for working with plain text files. Files that contain code, but also documentation or scientific texts: LaTeX, MarkDown, tekst, docx, but NOT PDF.

Basically you want to keep source files under version control and NOT the generated stuff, because with a correct source file you can always generate the output.

In a project under Git version control, you can inform Git about the things it can skip when keeping track of things. There is a "hidden" file '.gitignore' in the root directory of the project that can be used to specify the directories and/or files to be ignored.

It pays off to do this at the beginning of your project, because, while you can do it later on, since Git never throws anything away, you can end up with a large repo.

An example:

```Git
# Unit test / coverage reports
htmlcov/
.tox/
.coverage
.coverage.*
.cache
nosetests.xml
coverage.xml
*.cover
.hypothesis/
.pytest_cache/
```

--- BREAK ---

## II. Git: Working with a remote repository

### III. Git: Working with others