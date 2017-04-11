cm2git
======

cm2git is a conversion tool from Synergy repos to git.
It produces the output in the git-fast-import format from the
Synergy baseline project converting synergy tasks into git
commits.


Requirements
------------

- Linux
- Python 2.7 (Didn't test on previous versions).
- Working ccm synergy command-line tool.


Installation
------------

cm2git is a single python script.  It can be placed somewhere
on the PATH for conveniance or can be started from any place.


Configuration
-------------

### ccm session

cm2git requires an open ccm session to execute ccm queries.
Thus `ccm start ...` should be called in the terminal session before
running conversion.


### ccm diff for directories

cm2git expects the `ccm diff` output for directories in the diff Normal
output format.  Check whether smth. like:

    ccm diff somedir-v1:dir:db#1 somedir-v2:dir:db#1

would produce an output like:

    2c2
    < db#1/ascii/bye.txt
    ---
    > db#1/ascii/hello.txt


Usage examples
--------------

### Export a single project version

    cm2git export myproj-3:project:db#1 > out
    git init --bare test.git && cd test.git
    git fast-import < ../out


### Export a whole project history
The same commands repeated for all versions of baseline projects.

    cm2git export myproj-1:project:db#1 > out.1
    cm2git export myproj-2:project:db#2 > out.2

    git init --bare myproj.git && cd myproj.git
    cat out.1 out.2 ... | git fast-import

See below how to do it in parallel.


### Incremental update
The generated file for the git-fast-import should contain a special command
to specify on which commit to base the new ones.  This command is generated
with the `--update` parameter:

    # update on the latest commit
    cm2git export myproj-501:project:db#1 --update > out.501

    # update on the specific commit (tag, ...)
    cm2git export --update ab23e1e myproj-502:project:db#1 > out.502

Then the output file can be passt to the git-fast-import as before:

    cd myproj.git
    git fast-import < out.501


### Import to the git branch
The '--branch' parameter specifies the name of a git branch.  Export the
1st `test` branch project which was created from the proj-200 tag:

    cm2git export --branch test -u proj-200 proj-test1:project:db#1 > out

Update the `test` branch:

    cm2git export --branch test proj-test2:project:db#1 --update > out


### Export the whole project in parallel
First need to create a list of baseline versions of the project:

    cm2git show --parents myproj-500:project:db#1 > projects

The `projects` file would contain now a list of all parent projects of the
myproj-500 up to the very first version.

Now we can run the export on every version and store the respected outputs.
We would achieve a better performance when run these exports in parallel:

    cat projects | xargs -n 1 -P 4 -I{} cm2git export {} --directory --output

This command runs 4 parallel exports storing the projects' artifacts into
separate directories named after the projects.
The final git repo can be created as before:

    git init --bare proj.git && pushd proj.git
    tac ../projects | xargs -I{} cat "../{}/{}.out" | git fast-import


### Other commands
There are additional commands and parameters to query the Synergy database.
They can be used to debug, check intermediate stages or export projects step
by step (see cm2git --help for more information).


Alternatives
------------

[PySynergy](https://github.com/emanuelez/PySynergy) ccm -> git converter


License
-------
MIT License
