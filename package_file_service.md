# package, file and service are resources ! => resource ordering ?

### "package, file, service" => pattern because "always" use together
```
 /!\

 A package resource manages the software package itself, a file resource allows you to customize a related configuration file, and a service resource starts the service provided by the software you've just installed and configured.
 
 /!\
```


# Exemple with pasture
## Package
```
vim pasture/manifests/init.pp
```
So far, this should look just like the cowsay class, except that the package resource will manage the pasture package instead of cowsay.
```puppet
class pasture {
  package { 'pasture':
    ensure   => present,
    provider => gem,
  }
}
```
```
vim /etc/puppetlabs/code/environments/production/manifests/site.pp
```
Create a new node definition for the pasture.puppet.vm node we've set up for this quest. Include the pasture class you just wrote.
```puppet
node 'pasture.puppet.vm' {
  include pasture
}
```

## File

### first
```
vim pasture/files/pasture_config.yaml
```
Include a line here to set the default character to elephant.
```
---
:default_character: elephant
```
```
vim pasture/manifests/init.pp
```
And add a file resource declaration.

```puppet
class pasture {

  package {'pasture':
    ensure   => present,
    provider => 'gem',
  }

  file { '/etc/pasture_config.yaml':
    source => 'puppet:///modules/pasture/pasture_config.yaml',
  }
}
```


## Service

```sh
vim pasture/files/pasture.service
```
Include the following contents:

```sh
[Unit]
Description=Run the pasture service

[Service]
Environment=RACK_ENV=production
ExecStart=/usr/local/bin/pasture start

[Install]
WantedBy=multi-user.target
```

```
vim pasture/manifests/init.pp
```

```puppet
class pasture {

  package {'pasture':
    ensure   => present,
    provider => 'gem',
  }

  file { '/etc/pasture_config.yaml':
    source => 'puppet:///modules/pasture/pasture_config.yaml',
  }

  file { '/etc/systemd/system/pasture.service':
    source => 'puppet:///modules/pasture/pasture.service',
  }

  service { 'pasture':
    ensure => running,
  }

}
```

## Ordering:

```
vim pasture/manifests/init.pp
```
Add relationship metaparameters to define the dependencies among your package, file, and service resources. Take a moment to think through what each of these relationship metaparameters does and why.

```puppet
class pasture {

  package {'pasture':
    ensure   => present,
    provider => 'gem',
    before   => File['/etc/pasture_config.yaml'],
  }

  file { '/etc/pasture_config.yaml':
    source  => 'puppet:///modules/pasture/pasture_config.yaml',
    notify  => Service['pasture'],
  }

  file { '/etc/systemd/system/pasture.service':
    source  => 'puppet:///modules/pasture/pasture.service',
    notify  => Service['pasture'],
  }

  service { 'pasture':
    ensure => running,
  }

}
```