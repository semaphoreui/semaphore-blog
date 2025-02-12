With snap install method for SemaphoreUI going deprecated, I recently endeavored on the process to migrate to another hosting method. I chose to install package version of Semaphore on the same system. I had originally intended use the backup and restore capabilities on a fresh install but ran into multiple issues which all appear to be bugs that have been fixed but impact the latest version of Semaphore in the snap. I then went a different route that I'm suggesting here be added as a documented path for others to migrate.

**Steps to Migrate Semaphore Snap files to be be used with Package Manager or Binary Install:**
_NOTE: This method keeps boltdb as the database for SemaphoreUI_

1. Log in as `root`  via `sudo su -` or another valid method
2. Stop the snap version of SemaphoreUI
`snap stop semaphore.semaphored`
3. Copy files from snap over to new directory which will be the home directory a new system user called `semaphore`
`cp -rf /root/snap/semaphore/common /home/semaphore` 
4. Move the `reponsitories` folder to the new temp folder location
`mv -f /home/semaphore/repositories /tmp/semaphore`
5. Update `config.json` with new paths to `database.boltdb` and temp folder
`sed -i 's/\/root\/snap\/semaphore\/common\/database.boltdb/\/home\/semaphore\/database.boltdb/' /home/semaphore/config.json`
`sed -i 's/\/root\/snap\/semaphore\/common\/repositories/\/tmp\/semaphore/' /home/semaphore/config.json`
6. Create new system user and group called `semaphore` with `/home/semaphore` as its home folder
`adduser --system --group --home /home/semaphore --no-create-home --shell /bin/bash semaphore`
7. Transfer ownership of  `/home/semaphore` and all files to the new system uses
`chown -R semaphore:semaphore /tmp/semaphore`
`chmod o-rwx /home/semaphore`
`chown -R semaphore:semaphore /home/semaphore` 

With these steps complete, install and setup semaphore via package manager using `semaphore` as the service user or group. All settings and previous runs should show up in the new SemaphoreUI instance. Once satisfied, the snap instance can be removed via `sudo snap remove semaphore`

I ended up automating all these steps through this bash file:
```migrate-semaphore-snap.sh
#/bin/bash
#File: migrate-semaphore-snap.sh
#Migrate SemaphoreUI from snap as root to package/binary as semaphore
sudo su -l root <<EOF
set -x
snap stop semaphore.semaphored
cp -rf /root/snap/semaphore/common /home/semaphore
mv -f /home/semaphore/repositories /tmp/semaphore`
sed -i 's/\/root\/snap\/semaphore\/common\/database.boltdb/\/home\/semaphore\/database.boltdb/' /home/semaphore/config.json
sed -i 's/\/root\/snap\/semaphore\/common\/repositories/\/tmp\/semaphore/' /home/semaphore/config.json
adduser --system --group --home /home/semaphore --no-create-home --shell /bin/bash semaphore
cp /etc/skel/.* /home/semaphore/
chown -R semaphore:semaphore /tmp/semaphore
chmod o-rwx /home/semaphore
chown -R semaphore:semaphore /home/semaphore
EOF
```
