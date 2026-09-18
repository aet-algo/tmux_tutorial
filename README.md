
# Running Multiple Python Trading Bots on a Linux Server (via SSH)

 This guide covers how to run multiple Python algorithmic trading strategies simultaneously on a Linux server using `tmux`. By using a terminal multiplexer, you can monitor all your scripts side-by-side and safely close your SSH connection without terminating the processes.

---

 ## 🚀 Manual Setup (6 Steps)

 Here is the breakdown of the 6 steps, including the exact terminal commands and keyboard shortcuts you need.

 ### 1\. Install tmux

 Update your package manager and install `tmux` based on your Linux distribution.

 **For Ubuntu or Debian-based systems:**

```
sudo apt update && sudo apt install tmux -y
```

 **For CentOS, RHEL, or Amazon Linux:**

```
sudo yum install tmux -y
```

---

 ### 2\. Start a Dedicated Session

 Create a new session and name it `trading` so it is easy to identify later.

```
tmux new-session -s trading
```

---

 ### 3\. Split the Screen into Panes

 Once inside the `trading` session, you can split your screen using keyboard shortcuts or by typing commands directly into the terminal.

 #### Using Keyboard Shortcuts

 - **Vertical split (left/right):** Press `Ctrl+b`, release, then press `%`.
- **Horizontal split (top/bottom):** Press `Ctrl+b`, release, then press `"`.
- **Switch between panes:** Press `Ctrl+b`, release, then use the Arrow Keys (`↑`, `↓`, `←`, `→`).

 #### Using Commands

 These commands can also be used when scripting your setup:

 **Split the current pane left/right:**

```
tmux split-window -h
```

 **Split the current pane top/bottom:**

```
tmux split-window -v
```

---

 ### 4\. Run Your Strategies

 Navigate to your respective panes and start your Python scripts.

 The `-u` flag prevents Python from buffering the output, ensuring you see trade executions immediately.

 **Inside Pane 1 — Bitcoin strategy:**

```
python3 -u btc_momentum.py
```

 **Inside Pane 2 — Ethereum strategy:**

```
python3 -u eth_arbitrage.py
```

---

 ### 5\. Detach From the Session

 Detaching leaves the `tmux` session running safely in the background so you can close your SSH connection.

 #### Using Keyboard Shortcut

 Press:

```
Ctrl+b
```

 Release the keys, then press:

```
d
```

 #### Using Command

 Alternatively, run:

```
tmux detach
```

---

 ### 6\. Reattach Later

 When you log back into your server via SSH, you can view your running sessions and re-enter your trading environment.

 **View all running background sessions:**

```
tmux ls
```

 Example output:

```
trading: 1 windows (created Sat Sep 19 12:00:00 2026)
```

 **Reattach to your specific session:**

```
tmux attach-session -t trading
```

---

 ## ⚡ Pro Tip: Automate the Setup

 Instead of typing these commands manually every time your server reboots, you can create a single Bash script named `start_trading.sh`.

 The script can:

 - Create your `tmux` session.
- Split the windows/panes.
- Launch your Python scripts.
- Attach you to the session automatically.

 ### `start_trading.sh`

 Create the file:

```
nano start_trading.sh
```

 Then add:

```
#!/bin/bash

# 1. Create a new detached session named 'trading'
tmux new-session -d -s trading

# 2. Send the first Python command to the first pane
tmux send-keys -t trading:0 'python3 -u btc_momentum.py' C-m

# 3. Split the window vertically
tmux split-window -h -t trading

# 4. Send the second Python command to the new right-side pane
tmux send-keys -t trading:0.1 'python3 -u eth_arbitrage.py' C-m

# 5. Attach to the session to see them running
tmux attach-session -t trading
```

 Save the file and make it executable:

```
chmod +x start_trading.sh
```

 Run it with:

```
./start_trading.sh
```

---

 ## 🛠️ Advanced: The Multi-Bot Background Script

 Here is a complete Bash script that automatically:

 - Creates a detached `tmux` session.
