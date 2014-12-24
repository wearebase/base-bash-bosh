# Base Orchestration & Synchronisation Hippopotamus

B.O.S.H is a collection of orchestration & synchronisation scripts which can be used to deploy pre-built code and synchronise or backup databases and assets

## Installation

Add the following to your `composer.json` file and run `composer install`

    {
        "require": {
            "wearebase/bosh" : "dev-master"
        },
        "repositories": [
            {
                "type": "package",
                "package": {
                    "name": "wearebase/bosh",
                    "bin": ["bin/bosh"],
                    "version": "dev-master",
                    "source": {
                        "url": "https://bitbucket.org/wearebase/base-bash-bosh.git",
                        "type": "git",
                        "reference": "origin/master"
                    }
                }
            }
        ]
    }

## Setup

Once installed (see `install` below), there will be a set a configuration files in the root of your project.
`/config/bosh-config` contains the key settings for the scripts. `/config/environments/` contains `.env` files which are the *environments* the bosh scripts use as parameters. 
For example the `production` environment uses settings from the `/config/environments/production.env` file.

The environment name `local` does not need an environment file as this is your local environment. 

An example `/config/environments` folder could contain

    local.env
    vm.env
    dev.env
    staging.env
    production.env

## Usage

From the command line run the following:

    ./vendor/bin/bosh <command> <parameters>
    
The following commands can be run:

### Install

    ./vendor/bin/bosh install
     
No other commands can be run until your have run the install command.
This copies the example configuration and environment files to the root of your project.

### Deploy

Builds and deploys code to a server

    ./vendor/bin/bosh deploy <environment-name> <git-branch-or-tag>
    
The script is called with two parameters. The first is the name of the environment you wish to deploy to. The configuration for which can be found in `/config/environments/<environment-name>.env`. The second is the git hash, or a tag or branch name you wish to deploy. Only committed code will be deployed.

#### What it does

Firstly, the code base will be cloned from git using your commits that you have made locally (instead of the remote git server). This clone will be placed in `./dist` temporarily

This deploy script will call the build script (the location of which is defined in the `bosh-config` file) to prepare the site for deployment. This means that dependencies, pre-compiled files, minified scripts and anything else not source controlled will all have been placed in the `./dist` directory alongside your source controlled code.

This code will then be zipped, moved to the remote server and unzipped and placed into the releases folder under the timestamp it was deployed at. Any shared directories as defined in `shared-paths.sh` (for files which are not source controlled such as user uploads) will be symbolically linked into the release folder, and a symbolic link which points to the current release will be updated so that `current` now points to your new release.

Within the `bosh-config` file there are arrays for `PRE_PUBLISH` and `POST_PUBLISH` commands which are run before the symbolic link `current` is updated and after. This is where unit tests and other checking tools will be run. If any of part of the script fails to pass the script will bail out.


### Sync Database

This script will backup, copy and import databases from one server to another

    ./vendor/bin/bosh sync-db <origin-environment> <destination-environment>
    
The script is called with two parameters. The first is the environment name you with to copy FROM. The second is also an environment name - the one you wish to copy the database TO.

There is an optional parameter `-b` or `--backup` which will skip the import step.

You cannot sync to `production`. You can only sync TO or FROM `vm` with `local`

#### What it does

The export from the FROM server simply runs the `mysqldump` command and does an entire database dump. This file is then copied to the TO server you specified. The database will then be imported and apply the sql commands from the `POST_IMPORT_DB_COMMAND` item in the `bosh-config` file . These commands alter database values to turn the application into a development state from a production state, for example changing email addresses and sandboxing payment gateways.

### Sync Uploads (Wordpress Only)

Copies upload files from one server to another

    ./vendor/bin/bosh sync-uploads <origin-environment> <destination-environment>
    
The script is called with two parameters. The first is the environment name you with to copy FROM. The second is also an environment name - the one you wish to copy the database TO.

The location of the uploads folder may be different depending on your wordpress setup, so these locations must be defined in the corresponding environment file in `/config/environments/`

