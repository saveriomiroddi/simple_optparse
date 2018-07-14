[![Gem Version][GV img]](https://rubygems.org/gems/simple_scripting)
[![Build Status][BS img]](https://travis-ci.org/saveriomiroddi/simple_scripting)
[![Code Climate][CC img]](https://codeclimate.com/github/saveriomiroddi/simple_scripting)
[![Coverage Status][CS img]](https://coveralls.io/r/saveriomiroddi/simple_scripting)

# SimpleScripting

*The TabCompletion component is under development; please check this project again on (or before) Sat 21/Jul/2018.*

`SS` is a library composed of two modules (`Argv` and `Configuration`) which simplify two common scripting tasks:

- implementing the commandline options parsing (and the related help)
- loading and decoding the configuration for the script/application

`SS` is an interesting (and useful) exercise in design, aimed at finding the simplest and most expressive data/structures which accomplish the given task(s). For this reason, the library can be useful for people who frequently write small scripts (eg. devops or nerds).

## SimpleScripting::Argv

`SS::A` is a module which acts as frontend to the standard Option Parser library (`optparse`), giving a very convenient format for specifying the arguments. `SS::A` also generates the help.

This is a definition example:

    require 'simple_scripting/argv'

    result = SimpleScripting::Argv.decode(
      ['-s', '--only-scheduled-days',     'Only print scheduled days'                           ],
      ['-d', '--print-defaults TEMPLATE', 'Print the default activities from the named template'],
      'schedule',
      '[weeks]',
      long_help: 'This is the long help! It can span multiple lines.'
    ) || exit

which:

- optionally accepts the `-s`/`--only-scheduled-days` switch, interpreting it as boolean,
- optionally accepts the `-d`/`--print-defaults` switch, interpreting it as string,
- requires the `schedule` argument,
- optionally accepts the `weeks` argument,
- automatically adds the `-h` and `--help` switches,
- prints all the options and the long help if the help is invoked,
- exits with a descriptive error if invalid parameters are passed.

This is a sample result:

    {
      only_scheduled_days: true,
      print_defaults:      'my_defaults',
      schedule:            'schedule.txt',
      weeks:               '3',
    }

This is the corresponding help:

    Usage: tmpfile [options] <schedule> [<weeks>]
        -s, --only-scheduled-days        Only print scheduled days
        -d, --print-defaults TEMPLATE    Print the default activities from the named template
        -h, --help                       Help

    This is the long help! It can span multiple lines.

Commands are also supported (with unlimited depth), by using a hash:

    commands, result = SimpleScripting::Argv.decode(
      'pr' => {
        'create' => [
          'title',
          'description',
          long_help: 'This is the create PR command help.'
        ]
      },
      'issues' => {
        'list' => []
      }
    ) || exit

For the guide, see the [wiki page](https://github.com/saveriomiroddi/simple_scripting/wiki/SimpleScripting::Argv-Guide).

## SimpleScripting::Configuration

`SS::C` is a module which acts as frontend to the ParseConfig gem (`parseconfig`), giving compact access to the configuration and its values, and adding a few helpers for common tasks.

Say one writes a script (`foo_my_bar.rb`), with a corresponding (`$HOME/.foo_my_bar`) configuration, which contains:

    some_relative_file_path=foo
    some_absolute_file_path=/path/to/bar
    multiple_paths=foo:/path/to/bar
    my_password=uTxllKRD2S+IH92oi30luwu0JIqp7kKA

    [a_group]
    group_key=baz

This is the workflow and functionality offered by `SS::C`:

    require 'simple_scripting/configuration'

    # Picks up automatically the configuration file name, based on the calling program
    #
    configuration = SimpleScripting::Configuration.load(passwords_key: 'encryption_key')

    configuration.some_relative_file_path.full_path # '$HOME/foo'
    configuration.some_absolute_file_path           # '/path/to/bar'
    configuration.some_absolute_file_path.full_path # '/path/to/bar' (recognized as absolute)
    configuration.multiple_paths.full_paths         # ['$HOME/foo', '/path/to/bar']

    configuration.my_password.decrypted             # 'encrypted_value'

    configuration.a_group.group_key                 # 'baz'; also supports #full_path and #decrypted

### Encryption note

The purpose of encryption in this library is just to avoid displaying passwords in plaintext; it's not considered safe against attacks.

## Help

See the [wiki](https://github.com/saveriomiroddi/simple_scripting/wiki) for additional help.

[GV img]: https://badge.fury.io/rb/simple_scripting.png
[BS img]: https://travis-ci.org/saveriomiroddi/simple_scripting.svg?branch=master
[CC img]: https://codeclimate.com/github/saveriomiroddi/simple_scripting.png
[CS img]: https://coveralls.io/repos/saveriomiroddi/simple_scripting/badge.png?branch=master
