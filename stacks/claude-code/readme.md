# claude-code
Sandboxed Claude Code you drive from https://claude.ai/code (or the Claude mobile app) via Remote Control.
Uses the community image [beevelop/claude](https://github.com/beevelop/docker-claude), pinned by digest. No ports are published; the container only connects outbound.

## First-time setup
1. Create data dirs on the host and give them to the image's `developer` user
   (uid is probably 1001 on ubuntu:24.04; check with `docker run --rm --entrypoint id beevelop/claude developer`):
   ```sh
   mkdir -p /home/luca/claude-code/{claude,workspace} && sudo chown -R 1001:1001 /home/luca/claude-code
   ```
2. Deploy the stack (Portainer git stack, env vars from `example.env`).
3. One-time login: attach to the container (Portainer: container > Attach console, or `docker attach claude-code`),
   open the URL it prints, finish OAuth, press Ctrl+C, then detach with Ctrl+P Ctrl+Q.
4. Open claude.ai/code and pick the session named `CLAUDE_SESSION_NAME`.

## Notes
- Needs a Claude subscription login; API keys don't support Remote Control. Login persists in `${DATA_PATH}claude`.
- Hardening: all caps dropped, no-new-privileges (the image's `developer` has passwordless sudo, this blocks it), memory/pids limits, no docker socket.
- Git: use a deploy key scoped to a single repo. It sits in plain text in the Portainer stack env.
- Updating: replace the digest in the compose file with a newer one from Docker Hub (only `latest`/`nightly` tags exist).
- The network is not isolated from the LAN.
