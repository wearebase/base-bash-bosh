# Base Orchestration & Synchronisation Helper [BETA]

B.O.S.H is a collection of orchestration & synchronization scripts which can be used to deploy pre-built code and synchronise or backup databases and assets.

We built this as a tool to solve a problem, and it works great for us. However it has only been used with a small number of projects and as a result there maybe still be some bugs. We do not recommend its use on a production environment until it has been tested against a similar, non-production environment. If you find a problem please raise an issue for us.

## Prerequisites:
 
* You will need the following on your machine `sshpass` `git`

        ubuntu: sudo apt-get install sshpass git

## Installation

Run the following commands

    composer require wearebase/bosh:~1.0.16
    composer install

## Setup

Once installed (see `install` below), there will be a set a configuration files in the root of your project.
`config/bosh-config` contains the key settings for the scripts. `config/environments/` contains `.env` files which are the *environments* the bosh scripts use as parameters. 
For example the `production` environment uses settings from the `config/environments/production.env` file.

The environment `local` does not need a user/hostname/project-root (see `config/environments/local.env` for an example). 

An example `config/environments` folder could contain

    local.env
    vagrant.env
    development.env
    staging.env
    production.env

## Usage

From the command line run the following:

    ./bin/bosh <command> <parameters>
    
The following commands can be run:

### Install

    ./bin/bosh install
     
No other commands can be run until your have run the install command.
This copies the example configuration and environment files to the root of your project.

### Deploy

Builds and deploys code to a server

    ./bin/bosh deploy <environment-name> <git-branch-or-tag>
    
The script is called with two parameters. The first is the name of the environment you wish to deploy to. The configuration for which can be found in `config/environments/<environment-name>.env`. The second is the git hash, or a tag or branch name you wish to deploy. Only committed code will be deployed.

#### What it does

Firstly, the code base will be cloned from git using your commits that you have made locally (instead of the remote git server). This clone will be placed in `./dist` temporarily

This deploy script will call the build script (the location of which is defined in the `bosh-config` file. This location is local to the root of your project) to prepare the site for deployment. This means that dependencies, pre-compiled files, minified scripts and anything else not source controlled will all have been placed in the `./dist` directory alongside your source controlled code. If you wish to call a build script which is outside the project root, you will need to create a script inside which then proxies to the external script.

This code will then be zipped, moved to the remote server and unzipped and placed into the releases folder under the timestamp it was deployed at. Any shared directories as defined in `shared-paths.sh` (for files which are not source controlled such as user uploads) will be symbolically linked into the release folder, and a symbolic link which points to the current release will be updated so that `current` now points to your new release.

Within the `bosh-config` file there are arrays for `PRE_PUBLISH` and `POST_PUBLISH` commands which are run before the symbolic link `current` is updated and after. This is where unit tests and other checking tools will be run. If any of part of the script fails to pass the script will bail out.

### Sync Database

This script will backup, copy and import databases from one server to another

    ./bin/bosh sync-db <origin-environment> <destination-environment>
    
The script is called with two parameters. The first is the environment name you with to copy FROM. The second is also an environment name - the one you wish to copy the database TO.

There is an optional parameter `-b` or `--backup` which will skip the import step.

~~You cannot sync to `production`. You can only sync TO or FROM `vagrant` with `local`~~
Update: You can now! It uses your local machine as a proxy.

#### What it does

The export from the FROM server simply runs the `mysqldump` command and does an entire database dump. This file is then copied to the TO server you specified. The database will then be imported and apply the sql commands from the `POST_IMPORT_DB_COMMAND` item in the `bosh-config` file . These commands alter database values to turn the application into a development state from a production state, for example changing email addresses and sandboxing payment gateways.

**Important note:** Cancelling the script does not stop any processes that are running on the remote. You will need to let these finish or kill them manually be logging into the remote server yourself.

### Sync Uploads (Wordpress Only)

Copies upload files from one server to another

    ./bin/bosh sync-uploads <origin-environment> <destination-environment>
    
The script is called with two parameters. The first is the environment name you with to copy FROM. The second is also an environment name - the one you wish to copy the database TO.

The location of the uploads folder may be different depending on your wordpress setup, so these locations must be defined in the corresponding environment file in `config/environments/`

You cannot sync to `production`. You can only sync TO or FROM `vagrant` with `local`

**Important note:** Cancelling the script does not stop any processes that are running on the remote. You will need to let these finish or kill them manually be logging into the remote server yourself.
