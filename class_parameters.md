**Puppet should let you customize all its important variables without editing the module itself. This is done with class parameters.**

## Writing a parameterized class

```puppet
class pasture (

  $port                = '80',
  $default_character   = 'sheep',
  $default_message     = '',
  $pasture_config_file = '/etc/pasture_config.yaml',
)

  {

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

## Resource-like class declarations

Open your site.pp manifest.
```
vim /etc/puppetlabs/code/environments/production/manifests/site.pp
```

And modify your node definition for pasture.puppet.vm to include a resource-like class declartion. We'll set the default_character parameter to the string 'cow', and leave the other two parameters unset, letting them take their default values.
```puppet
node 'pasture.puppet.vm' {
  class { 'pasture':
    default_character => 'cow',
  }
}
```