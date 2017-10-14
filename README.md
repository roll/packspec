# PackSpec

[![Python](https://img.shields.io/travis/packspec/packspec-py/master.svg?label=Python)](https://travis-ci.org/packspec/packspec-py)
[![JavaScript](https://img.shields.io/travis/packspec/packspec-js/master.svg?label=JavaScript)](https://travis-ci.org/packspec/packspec-js)
[![Ruby](https://img.shields.io/travis/packspec/packspec-rb/master.svg?label=Ruby)](https://travis-ci.org/packspec/packspec-rb)
[![PHP](https://img.shields.io/travis/packspec/packspec-php/master.svg?label=PHP)](https://travis-ci.org/packspec/packspec-php)

Cross-language testings made easy: unified YML-based package specification and Python/JavaScript/Ruby/PHP test runners.

## Features

- write one YML-based specification and run for different languages
- test runners for Python, JavaScript, Ruby and PHP
- simple but powerful abstract language
- filter statements by language
- custom functions

---

#### Specification

![](https://i.imgur.com/0IxLi19.png)

---

#### Test Runners

![](https://i.imgur.com/cC7CUvn.png)

## Getting Started

### Installation

The packages use semantic versioning. It means that major versions could include breaking changes. It’s highly recommended to specify a package version range in your dependencies.

```bash
$ pip install packspec # Python
$ npm install packspec # JavaScript
$ gem install packspec # Ruby
$ composer require packspec/packspec # PHP

```

### Introduction

We will write a simple specification for a package called `tableschema` (it should be installed for every target language - use `pip/npm/gem/composer`). This package has a similar API for all these languages. Create file `packspec.yml` with following contents. It should be a valid YAML and the syntax is almost self-explanatory:

> packspec.yml

```yaml
- tableschema

- tableschema=$import: ['tableschema']

- Field - integer

- descriptor=: {'name': 'name', 'type': 'integer'}
- (py|js|rb)field=tableschema.Field: [{descriptor}]
- (php)field=tableschema.FieldsFactory.field: [{descriptor}]
- field.type==: 'integer'
- field.format==: 'default'
- (py|js|rb)field.cast_value: ['', ==: null]
- field.cast_value: [1, ==: 1]
- field.cast_value: ['1', ==: 1]
- field.cast_value: ['3.14', ==: ERROR]

```

Now we run a Python test runner for this specification (we don't provide any path because PackSpec test runner will run `packspec.yml` by default):

```bash
$ packspec-py

 #  Python

➖➖➖

 #  tableschema

 ✔  tableschema = $import("tableschema")

 #  Field - integer

 ✔  descriptor = {"name": "name", "type": "integer"}
 ✔  field = tableschema.Field(descriptor)
 ➖  field = tableschema.FieldsFactory.field(descriptor)
 ✔  field.type == "integer"
 ✔  field.format == "default"
 ✔  field.cast_value("")
 ✔  field.cast_value(1) == 1
 ✔  field.cast_value("1") == 1
 ✔  field.cast_value("3.14") == ERROR

 ✔  tableschema: 9/9
```

As we could see we pass all tests. Under the hood PackSpec specification has been translated to Python language and then evaluated. Ok. Let's test against JavaScript:

```bash
$ packspec-js # or ./node_modules/.bin/packspec-js

 #  JavaScript

➖➖➖

 #  tableschema

 ✔️  tableschema = $import("tableschema")

 #  Field - integer

 ✔️  descriptor = {"name":"name","type":"integer"}
 ✔️  field = tableschema.Field(descriptor)
 ➖  field = tableschema.FieldsFactory.field(descriptor)
 ✔️  field.type == "integer"
 ✔️  field.format == "default"
 ✔️  field.castValue("")
 ✔️  field.castValue(1) == 1
 ✔️  field.castValue("1") == 1
 ✔️  field.castValue("3.14") == ERROR

 ✔️  tableschema: 9/9
```

We've got the same result. It passes all test for JavaScript. Let's try Ruby:

```bash
$ packspec-rb

 #  Ruby

➖➖➖


 # tableschema

 ✔️  tableschema = $import("tableschema")

 # Field - integer

 ✔️  descriptor = {"name":"name","type":"integer"}
 ✔️  field = tableschema.Field({"descriptor":null})
 ➖  field = tableschema.FieldsFactory.field({"descriptor":null})
 ✔️  field.type == "integer"
 ✔️  field.format == "default"
 ✔️  field.cast_value("")
 ✔️  field.cast_value(1) == 1
 ✔️  field.cast_value("1") == 1
 ✔️  field.cast_value("3.14") == ERROR

 ✔️  tableschema: 9/9
```

Test for Ruby also work. Only PHP is left:

```bash
$ packspec-php # or php vendor/packspec/packspec/bin/packspec-php

 #  PHP

➖➖➖

 #  tableschema

 ✔  tableschema = $import("tableschema")

 #  Field - integer

 ✔  descriptor = {"name":"name","type":"integer"}
 ➖  field = tableschema.Field(descriptor)
 ✔  field = tableschema.FieldsFactory.field(descriptor)
 ✔  field.type == "integer"
 ✔  field.format == "default"
 ➖  field.castValue("")
 ✔  field.castValue(1) == 1
 ✔  field.castValue("1") == 1
 ✔  field.castValue("3.14") == "ERROR"


 ✔  tableschema: 8/8
```

As we could see the PHP `tableschema` implementation has a slightly different API so we use an alternative call. That's it! We was able to write the test specification once and then run it against 4 different languages!

### Examples

Take a look how it works on a real world projcts:

- [Frictionless Data's testsuite](https://github.com/frictionlessdata/testsuite-basic)


## Documentation

### Specification

The PackSpec language is an YML-based high-level abstract language for writing a package specification. In the most cases it's self-explanatory so let's do a quick dive using a `learn X in Y minutes` style:

```yml
# It's a valid YAML so comments are allowed
# All names are normalized to be snake_cased
# Specification is a list of statements

# Section name
- My package

# Assignment
- name=: 'value'

# Dereference
- other_name=: {name}

# Assertion
- other_name==: 'value'

# Function call and assertion
- result=function: ['value1', kwarg=: 'value2']
- result==: 'value3'

# Function call with assertion
- function: ['value1', kwarg=: 'value2', ==: 'value3']

# Function call expected to fail
- function: ['value1', kwarg=: 'value2', ==: ERROR]

# Object instantiation and property access
- object=: module.Class('value1')
- object.property=: 'value2'
- object.property==: 'value2'
- object.method: ['value3', ==: 'value4']

# Working with collections
- list=: [1, 2, 3]
- list.0==: 1
- dict=: {key: 'value1'}
- dict.key=: 'value2'
- dict.key==: 'value2'

# Builtin import
- package=: $import('package')

# Language filters
- (py|js)Section
- (py)a=: 1
- b=: 2

# Use custom function (see below)
- dt=$make_datetime: [2015, 1, 1]

---

# It could contain second document with custom funcitons

# Add `make_datetime` function
py: |
  import datetime
  make_datetime = datetime.datetime

# Add `make_datetime` function
js: |
  const moment = require('moment')
  makeDatetime = (year, month, ...args) => {
    return new Date(year, month - 1, ...args)
  }
```

### Test Runners

The runners provide unified interface and output so we could talk about abstract `packspec-<lang>` runner (for every language just use corresponding executable):

#### `$ packspec-<lang> [-x/--exit-first] [PATH]`

- `PATH (str)` - file or directory path where PackSpec YML file(s) are stored. By default it will test `packspec.yml` or all YML files in `packspec` directory.
- `-x/--exit-first (bool)` - exit on first failed test and show an extended error information (e.g. an exception traceback)

For now these runners are implemented:

- Python
- JavaScript
- Ruby
- PHP

## Contributing

This project uses consolidated issue tracker and development model. Please use this repository for everything except test runners code.

### Python

Requirements:
- installed `virtualenv` - https://virtualenv.pypa.io/en/stable/installation/

```bash
git clone git@github.com:packspec/packspec-py.git
cd packspec-py
virtualenv .python -ppython3.5
source .python/bin/activate
make install
make test
```

### JavaScript

Requirements:
- installed `nvm` - https://github.com/creationix/nvm#installation

```bash
git clone git@github.com:packspec/packspec-js.git
cd packspec-js
nvm install 8
nvm use 8
npm install
npm test
```

### Ruby

Requirements:
- installed `rvm` - https://rvm.io/rvm/install

```bash
git clone git@github.com:packspec/packspec-rb.git
cd packspec-rb
rvm install 2.4
rvm use 2.4
gem install bundler
./bin/setup
rake spec
```

### PHP

Requirements:
- installed `phpbrew` - http://phpbrew.github.io/phpbrew/

```bash
git clone git@github.com:packspec/packspec-php.git
cd packspec-php
phpbrew install 7.1.0
phpbrew use 7.1.0
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php -r "if (hash_file('SHA384', 'composer-setup.php') === '544e09ee996cdf60ece3804abc52599c22b1f40f4323403c44d44fdfdd586475ca9813a858088ffbc1f233e9b180f061') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
php composer-setup.php --filename=composer
php -r "unlink('composer-setup.php');"
php composer install
php composer test
```

## Changelog

### v0.x

Proof of concept version of the standard and test runners.
