# media-init
One-shot container that prepares the shared media library so jellyfin, audiobookshelf and nextcloud agree on ownership.
Use the SAME `MEDIA_UID`, `MEDIA_GID`, `MEDIA_PATH`, `APPDATA_PATH` values in all four stacks.

1. Deploy this stack first. It exits when done; "Exited (0)" in Portainer is success.
2. Exited (1)? Open the container logs, the first line says what failed.
3. Then deploy jellyfin, audiobookshelf, nextcloud.

Layout it enforces (owner `MEDIA_UID:MEDIA_GID`, mode 2775 setgid):
- `MEDIA_PATH/{movies,tv,music}` -> jellyfin (read-only)
- `MEDIA_PATH/{audiobooks,podcasts,books}` -> audiobookshelf (read-only)
- all of `MEDIA_PATH` -> nextcloud (read-write, the only writer; mounted at `/media`, add it as External Storage)
- `APPDATA_PATH/{jellyfin,audiobookshelf}` -> app config/cache/metadata (never shared between apps)

Redeploy it after changing any of the variables. Rootless docker: chown to another uid will fail the preflight.
