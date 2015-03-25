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

### Boilerplate setup for AWS

Instructions for: ubuntu-trusty-14.04-amd64-server-20150123 (ami-9a562df2)

#### Setup ZSH and GIT

    sudo apt-get update
    sudo apt-get install language-pack-en
    sudo apt-get install zsh
    sudo apt-get install git
    curl -L https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh | sh

Note, the last line usually has a PAM authentication error when trying to change the default shell. To solve this, change shell using this method after the above:

    sudo chsh -s $(which zsh) ubuntu

Finally, `exit` the session and re-connect for changes to apply.

#### Install private key for service account

On your local machine (i.e. not in the above SSH shell):

1.  Download the `fifthweek-services` private key to your machine (we'll assume you save it to `~/.ssh/fifthweek-services`).

2.  Transfer the file to the test machine:

        scp ~/.ssh/fifthweek-services ubuntu@XX.XX.XX.XX:~/.ssh/fifthweek-services

3.  Install the above key by entering the following (in the SSH session this time):

        echo IdentityFile ~/.ssh/fifthweek-services >> ~/.ssh/config

4.  Prevent SSH complaining about GitHub's signature:

        ssh -o StrictHostKeyChecking=no git@github.com

#### Install Java and NPM

    sudo apt-get -q -y install default-jre
    sudo apt-get -q -y install nodejs
    sudo apt-get -q -y install nodejs-legacy
    sudo apt-get -q -y install npm
    npm config set prefix ~/npm

    echo . ~/.environment-variables >> ~/.zshrc
    echo export PATH="\$PATH:\$HOME/npm/bin" >> ~/.environment-variables

    source ~/.zshrc

#### Set BrowserStack environment variables

Substitute `<key>` and `<username` with service account credentials.

    echo export BROWSER_STACK_ACCESS_KEY=<key> >> ~/.environment-variables
    echo export BROWSER_STACK_USERNAME=<username> >> ~/.environment-variables
    source ~/.zshrc

#### Install the scheduled task

    git clone git@github.com:fifthweek/fifthweek-web-test-machine.git
    cd fifthweek-web-test-machine
    ./install

