# I have an unused laptop at home which has 500 GB of storage. I would like to store photos in that and I am ok with syncing it every night with my mobile. But I would prefer the sync to happen automatically. I can keep the laptop switched on for one hour everyday in the night. What software do I need?

Since you are switching from a Cloud/Serverless approach to a Local Network approach, you no longer need AWS. You need a **Peer-to-Peer (P2P) synchronization tool**.

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
