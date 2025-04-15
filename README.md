# Mac workstation playbook

_Fork from Jeff Geerling's mac-dev-playbook._

_Let's be honest, this is overkill if you only have a single mac. However, it's very useful as soon as you have multiple. For me, it has meant that I no longer need to lug around a large Macbook Pro; instead, I have a Mac Studio at home and a Macbook Air when travelling._

_Perhaps more importantly, it is the perfect project for learning about Ansible, which is one of the biggest open-source projects in the world, and a great skill to have if you have any interest in server management, "home labs" or anything like that._

_I recommend watching some of  Jeff's ["Ansible 101" series](https://www.youtube.com/playlist?list=PL2_OBreMn7FqZkvMYt6ATmgC0KAGGJNAN) if you'd like to learn about Ansible, and get enough of an understanding to understand this repo. The first six episodes are necessary to get the basics._

## Installation and new Mac setup

1. Run through Apple's mandtory setup wizard.

1. Avoid opening too many Finder windows before you have set Finder defaults that you are happy with, because the windows you have already opened will retain their config. (My `.osx-defaults.sh` script, which is invoked by this playbook, sets most of my preferred defaults.)

1. Sign into Internet Accounts:
	- iCloud for everything except for Mail, Contacts, Calendar.
	- Personal Gmail for Contacts

1. Sign into App Store (since `mas` can't sign in automatically).

1. Install Command Line Tools, update PATH, install Ansible.

	`xcode-select --install`
	
	`export PATH="$HOME/Library/Python/3.9/bin:/opt/homebrew/bin:$PATH"`
	
	`pip3 install ansible`

1. Set up SSH key for GitHub:

	`ssh-keygen -t ed25519 -C "8040671+mjfdevere5@users.noreply.github.com"`

	`pbcopy < ~/.ssh/id_ed25519.pub`

	Paste this into [GitHub key settings](https://github.com/settings/keys), and test connection with:
	
	`ssh -T git@github.com`

1. Clone this repo, e.g.:
	```sh
	mkdir -p ~/Coding && git clone git@github.com:mjfdevere5/mac-setup.git ~/Coding/mac-setup
	```

1. Install requirements:
	```sh
	cd ~/Coding/mac-setup && ansible-galaxy install -r requirements.yml
	```

1. Configure the playbook, by editing `config.yml`  to suit your needs. This will override the variables in `default.config.yml`. (I try not to touch the defaults, for ease of tracking the upstream repo.)

1. Run the playbook, entering macOS password (must be admin) when prompted:
	```sh
	ansible-playbook main.yml --ask-become-pass
	```

	This playbook will:
	- Load `default.config.yml`, and `config.yml` if it exists.
	- Ensure the OSX command line tools are installed.
	- Ensure Homebrew is installed, and uses it to install packages/casks.
	- Ensure dotfiles are installed.
	- Ensure mas is installed, and uses it to install App Store apps.
	- Ensure dockutil is installed, and uses it to configure the dock.
	- Configure sudoers (I have this switched off).
	- Configure the Terminal app (I have this switched off).
	- Configure OSX settings and defaults via a script.
	- Ensure extra packages are installed, e.g. Composer, NPM, Pip, Ruby gems.
	- Configure Sublime Text.
	- Runs post-provision task files (nothing by default).

	> Note: If some Homebrew commands fail, you might need to agree to Xcode's license or fix some other Brew issue. Run `brew doctor` to see if this is the case.

1. Start Synchronization tasks:
	- Open Photos and make sure iCloud sync options are correct.
	- Ensure iCloud is syncing Documents and Desktop.

1. 🚧 TODO: Transfer a copy of all non-Documents non-Desktop files into the `$HOME` folder.
	- Archives [SHOULD LIVE ON A NAS]
	- Coding [CAN ALSO BE CLONED IN, BUT MANUAL TRANSFER IS QUICKER]
	- Downloads [CONFIGURE TO SYNC WITH NAS]
	- Movies (ignoring Apple's "TV" folder) [SHOULD LIVE ON A NAS]
	- Music (ignoring Apple's "Music" folder) [SHOULD LIVE ON A NAS]

1. 🚧 TODO: For apps which need to have their settings configured manually, do so. For me, these are:
	- App1
	- App2
	- App3
	- ...

## Thereafter, keep macs in sync

Periodically, pull any changes and run the playbook:
```sh
ansible-playbook main.yml --ask-become-pass
```

> Note: You can filter which part of the provisioning process to run by specifying a set of tags using ansible-playbook's `--tags` flag, e.g. `--tags "osx,homebrew,dotfiles"`. See `main.yml` for all the tags. This shouldn't be necessary, since everything is idempotent and that is largely the point of using Ansible in the first place.