ansible_role_postgresql
=========


This ansible deploys postgres database for redhat based distro or debian based.

---


This ansible template is using jenkins multibranch pipeline to test ansible-playbooks and you need docker installed on the build node.

The Jenkins file looks like this you can call it what you want but give the name name on jenkins multibranch pipeline.

```
pipeline {

  agent {
    // the label for the build node.
    label 'hf-t-build02-docker'
  }

  stages {

    stage ('Get latest code') {
      steps {
        checkout scm
      }
    }

    stage ('Setup Python virtual environment') {
      steps {
        sh ''' #!/bin/bash
          python3 -m venv virtenv
          . ./virtenv/bin/activate
          python3 -m pip install --upgrade ansible molecule docker
          pip3 install 'molecule[docker]'
        '''
      }
    }

    stage ('Display versions') {
      steps {
        sh '''
          . ./virtenv/bin/activate
          docker -v
          python -V
          ansible --version
          molecule --version
        '''
      }
    }

    stage ('Molecule test') {
      steps {
        sh '''
          . ./virtenv/bin/activate
          molecule test
        '''
      }
    }

  }

}

```


There is also .ansible-lint file for linting ansible.

To install molecule locally with docker driver.

```
pip3  install --user molecule[docker]

```

Molecule settings are under molecule/default/
there you have the files converge.yml, molecule.yml .

converge.yml here you call the role so molecule can run it 
just like a site.yml file.

When using jenkins it looks like this.

```
---
- name: Converge
  hosts: all
  become: true
  gather_facts: yes
  # import a variable file
  #vars_files:
  #  - ../../defaults/main.yml
  # variables
  # add vars if you need it
  #vars:


  tasks:
        
  roles:
    - role: "{{ lookup('env', 'MOLECULE_PROJECT_DIRECTORY') | basename }}"
```

molecule.yml is the central configuration file for testing ansible playbooks.
Here you can choose what docker image it will use for testing. The line test_sequence here 
you choose testing scenario.

With jenkins you need multipule instance to run a test for a spesific distro and this is the same way you can test on your laptop.

```
dependency:
  name: galaxy
driver:
  name: docker
platforms:
  - name: instance-centos7
    image: "habbis0/docker-centos7-ansible:latest"
    command: ${MOLECULE_DOCKER_COMMAND:-""}
    volumes:
      - /sys/fs/cgroup:/sys/fs/cgroup:ro
    privileged: true
    pre_build_image: true
  - name: instance-ubi8-rhel
    image: "habbis0/docker-uib8-ansible:latest"
    command: ${MOLECULE_DOCKER_COMMAND:-""}
    volumes:
      - /sys/fs/cgroup:/sys/fs/cgroup:ro
    privileged: true
    pre_build_image: true
  - name: instance-rockylinux8
    image: "habbis0/docker-rockylinux8-ansible:latest"
    command: ${MOLECULE_DOCKER_COMMAND:-""}
    volumes:
      - /sys/fs/cgroup:/sys/fs/cgroup:ro
    privileged: true
    pre_build_image: true
  - name: instance-debian11
    image: "habbis0/docker-debian11-ansible:latest"
    command: ${MOLECULE_DOCKER_COMMAND:-""}
    volumes:
      - /sys/fs/cgroup:/sys/fs/cgroup:ro
    privileged: true
    pre_build_image: true
  - name: instance-debian10
    image: "habbis0/docker-debian10-ansible:latest"
    command: ${MOLECULE_DOCKER_COMMAND:-""}
    volumes:
      - /sys/fs/cgroup:/sys/fs/cgroup:ro
    privileged: true
    pre_build_image: true
  - name: instance-ubuntu2004
    image: "habbis0/docker-ubuntu2004-ansible:latest"
    command: ${MOLECULE_DOCKER_COMMAND:-""}
    volumes:
      - /sys/fs/cgroup:/sys/fs/cgroup:ro
    privileged: true
    pre_build_image: true
provisioner:
  name: ansible
  playbooks:
    converge: ${MOLECULE_PLAYBOOK:-converge.yml}
scenario:
  test_sequence:
    - lint
    - destroy
    - dependency
    - syntax
    - create
    - prepare
    - converge
    - idempotence
    - check
    - side_effect
    - verify
    - destroy

```

To test on your machine with molecule since the driver is docker 
that need to be installed.

Run the molecule test.

```
molecule test
```

To just run the playbook to see if you have any problems.

```
molecule converge
```

See [molecule doc](https://molecule.readthedocs.io/en/latest/getting-started.html) for more info.


To change role name in all files in the folder.

```
find . -type f -print0 | xargs -0 sed -i "s/ansible_role_postgresql/new_role/g"

```

And for macos since apple is special.

```
find . -type f -print0 | LC_ALL=C  xargs -0  sed -i "" 's/ansible_role_postgresql/new_role/g'
```

Example site.yml

```
---
- name: setup puppet agent
  gather_facts: yes
  remote_user: root
  #remote_user: ansible
  #become: yes
  #become_method: sudo
  hosts: test
  #hosts: puppet-clients
  #hosts: all
  vars_files:
    -  defaults/main.yml
    #-  defaults/secrets.yml

  roles:
    - { role: ../ansible_role_postgresql }
```
