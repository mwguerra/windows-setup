# windows-setup
Steps to reproduce the initial setup of a Windows backup development notebook.

- Update Windows
- Install Google Chrome
- Install Visual Studio Code
- Install GitHub Desktop
- Install Oracle Virtual Box
- Install Hyper Terminal
- Setup Windows developer mode and turn on the Windows Subsystem for Linux (WSL) feature
- Install Ubuntu from the Windows Store and run/install it

- Windows path to the WSL Ubuntu root
```
%LOCALAPPDATA%\Packages\CanonicalGroupLimited.UbuntuonWindows_79rhkp1fndgsc\LocalState\rootfs
```

# Hyper Terminal

- Open the bash prompt
- Install Zsh and Oh My Zsh

```
sudo apt install zsh

sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

- Download the Spaceship theme for Zsh:

```
git clone https://github.com/denysdovhan/spaceship-prompt.git "$ZSH_CUSTOM/themes/spaceship-prompt"

ln -s "$ZSH_CUSTOM/themes/spaceship-prompt/spaceship.zsh-theme" "$ZSH_CUSTOM/themes/spaceship.zsh-theme"
```

- Setup Zsh to be called automatically by Bash. Edit the ~/.bashrc file and append to the file:

```
cd ~
bash -c zsh
exit
```

- Paste the lines below at the end of the ~/.zshrc file:

```
export PATH="$PATH:/mnt/c/Program Files/Oracle/VirtualBox"
export PATH=$PATH:~/.config/composer/vendor/bin
export VAGRANT_WSL_ENABLE_WINDOWS_ACCESS="1"

alias ll='ls -alFh'
alias code='cd /mnt/c/Users/[yourusername]/Documents/code'

ZSH_THEME="spaceship"
SPACESHIP_PROMPT_ORDER=(
  user          # Username section
  dir           # Current directory section
  host          # Hostname section
  git           # Git section (git_branch + git_status)
  hg            # Mercurial section (hg_branch + hg_status)
  exec_time     # Execution time
  line_sep      # Line break
  vi_mode       # Vi-mode indicator
  jobs          # Background jobs indicator
  exit_code     # Exit code section
  char          # Prompt character
)
SPACESHIP_USER_SHOW=always
SPACESHIP_PROMPT_ADD_NEWLINE=false
SPACESHIP_CHAR_SYMBOL="❯"
SPACESHIP_CHAR_SUFFIX=" "

function homestead() {
    ( cd /mnt/c/Users/[yourusername]/Homestead && vagrant $* )
}
```

- Download the Fira Code fonts at https://github.com/tonsky/FiraCode. Unzip and double click the TTF to install.

- Open the Hyper config file (CTRL+',') and change the default shell to be bash and the font family (just add "Fira Code" before Menlo):

```
shell: 'C:\\Windows\\System32\\bash.exe'
fontFamily: '"Fira Code", Menlo, ...
```

# Update packages and upgrade the WSL Ubuntu
```
sudo apt-get update
sudo apt-get upgrade
sudo -S apt-mark hold procps strace sudo
sudo -S env RELEASE_UPGRADER_NO_SCREEN=1 do-release-upgrade
```

# Install PHP, Composer, Node, Yarn and Vagrant

- PHP Installation

```
sudo apt-get update
sudo apt -y install software-properties-common
sudo add-apt-repository ppa:ondrej/php
sudo apt-get update
sudo apt -y install php7.4
sudo apt-get install -y php7.4-{bcmath,bz2,intl,gd,mbstring,mysql,zip,dom}
sudo apt-get install -y php7.4-dom
```

- Composer Installation

```
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php -r "if (hash_file('sha384', 'composer-setup.php') === 'e0012edf3e80b6978849f5eff0d4b4e4c79ff1609dd1e613307e16318854d24ae64f26d17af3ef0bf7cfb710ca74755a') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
php composer-setup.php
php -r "unlink('composer-setup.php');"
sudo mv composer.phar /usr/local/bin/composer
```

- Laravel Installer Installation

```
sudo apt-get install unzip
composer global require laravel/installer
```

- NodeJs 13.x Installation

```
curl -sL https://deb.nodesource.com/setup_13.x | sudo -E bash -
sudo apt-get install -y nodejs
```

- Yarn Installation (without Node)

```
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
sudo apt update && sudo apt install --no-install-recommends yarn
```

- Vagrant and Homestead Installation

```
sudo apt update
curl https://releases.hashicorp.com/vagrant/2.2.7/vagrant_2.2.7_x86_64.deb -O
sudo dpkg -i vagrant_2.2.7_x86_64.deb
rm vagrant_2.2.7_x86_64.deb
vagrant -v
vagrant box add laravel/homestead
git clone https://github.com/laravel/homestead.git /mnt/c/Users/[yourusername]/Homestead
cd /mnt/c/Users/[yourusername]/Homestead
bash init.sh
```

- Append to the Vagrantfile file in the Homestead folder

```
config.vm.provider "virtualbox" do |v|
    v.customize ["setextradata", :id, "VBoxInternal2/SharedFoldersEnableSymlinksCreate/v-root", "1"]
end
```

- Create the ssh key

```
ssh-keygen -o
```

- Setup the local folder at the Homestead.yaml file to be /mnt/c/Users/[yourusername]/Documents/code

```
folders:
    - map: /mnt/c/Users/[yourusername]/Documents/code
      to: /home/vagrant/code
```


- Start the Homestead VM

```
homestead up
```

# Warning message suppression

- In order to supress the "Insecure world writable dir /home/[yourusername]/.config/composer/vendor/bin" warning:

```
sudo chmod 755 ~/.config -R
```

- In order to supress the "Insecure world writable dir /mnt/c" warning, create the /etc/wsl.conf file:

```
sudo nano /etc/wsl.conf
```
```
[automount]
options="metadata,umask=0033"
```


