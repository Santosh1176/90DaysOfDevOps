# An Introduction to Ansilbe

With some basic understanding of Terraform, let's digress a bit to another aspect of infrastructure — configuring the infrastructure provisioned by Terraform. In this session, we shall look at one of the most used configuration management too [**Ansible**](https://www.ansible.com/).

Ansible is an open-source configuration management and automation tool, released by [Michael DeHaan](https://twitter.com/laserllama) in 2012 and later acquired by Red Hat in 2015. uses **YAML** to describe the desired state of a system, and then uses that description to ensure that the system is configured properly.

One of the greatest strengths of Ansible is its ability to run regular shell commands, so we can take some existing scripts and commands and work on converting them into *idempotent playbooks*. One of the key things to remember while dealing with Ansible is it's idempotent — The same code will produce the exact same result immaterial of the number of times its executed. Ansible uses a metaphor to describe configuration files called **Playbooks**,  they are a list of tasks (*Plays* in Ansible's terminology) that will run on a pre-defined set of servers. 

Ansible calls this *inventories*, which is a list of servers that are managed in `/etc/ansible/hosts` directory and Ansible looks for them when configuring servers using **Playbooks**.

Let's install Ansible on our local enviornment. You can find all the instructions for installing Ansible using Python's `pip` package manager on its [official docs](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#using-argcomplete-with-zsh-or-tcsh).

Check for the availability of `pip` on your system:
```
santosh@90DaysOfDevOps:main$ python3 -m pip -V
pip 22.0.2 from /usr/lib/python3/dist-packages/pip (python 3.10)
```
If you don't have `pip` installed, you can install the latest pip directly from the Python Packaging Authority by running the following:
```
$ curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
$ python3 get-pip.py --user
```

Once the `pip` is available and configured, we can move ahead and install the Ansible package using the `pip`.

`python3 -m pip install --user ansible`

That's great, now we can check if the installation was correct by checking the version:
```bash
santosh@90DaysOfDevOps:main$ ansible --version
ansible [core 2.14.0]
  config file = None
  configured module search path = ['/home/santosh/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /home/santosh/.local/lib/python3.10/site-packages/ansible
  ansible collection location = /home/santosh/.ansible/collections:/usr/share/ansible/collections
  executable location = /home/santosh/.local/bin/ansible
  python version = 3.10.6 (main, Nov  2 2022, 18:53:38) [GCC 11.3.0] (/usr/bin/python3)
  jinja version = 3.1.2
  libyaml = True
santosh@90DaysOfDevOps:main$
```

# Resource

- [How To Use Ansible with Terraform for Configuration Management - DigitalOcean Tutorial](https://www.digitalocean.com/community/tutorials/how-to-use-ansible-with-terraform-for-configuration-management)

- [The Most Simplified Integration of Ansible and Terraform](https://medium.com/geekculture/the-most-simplified-integration-of-ansible-and-terraform-49f130b9fc8)