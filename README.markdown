# mcwrapper

A Minecraft Server wrapper for POSIX compatible operating systems (OSX, Linux, BSD, etc). This enables a Minecraft server admin to easily start and stop their server, send commands and also do safe, automatic world data backups.

*I'm MC Wrapper and I'm here to say that I wanna wrap up minecraft so everything'll be okay.*

## Quickstart

Getting up and running with `mcwrapper` is easier than ever. Provided you have git installed:

 1. cd into the directory with minecraft_server.jar in it.
 2. git clone git://github.com/spikegrobstein/mcwrapper.git
 3. cd mcwrapper
 4. ./mcwrapper start
 
If you don't have git:

 1. download a zip or tarball of the latest version of mcwrapper from https://github.com/spikegrobstein/mcwrapper
 2. unzip/untar it
 3. rename resulting directory to 'mcwrapper'
 4. copy resulting 'mcwrapper' directory into the same directory as minecraft_server.jar.
 5. cd into the 'mcwrapper' directory
 6. ./mcwrapper start
 
The above will start up a Minecraft server instance using default settings. All support files related to mcwrapper will be stored in this new mcwrapper directory. Updating to the latest version is as easy as cd'ing into the mcwrapper directory and typing `git pull`

If you're starting without a Minecraft server installed and want to get everything running quickly, once you have unpacked a tarball or cloned the `git` repository, run the following command:

    ./mcwrapper install

This will download the current version of `minecraft_server.jar` from Mojang and place it in the correct location. You can then run the `start` action to start it up.

Basic usage is:

    ./mcwrapper <action> [ <action_params> ]
    
Use the help action to see a full breakdown of usage:

    ./mcwrapper help

See the Configuration section below for instructions on modifying the default configuration.

### Commands

As seen above, you can start the server by running the following:

    ./mcwrapper start
    
The Minecraft server will be started in the background. You can then execute arbitrary commands to the server using the `command` action. The `command` action is also aliased as `cmd`:

    ./mcwrapper cmd op spizzike
    ./mcwrapper cmd say hi
    ./mcwrapper cmd save-all
    ./mcwrapper cmd save-off
    ./mcwrapper cmd save-on

If you'd like to stop the server, issue the following command:

    ./mcwrapper stop
    
You can also find out whether the server is running:

    ./mcwrapper status
    
The status action will return 0 if the server is running or 1 if it's stopped.

There is also support for reading information about the running configuration for cases where an external script may want to interact with Minecraft or `mcwrapper`. Example:

    ./mcwrapper config serverpath
    
The above will output the path to the `minecraft_server.jar` file that it will wrap. All config commands can be run whether `mcwrapper` is running a server or not. A bug currently exists where if `mcwrapper` cannot locate the minecraft_server.jar, no commands will work.

### Convenience Scripts

`mcwrapper` comes with a set of convenience scripts for the commandline-impared user. These scripts can be run by double-clicking them in OSX or Linux and will execute the specified command. The scripts should not be moved outside of that directory or they will not work.
    
## Configuration

Although `mcwrapper` has a built-in default configuration, there are times when you may want to have it operate slightly differently. `mcwrapper` is bundled with an example mcwrapper.conf example file called `mcwrapper.conf-example`. See that file for full configuration documentation.

## Details

When running, `mcwrapper` uses 2 files:

 * mcwrapper.pid -- the pid of the currently running process. This is used by `mcwrapper` for sanity checks but can also be used by 3rd party scripts to see if minecraft_server is running.
 * command_input -- the FIFO used for communicating with the server.
 
The names and locations of the above files are both configurable in `mcwrapper.conf`.

You can run arbitrary commands either through the mcwrapper script as seen in the Quickstart or you can output commands directly to the command_input FIFO. This is handy if you write re-usable Minecraft scripts.

Examples of working directly with the FIFO:

    echo "tell spizzike you are awesome" > command_input

If you have a file called "gimmie_diamond.mcs" containing the following text:

    give spizzike 264
    give spizzike 264
    give spizzike 264
    give spizzike 264
    give spizzike 264
    give spizzike 264

You can run that all through the Minecraft server with the following command:

    command_input < gimmie_diamond.mcs

## Backing Up Minecraft Data

Since it's not safe to back up the world data while the server is running, you need to force a save, then disable writing world data to disk during a backup.

`mcwrapper` contains a backup action for just this purpose. To back up your current world data directory, run the following command:

    ./mcwrapper backup
    
`mcwrapper` will read your `server.properties` file to learn the location of your world data and, after flushing anything in memory, creates a timestamped directory in the minecraft server directory and creates a symlink to the latest backup called `latest`.

By default, the backup action will simply copy your world data and server configuration (white lists, server.properties, ban lists, etc) into the backups directory, but it can be configured to zip or tgz the backup data. See `mcwrapper.conf-example` for information on this.

Assuming your `mcwrapper` lives in `/usr/local/minecraft/mcwrapper`, `backup` will do the following:

 1. create `/usr/local/minecraft/backups/YYYYMMDDHHMMSS` where YYYYMMDDHHMMSS is the current timestamp
 2. copy `/usr/local/minecraft/world` and any other configuration data in `/usr/local/minecraft` into the above directory
 3. create a symlink to the latest backup at `/usr/local/minecraft/backups/latest`
 4. delete old backups. Defaults to keeping the latest 5.
 
The name of the `latest` backup can be configured by editing that setting in `mcwrapper`. You can also configure how many previous backups are kept. Again, see `mcwrapper.conf-example` for information on doing this.

## Restoring from a Minecraft backup

If you ever find a need to restore from a previous world data backup, `mcwrapper` now using the `restore` action and passing a path (full or relative) to the backup you wish to restore:

    ./mcwrapper restore backups/20111118123456

The above example will perform the following actions:

 1. gracefully stop the Minecraft server if it's running.
 2. do a non-clobbering backup of the current world data. This means that regardless of what your backup retention settings are, it will not delete any while creating a backup of the current world data.
 3. copy the specified backup directory into the servers's world directory.
 4. start Minecraft server up if it was previously running.

In addition to the above, a file is also created inside the world directory called `RESTORED_FROM` which contains the argument specified to the `restore` action.
    
## The Future (Todo List)

In the future I aim to create sysV init scripts for Linux (Ubuntu flavoured) and OSX launchd configs. I also plan on including Minecraft backup support to SnapBackup, my backup script (http://github.com/spikegrobstein/snapbackup).

## About

`mcwrapper` is written by Spike Grobstein <spikegrobstein@mac.com>    
http://sadistech.com    
http://spike.grobste.in    
http://github.com/spikegrobstein
