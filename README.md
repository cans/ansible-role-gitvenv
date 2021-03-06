cans.gitvenv
============

Clone Python project git repositories and setup a virtual environment for each


## How does it works

This roles works on a list of repositories. For each repository in the list,
it will clone it, create a virtualenv for it and install checked-out code
requirements inside the virtualenv.

It will also setup the user shell config so it can use the created virtual
environment with virtualwrapper's `workon` command.

Each repository to clone and setup has to be specified as follows:

      - name: myapp
        bare: "yes"                                 # default: false
        default: true                               # default: {{omit}}
        dest: "/var/lib"                            # default: {{gitvenv_clone_dir}}
        requires: "dev-requirements.txt"            # default: {{gitvenv_requirements}}
        site_packages: true                         # default: false
        url: git@github.com:Account/repository.git  # default: "{{gitvenv_base_url + '/' + name + '.git'}}"
        install: true                               # default: false
        branch: feature/fancy-new-one               # default: "{{gitvenv_branch}}"


Values for which no default value is indicated are *mandatory*.

This will create a clone of the repository found at `{{url}}`
(`git@github.com/Account/repository.git` here) in `{{dest}}/{{name}}`
(`/var/lib/myapp` here). Which is roughly equivalent to firing the
command:

    git clone {{url}} {{dest}}/{{name}}

Or when expanded:

    git clone git@github.com:Account/repository.git /var/lib/myapp

Then it will create a virtualenv in `{{workon_home}}` named `{{name}}`,
(that is `/opt/virtualenv/myapp`). Which boils down to:

    virtualenv /opt/virtualenv/myapp
    cd /var/lib/myapp
    pip install --upgrade -r requirements.txt

On the virtualenv is setup, you may want to install the code you checked
out from git inside the virtualenv. If you set the `install` variable
to `true`, then this role will install do so, in development mode:

    cd /var/lib/myapp
    pip install -e .


And finally it will update the users `~/.bashrc` to define and export
the `WORKON_HOME` variable, set to the value of`gitvenv_workon_home_dir`;
as well as add a call to the `workon` command with the name of the
virtual environment which definition contains the `default` variable
set to `true`. This will ensure that virtual environment is active upon
login.


## Notes

To be able to set-up the user's shell in a consistent manner, this role
assumes all virtual environment will be created in the same place and
for a unique user. This means you cannot override the `gitvenv_user`
and `gitvenv_workon_home_dir` on a per-repository basis.

You can however use this roles several times in a single playbook to
clone several git repositories and setup their respective virtual
environments and for different users.


Requirements
------------

This role assumes `git` and `pip` are available on the target host.


Role Variables
--------------

All the variables of this role are namespaced using the prefix `gitvenv_`


Role input variables:

- `gitvenv_base_url`: the base url to use if not url is given. The url
  to use for the repository to clone is then:

       "{{ gitvenv_base_url + '/' + name + '.git'}}"

  Where name is the name given in the repository defition (see above).
- `gitvenv_clone_dir`: the directory in which store git clones
- `gitvenv_home_dir`:
- `gitvenv_git_force`: whether to allow git to override local changes (default: `false`);
- `gitvenv_python`: the version of python with which create the virtual
  environment(s);
- `gitvenv_requirements_file`: the default filename (or path, relative
  to `{{dest}}/{{name}}`) of the pip requirements file to use.
- `gitvenv_user`: the user for which install and setup the virtual
  environment(s) (default: `ansible_user_id`);
- `gitvenv_workon_home_dir`: the directory in which install the virtual
  environment(s) (default: `"{{gitvenv_workon_home_dir}}/{{gitvenv_virtualenvs_dirname}}"`);
- `gitvenv_workon_regex`: the pattern used to find the virtualenv activation command in the
  users shell profile.

All those variables are defined in the `defaults/main.yml` file. They can
be overriden as you need either globally, _e.g._ in you playbook's `vars`
section or at the role level (_cf._ [examples](#Example Playbook) below)


Variables intended to be used in other roles:


- `gitvenv_virtualenvs_dirname`: the name of the directory in which store
  Python virtual environments. Helper (default: `.virtualenvs`);

All those variables are defined in the `vars/main.yml` file.


Dependencies
------------

This role has no dependencies


Example Playbook
----------------

Here is an example playbook to clone and setup a virtualenv for a project.

    - hosts: devel
      vars:
        gitvenv_clone_dir: "/home/cans/projects/aurora"  # All repo. will be cloned here
        project_aurora_repositories:
          - name: "ssh-harness"
            url: "https://github.com/cans/ssh-harness.git"
          - name: "vcs-ssh"
            url: "https://github.com/cans/vcs-ssh.git"
          - name: "vcsd"
            url: "https://github.com/cans/vcsd.git"
            dest: "/home/cans/projects/aurora"  # Override clone destination directory.

      roles:
        - role: cans.gitvenv
          gitvenv_repositories: "{{project_aurora_repositories}}"
          gitvenv_user: "cans"
          gitvenv_workon_home_dir: "~/.virtualenvs/"
          gitvenv_python: python2.7

Of course most variable would be better defined in a seperate file under `vars/` and
included in the playbook.


License
-------

GPLv2

Author Information
------------------

Copyright © 2017, Nicolas CANIART
