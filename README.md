# Fifthweek Web Test Machine

Provisions a machine to perform scheduled runs of the entire test suite for `fifthweek-web` and to communicate results back to developers.

This is not part of the CI pipeline, so errors will likely be reported after change sets have been deployed live.

## Connecting to the test machine

1.  Save the `fifthweek-web-test-machine` private key to a location on your development machine.

2.  Add this to the `ssh-agent`:

        chmod 400 ~/.ssh/fifthweek-web-test-machine.pem
        ssh-add ~/.ssh/fifthweek-web-test-machine.pem

3.  Connect to the machine:

        ssh ubuntu@XX.XX.XX.XX

## Deploying to Ubuntu 14.04 (64 bit)

Instructions have been tested against AWS on the following AMI: 

    ubuntu-trusty-14.04-amd64-server-20150123 (ami-9a562df2)

### Setup ZSH and GIT

    sudo apt-get update
    sudo apt-get install language-pack-en
    sudo apt-get install zsh
    sudo apt-get install git
    curl -L https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh | sh

Note, the last line usually has a PAM authentication error when trying to change the default shell. To solve this, change shell using this method after the above:

    sudo chsh -s $(which zsh) ubuntu

Finally, `exit` the session and re-connect for changes to apply.

### Install private key for service account

On your local machine (i.e. not in the above SSH shell):

1.  Download the `fifthweek-services` private key to your machine (we'll assume you save it to `~/.ssh/fifthweek-services`).

2.  Transfer the file to the test machine:

        scp ~/.ssh/fifthweek-services ubuntu@XX.XX.XX.XX:~/.ssh/fifthweek-services

3.  Install the above key by entering the following (in the SSH session this time):

        echo IdentityFile ~/.ssh/fifthweek-services >> ~/.ssh/config

4.  Prevent SSH complaining about GitHub's signature:

        ssh -o StrictHostKeyChecking=no git@github.com

### Install Java and NPM

    sudo apt-get -q -y install default-jre
    curl -sL https://deb.nodesource.com/setup | sudo bash -
    sudo apt-get install -y nodejs
    sudo npm install npm -g
    npm config set prefix ~/npm

    echo . ~/.environment-variables >> ~/.zshrc
    echo export PATH="\$PATH:\$HOME/npm/bin" >> ~/.environment-variables

    source ~/.zshrc

### Set BrowserStack environment variables

Substitute `<key>` and `<username` with service account credentials.

    echo export BROWSER_STACK_ACCESS_KEY=<key> >> ~/.environment-variables
    echo export BROWSER_STACK_USERNAME=<username> >> ~/.environment-variables
    source ~/.zshrc

### Install the scheduled task

    git clone git@github.com:fifthweek/fifthweek-web-test-machine.git
    cd fifthweek-web-test-machine
    ./install

## Updating the test script

    cd ~/fifthweek-web-test-machine
    git pull
    ./install

## Manually running the tests

Replace `<branch>` with the branch name. Note: when testing against new branches, ensure a same-named branch exists in `fifthweek/fifthweek-web-test-machine-logs`.

    cd ~/fifthweek-web-test-machine
    ./test-branch <branch>

## Adding a new scheduled test suite / branch

The test script is scheduled via `crontab` jobs configured in the `install` script. To add a new scheduled test run (perhaps for a different branch), simply duplicate a `crontab` line in the `install` script and be sure to change the branch and schedule time -- you cannot run multiple tests in parallel.
