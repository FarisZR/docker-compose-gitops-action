# Docker Compose Gitops Action
A [GitHub Action](https://github.com/marketplace/actions/docker-compose-gitops) making GitOps with the simplicity of docker-compose possible, using SSH or optionally Tailscale SSH, with support for docker swarm, uploading directory for bind mounts and other features!

**Note:** This action has been refactored into a [composite action](https://docs.github.com/en/actions/creating-actions/creating-a-composite-action). This change aims to speed up workflow execution by eliminating the need to pull a Docker image for the action itself, running steps directly on the runner. The functionality and inputs remain the same.

The Action is adapted from work by [TapTap21](https://github.com/TapTap21/docker-remote-deployment-action) and [wshihadeh](https://github.com/marketplace/actions/docker-deployment).

## Example

Here is an example of how to use the action. Usage remains the same as before:

```yaml
- name: Tailscale
  uses: tailscale/github-action@v2 # Consider using a more specific version/commit SHA
  with:
    authkey: ${{ secrets.TAILSCALE_AUTHKEY }}
    hostname: Github-actions # Optional: set a hostname for the Tailscale node

- name: Start Deployment
  uses: FarisZR/docker-compose-gitops-action@v1 # Or your current version
  with:
    remote_docker_host: root@your_tailscale_ip_or_hostname # e.g., root@100.x.x.x or your custom hostname
    tailscale_ssh: true # Set to true if using Tailscale for SSH
    compose_file_path: postgres/docker-compose.yml
    upload_directory: true
    docker_compose_directory: postgres
    # Example Docker login (optional)
    # docker_login_user: ${{ secrets.DOCKER_HUB_USER }}
    # docker_login_password: ${{ secrets.DOCKER_HUB_PASSWORD }}
    # docker_login_registry: docker.io # Optional, defaults to Docker Hub
    args: -p postgres up -d --remove-orphans
```

## Action Inputs

The inputs remain unchanged:

- `args` - Docker compose/stack command arguments. Example: `-p app_stack_name -d up` (required)
- `remote_docker_host` - Specify Remote Docker host. The input value must be in the following format `user@host` (required)
- `tailscale_ssh` - Enables Tailscale SSH mode, which leverages Tailscale's managed SSH connections. If `true`, the `ssh_public_key` and `ssh_private_key` inputs are not required by this action (though your Tailscale setup handles authentication). Default: `false`
- `ssh_public_key` - Remote Docker SSH public key. Required when `tailscale_ssh` is `false`.
- `ssh_private_key` - SSH private key used in PEM format to connect to the docker host. Required when `tailscale_ssh` is `false`.
- `ssh_port` - The SSH port to be used. Default is `22`.
- `compose_file_path` - Docker compose file path. Default is `docker-compose.yml` (in the repo root). Example for a sub-directory: `caddy/docker-compose.yml`
- `upload_directory` - If `true`, uploads the `docker_compose_directory`. Useful for configuration files needed alongside your containers. Default: `false` (Optional)
- `docker_compose_directory` - Specifies which directory in the repository to upload. Required if `upload_directory` is `true`.
- `post_upload_command` - Optional command to execute on the remote host after a successful upload (if `upload_directory` is `true`). Useful for tasks like setting file permissions.
- `docker_swarm` - If `true`, uses `docker stack deploy` for Docker Swarm mode instead of `docker compose`. Default: `false`
- `docker_login_user` - The username for your container registry (e.g., Docker Hub, GHCR, ECR). (Optional)
- `docker_login_password` - The password or access token for your container registry user. (Optional)
- `docker_login_registry` - The container registry hostname (e.g., `ghcr.io`, `your_aws_account_id.dkr.ecr.your_region.amazonaws.com`). If not specified, defaults to Docker Hub (`docker.io`). (Optional)

## Development & Testing

This action includes a testing workflow located at `.github/workflows/test.yml`. This workflow automatically tests various functionalities of the action upon pushes and pull requests. Key features of the testing setup include:

- **Service Container:** An SSH server (`rastasheep/ubuntu-sshd`) is run as a service container to act as a mock remote host.
- **SSH Key Management:** SSH keys are dynamically generated and configured for communication between the action and the mock SSH server.
- **Docker Command Mocking:** The `docker` command is replaced with a mock script during tests. This script logs the calls made to `docker` (e.g., `context create`, `login`, `compose`, `stack deploy`), allowing verification of the action's command construction logic without requiring a full Docker-in-Docker setup.
- **Test Cases:** The workflow includes tests for:
    - Basic SSH connectivity and Docker context setup.
    - File uploads using `upload_directory` and `post_upload_command`.
    - Docker Swarm mode (`docker_swarm: true`).
    - Docker registry login.
    - Tailscale SSH mode (verifying that SSH key inputs are not strictly required by the action).

This testing suite helps ensure the reliability of the action and serves as a reference for future development.

## License

This project is licensed under the MIT license. See the [LICENSE](LICENSE) file for details.
