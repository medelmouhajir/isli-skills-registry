# ISLI AI Skills Registry

This is the central registry for ISLI AI skills. It acts as a directory that the ISLI Board UI fetches to discover and install new capabilities.

## Universal Skill Runtime (USR) - ISLI v2.0

As of ISLI v2.0, skills are now **standalone Dockerized HTTP microservices**. This "Universal Skill Runtime" allows for zero-code integration of any capability, regardless of the language it's written in.

### Adding a Skill

To add a skill to this registry:

1.  Create a public Git repository for your skill.
2.  Ensure your skill repo contains:
    - `isli-skill.yaml`: The manifest defining your skill and its tools.
    - `Dockerfile`: Instructions to build your skill container.
    - A web server exposing the required endpoints.
3.  Add an entry to `index.json` in this repository:

```json
{
  "id": "your-skill-id",
  "name": "Your Skill Name",
  "description": "What your skill does.",
  "author": "Your Name",
  "git_url": "https://github.com/your-username/your-skill-repo",
  "tags": ["utility", "web"]
}
```

4.  Submit a Pull Request.

### Skill Manifest (`isli-skill.yaml`)

Every skill must provide an `isli-skill.yaml` manifest at its root:

```yaml
isli_version: "2.0"
id: "my-skill"                      # Globally unique, kebab-case
name: "My Cool Skill"
description: "Briefly explain the capability"
version: "1.0.0"
author: "YourName"
category: "web"                        # web, content, workspace, etc.

runtime:
  port: 8000                           # Port exposed by your Docker container
  build:
    context: "."
    dockerfile: "Dockerfile"

auth:
  type: "internal_jwt"               # Required for verification by Core

tools:
  - name: "my_tool_action"
    description: "What this specific tool does"
    endpoint: "my-action"            # The HTTP path (e.g., POST /my-action)
    method: "POST"
    parameters:
      type: "object"
      properties:
        query: { type: "string" }
      required: ["query"]
```

### Container Contract

Your container must expose the following HTTP endpoints:

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/health` | `GET` | Health check. Return `{"status": "ok"}`. |
| `/.well-known/isli-manifest` | `GET` | Return the `isli-skill.yaml` content as JSON. |
| `/{endpoint}` | `POST` | One per tool defined in the manifest. Accepts and returns JSON. |

#### Authentication

All incoming requests from the ISLI Core API will include an `X-Internal-Auth` header containing a JWT. Your skill **must** verify this JWT using the `JWT_SECRET` injected into your container environment during the installation/enable flow.

### How it works

1.  **Discovery**: The ISLI Board UI fetches `index.json` to show available skills.
2.  **Installation**: When "Install" is clicked, ISLI Core clones the Git repo and validates the `isli-skill.yaml`.
3.  **Activation**: ISLI Core builds the Docker image and starts the container on the internal `isli_default` network.
4.  **Auto-Registration**: The Agent SDK automatically fetches the manifest from Core and registers the tools with the LLM.

### Legacy Support

The previous file-system-based dynamic skill system (`skill.json` + `main.py`) is still supported for internal backward compatibility but is **deprecated** for new third-party skills. Please use the Universal Skill Runtime (USR) for all new contributions.
