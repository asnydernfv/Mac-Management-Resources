# Zero Touch Deployment with jamf
## 1. Ensure device is in Automated Device Enrollment
Ensure that your device has been assigned to your mdm in from the Apple School Manager portal, or the Automated Device Enrollment section in jamf.
## 2. Assign device to prestage enrollment
Assign the device to a prestage enrollment. Make sure that the name makes sense, it will be needed later.
## 3. Create smart group
Create a smart group with the criteria: Enrollment Method: PreStage enrollment is [The name of the prestage enrollment]
## 4. Create enrollment script policy
Create an enrollment script to kick things off. This will be a script in a policy with the trigger set to enrollment complete and the frequency set to once per computer. The below example assumes you're using nomad login, and depnotify. I've included comments for relevent sections. The software I install in my first run script is specific to my enviroment and your needs may be different. I only install the necessities to keep provisioning quick. Everything else will get installed by munki when it gets installed.

```bash
#!/bin/bash
# NFV First Run DEP Script
# 11/1/19

# Install nomad, nomad login, and custom branding images
# Config profiles are already installed at this point from being included to the prestage enrollment and from being scoped to the device department
jamf policy -event nomad
# Restart the login window to get the nomad login screen
killall -HUP loginwindow
# Install DEPNotify. This usually finishes before the user account gets created by nomad login
jamf policy -event depnotify

# Wait for user to be logged in

dockStatus=$(pgrep -x Dock)
log "Waiting for Desktop"
while [ "$dockStatus" == "" ]; do
  log "Desktop is not loaded. Waiting."
  sleep 2
  dockStatus=$(pgrep -x Dock)
done

# Stare DEPnotify in fullscreen and read the jamf logs
/Applications/Utilities/DEPNotify.app/Contents/MacOS/DEPNotify -fullScreen -jamf &>/dev/null &

# set computer name
jamf policy -event setname

# Enable ARD and hide <500 users
jamf policy -event hide500

jamf policy -event ARD

# Set Preferences
# Add the user to the lpadmin group so they can add printers
jamf policy -event addprint
# Set the timezone. This can be done with a config profile now
jamf policy -event settz
# This runs a security script that enables/disables features based on CIS benchmarks
jamf policy -event security
# This installs dockutil for cleaning up the user dock
jamf policy -event dockutil

# Install Bit Bar
jamf policy -event bitbar
# Install munki
jamf policy -event munki
# Update inventory
jamf recon

sleep 10

# Remove DEPNotify Log
echo "Command: Restart: Your computer will now restart." >> /var/tmp/depnotify.log
```
## 5. Watch the magic
Turn on the device, connect to the internet, sit back and relax.