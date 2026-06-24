 Hi everyone!

Today i wanted to show to you how to fix the fingerptint scanner **ELAN 04f3:0c00.**
It won't work out of the box on Fedora 44. Here is the complete guide to compile the community driver, fix the dependencies, and solve the "dead sensor after suspend" bug.

⚠️**I'M NOT RESPONSABLE OF ANY DAMAGES TO YOUR SYSTEM**⚠️

**Step 1: Install Build Dependencies**

Open your terminal and install all required packages (including Cairo and OpenSSL which are missing from standard install):

```sudo dnf install git gcc glib2-devel libgusb-devel gobject-introspection-devel pixman-devel nss-devel libgudev-devel gtk-doc meson ninja-build cairo-devel openssl-devel```

**Step 2: Clone the Community Patched Driver**

We need the elanmoc2 branch from the community repository:

```cd ~```

```git clone -b elanmoc2 https://gitlab.freedesktop.org/Depau/libfprint.git```

```cd libfprint```

**Step 3: Compile and Install**

Run these commands to build the custom driver:

```meson setup builddir```

```cd builddir```

```ninja```

```sudo ninja install```

**Step 4: Configure Fedora Library Path**

Tell Fedora to look into /usr/local/lib for the new driver:

```echo -e "/usr/local/lib\n/usr/local/lib64" | sudo tee /etc/ld.so.conf.d/local.conf```

```sudo ldconfig```

**Step 5: Enable Biometrics & Register Fingers**

Restart the daemon and register your main finger:

```sudo systemctl restart fprintd.service```

```fprintd-enroll```

When you insert the command "fprintd-enroll" you must tap the fingerprint sensor to register your finger. You must do this right now, than if you want to register another finger you can do it by graphics intarface by going into settings>users>setup fingerprint or similar things (i'm italian so my pc is in italian language :) ) or by the terminal with the same command

Enable the fingerprint feature system-wide for login and sudo:

```sudo authselect enable-feature with-fingerprint```

```sudo authselect apply-changes```

**Step 6: Fix the "Not working after Sleep/Suspend" Bug**

If your sensor disappears or stops responding when you wake up your laptop, create this systemd service to automatically restart it:

```sudo nano /etc/systemd/system/restart-fprintd-after-suspend.service```

Paste this configuration inside, then save and exit:

[Unit]
Description=Restart fprintd after suspend
After=suspend.target

[Service]
Type=oneshot
ExecStart=/bin/systemctl restart fprintd.service

[Install]
WantedBy=suspend.target```

Enable it with:

```sudo systemctl enable --now restart-fprintd-after-suspend.service```

**Tip for KDE Plasma Users**

If you use KDE Plasma like me If the sensor doesn't recognize your finger on the first try, go to System Settings > Users > Configure Fingerprints and register your same physical finger under a different slot (e.g., register your right index finger again but label it as "right middle finger"). This maps multiple angles of your finger and drastically improves accuracy.

Unfortunately, on the lock screen, there is a guide that tells you to put your finger and you must do that before it disappears, **otherwise it won't work** and you'll need to enter your password.

If someone knows how to do it, I'll be grateful.


Unfortunately, on the lock screen, there is a guide that tells you to put your finger and you must do that before it disappears, otherwise it won't work and you'll need to enter your password.

If someone knows how to do it, I'll be grateful.

Have a nice day! :)

(I used some old guides, some old reddit posts on old versions of fedora and some AI. I rebuild the commands so if you see something similar is because i ridesigned it.) 
