## Overview

- Creates a Jenkins worker using vagrant and Ansible Provisioning
- Vagrantfile installs dependencies: Java, Python, Ansible
- Creates Jenkins user and group
- Downloads a Jenkins slave docker image for the containers to run the workers
- Sets up access to the Docker Socket
- Creates and configures a Swap file to improve server performance

### Setup

1. Install Vagrant

2. Initialise Vagrant
```
vagrant init
```

3. To create multiple workers, modify the json file
```
jenkins_worker/json/vconfig.json
```

4. Create Jenkins Workers
```
vagrant up
```

#### License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details
