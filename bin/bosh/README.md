# Base Orchestration & Synchronisation Hippopotamus

## Usage

After cloning/installing via composer, move the /bin and /config folders and the BoshFile to the root of your project.
copy the example.env file and rename to production, update the file with your production details. Repeat for any staging, dev, vm etc environments you may have.

    ./bin/bosh/deploy <environment-name> <git-branch-or-tag>