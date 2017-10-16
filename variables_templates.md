# Variables:
```
vim pasture/manifests/init.pp
```
Assign these variables at the top of your class. Replace the hard-coded references to the /etc/pasture_config.yaml configuration filepath with the variable.
```puppet
class pasture {

  $port                = '80'
  $default_character   = 'sheep'
  $default_message     = ''
  $pasture_config_file = '/etc/pasture_config.yaml'

  package {'pasture':
    ensure   => present,
    provider => 'gem',
    before   => File[$pasture_config_file],
  }
  file { $pasture_config_file:
    source  => 'puppet:///modules/pasture/pasture_config.yaml',
    notify  => Service['pasture'],
  }
  file { '/etc/systemd/system/pasture.service':
    source => 'puppet:///modules/pasture/pasture.service',
    notify  => Service['pasture'],
  }
  service { 'pasture':
    ensure    => running,
  }
}
```

# Templates:

Next, create a pasture_config.yaml.epp template file.

vim pasture/templates/pasture_config.yaml.epp
```puppet
<%- | $port,
      $default_character,
      $default_message,
| -%>
# This file is managed by Puppet. Please do not make manual changes.
---
:default_character: <%= $default_character %>
:default_message:   <%= $default_message %>
:sinatra_settings:
  :port: <%= $port %>
```

Create
```sh
vim pasture/templates/pasture.service.epp
```

```puppet
<%- | $pasture_config_file = '/etc/pasture_config.yaml' | -%>
# This file is managed by Puppet. Please do not make manual changes.
[Unit]
Description=Run the pasture service

[Service]
Environment=RACK_ENV=production
ExecStart=/usr/local/bin/pasture start --config_file <%= $pasture_config_file %>

[Install]
WantedBy=multi-user.target
```
```
vim pasture/manifests/init.pp
```

```puppet
class pasture {

  $port                = '80'
  $default_character   = 'sheep'
  $default_message     = ''
  $pasture_config_file = '/etc/pasture_config.yaml'

  package {'pasture':
    ensure   => present,
    provider => 'gem',
    before   => File[$pasture_config_file],
  }
  $pasture_config_hash = {
    'port'              => $port,
    'default_character' => $default_character,
    'default_message'   => $default_message,
  }
  file { $pasture_config_file:
    content => epp('pasture/pasture_config.yaml.epp', $pasture_config_hash),
    notify  => Service['pasture'],
  }
  $pasture_service_hash = {
    'pasture_config_file' => $pasture_config_file,
  }
  file { '/etc/systemd/system/pasture.service':
    content => epp('pasture/pasture.service.epp', $pasture_service_hash),
    notify  => Service['pasture'],
  }
  service { 'pasture':
    ensure    => running,
  }
}
```

=> Test

```
curl 'pasture.puppet.vm/api/v1/cowsay?message=Hello!'
```