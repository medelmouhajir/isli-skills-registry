# ISLI AI Skills Registry

This is the central registry for ISLI AI skills. It acts as a directory that the ISLI Board UI fetches to discover and install new capabilities.

## Structure

- `index.json`: The main directory of skills.
- `schemas/`: (Optional) JSON schemas for skill validation.

## Adding a Skill

To add a skill to this registry:

1.  Create a public Git repository for your skill.
2.  Ensure your skill repo contains:
    - `skill.json`: Metadata about your skill.
    - `main.py`: The entry point with a `handle(action, payload, db)` function.
3.  Add an entry to `index.json` in this repository:

```json
{
  "id": "your-skill-id",
  "name": "Your Skill Name",
  "description": "What your skill does.",
  "author": "Your Name",
  "git_url": "https://github.com/your-username/your-skill-repo",
  "tags": ["utility", "etc"]
}
```

4.  Submit a Pull Request.

## How it works

The ISLI Board UI fetches `index.json` directly from GitHub. When a user clicks "Install", the ISLI Core API clones the repository and loads it dynamically.
