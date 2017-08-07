# logrotate Cookbook

[![Build Status](https://secure.travis-ci.org/stevendanna/logrotate.svg?branch=master)](http://travis-ci.org/stevendanna/logrotate) [![Cookbook Version](https://img.shields.io/cookbook/v/logrotate.svg)](https://supermarket.chef.io/cookbooks/logrotate)

Manages the logrotate package and provides a resource to manage application specific logrotate configuration.

## Requirements

### Platforms

Should work on any platform that includes a 'logrotate' package and writes logrotate configuration to /etc/logrotate.d. Tested on Ubuntu and Centos.

### Chef

- Chef 12.5+

### Cookbooks

- none

## Recipes

### global

Generates and controls a global `/etc/logrotate.conf` file that will include additional files generated by the `logrotate_app` resource (see below). The contents of the configuration file is controlled through node attributes under `node['logrotate']['global']`. The default attributes are based on the configuration from the Ubuntu logrotate package.

To define a valueless directive (e.g. `compress`, `copy`) simply add an attribute named for the directive with a truthy value:

```ruby
node['logrotate']['global']['compress'] = 'any value here'
```

Note that defining a valueless directive with a falsey value will not make it false, but will remove it:

```ruby
# Removes a defaulted 'compress' directive; does not add a 'nocompress' directive.
node.override['logrotate']['global']['compress'] = false
```

To fully override a booleanish directive like `compress`, you should probably remove the positive form and add the negative form:

```ruby
node.override['logrotate']['global']['compress'] = false
node.override['logrotate']['global']['nocompress'] = true
```

The same is true of frequency directives; to be certain the frequency directive you want is included in the global configuration, you should override the ones you don't want as false:

```ruby
%w[ daily weekly yearly ].each do |freq|
  node.override['logrotate']['global'][freq] = false
end
node.override['logrotate']['global']['monthly'] = true
```

To define a parameter with a value (e.g. `create`, `mail`) add an attribute with the desired value:

```ruby
node['logrotate']['global']['create'] = '0644 root adm'
```

To define a path stanza in the global configuration (generally unneeded because of the `logrotate_app` resource) just add an attribute with the path as the name and a hash containing directives and parameters as described above:

```ruby
node['logrotate']['global']['/var/log/wtmp'] = {
  'missingok' => true,
  'monthly'   => true,
  'create'    => '0660 root utmp',
  'rotate'    => 1
}
```

`firstaction`, `prerotate`, `postrotate`, and `lastaction` scripts can be defined either as arrays of the lines to put in the script or multiline strings:

```ruby
node['logrotate']['global']['/var/log/foo/*.log'] = {
  'missingok'  => true,
  'monthly'    => true,
  'create'     => '0660 root adm',
  'rotate'     => 1,
  'prerotate'  => ['service foo start_rotate', 'logger started foo service log rotation'],
  'postrotate' => <<-EOF
    service foo end_rotate
    logger completed foo service log rotation
  EOF
}
```

## Resources

### logrotate_app

This resource can be used to drop off customized logrotate config files on a per application basis.

The resource takes the following properties:

- `path`: specifies a single path (string) or multiple paths (array) that should have logrotation stanzas created in the config file. No default, this must be specified.
- `cookbook`: The cookbook that continues the template for logrotate_app config resources. By default this is `logrotate`. Users can provide their own template by setting this attribute to point at a different cookbook.
- `template_name`: sets the template source, default is "logrotate.erb".
- `template_mode`: the mode to create the logrotate template with (default: "0644")
- `template_owner`: the owner of the logrotate template (default: "root")
- `template_group`: the group of the logrotate template (default: "root")
- `frequency`: sets the frequency for rotation. Default value is 'weekly'. Valid values are: hourly, daily, weekly, monthly, yearly, see the logrotate man page for more information. Note that usually logrotate is configured to be run by cron daily. You have to change this configuration and run logrotate hourly to be able to really rotate logs hourly. Hourly rotation requires logrotate v3.8.5 or higher.
- `options`: Any logrotate configuration option that doesn't specify a value. See the logrotate(8) manual page of v3.9.2 or earlier for details.

In addition to these properties, any logrotate option that takes a parameter can be used as a logrotate_app property. For example, to set the `rotate` option you can use a resource declaration such as:

```ruby
logrotate_app 'tomcat-myapp' do
  path      '/var/log/tomcat/myapp.log'
  frequency 'daily'
  rotate    30
  create    '644 root adm'
end
```

See the logrotate(8) manual page of v3.9.2 or earlier for the list of available options.

## Usage

The default recipe will ensure logrotate is always up to date.

To create application specific logrotate configs, use the `logrotate_app` resource. For example, to rotate logs for a tomcat application named myapp that writes its log file to `/var/log/tomcat/myapp.log`:

```ruby
logrotate_app 'tomcat-myapp' do
  path      '/var/log/tomcat/myapp.log'
  frequency 'daily'
  rotate    30
  create    '644 root adm'
end
```

To rotate multiple logfile paths, specify the path as an array:

```ruby
logrotate_app 'tomcat-myapp' do
  path      ['/var/log/tomcat/myapp.log', '/opt/local/tomcat/catalina.out']
  frequency 'daily'
  create    '644 root adm'
  rotate    7
end
```

To specify which logrotate options, specify the options as an array:

```ruby
logrotate_app 'tomcat-myapp' do
  path      '/var/log/tomcat/myapp.log'
  options   ['missingok', 'delaycompress', 'notifempty']
  frequency 'daily'
  rotate    30
  create    '644 root adm'
end
```

## License & Authors

- Author:: Steven Danna ([steve@chef.io](mailto:steve@chef.io))
- Author:: Scott M. Likens ([scott@likens.us](mailto:scott@likens.us))
- Author:: Joshua Timberman ([joshua@chef.io](mailto:joshua@chef.io))

```text
Copyright 2009, Scott M. Likens
Copyright 2011-2012, Chef Software, Inc.
Copyright 2016, Steven Danna

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```
