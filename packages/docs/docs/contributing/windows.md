# Building on Windows

Many of the build scripts are bash scripts and not natively invocable in Windows. To solve this, you can build the project using Git Bash.

1. Install [Git & Git Bash for Windows](https://git-scm.com/downloads).
2. Ensure you have activated [Developer Mode](https://docs.microsoft.com/en-us/windows/apps/get-started/enable-your-device-for-development).
3. Install Node v22.x or greater.
4. Clone this repo.
5. Using Git Bash (run as administrator), change to the root of this repo.
6. From inside the bash shell, run `yarn install`.
7. From still inside the shell, run `yarn start:browser`.
8. Open your browser to [http://localhost:3001](http://localhost:3001).

# Errors

## `rsync: command not found`

If you run into this error, you will need to install the rsync binary to Git Bash. Follow the [directions here](https://prasaz.medium.com/add-rsync-to-windows-git-bash-f42736bae1b3).

## `ln: failed to create symbolic link '../../desktop-client/public/kcab': Operation not permitted`

Bash needs to be run as administrator.
