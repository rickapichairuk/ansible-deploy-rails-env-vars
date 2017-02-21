# ansible-deploy-rails-env-vars

A role to deploy new env vars for configuring a rails app by interpolating
values into template files and uploading them to servers.

# INSTALLATION

Using ansible-galaxy:

```sh
$ ansible-galaxy install rickapichairuk.deploy-rails-env-vars
```

Using `requirements.yml`:

```yaml
- src: rickapichairuk.deploy-rails-env-vars
  path: roles/
  name: rickapichairuk.deploy-rails-env-vars
```


# INTRODUCTION

In the spirit of [12 Factor Apps](https://12factor.net/) recommendation for
[configuring apps](https://12factor.net/config), this ansible role provides
mechanisms to manage the following commonly used rails configuration files.
You can use ansible-vault to encrypt configuration key/values.

# CONFIGURATION FILES SUPPORTED

* application.yml (used in conjunction with [Figaro](https://github.com/laserlemon/figaro))
* database.yml (default [rails database configuration file](http://edgeguides.rubyonrails.org/configuring.html#configuring-a-database))
* newrelic.yml ([newrelic configuration](https://docs.newrelic.com/docs/agents/ruby-agent/configuration/ruby-agent-configuration) file)
* .rbenv-vars (used in conjunction with [rbenv-vars](https://github.com/rbenv/rbenv-vars))

# EXAMPLE PLAYBOOK

```yaml
- name: Playbook to deploy rails related configuration files to rails app servers
  hosts: rails_app_servers
  roles:
    - role: rickapichairuk.deploy-rails-env-vars
      application_yml:
        deploy: true
        template: /local/path/to/your/application.yml.j2
        dest: /server/path/to/put/application.yml
      dot_rbenv_vars:
        deploy: true
        template: /local/path/to/your/dot.rbenv-vars.j2
        dest: /server/path/to/put/.rbenv-vars
        create_symlink: true
        symlink_dest: /server/path/to/create/symlink/to/.rbenv-vars
        symlink_owner: username
        symlink_group: group
      database_yml:
        deploy: true
        template: /local/path/to/your/database.yml.j2
        dest: /server/path/to/put/database.yml
      newrelic_yml:
        deploy: true
        template: /local/path/to/your/newrelic.yml.j2
        dest: /server/path/to/put/newrelic.yml
```

# ONE FILE CONFIG

If you really want to use just one file to config your app, you can probably get
away with using .rbenv-vars and only deploy this one file. In order for this to 
work, you will have to setup rbenv on your server for the user that puma (and
sidekiq) run as. You will have to have to also write all your configuration
files (application.yml, database.yml, etc) to refer to ENV.

# USING WITH CAPISTRANO AND SYMLINKED CONFIG FILES

You can deploy config files that have the configuration values interpolated into
configuration files.

Be sure to setup capistrano to use `:linked_files` and `:linked_dirs` to symlink
the files deployed by ansible and this role to the rails app deploy directory.

# WHY IS .rbenv-vars TREATED DIFFERENTLY?

The reason that .rbenv-vars can be configured to be symlinked is because the
author usually has the destination be the shared directory so it is in the same
directory as all the other configuration files. However, capistrano cannot
symlink the .rben-vars to the $HOME directory because it's not in the app
directory. Ansible is used to create the symlink from the shared directory to
the home directory.

For example:

```sh
$ ln -s /home/deploy_user/apps/your_app/shared/dot.rbenv-vars /home/deploy_user/.rbenv-var
```

# AUTHOR

Rick Apichairuk

# LICENSE

BSD 3-Clause License

Copyright (c) 2017, Rick Apichairuk
All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:

* Redistributions of source code must retain the above copyright notice, this
  list of conditions and the following disclaimer.

* Redistributions in binary form must reproduce the above copyright notice,
  this list of conditions and the following disclaimer in the documentation
  and/or other materials provided with the distribution.  
* Neither the name of the copyright holder nor the names of its
  contributors may be used to endorse or promote products derived from
  this software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
