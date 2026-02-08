# I have an unused laptop at home which has 500 GB of storage. I would like to store photos in that and I am ok with syncing it every night with my mobile. But I would prefer the sync to happen automatically. I can keep the laptop switched on for one hour everyday in the night. What software do I need?

Given your constraints (Laptop on for 1 hour, local network, automatic sync), the best software for this is **Syncthing**.

### The Recommendation: Syncthing

Syncthing is open-source, completely free, and private. It connects your phone and laptop directly over your Wi-Fi without sending data to the internet.

**Why it fits your scenario:**
1.  **Intermittent Connectivity:** It is designed to handle devices going offline. If your laptop is off, nothing happens. The moment you turn the laptop on, Syncthing detects it, connects, and resumes the transfer exactly where it left off.
2.  **Speed:** Since it transfers over local Wi-Fi, it is extremely fast (much faster than uploading to the internet).
3.  **Storage:** It utilizes your full 500GB local drive.

---

### How to set it up (Step-by-Step)

#### 1. Install on Laptop (The Destination)
*   Download **Syncthing** (Windows/Mac/Linux) from `syncthing.net`.
*   Alternatively, for Windows, use **SyncTrayzor** (a wrapper that makes it easier to use and minimizes it to the system tray).
*   Run it. It will open a dashboard in your web browser (usually `127.0.0.1:8384`).

