🎨 GRUB Background Image Cycling (with Wuthering Theme)
=======================================================

This guide walks you through installing the beautiful [Wuthering GRUB2 theme](https://github.com/vinceliuice/Wuthering-grub2-themes) and setting it up to display a **different background image on every reboot**.

* * *

✅ Step 1: Install the Wuthering GRUB Theme
------------------------------------------

1. Clone and install the theme:
    
    ```bash
    git clone https://github.com/vinceliuice/Wuthering-grub2-themes.git
    cd Wuthering-grub2-themes
    sudo ./install.sh
    ```
    
2. Choose your preferred variant (e.g., `Wuthering-jinxi`). The theme will be installed in:
    
    ```
    /usr/share/grub/themes/Wuthering-jinxi/
    ```
    

* * *

✅ Step 2: Prepare Background Images
-----------------------------------

1. Create a folder for GRUB background images:
    
    ```bash
    mkdir -p ~/Pictures/GRUB-Backgrounds/1920x1080
    ```
    
2. Add `.jpg` or `.png` images of **1920x1080** resolution into that folder.
    
3. Make sure they are readable:
    
    ```bash
    chmod a+r ~/Pictures/GRUB-Backgrounds/1920x1080/*
    ```
    

* * *

✅ Step 3: Create the Cycling Script
-----------------------------------

We’ll use a script that randomly or sequentially selects an image at boot and applies it as GRUB’s background.

> Place the script in `/usr/local/bin/`.

```bash
sudo nano /usr/local/bin/grub-bg-cycle.sh
```

### 🔁 Option A: **Sequential Rotation**

```bash
#!/bin/bash

IMAGE_DIR="/home/hannan/Pictures/GRUB-Backgrounds/1920x1080"
DEST="/usr/share/grub/themes/Wuthering-jinxi/background.jpg"
LOG_FILE="/var/log/grub-bg-cycle.log"
STATE_FILE="/var/lib/grub-bg-index"

mkdir -p "$(dirname "$STATE_FILE")"

# Read images into array
mapfile -t IMAGES < <(find "$IMAGE_DIR" -type f \( -iname '*.jpg' -o -iname '*.png' \) | sort)
NUM_IMAGES=${#IMAGES[@]}

if [ "$NUM_IMAGES" -eq 0 ]; then
    echo "[$(date)] No images found in $IMAGE_DIR" >> "$LOG_FILE"
    exit 1
fi

# Read last index or initialize
if [[ -f "$STATE_FILE" ]]; then
    INDEX=$(<"$STATE_FILE")
    [[ "$INDEX" =~ ^[0-9]+$ ]] || INDEX=-1
else
    INDEX=-1
fi

# Increment and loop index
INDEX=$(( (INDEX + 1) % NUM_IMAGES ))
echo "$INDEX" > "$STATE_FILE"

SELECTED_IMAGE="${IMAGES[$INDEX]}"
cp "$SELECTED_IMAGE" "$DEST"
update-grub

echo "[$(date)] Set GRUB background to: $SELECTED_IMAGE (index $INDEX)" >> "$LOG_FILE"

# Optional: Keep log file to last 50 entries
LOG_MAX=100
tail -n "$LOG_MAX" "$LOG_FILE" > "${LOG_FILE}.tmp" && mv "${LOG_FILE}.tmp" "$LOG_FILE"
```

> Replace `YOUR_USERNAME` in `IMAGE_DIR` with your actual username.

Make it executable and set permissions:

```bash
sudo chmod +x /usr/local/bin/grub-bg-cycle.sh
sudo chown root:root /usr/local/bin/grub-bg-cycle.sh
sudo touch /var/lib/grub-bg-index
sudo chown root:root /var/lib/grub-bg-index
```

* * *

✅ Step 4: Create the `systemd` Service
--------------------------------------

Create the service unit:

```bash
sudo nano /etc/systemd/system/grub-bg-cycle.service
```

Paste the following:

```ini
[Unit]
Description=Cycle GRUB Background Image at Boot
After=network.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/grub-bg-cycle.sh
StandardOutput=append:/var/log/grub-bg-cycle.service.log
StandardError=append:/var/log/grub-bg-cycle.service.log

[Install]
WantedBy=multi-user.target
```

* * *

✅ Step 5: Enable the Service
----------------------------

1. Reload systemd:
    
    ```bash
    sudo systemctl daemon-reload
    ```
    
2. Enable the service to run on boot:
    
    ```bash
    sudo systemctl enable grub-bg-cycle.service
    ```
    
3. Test the service manually:
    
    ```bash
    sudo systemctl start grub-bg-cycle.service
    ```
    
4. Check the logs:
    
    ```bash
    cat /var/log/grub-bg-cycle.service.log
    cat /var/log/grub-bg-cycle.log
    ```
    

* * *

🔁 Bonus: Use Random Instead of Sequential (Optional)
-----------------------------------------------------

If you'd rather have a **random image** each time, replace the script body with this version:

```bash
mapfile -t IMAGES < <(find "$IMAGE_DIR" -type f \( -iname '*.jpg' -o -iname '*.png' \))

if [ ${#IMAGES[@]} -eq 0 ]; then
  echo "[$(date)] No images found in $IMAGE_DIR" >> "$LOG_FILE"
  exit 1
fi

SELECTED_IMAGE="${IMAGES[RANDOM % ${#IMAGES[@]}]}"
cp "$SELECTED_IMAGE" "$DEST"
update-grub
echo "[$(date)] Random GRUB background set to: $SELECTED_IMAGE" >> "$LOG_FILE"
```

* * *

✅ Step 6: Reboot and Enjoy
--------------------------

```bash
sudo reboot
```

On each reboot, your GRUB background image will update automatically! 🎉

* * *
