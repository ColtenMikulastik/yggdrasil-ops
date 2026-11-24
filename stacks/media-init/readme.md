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

# Adding media
Nextcloud is the only app that writes to the library; jellyfin and audiobookshelf mount it read-only.
The normal procedure is: upload through nextcloud, rescan, then refresh the app that plays it.

## 1. One-time: expose the library in Nextcloud
1. Admin > Apps > enable **External storage support**.
2. Admin > External storage > add folder `media`, type **Local**, configuration `/media`, available for the users you want.
3. Optionally set "check for changes" to "Once per direct access" so outside changes show up.

## 2. Put files in the right folder
Folder names matter, the apps parse them:

| what | folder | layout |
| ---- | ------ | ------ |
| audiobooks | `audiobooks/` | `Author/Title/files.m4b` (series: `Author/Series/Title/`) |
| podcasts | `podcasts/` | `Podcast name/episode.mp3` |
| ebooks | `books/` | `Author/Title.epub` |
| movies | `movies/` | `Movie Name (2019)/Movie Name (2019).mkv` |
| tv | `tv/` | `Show Name/Season 01/Show Name S01E01.mkv` |
| music | `music/` | `Artist/Album/01 Track.flac` |

Keep audiobooks out of `music/` and vice versa, jellyfin will try to import them as music.

## 3. Rescan
Files added through the Nextcloud web UI or sync clients are seen immediately.
Files copied directly onto the host (scp, rsync, mv) are not, run:
```sh
docker exec -u www-data <nextcloud-app-container> php occ files:scan --all
```

## 4. Refresh the player
- audiobookshelf: Settings > Libraries > the library > **Scan** (point libraries at `/audiobooks`, `/podcasts`, `/books`).
- jellyfin: Dashboard > Libraries > **Scan All Libraries** (libraries point at `/data/movies`, `/data/tv`, `/data/music`). Turn off "Save artwork/nfo into media folders" since the mount is read-only.

## Copying directly on the host
Prefer Nextcloud. If you copy on the host, files must be readable by the media group, otherwise the apps see an empty library:
```sh
sudo chown -R <MEDIA_UID>:<MEDIA_GID> <MEDIA_PATH>/audiobooks/NewAuthor
sudo chmod -R u+rwX,g+rwX,o+rX <MEDIA_PATH>/audiobooks/NewAuthor
```

## Troubleshooting
- File shows in nextcloud but not in the player: rescan the player, then check `ls -ln` on the file (group should be `MEDIA_GID` or the file world-readable).
- Player library empty after deploy: check that `media-init` exited 0.
- Nextcloud "permission denied" on upload: check the folder is `drwxrwsr-x` and the nextcloud container has `group_add` for `MEDIA_GID`.
