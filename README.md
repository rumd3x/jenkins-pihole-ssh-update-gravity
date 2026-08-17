# jenkins-pihole-ssh-update-gravity

A Jenkins declarative pipeline that remotely triggers a [Pi-hole](https://pi-hole.net/) gravity update (`pihole -g`) over SSH, with support for multiple Pi-hole containers on the same host.

## Features

- Connects to any host via SSH using username/password credentials stored in Jenkins
- Automatically detects whether `docker` requires `sudo` on the target host
- Discovers **all** running `pihole/pihole` containers and updates each one
- Sets a descriptive build title and description in the Jenkins UI

## Requirements

- Jenkins agent with label `docker`
- `sshpass` installed on the Jenkins agent
- SSH access (username + password) to the target host
- Pi-hole running as a Docker container (`pihole/pihole` image) on the target host

## Parameters

| Parameter | Type | Description |
|---|---|---|
| `HOST` | String | Hostname or IP address of the Pi-hole host |
| `SSH_CREDENTIALS` | Credentials | Jenkins username+password credential for SSH access |

## Usage

1. Create a new Jenkins Pipeline job and point it at this `Jenkinsfile`.
2. Add a username+password credential in Jenkins for the target host (e.g. `pihole-host-ssh`).
3. Run the job, providing the host and selecting the credential.

The pipeline will SSH into the host, find every running `pihole/pihole` container, and exec `pihole -g` inside each one.

## License

[MIT](LICENSE)
