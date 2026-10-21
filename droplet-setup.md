# Droplet Setup

A DigitalOcean Droplet running Linux, used only for bug bounty recon. Keeping scans on a cloud server keeps them off my home network and lets long scans run while my laptop is closed.

## Access

- **SSH keys only.** Password login is disabled; the key is protected with a passphrase.
- From Windows, I connect with a saved SSH host alias:

  ```
  ssh recon
  ```

- **VS Code Remote-SSH** opens the Droplet's files in my editor, so I can edit scripts, read output, and download results in one place.

## Keeping scans alive with tmux

Long scans keep running after I disconnect.

```
tmux new -s recon          # start a named session
Ctrl-b  d                  # detach; everything keeps running
tmux ls                    # list sessions
tmux attach -t recon       # reattach
tmux kill-session -t recon # end the session

Ctrl-b  c                  # new window
Ctrl-b  n / p              # next / previous window
Ctrl-b  0-9                # jump to window by number
Ctrl-b  ,                  # rename window
Ctrl-b  "                  # split pane horizontally
Ctrl-b  %                  # split pane vertically
```

## Temporary file server

To view or grab output files in a browser, I start a short-lived web server, open the port, and close it again when I'm done.

```
python3 -m http.server 8000

sudo ufw allow 8000/tcp          # open the port
# browse to http://<droplet-ip>:8000
sudo ufw delete allow 8000/tcp   # close it again afterward
```

The server has no authentication, so the port stays closed except while I'm using it.
