# Ansible environment setup with Docker

1. Create a ```docker-compose.yml``` file with
    1. Ansible Controller (Debian 9) (install openssh-server, Docker and ansible with Dockerfile)
    2. Target Server 1 (Debian 9) (install openssh-server with Dockerfile.target1)
    3. Target Server 2 (Debian 9) (install openssh-server with Dockerfile.target2)
    4. Target Server 3 (Debian 9) (install openssh-server with Dockerfile.target3)
2. Create volume of "Ansible Controller" in local computer

