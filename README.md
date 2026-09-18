
Here is the breakdown of the 6 steps, including the exact terminal commands and keyboard shortcuts you need.
As a bonus, I have included the direct tmux command-line equivalents for screen splitting and detaching, which allows you to automate your entire setup in a bash script.
 1. Install tmux
   Update your package manager and install the application based on your Linux distribution:
   # For Ubuntu or Debian-based systems
sudo apt update && sudo apt install tmux -y

# For CentOS, RHEL, or Amazon Linux
sudo yum install tmux -y

 2. Start a dedicated session
   Create a new session and name it trading so it is easy to identify later.
   tmux new-session -s trading

 3. Split the screen into panes
   Once inside the trading session, you can split your screen using keyboard shortcuts or by typing commands directly into the terminal prompt.
   Using Keyboard Shortcuts:
   * Vertical split (left/right): Press Ctrl+b, release, then press %
   * Horizontal split (top/bottom): Press Ctrl+b, release, then press "
   * Switch between panes: Press Ctrl+b, release, then use your Arrow Keys (↑ ↓ ← →)
   Using Commands (can be scripted):
   # Split the current pane left/right
tmux split-window -h

# Split the current pane top/bottom
tmux split-window -v

 4. Run your strategies
   Navigate to your respective panes and start your Python scripts. The -u flag prevents Python from buffering the output, ensuring you see trade executions immediately.
   # Inside Pane 1 (e.g., your Bitcoin strategy)
python3 -u btc_momentum.py

# Inside Pane 2 (e.g., your Ethereum strategy)
python3 -u eth_arbitrage.py

 5. Detach from the session
   Detaching leaves the session running safely in the background so you can close your SSH connection.
   Using Keyboard Shortcut:
   Press Ctrl+b, release, then press d
   Using Command:
   tmux detach

 6. Reattach later
   When you log back into your server via SSH, you can view your running sessions and re-enter your trading environment.
   # View all running background sessions
tmux ls
# Output example: trading: 1 windows (created Sat Sep 19 12:00:00 2026)

# Reattach to your specific session
tmux attach-session -t trading

Pro-Tip: Automate the Setup
Instead of typing these commands manually every time your server reboots, you can create a single bash script (start_trading.sh) that builds your session, splits the windows, launches the Python scripts, and attaches you to it automatically:
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



Here is a complete bash script that automatically creates a detached tmux session, splits the screen into three panes, launches a different Python strategy in each, and elegantly arranges them.
Because the session is launched in detached mode (-d), it runs entirely in the background. You can run this script and immediately close your SSH console without any fear of the programs terminating.
The Bash Script (start_bots.sh)
Create a new file on your server (e.g., nano start_bots.sh) and paste the following code:
#!/bin/bash

# Define the name of your tmux session
SESSION_NAME="algo_trading"

# Check if the session already exists to prevent duplicates
tmux has-session -t $SESSION_NAME 2>/dev/null
if [ $? == 0 ]; then
  echo "Error: Session '$SESSION_NAME' is already running."
  echo "Attach to it using: tmux attach -t $SESSION_NAME"
  exit 1
fi

echo "Initializing tmux session: $SESSION_NAME..."

# 1. Create a new detached session (-d) named $SESSION_NAME
tmux new-session -d -s $SESSION_NAME

# 2. Run the 1st Python script in the initial pane (Pane 0)
# 'C-m' simulates pressing the Enter key
tmux send-keys -t $SESSION_NAME:0 'echo "Starting Strategy 1..." && python3 -u strategy_1.py' C-m

# 3. Split the screen horizontally (creates a right pane) and run the 2nd script (Pane 1)
tmux split-window -h -t $SESSION_NAME:0
tmux send-keys -t $SESSION_NAME:0.1 'echo "Starting Strategy 2..." && python3 -u strategy_2.py' C-m

# 4. Split the right pane vertically (creates a bottom-right pane) and run the 3rd script (Pane 2)
tmux split-window -v -t $SESSION_NAME:0
tmux send-keys -t $SESSION_NAME:0.2 'echo "Starting Strategy 3..." && python3 -u strategy_3.py' C-m

# (Optional) If you want to run a 4th script, uncomment the lines below:
# tmux split-window -v -t $SESSION_NAME:0.0  # Splits the left pane
# tmux send-keys -t $SESSION_NAME:0.3 'python3 -u strategy_4.py' C-m

# 5. Arrange the panes into a neat, balanced layout
tmux select-layout -t $SESSION_NAME tiled

echo "✅ Success! All strategies have been launched in the background."
echo "🔌 You can safely close your SSH connection now."
echo "🖥️  To view your live terminals later, run: tmux attach -t $SESSION_NAME"

How to Use This Script
1. Make the script executable
Before Linux will let you run the script, you must give it execution permissions. Run this in your terminal:
chmod +x start_bots.sh

2. Run the script
Execute the script to start your bots in the background:
./start_bots.sh

3. Safely Logout
Because the script tells tmux to start in the background, it instantly returns you to your normal terminal prompt. You can now type exit or close your SSH window. Your Python scripts will happily continue trading on the server.
4. Check on your bots later
Whenever you log back into your server via SSH, simply type this command to bring your 3-pane trading dashboard to the foreground:
tmux attach -t algo_trading

(To leave the session again and keep them running, press Ctrl+b, release, then press d).
