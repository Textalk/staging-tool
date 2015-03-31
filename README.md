staging-tool
============

A shell script for publishing to multiple servers using SSH.

Current status: Working.

If you're skeptical, check the source code. It is pretty small. This is free
software. Contributions are welcome.

Features
--------

* Upload a directory to multiple servers.
* Activate a version by pointing a symlink to the recently uploaded
  directory.
* Determine the current version using the last git commit timestamp or
  a custom command.

Introduction by examples
------------------------

Create a config file `staging.config` in the project directory with the contents
inspired by that of `staging.config.sample`:

```sh
SERVERS='www1.example.com www2.example.com www3.example.com'
REMOTE_USER=webmaster
LOCAL_DIR=webpages
TARGET=/var/www/www.example.com/www-root
VERSION_SEPARATOR='.'
VERSION_CMD='stat --format=%Y webpages'
```

In this example, the value for VERSION_CMD is a command that takes the last
modification timestamp of the `webpages` directory as a unix timestamp.

If the version is 1410540229, with this configuration `staging-tool upload`
will upload the local directory `webpages` to the remote directory
`/var/www/www.example.com/www-root.1410540229`. The activation command
`staging-tool activate` will activate the current version by turning `www-root`
into a symbolic link to `www-root.1410540229`. In this way, activating a new
version is an atomic operation in the file system.

To list the currently uploaded versions on all servers, use `staging-tool list`.

To activate another version, use `staging-tool -v VERSION activate`.

To check if your configuration seems right, use the command `staging-tool info`.

```sh
$ staging-tool info
Version:        1410540229
Local dir:      webpages
Remote hosts:   www1.example.com www2.example.com www3.example.com
Remote user:    webmaster
Remote target:  /var/www/www.example.com/www-root
Upload dir:     /var/www/www.example.com/www-root.1410540229
```

Possible commands for generating a current version
--------------------------------------------------

Here are some suggestions for generating version names.

```sh
# The git tag of the current HEAD. Fails if there is no such tag.
VERSION_CMD='git describe --tags --abbrev=0 --exact-match'

# The same, but fails also if there are any uncommitted changes.
VERSION_CMD='git diff --quiet HEAD && git describe --tags --abbrev=0 --exact-match'

# A timestamp of the last git commit on the form YYYY-MM-DD_HHMMSS_ZZZZ.
VERSION_CMD="git log -1 --format='%ad' --date=iso | sed -e 'y/ /_/ ; s/[:+]//g'"

# The last modification timestamp of the current directory in seconds since the
# epoch.
VERSION_CMD='stat --format=%Y .'

# The last modification timestamp of the current directory on the form
# YYYY-MM-DD_HHMMSS_ZZZZ
VERSION_CMD="stat --format=%y . | sed 's/\..*//; s/ /_/; s/://g'"
```

Use ssh-agent
-------------

Since this script makes one, sometimes two, ssh connections to each server on
every command, it is useful to start `ssh-agent`. This lets you type your
password only once.

```sh
$ eval `ssh-agent`
Agent pid 7297
$ ssh-add
Enter passphrase for /home/viktor/.ssh/id_rsa:
Identity added: /home/viktor/.ssh/id_rsa (/home/viktor/.ssh/id_rsa)
```

Now you can use any staging-tool commands without typing your password all the
time. To use an identity file other than ssh's default one, use `-i ID_FILE`
with staging-tool or set the `ID_FILE` environment variable.

When you're done, do `ssh-agent -k` to kill the current ssh-agent.

Wishlist
--------

* Use rsync instead of scp
* Possibility to specify rsync options in the configuration
* A command to delete a version on remote servers
* Run a custom command over ssh on all servers
* Customizable activation command to run when activating a version instead of
  making a symlink

More documentation
------------------

For more documentation, use `staging-tool --help` or look at the source code
of the file [staging-tool](staging-tool).
