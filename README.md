# GitLab Omnibus project

This project creates full-stack platform-specific packages for
GitLab!

## How to manage an Omnibus-installed GitLab

### Administrative commands

You can make configuration changes by editing `/etc/gitlab/gitlab.rb` and
`/etc/gitlab/gitlab-secrets.json`, followed by running

```
sudo gitlab-ctl reconfigure
```

To start/stop a component of GitLab, run e.g.
`sudo gitlab-ctl stop sidekiq`. To permanently disable e.g. Sidekiq, add
`sidekiq['enable'] = false` to `/etc/gitlab/gitlab.rb`, and run
`sudo gitlab-ctl reconfigure` for the change to take effect.

To invoke a GitLab rake task, use `gitlab-rake`. For example:

```
sudo gitlab-rake gitlab:backup:create
```

There is no need to change the user or the `RAILS_ENV` environment variable;
this is taken care of by the `gitlab-rake` wrapper script.

### Directory structure

Omnibus-gitlab uses four different directories.

- `/opt/gitlab` holds application code for GitLab and its dependencies.
- `/var/opt/gitlab` holds application data and configuration files that
  `gitlab-ctl reconfigure` writes too.
- `/etc/gitlab` holds configuration files for omnibus-gitlab. These are
  the only files that you should ever have to edit manually.
- `/var/log/gitlab` contains all log data generated by components of
  omnibus-gitlab.

## Preparing a build environment on Ubuntu 12.04

To create builds you will need a build user (`ubuntu:ubuntu` in our example).
Preparing the build machine requires sudo access.

```shell
# Install dependencies
sudo apt-get install ruby1.9.1 ruby1.9.1-dev git build-essential
sudo gem install --no-ri --no-rdoc bundler

# Create build directories for use by the build user
sudo mkdir -p /opt/gitlab /var/cache/omnibus
# We assume the build user and group is 'ubuntu'
sudo chown ubuntu:ubuntu /opt/gitlab /var/cache/omnibus
```

Then, as the build user:

```shell
# Clone the omnibus repo
git clone https://gitlab.com/gitlab-org/omnibus-gitlab.git

# Install gem dependencies for omnibus-ruby
cd omnibus-gitlab
bundle install --path .bundle --binstubs
```

## Usage

### Build

You create a platform-specific package using the `build project` command:

```shell
$ bin/omnibus build project gitlab
```

The platform/architecture type of the package created will match the platform
where the `build project` command is invoked. So running this command on say a
MacBook Pro will generate a Mac OS X specific package. After the build
completes packages will be available in `pkg/`.

### Clean

You can clean up all temporary files generated during the build process with
the `clean` command:

```shell
$ bin/omnibus clean
```

Adding the `--purge` purge option removes __ALL__ files generated during the
build including the project install directory (`/opt/gitlab`) and
the package cache directory (`/var/cache/omnibus/pkg`):

```shell
$ bin/omnibus clean --purge
```

### Help

Full help for the Omnibus command line interface can be accessed with the
`help` command:

```shell
$ bin/omnibus help
```

## Vagrant-based Virtualized Build Lab

Every Omnibus project ships will a project-specific
[Berksfile](http://berkshelf.com/) and [Vagrantfile](http://www.vagrantup.com/)
that will allow you to build your projects on the following platforms:

* CentOS 5 64-bit
* CentOS 6 64-bit
* Ubuntu 10.04 64-bit
* Ubuntu 11.04 64-bit
* Ubuntu 12.04 64-bit

Please note this build-lab is only meant to get you up and running quickly;
there's nothing inherent in Omnibus that restricts you to just building CentOS
or Ubuntu packages. See the Vagrantfile to add new platforms to your build lab.

The only requirements for standing up this virtualized build lab are:

* VirtualBox - native packages exist for most platforms and can be downloaded
from the [VirtualBox downloads page](https://www.virtualbox.org/wiki/Downloads).
* Vagrant 1.2.1+ - native packages exist for most platforms and can be downloaded
from the [Vagrant downloads page](http://downloads.vagrantup.com/).

The [vagrant-berkshelf](https://github.com/RiotGames/vagrant-berkshelf) and
[vagrant-omnibus](https://github.com/schisamo/vagrant-omnibus) Vagrant plugins
are also required and can be installed easily with the following commands:

```shell
$ vagrant plugin install vagrant-berkshelf
$ vagrant plugin install vagrant-omnibus
```

Once the pre-requisites are installed you can build your package across all
platforms with the following command:

```shell
$ vagrant up
```

If you would like to build a package for a single platform the command looks like this:

```shell
$ vagrant up PLATFORM
```

The complete list of valid platform names can be viewed with the
`vagrant status` command.