- Splits the screen into three panes.
- Launches a different Python strategy in each pane.
- Arranges the panes into a balanced layout.

 Because the session is launched in detached mode (`-d`), the `tmux` session continues running after you disconnect from SSH.

 ### The Bash Script — `start_bots.sh`

 Create a new file on your server:

```
nano start_bots.sh
```

 Paste the following code into the file:

```
#!/bin/bash

# Define the name of your tmux session
SESSION_NAME="algo_trading"

# Check if the session already exists to prevent duplicates
tmux has-session -t "$SESSION_NAME" 2>/dev/null

if [ $? -eq 0 ]; then
    echo "Error: Session '$SESSION_NAME' is already running."
    echo "Attach to it using: tmux attach -t $SESSION_NAME"
    exit 1
fi

echo "Initializing tmux session: $SESSION_NAME..."

# 1. Create a new detached session (-d) named $SESSION_NAME
tmux new-session -d -s "$SESSION_NAME"

# 2. Run the 1st Python script in the initial pane (Pane 0)
# 'C-m' simulates pressing the Enter key
tmux send-keys -t "$SESSION_NAME:0.0" \
    'echo "Starting Strategy 1..." && python3 -u strategy_1.py' C-m

# 3. Split the screen horizontally
# Creates a right-side pane (Pane 1)
tmux split-window -h -t "$SESSION_NAME:0.0"

tmux send-keys -t "$SESSION_NAME:0.1" \
    'echo "Starting Strategy 2..." && python3 -u strategy_2.py' C-m

# 4. Split the right pane vertically
# Creates a bottom-right pane (Pane 2)
tmux split-window -v -t "$SESSION_NAME:0.1"

tmux send-keys -t "$SESSION_NAME:0.2" \
    'echo "Starting Strategy 3..." && python3 -u strategy_3.py' C-m

# 5. Arrange the panes into a neat, balanced layout
tmux select-layout -t "$SESSION_NAME:0" tiled

echo "✅ Success! All strategies have been launched in the background."
echo "🔌 You can safely close your SSH connection now."
echo "🖥️ To view your live terminals later, run:"
echo "   tmux attach -t $SESSION_NAME"
```

---

 ## How to Use This Script

 ### 1\. Make the Script Executable

 Before Linux will let you run the script, give it execution permissions.

 Run:

```
chmod +x start_bots.sh
```

---

 ### 2\. Run the Script

 Execute the script to start your bots in the background:

```
./start_bots.sh
```

---

 ### 3\. Safely Logout

 Because the script tells `tmux` to start in detached mode, it returns you to your normal terminal prompt.

 You can now type:

```
exit
```

 Or simply close your SSH window.

 Your Python scripts will continue running inside the `tmux` session on the server.

---

 ### 4\. Check on Your Bots Later

 Whenever you log back into your server via SSH, run:

```
tmux attach -t algo_trading
```

 This brings your 3-pane trading dashboard back to the foreground.

 To leave the session again while keeping the bots running:

```
Ctrl+b
```

 Release the keys, then press:

```
d
```

---

 ## Quick Command Reference

 | Action | Command / Shortcut |
| --- | --- |
| Create session | `tmux new-session -s trading` |
| List sessions | `tmux ls` |
| Attach to session | `tmux attach -t trading` |
| Detach | `Ctrl+b`, then `d` |
| Split left/right | `Ctrl+b`, then `%` |
| Split top/bottom | `Ctrl+b`, then `"` |
| Switch panes | `Ctrl+b`, then Arrow Key |
| Split horizontally | `tmux split-window -h` |
| Split vertically | `tmux split-window -v` |
| Detach via command | `tmux detach` |
| Make Bash script executable | `chmod +x start_bots.sh` |
| Run Bash script | `./start_bots.sh` |
| Logout SSH | `exit` |

 I also made a small robustness improvement in the advanced script: the `tmux` targets are explicitly quoted and the pane targets are clearer (`0.0`, `0.1`, `0.2`), which makes the script less error-prone when copied directly to a server.
