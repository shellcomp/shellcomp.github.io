# Command line completion

Command line (aka tab) completion is popular in the Unix world as it helps typing speed, prevents typos and makes the shell more user-friendly.

## The problem

Impementing filename completion is easy.
Implementing command-specific completion like `git com<tab>` is not.

Completion scripts are different across Bash, Zsh and Fish.

Time consuming to implement, sometimes out of date, hacky.
  https://anonscm.debian.org/cgit/bash-completion/debian.git/tree/completions

## A modest proposal

Completion is implemented in the command about to be run:
The shell run the command with a specific `--tabcomplete '<partial_string>'` option.
The command responds with simple JSON structure that the shell will parse to perform completion or display help messages.

## Benefits

* Shell-independent
* Able to suggest alternative commands and provide contextual help
* Discoverable: the completion mechanism can be queried programmatically to generate...
* ...documentation,
* ...manpages,
* ...simple GUIs,
* ...functional tests.

### The JSON format

The JSON document is to be printed to stdout; All fields are optional:

v
  version.
completions
  a list of valid completion values with an optional help message.
alt
  alternative completions: other commands that the user might want to use.
help
  contextual help message.

An example response for "git con<TAB>"

```javascript
    {
        "v": 1,
        "completions": [
            ["config", "get and set repository or global options"],
        ]
        "alt": [
            ["commit", "record changes to repository"]
        ]
        "help": "",
    }
```

**Fields usage:**

- v

   format version. Should be 1.

- completions

   a list of valid completion values in format ["completion string", "help string"]. The help string can contain multiple lines or be empty. An item with an empty completion string means that non-completable user input is expected (e.g. a name for a new file). An empty list means that no more arguments are expected.

- alt

   alternative completions. Some structure as `completions`, contains suggestions about other commands that the user might want to use instead.
 
- help

   contextual help message. It can contain multiple lines or be empty.
   
- errpos

   if the user-supplied string contains a syntax error, this is the position of the first error, counting from 0 from the left.
   
- files

  a list of filenames, in case the current argument is meant to be an existing file. Allow shells to display the filenames in different colors than `completions`

## How to implement it

In the long term, popular shells like Bash, Zsh, Fish could implement the standard internally.

In the short term, a simple wrapper could run the application on behalf of the shell and perform "traditional" completion. 

Applications could implement completion autonomously or through helper libraries
like [DocOpt](http://docopt.org) or [Click](http://click.pocoo.org/5/)
