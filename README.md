Simple Compose
=========

A role meant to be kept as simple as possible whilst still trying to cover various different Docker Compose deployment variations. For now, assumes Docker Compose v2 (i.e. the plugin) is used. This will **NOT** work with Podman or similar container runtimes. The project makes heavy use of the built-in docker API provided by Ansible. It is mearly meant to reduce the amout of boilerplate code needed to deploy a Docker Compose project.

Planned
------------

There are some things I have planned for this role, namely:

- Add support for Compose v1 (even though it is deprecated, it's still used here and there, and I think it's worth supporting)
- Add support for handlers (i.e. when a Compose project is updated, notify any given handler(s), can be useful for some Compose projects)
- Add more configuration options (e.g. use variables for folder/file permissions, right now they're hardcoded to `0700` for the project dir and `0400` for the project files)

If you have any suggestions or want to contribute, feel free to open an issue or a pull request.

Requirements
------------

You will need a working Docker installation and use Docker Compose v2. Other than that, there is none.

Role Variables
--------------

This roles uses the following variables together with its default values:

```yaml
simple_compose_username: "{{ username }}"
simple_compose_projects: []
```

Set `simple_compose_username` to the username of your Docker administering user. This can be any user that has access to the Docker socket (i.e. is in the `docker` group).

`simple_compose_projects` is a list of dictionaries that define the projects you want to deploy. Each dictionary should have the following keys:
- `name`: The name of the project. This will be used to differentiate the loop runs in Ansible's CLI output, and also dicates the location to the template files used for the role.
- `directory`: The directory where the project is located. This should be an absolute path.
- `files`: All files belonging to the compose project, like `docker-compose.yaml`, `.db.env`, `.web.env`, `.env`, etc.

Dependencies
------------

None.

(Optional): `geerlingguy.docker` if you want to easily install Docker using Ansible.

Example Playbook
----------------

Simple playbook example:

```yaml
    - hosts: all
      vars:
        # Global variables shared between roles
        docker_username: "my_docker_user"

        # Varialbes specific to Geeklingguy's Docker role
        docker_users:
          - "{{ docker_username }}"

        # Varialbes specific to the simple_compose role
        simple_compose_projects:
          - name: "my_project"
            directory: "/compose_projects/my_project"
            files:
              - "docker-compose.yaml"
              - ".my_project.env"
          - name: "nextcloud"
            directory: "/compose_projects/nextcloud"
            files:
              - "docker-compose.yaml"
              - ".nextcloud.env"
        simple_compose_username: "{{ docker_username }}"
      
      roles:
        # This role is optional, but it makes it easier to install Docker
        - role: geerlingguy.docker
        # Actual magic happens here
        - role: tibiiius.simple_compose
```

Afterward, make sure to create template files in the `template` directory. The role uses Ansible's built-in template module to render the files, so just stick to Ansible's default directory structure. The role will look for the files in `templates/{{ name }}/{{ file }}` where `name` is the name of the project and `file` is the file name. It will do so for each project specified.

License
-------

MIT
