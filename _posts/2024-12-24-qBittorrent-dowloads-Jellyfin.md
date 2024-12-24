---
title: "Fix qBittorrent downloads for Jellyfin"
categories:
  - self-host
comments: true
---

Jellyfin requires the folders to and subfolders in each collection to have 755 (executable) permissions to traverse them.

qBittorrent uses the `umask` set by default (`0022`) which sets the permissions to 644.

Let's create the script that fixes the permissions for torrents downloaded and tagged with the categories `Films` or `TV Series`:

```bash
#!/usr/bin/env bash

F=$1  # Passes the first command-line argument to F
L=$2  # Passes the second command-line argument to L

if [[ "$L" == "Films" || "$L" == "TV Series" ]]; then  
    find "$F" -type d -exec chmod 755 {} \;  # Apply 755 to all directories recursively
    find "$F" -type f -exec chmod 644 {} \;  # Apply 644 to all files recursively
fi

```

Save the file, grant exec permissions (`+x`).

Open qBittorrent, go to Downloads -> Run on torrent finished

Insert:
`/path/to/your/script %F %L`

Now the file structure and its permissions should be correct for Jellyfin to use.



