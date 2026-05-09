# pomodoro
Pomodoro shell script
## Install
1. Place the scripts `pomodoro` and `pomodoro_d` into a scripts directory. For my use it's `$HOME/.config/hypr/scripts/` so it may be accessed via Hyprland keybinds
2. Ensure there is a directory for cache files. For my usage, it's `$HOME/.config/hypr/cache/`
3. Call the handler script `pomodoro` along with a command `{ start | skip | stop }`

## Explanation
### pomodoro_d
This script is the actual pomodoro timer. When called it manages its states via cache files, and the timer itself. It runs in the background and is interacted with by the handler script with signals.
The `pkill -RTMIN+2 waybar` is a way for the script to communicate with the waybar module to display the timer in the module. If not being used, this can be commented out.
### pomodoro
This script is the handler script. It parses user command along with the cache files to determine which signals to send to `pomodoro_d`. It is used by calling it along with an argument (Ex. `path/to/pomodoro start`).

## Configuration
### Time and Breaks
There are variables in `pomodoro_d` that can be altered to configure the pomodoro timer, namely 
```bash
declare -ir WORK_TIME=1500
declare -ir SHORT_BREAK=300
declare -ir LONG_BREAK=900
declare -ir NUM_BREAKS=5
```
The first three `{ WORK_TIME, SHORT_BREAK, LONG_BREAK }` are times in seconds equating to 25, 5, and 15 minutes respectively. The last one `NUM_BREAKS` is on which session to have the long break. With it set to 5, this means 4 sessions of 5 minute breaks will be followed by a long break session.
### Cache Directory and Cache Files
This script has a cache directory and 4 cache files:
```bash
CACHE_DIR="$HOME/.config/hypr/cache"
STATE_FILE="$CACHE_DIR/pomodoro_state"
MODE_FILE="$CACHE_DIR/pomodoro_mode"
COUNT_FILE="$CACHE_DIR/pomodoro_count"
PID_FILE="$CACHE_DIR/pomodoro.pid"
```
The `CACHE_DIR` can be changed to fit usecase. The cache files `{ STATE_FILE, MODE_FILE, COUNT_FILE, PID_FILE }` store information regarding the pomodoro timer, namely the state it is in `{ dead, running, paused, stopped }`, the mode it is in `{ work, break }`, the count (# of work + breaks), and the process id of the script so the handler may send signals.
A handful of these files are also within the handler script `pomodoro` and should be configured to match the values in `pomodoro_d`.
### Asset Directory
The `pomodoro` script will send notifications depending on updating actions like starting, pausing and stopping the timer. These notifications also play a sound using `ffplay` calling mp3 files in the asset folder. The asset folder must be correctly referenced in the `pomodoro` script for the sounds to play.

## Usage
- Call `path/to/pomodoro start` either via terminal or via a keybind. This will send a notification that the work timer has started.
- Calling `path/to/pomodoro start` again while the script is running will toggle pause on the pomodoro timer.
- Calling `path/to/pomodoro skip` will skip to and start to the next timer, either ` work -> break ` or ` break -> work `.
- Calling `path/to/pomodoro stop` will stop the current timer and reset it. This does not reset count or mode, but does restart the timer, so calling it in the middle of `#2 break` will stop and when started again via `path/to/pomodoro start` will start `#2 break` from the beginning.
- Calling `path/to/pomodoro stop` a second time will completely reset and kill the timer.
- Upon ending a timer, the end notification will send, and the timer will stay at `00:00` until a `start` or `skip` call is sent.

## Troubleshoot
### 1. The timer crashed and wont start again
If the timer doesn't exit gracefully, it may not run the `cleanup ()` method.
1. Ensure the process is truly terminated, either using `ps axf | grep pomodoro` and preferably, if running, `kill -TERM <PID>`. This should call the cleanup command, but if it still wont start procede to step 2.
2. Run `echo "dead" > "path/to/cache/pomodoro_state"`. The handler script will only start the `pomodoro_d` script if the statefile has the dead state, indicating that the program terminated gracefully.
3. Submit an issue explaining what was happening, and I'll try my best to recreate it and fix it.
### 2. Notifications aren't coming through
Ensure both `notify-send` and `ffplay` are installed, and the script can reach the mp3 files
### 3. The program wont start at all
Ensure the cache directory and asset directory are correctly set in both `pomodoro` and `pomodoro_d` and that the path to `pomodoro_d` is correct in `pomodoro`. The script should create the cache files automatically, but just in case, check if the files exist and if they don't, create them with 
```bash
echo "dead" > "path/to/cache/pomodoro_state"
echo "work" > "path/to/cache/pomodoro_mode"
echo 1 > "path/to/cache/pomodoro_count"
touch "path/to/cache/pomodoro.pid"
```
### 4. The timer froze
Follow the same steps as troubleshoot #1. Running the `pomodoro` script in terminal may print errors as to why the script froze. Submit an issue explaining what happens and with the log if applicable, and I will try my best to fix it.
