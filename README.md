#puppet-zookeeper

[![Puppet
Forge](http://img.shields.io/puppetforge/v/deric/zookeeper.svg)](https://forge.puppetlabs.com/deric/zookeeper) [![Build Status](https://travis-ci.org/deric/puppet-zookeeper.png?branch=master)](https://travis-ci.org/deric/puppet-zookeeper)

A puppet receipt for [Apache Zookeeper](http://zookeeper.apache.org/). ZooKeeper is a high-performance coordination service for maintaining configuration information, naming, providing distributed synchronization, and providing group services.

## Requirements

  * Puppet 2.7, Puppet 3.x
  * Ruby 1.8.7, 1.9.3, 2.0.0, 2.1.x
  * binary package of ZooKeeper

## Basic Usage:

```puppet
class { 'zookeeper': }
```

### Quorum

For setting up a quorum of ZooKeeper you should list all nodes in the quorum:

```puppet
class { 'zookeeper':
  servers => ['192.168.1.1', '192.168.1.2', '192.168.1.3']
}
```
Currently first ZooKeeper will have `ID = 1`. This would produce following configuration:
```
server.1=192.168.1.1:2888:3888
server.2=192.168.1.2:2888:3888
server.3=192.168.1.3:2888:3888
```
where first port is `election_port` and second one `leader_port`. Both ports could be customized for each ZooKeeper instance.

```puppet
class { 'zookeeper':
  election_port => 2889,
  leader_port   => 3889,
  servers       => ['192.168.1.1', '192.168.1.2', '192.168.1.3']
}
```

### Setting IP address

If `$::ipaddress` is not your public IP (e.g. you are using Docker) make sure to setup correct IP:

```puppet
class { 'zookeeper':
  client_ip => $::ipaddress_eth0
}
```

or in Hiera:

```yaml
zookeeper::client_ip: "%{::ipaddress_eth0}"
```

This is a workaround for a a [Facter issue](https://tickets.puppetlabs.com/browse/FACT-380).

##  Parameters

   - `id` - cluster-unique zookeeper's instance id (1-255)
   - `datastore`
   - `log_dir`
   - `purge_interval` - automatically will delete zookeeper logs (available since 3.4.0)
   - `snap_retain_count` - number of snapshots that will be kept after purging (since 3.4.0)

and many others, see the `init.pp` file for more details.

If your distribution has multiple packages for ZooKeeper, you can provide all package names
as an array.

```puppet
class { 'zookeeper':
  packages => ['zookeeper', 'zookeeper-java']
}
```

## Hiera Support

All parameters could be defined in hiera files, e.g. `common.yaml`, `Debian.yaml` or `zookeeper.yaml`:

```yaml
zookeeper::id: 1
zookeeper::client_port: 2181
zookeeper::datastore: '/var/lib/zookeeper'
```

## Cloudera package

In Cloudera distribution ZooKeeper package does not provide init scripts (same as in Debian). Package containing init scripts
is called `zookeeper-server` and the service as well. Moreover there's initialization script which should be called after installation.
So, the configuration might look like this:

```puppet
class { 'zookeeper':
  packages             => ['zookeeper', 'zookeeper-server'],
  service_name         => 'zookeeper-server',
  initialize_datastore => true
}
```

### Managing repository

For RedHat family curretly we support also managing an yum repo. It can be enabled with `repo` parameter:

```puppet
class { 'zookeeper':
  repo => 'cloudera'
}
```


## Java installation

Default: `false`

By changing these two parameters you can ensure, that given Java package will be installed before ZooKeeper packages.

```puppet
class { 'zookeeper':
  install_java => true,
  java_package => 'openjdk-7-jre-headless'
}
```

## Install

### librarian (recommended)

For [puppet-librarian](https://github.com/rodjek/librarian-puppet) just add to `Puppetfile`

from Forge:
```ruby
mod 'deric/zookeeper'
```

latest (development) version from GitHub
```ruby
mod 'deric/zookeeper', :git => 'git://github.com/deric/puppet-zookeeper.git'
```

### submodules

If you are versioning your puppet conf with git just add it as submodule, from your repository root:

    git submodule add git://github.com/deric/puppet-zookeeper.git modules/zookeeper

## Dependencies

  * stdlib `> 2.3.3` - function `ensure_resources` is required

## Supported platforms

  * Debian/Ubuntu
    * Debian 6 Squeeze: you can get ZooKeeper package from [Wheezy](http://packages.debian.org/wheezy/zookeeper) or [Sid](http://packages.debian.org/sid/zookeeper) repo.
    * Debian 7 Wheezy: available in apt repository
  * RedHat/CentOS/Fedora

### Tested on:

  * Debian 6 Squeeze, 7 Wheezy
  * Ubuntu 12.04.03 LTS, 14.04
  * CentOS 6