#### 2. Install on Mobile (The Source)
*   **Android:** Download the official **Syncthing** app.
*   **iOS (iPhone):** Download **Möbius Sync** (This is a third-party client for Syncthing, as Apple doesn't allow the official one. The free version has a file limit, but the Pro version is cheap and works well).

#### 3. Pair the Devices
1.  Open the app on the phone and the dashboard on the laptop.
2.  On the Laptop, click **"Add Remote Device"**. It will show a QR code.
3.  On the Phone, scan that QR code.
4.  Accept the connection on both devices.

#### 4. Configure the "Sync" (Crucial Step)
You don't want a "Two-Way Sync" (where deleting a photo on the laptop deletes it from the phone). You likely want a "One-Way Backup."

1.  **On the Phone:** Create a folder in Syncthing and point it to your `DCIM/Camera` folder.
2.  Set the **Folder Type** to **"Send Only"**.
3.  **On the Laptop:** Accept the folder. Point it to your 500GB drive (e.g., `D:\Mobile_Backup`).
4.  Set the **Folder Type** to **"Receive Only"**.
5.  **Important:** On the Laptop side, enable **"File Versioning"** (Trash Can File Versioning).
    *   *Why?* If you accidentally delete a photo on your phone, Syncthing might try to delete it on the laptop (even in receive mode, sometimes sync logic is tricky). Versioning ensures that if a file is "deleted" by the sync, it actually just moves to a hidden `.stversions` folder on your laptop, so you never lose data.

---

### Handling the "One Hour" Window

Since the laptop is only on for a short time, you need to ensure the sync actually happens during that window.

**1. Automate the Power (Optional)**
*   **Wake Up:** Enter your Laptop's BIOS (usually F2 or Del key at startup) and look for **"Wake on Alarm"** or **"RTC Alarm"**. You can set the laptop to automatically boot up at, say, 10:00 PM.
*   **Shut Down:** On Windows, you can use Task Scheduler to run `shutdown.exe /s /t 0` at 11:00 PM.

**2. Optimize the Mobile App**
*   **Android:** Go to Battery Settings and set Syncthing to **"Unrestricted"** or **"No Battery Optimization"**. Tell the app to "Run on Wi-Fi only."
*   **iOS:** Apple restricts background activity. You might need to open the Möbius Sync app and leave the screen on (plugged in) during that hour to ensure the transfer completes.

### Alternative Options

If Syncthing feels too technical:

1.  **Resilio Sync (formerly BitTorrent Sync):**
    *   Very similar to Syncthing but has a more polished, user-friendly interface.
    *   It is proprietary (not open source).
    *   The free version is great, but some advanced features require a paid license.

2.  **Photosync (Android/iOS) + SMB Share (Windows):**
    *   **Laptop:** Right-click a folder -> Properties -> Sharing -> Share. (This creates a Windows SMB share).
    *   **Mobile:** Install the **PhotoSync** app. Connect to "SMB / Windows Share."
    *   **Feature:** You can set a "trigger" so that the moment your phone connects to your home Wi-Fi (and is charging), it starts uploading to the laptop.

# Give the complete solution for photos sync and viewing using Immich

This is the best route. **Immich** effectively turns your unused Linux laptop into a private Google Photos clone. It handles the **backup** (syncing) and the **viewing** (timeline, map view, facial recognition) in a single interface.

Here is the complete, step-by-step guide to setting this up on your Linux laptop.

### Phase 1: Prepare the Laptop (One-time Setup)

Since your mobile needs to find the laptop every night, the laptop needs a fixed address on your Wi-Fi.

**1. Find your IP Address:**
Open the terminal on your Linux laptop and type:
```bash
ip addr show
```
Look for `inet 192.168.1.X` (or similar). Note this IP down.
*Tip: It is best to go into your Router settings and set a "Static IP Reservation" for this laptop so the IP never changes.*

**2. Install Docker:**
Immich runs inside a "container" (Docker), which makes it easy to install and update.
```bash
# Update your system
sudo apt update && sudo apt upgrade -y

# Install Docker and Docker Compose
sudo apt install docker.io docker-compose-v2 -y

# Allow your user to run Docker without "sudo" (Replace 'your_username' with your actual user)
sudo usermod -aG docker $USER

# RESTART your laptop now for this to take effect.
sudo reboot
```

---

### Phase 2: Install Immich

**1. Create a folder for Immich:**
After the reboot, open the terminal:
```bash
mkdir ~/immich-app
cd ~/immich-app
```

**2. Download the configuration files:**
```bash
wget https://github.com/immich-app/immich/releases/latest/download/docker-compose.yml
wget https://github.com/immich-app/immich/releases/latest/download/example.env -O .env
```

**3. Configure the Storage (Important):**
You need to edit the `.env` file to tell Immich where to store your photos on that 500GB drive.
```bash
nano .env
```
*   Look for the line `UPLOAD_LOCATION=./library`.
*   You can leave this as is (it will store photos inside `~/immich-app/library`).
*   **Or**, if you have a specific hard drive partition mounted elsewhere, change it to that path (e.g., `UPLOAD_LOCATION=/mnt/MyBigDrive/Photos`).

*   **Security:** In the same file, look for `DB_PASSWORD=postgres`. Change `postgres` to a secure password.

Press `Ctrl+O`, `Enter` to save, and `Ctrl+X` to exit.

**4. Start the Server:**
```bash
docker compose up -d
```
*It will take a few minutes to download the necessary files (Images, Database, AI models).*

---

### Phase 3: Connect the Mobile App

**1. Access the Dashboard:**
On your laptop browser, go to `http://localhost:2283`.
*   Click **"Getting Started"**.
*   Create your Admin Account (Email and Password).

**2. Install the Mobile App:**
*   **Android:** Download "Immich" from the Play Store.
*   **iOS:** Download "Immich" from the App Store.

**3. Login:**
*   **Server Endpoint:** `http://<your-laptop-ip>:2283` (e.g., `http://192.168.1.50:2283`).
*   **Email/Password:** Use the admin credentials you just created.

---

### Phase 4: Configure the "Sync" (For your 1-hour window)

Since your laptop is only on for one hour, we need to make sure the sync is aggressive when it connects.

**1. In the Mobile App:**
*   Tap the **Cloud Icon** (top right) or go to Settings > Backup.
*   **Select Albums:** Choose which folders to back up (Camera, WhatsApp, Instagram, etc.).
*   Tap **"Start Backup"**.

**2. Optimize for the "Nightly Window":**
*   **Android:** Go to Android Settings > Apps > Immich > Battery. Set it to **"Unrestricted"**. Enable **"Background Usage"**.
    *   *In Immich Settings:* Enable "Backup in background" and "Only on Wi-Fi".
*   **iOS:** Apple kills background processes strictly. For the best result, **leave the Immich app open** on your screen when you plug your phone in at night.

**3. How the Nightly Routine works:**
1.  **10:00 PM:** Laptop turns on (via BIOS alarm or you pressing the button). Immich starts automatically in the background.
2.  **10:05 PM:** You plug your phone into the charger.
3.  **The Sync:**
    *   **Android:** Should detect the server is online and start uploading silently.
    *   **iOS:** Open the app once to trigger the handshake.
4.  **11:00 PM:** Laptop shuts down.

---

### Phase 5: Viewing the Photos

You don't need to do anything extra here.
*   Open the Immich app on your phone.
*   The "Timeline" tab shows all your synced photos (fetched from the laptop database).
*   The "Search" tab lets you search by objects ("Car", "Tree") or Faces, powered by the AI running on your laptop.

### Maintenance (How to update)
Immich improves very fast. Once a month, when your laptop is on, run this command in the terminal to update it:

```bash
cd ~/immich-app
docker compose pull && docker compose up -d
```
