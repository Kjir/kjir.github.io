---
categories:
- grunt
date: 2016-02-14T21:02:00Z
tags:
- grunt
title: A modular Gruntfile
url: /2016/02/14/modular-gruntfile/
---

[Grunt](http://www.gruntjs.com/) is a very powerful task runner and if you are
developing in Javascript chances are you are probably using it. It's
configuration file defines all the tasks available and what they should do, but
have you ever felt that even for mid-sized projects it easily becomes too big
and hard to manage?

Well I felt that way, so that's when I decided to split it into many smaller
configuration files with the help of
[load-grunt-config](http://firstandthird.github.io/load-grunt-config/). By
following the excellent advice given by [Paul Bakaus](https://paulbakaus.com/)
on [his article on
html5rocks](http://www.html5rocks.com/en/tutorials/tooling/supercharging-your-gruntfile/),
I came up with a cleaner and much more manageable setup.

Here I'll try to build upon his article to add some more details about useful
tricks I used in my setup.

## The brevity of YAML
Being at heart a Python developer, I love terse syntax. Why bother with
unecessary brackets, commas or quotes which clutter your files making them
harder to read for the human eye? Hence a move from a Javascript syntax to a
YAML syntax for a configuration file is a big relief for me!

`load-grunt-config` supports many different kinds of syntax files, which can be
mixed, allowing me to greatly reduce the noise in my configuration. If I
initially had a `Gruntfile` like this:

```javascript
module.exports = function (grunt) {
  var pkg = grunt.file.readJSON('package.json');

  grunt.initConfig({
    ngconstant: {
      options: {
        name: 'config',
        dest: 'app/config/config.js',
        constants: {
          ENV: {
            apiLocation: '',
            appName: 'MyApp',
            appVersion: pkg.version
          }
        }
      },
      development: {
        constants: {
          ENV: {
            apiLocation: 'http://localhost:3000/api'
          }
        }
      },
      staging: {
        constants: {
          ENV: {
            apiLocation: 'https://staging.example.com/api'
          }
        }
      },
      production: {
        constants: {
          ENV: {
            apiLocation: 'https://api.example.com'
          }
        }
      }
    }
  });
}
```

I would now have this as a `Gruntfile`:

```javascript
module.exports = function (grunt) {
  require('load-grunt-config')(grunt);
};
```

And I would create a YAML configuration file in `grunt/ngconstant.yaml`:

```yaml
options:
  name: 'config'
  dest: 'app/config/config.js'
  constants:
    ENV:
      apiLocation: ''
      appID: 'MyApp'
      appVersion: '<%= package.version %>'
  development:
    constants:
      ENV:
        apiLocation: 'http://localhost:3000/api'
  staging:
    constants:
      ENV:
        apiLocation: 'https://staging.example.com/api'
  production:
    constants:
      ENV:
        apiLocation: 'https://api.example.com'
```

Isn't this cleaner? And it's shorter, too!

Also notice how I did not have to manually parse my `package.json` to access
its values? This is another nice feature of `load-grunt-config`'s which comes
in handy. All you need to do is to read your values using a grunt template like
`'<%=package.version%>'`.

## Sharing values between tasks

In my project I used to have some variables holding some values shared by many
different tasks. Lets take for instance the files list, which can be used by
`concat`, but also by `uglify` or `copy` or whatever. So I had something like
this:

```javascript
module.exports = function (grunt) {

  var files = [

    // Application files
    'app/**/*.module.js',
    'app/module/**/*.js',
    'app/component/**/*.js',
    'app/service/**/*.js',
    'app/app.js',

    // Do not include tests in application build
    '!app/**/*.test.js',
    '!app/**/*_test.js'
  ];

  grunt.initConfig({
    jshint: {
      options: {
        jshintrc: true
      },
      all: {
        src: [files]
      }
    },
    uglify: {
      production: {
        src: files,
        dest: 'dist/app/js/<%= package.name %>.min.js',
        sourceMap: true
      }
    }
  });
};

```

How can I do to share these values across tasks now that they each live in
their own file? Well first I created a `grunt/project.yaml` file  holding these
values:

```yaml
files:
  # Application files
  - 'app/**/*.module.js'
  - 'app/module/**/*.js'
  - 'app/component/**/*.js'
  - 'app/service/**/*.js'
  - 'app/app.js'
    # Do not include tests in application build
  - '!app/**/*.test.js'
  - '!app/**/*_test.js'
```

And here's the magic in the `grunt/jshint.yaml` file:

```yaml
options:
  jshintrc: true
all:
  src:
    - '<%= project.files %>'
```

And the same goes for `grunt/uglify.yaml`:

```yaml
production:
  src: '<%= project.files %>'
  dest: 'dist/app/js/<%= package.name %>.min.js'
  sourceMap: true
```

That was easy!

## Separating configuration files by environment

As you could see, in my project I use
[grunt-ng-constant](https://github.com/werk85/grunt-ng-constant) to create an
appropriate configuration file according to the environment. I also use
[grunt-env](https://github.com/jsoverson/grunt-env) in combination with
[grunt-preprocess](https://github.com/jsoverson/grunt-preprocess) to configure
my `index.html` file differently according to the environment. I wanted to have
a single file with all the configuration for a single environment so that it
would be easier to verify and change them, and once again `load-grunt-config`
came to the rescue!

I used [config
grouping](http://firstandthird.github.io/load-grunt-config/#toc7) to do that:

```yaml
# File: development-tasks.yaml
ngconstant:
  constants:
    ENV:
      appVersion: '<%= package.version %>-dev'
    apiLocation: 'https://devel.example.com/api'
env:
  ENV: 'DEVELOPMENT'
  THEME_CSS: 'css/mytheme.devel.css'
```

Using this configuration I can now have a separate file for each environment.
The `grunt/ngconstant.yaml` and `grunt/env.yaml` files become like this:

```yaml
# File: grunt/ngconstant.yaml
options:
  name: 'config'
  dest: 'app/config/config.js'
  constants:
    # These are all default values
    ENV:
      appName: 'MyApp'
      appVersion: '<%= package.version %>'
    apiLocation: 'https://api.example.com/api/v1'
```

```yaml
# File: grunt/env.yaml
options:
  add:
    THEME_CSS: 'css/mytheme.css'
```

As you can see, there are only the default options there and no environment
specific configuration.

I also wanted to separate the environment-specific configuration files from the
standard task configurations, so that the context would be clear. So I put them
all in a `config` directory, which I created, and modified our `Gruntfile.js`:

```javascript
module.exports = function (grunt) {

  var path = require('path');

  require('load-grunt-config')(grunt, {
    configPath: [
      path.join(process.cwd(), 'grunt'), // Task settings here
      path.join(process.cwd(), 'config') // Environment specific settings here
    ],

    /*
     * If you want to change your setting locally without changing what is on
     * the repo, you can define your configuration overrides in config/override
     */
    overridePath: [
      path.join(process.cwd(), 'config', 'override')
    ],

  });

};
```

`load-grunt-config` also supports overrides, which is another nice feature: as
I set it up, you can create a configuration file in `config/override` which
will override any previously set configuration option. This is very useful to
accomodate the needs of every developer which might be a little different,
without having them modify the files which are under versioning. Pretty neat!

## Conclusion

`load-grunt-config` is a very nice grunt plugin which really improves the
quality of your Gruntfile by splitting it into many smaller files. Not only
that: it adds a lot of useful features that will make you wonder why bother
writing plain `Gruntfile`s anymore!

Do you use `load-grunt-config` in your project? Is there some other great
feature I missed? Please leave a comment below!