You cannot sync to `production`. You can only sync TO or FROM `vm` with `local`




























                                                  $$$ 
                                                 $$$$$$$
                                                $$$$$$$$                 
                                                $$$$   $$ $$$$$ 
                                                   $$$$$$$$$$$$$$$$$$        $$$$$     
                                                   $$$$$$$$$$$$$$$$$$$     $$$$$$$$$                                                                                      $
                                                $$$$$$$$$$$$$$$$$$$$$$$$$$$   $$$                                                                                       $$$ 
                                             $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$    $$$$$$$$$$                                                                        $$$$$
                                           $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$        $$$$$$$$$$$$$$                         $$$$$$$$$$                                  $$$$$$
               $$$$$$$$$$$$$$$          $$$$$$$$$$$$$$$$$  $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$    $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$                           $$$$$$$$
            $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$ $$$$$$$$$$$$$$$$$$$$  $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$                         $$$$$$$
        $$$$$$ $$$$   $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$   $$$$$$$$$$$$$$$$$$$$$$$$$$$ $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$                       $$$$$
       $$$$$  $$$$$$$  $$$$$$$$$$      $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$  $$$$$$$$$$$$$$$$$$$$$$$$$$$$  $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$                      $$$$$
      $$$$$$$$$     $$$$$$$$$$$$$$$$$$$$  $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$  $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$  $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$                       $
      $$$$$$$$$$$$   $$$$$$$$$$$$      $$ $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$ $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$  $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$                     $ 
      $$$$$$$$$$$$   $$$$$$$$$$$ $$$   $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$  $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$  $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$                      $
      $$$$$$$$$$$$$$$$$$$$$$$$$ $$$$$  $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$  $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$ $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$                      $
      $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$  $$$$$$$$$$$$$$$$$$$   $$$$$$ $$$$$$$$$$$$$$$$$$$$$$$$$$ $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$                     $
      $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$   $$$$$$$$$$$$$$$$$$$  $$$$$$ $$$$$$$$$$$$$$$$$$$$$$$$$$ $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$                     $
       $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$  $$$$$$$$$$$$$$$$$$  $$$$$$  $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$                    $ 
         $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$      $$$$$$$$$$$$$$$$  $$$$$$  $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$                    $
               $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$         $$$$$$$$$$$$$$$$  $$$$$$  $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$                    $
              @@@            $$$$$$$$$$$$$$$$$$$$$$         @@     $$$$$$$$$$$$$$$  $$$$$$ $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$               $
              @@@@@@                                  @@@@@@@@@@    $$$$$$$$$$$$$$  $$$$$  $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$  $$$$$$$         $$ 
              @@@@@@            @@@@@@@               @@@@@@@@@@@  $$$$$$$$$$$$$$  $$$$$  $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$    $$$$$$$$$$$$$$$ 
               @@@@@             @@@@@@              @@@@@@@@@@@@ $$$$$$$$$$$$$$$ $$$$$   $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$        $$$$$$$$$$
                                 @@@@@@            @@@@@@@@@@@@@  $$$$$$$$$$$$$$ $$$$$ $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
                                                 @@@@@@@@@@@@@@  $$$$$$$$$$$$$  $$$$  $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
                                                @@@@@@@@@@@@@@@ $$$$$$$$$$$$$ $$$$   $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$ 
                                             @@@@@@@@@@@@@@@   $$$$$$$$$$$$  $$$   $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
                                       @@@@@@@@@@@@@@@@@@@@   $$$$$$$$$$$  $$   $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
                                    @@@@@@@@@@@@@@@@@@@@@   $$$$$$$$$$$$$     $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
                                    @@@@@@@@@@@@@@@@@    $$$$$$$$$$$$      $$$$$$$   $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$ $$$$$$$$$$$$$$$$$$$$$ 
                                     @@@@@@@@@@     $$$$$$$$$$       $$$$$$   $$$$   $$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$  $$$$$$$$$$$$$$$$$$$$$$
                                               $$$$$$$$$$$$$     $$$$$$$$$$   $$$$   $$$$$$$$$$$$$$$$$$$$   $$$$$$$$$$$$$$$$$   $$$$$$$$$$$$$$$$$$$$$
                                    $$$$$$$$$$$$$$$$$$$$       $$$$$$$$$$$$$$   $$$  $$$$$$$$$$$$$$$$$$$$   $$$$$$$$$$$$$$$    $$$$$$$$$$$$$$$$$$$$
                                             $$$$$$$$          $$$$$$$$$$$$$$$        $$$$$$$$$$$$$$$$$$$  $$$$$$$              $$$$$$$$$$$$$$$$$$
                                                                $$$$$$$$$$$$$$$$$$$   $$$$$$$$$$$$$$$$$$                //////  $$$$$$$$$$$$$$$$$
                                                                  $$$$$$$$$$$$$$$$$$  $$$$$$$$$$$$$$$$$     //////////////////  $$$$$$$$$$$$$$$$$
                                                                  $$$$$$$$$$$$$$$$$$  $$$$$$$$$$$$$$$$$       ////////////////  $$$$$$$$$$$$$$$$$
                                                                    $$$$$$$$$$$$$$$$   $$$$$$$$$$$$$$$         ///////////////   $$$$$$$$$$$$$$$$
                                                                    $$$$$$$$$$$$$$$$   $$$$$$$$$$$$$$$         ////////////////  $$$$$$$$$$$$$$$     
                                                                     $$$$$$$$$$$$$$$   $$$$$$$$$$$$$$$           ///  /////////  $$$$  $$$$$$$$$  
                                                                        $    $$$$$$$  $  $$$$$$$$$$$$            /@@@   @@  ///  $  @@@      $$$
                                                                     @@    @  $$$$$       $$    $$$$              @@@@  @@@  /   $  @@@@  @@@ $
                                                                                       @@@   @@@$$$

