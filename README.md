# pytocli

A Python lib to generate CLI commands
 
## Problem

You need generate some CLI commands and start simple as
```python
>>> def git():
...    return 'git'
>>> git()
'git'

```
But you realize it has many options:`

```python
>>> def git(p=None, v=None):
...    cmd = 'git' 
...    if p:
...        cmd += ' -p'
...    if v is not None:
...        cmd += ' -v'
...    return cmd
>>> git()
'git'
>>> git(p=True)
'git -p'
>>> git(v=True)
'git -v'
>>> git(p=True, v=True)
'git -p -v'

```
So the spaghetti code begins.

# Lib Goal

The lib goal is provide a fluent interface to generate cli commands:

```python
>>> from tests.test_base import Git
>>> str(Git())
'git'
>>> str(Git().start('foo.git'))
'git -C foo.git'
>>> str(Git().verbose().start('bar.git'))
'git -v -C bar.git'
>>> str(Git().verbose().start('baz.git').commit.message('great!'))
'git -v -C baz.git commit -m great!'

```

## How to extend the framework. Use Case: git like

### Step 1: inherit from CommandBuilder class

The command name must be overwritten as class attribute.
 
```python
from pytocli import CommandBuilder


class Git(CommandBuilder):
    name = 'git'
```

## Step 2: create command options
 
```python
from pytocli import (CommandBuilder, Option, NoValueOption, 
                     SingleValueOption, MultiValuesOption)


class Git(CommandBuilder):
    name = 'git'
    # Options
    verbose = Option(NoValueOption, '-v', 'Verbose mode')
    start = Option(SingleValueOption, '-C')
    multi = Option(MultiValuesOption, '--mu')

```
## Step 3: create a sub commands following steps 1 and 2
 
```python
from pytocli import (CommandBuilder, Option, NoValueOption, 
                     SingleValueOption, MultiValuesOption, 
                     SubCommandBuilder)



class GitSubCommand(SubCommandBuilder):
    @property
    def git(self):
        return self.parent_cmd


class Commit(GitSubCommand):
    name = 'commit'
    message = Option(SingleValueOption, '-m', 'Message to be added on commit')


class Revert(GitSubCommand):
    name = 'revert'

```

## Step 4: Connecting CommandBuilder connecting SubCommandBuilders
 
```python

class Git(CommandBuilder):
    name = 'git'
    # Options
    verbose = Option(NoValueOption, '-v', 'Verbose mode')
    start = Option(SingleValueOption, '-C')
    multi = Option(MultiValuesOption, '--mu')

    # SubCommands
    commit = SubCommand(Commit)
    revert = SubCommand(Revert)

```

## Step 5: Add parameters to Parent command

Passing parameters to parent command can be done even after subcommand creation:
 
```python
>>> commit = Git().commit
>>> str(commit)
'git commit'
>>> commit.parent_cmd
git
>>> commit.parent_cmd.verbose()
git -v
>>> str(commit)
'git -v commit'

```

# Exploring CLI from Python Command Line

```python

>>> from tests.test_base import *
>>> Git.options
('verbose', 'start', 'multi')
>>> Git.verbose
Option -v: Verbose mode
>>> Git.sub_commands
('commit', 'revert')
>>> Git.commit.options
('message',)

```

