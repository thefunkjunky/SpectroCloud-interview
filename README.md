# SpectroCloud-interview
Vagrant/ansible project to run "hello world" in a VM

## Requirements
You must have Vagrant, Python3, pip, and ansible installed on your system. Use homebrew to install vagrant, python3, and pip if you are on macos.  Otherwise, follow the installation instructions for your OS.

Installing ansible:
```
$ pip3 install ansible
```

## Bootstrapping VM
To bootstrap and provision the VM:

1. cd to `vagrant` directory.
1. Edit Vagrantfile line `config.vm.network`'s `host` to the local port you want to forward. Default is 8080.
1. Run `vagrant up`
1. Once provisioned, check if the application is running by running `curl localhost:8080` (be sure to change the port number if you edited it). It should return a "Hello World" page.

## Design
This is a simple Flask application that returns a static "Hello World" website.  It runs inside of the default VM for Vagrant.  Vagrant provisions the VM, using the Ansible provisioner to set up the guest system.  The system uses NGINX for the web server, and a number of Gunicorn workers to connect the NGINX server to the application.  Details can be found by reading the `Vagrantfile` and `playbook.yml` in the `vagrant` directory.

## Issues encountered
- The biggest problem I had was due to the Ansible provisioner. By default, it uses the local ansible on your system, but when installing apt resources, it would fail due to 404s on the ubuntu package resources.  When changing the provisioner to `ansible_local`, it would install and run ansible on the guest machine, which worked.  However, `ansible_local` is incapable of copying files from the local system, so I was originally forced to use a vagrant file provisioner for copying files, which is terrible practice.  Eventually I figured out that the reason I was getting 404s was because the apt cache had to be updated first, which fixed the problem and allowed me to use the regular ansible provisioner.

- Another big issue I had was the gunincorn service failing due to an inability to read the application.sock socket required to connect nginx to the application.  Eventually I realized that the application.sock was set to the parent directory of where it should have been, and that I needed to ensure that all folders and files were owned by the `www-data` user and groups used by the service.

- Finally, after everything was provisioned and running correctly, testing that the nginx server was return a default ngix page, not hello world.  Everything appeared to be set up correctly, which made troubleshooting it challenging.  After some time, I noticed that there was a `default` site configured in `/etc/nginx/sites-enabled/default`. I tried deleting it, after which the site was working as expected.

## Authors
- Garrett Anderson <garrett@devnull.rip> 
