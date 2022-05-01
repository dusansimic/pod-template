# Pod playbook template

Template for an Ansible playbook that creates a pod systemd service and is easily extendable to make
a good pod and container management system.

## Roles

### [pod](./ansible/roles/pod/)

The main and only role in the playbook is `pod` which can be renamed to anything else. It creates
a pod service and is has handlers for enabling and restarting the service. You can easily add tasks
to the role in order to start containers in said pod.

A recommended way to create unit files is to generate them using `podman` cli.

## Tasks

### [create units directory](./ansible/roles/pod/tasks/main.yaml#L3)

Makes sure the units directory is present. This is done in case you wan't to deploy the
pod in user scope (in rootless mode). It also sets SELinux context for the directory.

### [set context for units directory](./ansible/roles/pod/tasks/main.yaml#L12)

Sets the context for units directory in using `semanage` so default contenxt for all
files created there would be proper.

### [create pod service](./ansible/roles/pod/tasks/main.yaml#L18)

Creates a pod service (systemd unit file) from a template. I'd recommend doing it this way so
variables from ansible playbook could be used in the unit file.

## Container task example

In order to start containers inside a pod, you'll need to create service files for that container.
Here is an example task for doing that including necessary notifications to make sure all
configurations are reloaded on any change:

```yaml
- name: create container service
  ansible.builtin.template:
    src: container.j2
    dest: "{{ units_path }}/container-{{ container_name }}.service"
    mode: 0644
  notify:
    - reload daemon
    - restart pod service
```

## Handlers

### [reload daemon](./ansible/roles/pod/handlers/main.yaml#L3)

Reloads systemd daemon in order to load changes.

### [enable pod service](./ansible/roles/pod/handlers/main.yaml#L8)

Makes sure the pod service is enabled so it starts on boot.

### [restart pod service](./ansible/roles/pod/handlers/main.yaml#L14)

Restart the pod service. If unit files are generated using `podman` cli, all services for containers
in the pod are dependant on pod service so when it restarts, so do container services. That way you
can easily restart the whole pod with it's containers when some configuration is changed.

## License

This template is licensed under the [MIT license](./LICENSE).
