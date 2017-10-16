# agent_run

## On Master
###  affiche certificat des slaves ?
```sh
puppet cert list
```
### sign certif
```sh
puppet cert sign "{serverSlaveName}"
```

## On slave
### trigger a puppet agent
```sh
sudo puppet agent -t
```

## On master
### To see what your configured modulepath is, run the following command:
```sh
puppet config print modulepath
```
```sh
/etc/puppetlabs/code/environments/production/modules:/etc/puppetlabs/code/modules:/opt/puppetlabs/puppet/modules
```

# CLASSES / MODULES / MANIFESTS

## Find the configuration of puppet:
```sh
puppet config print modulepath
```

## Module are stock here:
```sh
cd /etc/puppetlabs/code/environments/production/modules
```

## Create a new module

```sh
vim cowsay/manifests/init.pp
```

```puppet
class cowsay {
  package { 'cowsay':
    ensure   => present,
    provider => 'gem',
  }
}
```
## Validate module

```sh
puppet parser validate cowsay/manifests/init.pp
```

## call the module in site.pp

```sh
vim /etc/puppetlabs/code/environments/production/manifests/site.pp
```

### At the end of the site.pp manifest, insert the following code:

```puppet   
node 'cowsay.puppet.vm' {
  include cowsay
}
```

##  the --noop flag to do a practice run of the Puppet agent.

```sh
sudo puppet agent -t --noop
```


## Composed classes and class scope
```puppet
class cowsay::fortune {
  package { 'fortune-mod':
    ensure => present,
  }
}
```
## add fortune to cowsay (scope scope)
```
vim cowsay/manifests/init.pp
```
## Add an include statement for your cowsay::fortune class to the cowsay class.
```puppet
class cowsay {
  package { 'cowsay':
    ensure   => present,
    provider => 'gem',
  }
  include cowsay::fortune
}
```