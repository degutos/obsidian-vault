

```sh
tmux # start new tmux session
tmux CTRL+b+d # detach from your tmux session 
tmux attach # attach to a previously session
CTRL+b+$ # rname your current tmux session
tmux attach -t client # attach to your session client
tmux attach -t S # attach to a session that starts with "S", since only one session with "S" that will work!
CTRL+b+( # move to next tmux session 
CTRL+b+"  # split screen horizontally
CTRL+b+arrow-up # move cursor to up panel 
CTRL+b+% # split screen vertically
watch 'df -hP' # good command to monitor filesystem usage
CTRL+b+z # zoom in and out the pannel 
CTRL+b+c # open new window
CTRL+b+w show all window and session on menu fancy way
CTRL+b+n # next windows
CTRL+b+p # previously window
CTRL+b:set -g mouse # to use the mouse to scroll up and not scroll through your history command
```

