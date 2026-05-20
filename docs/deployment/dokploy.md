# Deploy OpenHands with Dokploy

This guide deploys the local OpenHands web application with Dokploy using
Docker Compose.

## Requirements

- A Dokploy server with Docker available.
- A domain or subdomain pointing to the Dokploy server.
- Permission to mount `/var/run/docker.sock`. OpenHands uses it to start
  sandbox containers for agent tasks.

> Security note: mounting the Docker socket gives this service control over the
> host Docker daemon. Only deploy this for trusted users and protect it behind
> authentication or a private network.

## Dokploy service

1. In Dokploy, create a new **Compose** service.
2. Use this repository as the Git source.
3. Select **Docker Compose** as the compose type. Do not use Docker Stack,
   because this file builds the application image from the repository.
4. Set **Compose Path** to:

   ```text
   ./docker-compose.dokploy.yml
   ```

5. Add a domain in the Dokploy **Domains** tab and route it to port `3000`.
   Dokploy will add the required Traefik labels automatically.
6. Deploy the service.

## Environment variables

Dokploy writes variables from its UI to a `.env` file next to the compose file.
`docker-compose.dokploy.yml` loads that file with `env_file`.

Common variables:

```dotenv
AGENT_SERVER_IMAGE_REPOSITORY=ghcr.io/openhands/agent-server
AGENT_SERVER_IMAGE_TAG=1.22.1-python

# Optional LLM defaults. Users can also configure these in the Settings UI.
LLM_MODEL=anthropic/claude-sonnet-4-5
LLM_API_KEY=replace-me
```

By default, the compose file stores the OpenHands state in a Docker named volume
and workspaces under Dokploy's persistent `../files/workspace` directory.

If sandbox containers fail with workspace mount errors, set both of these
variables to the absolute host path for the same persistent workspace directory:

```dotenv
WORKSPACE_HOST_PATH=/absolute/path/to/dokploy/files/workspace
WORKSPACE_MOUNT_PATH=/absolute/path/to/dokploy/files/workspace
```

## Updating

Push changes to the selected branch or trigger the Dokploy webhook. Dokploy will
rebuild the image from `containers/app/Dockerfile` and restart the service.

## Troubleshooting

- **The site does not open:** verify the domain points to the Dokploy server and
  that the Dokploy Domains tab routes to port `3000`.
- **Variables do not appear in the app:** keep variables in the Compose service
  environment editor; Dokploy writes them to `.env`, which the compose file
  loads with `env_file`.
- **Sandbox startup fails:** confirm `/var/run/docker.sock` is mounted and the
  Dokploy server can pull `ghcr.io/openhands/agent-server:1.22.1-python`.
- **Workspace mount errors:** set `WORKSPACE_HOST_PATH` and
  `WORKSPACE_MOUNT_PATH` to an absolute persistent host path as shown above.
