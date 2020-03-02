# fixing-docker-permissions
## How to fix docker: Got permission denied while trying to connect to the Docker daemon socket

After a fresh install of Ubuntu 18.04 on AWS Lightsail, I installed successfully **docker** and **docker-compose**.
```
$ cat /etc/os-release 
NAME="Ubuntu"
VERSION="18.04.1 LTS (Bionic Beaver)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 18.04.1 LTS"
VERSION_ID="18.04"
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
VERSION_CODENAME=bionic
UBUNTU_CODENAME=bionic
```
The fun started when I attempted to run **docker ps** and **login to my dockerhub account**.

I ran a **docker ps** and received the error message below:
```
$ docker ps
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.39/containers/json: dial unix /var/run/docker.sock: connect: permission denied
```
[Check permissions on docker.sock, my user does not have permission to read/write to docker.sock](https://www.digitalocean.com/community/questions/how-to-fix-docker-got-permission-denied-while-trying-to-connect-to-the-docker-daemon-socket)
```
$ cd /var/run
$ ls -l | grep docker
drwx------  5 root root    120 Feb 27 19:42 docker
-rw-r--r--  1 root root      4 Feb 27 19:42 docker.pid
srw-rw----  1 root docker    0 Feb 27 19:42 docker.sock
```
Modified permissions on **/var/run/docker.sock** to give world read/write permission
```
$ cd /var/run
$ sudo chmod 666 docker.sock 
$ ls -l | grep docker
drwx------  5 root root    120 Feb 27 19:42 docker
-rw-r--r--  1 root root      4 Feb 27 19:42 docker.pid
srw-rw-rw-  1 root docker    0 Feb 27 19:42 docker.sock
```
Ran **docker ps** and command executed as expected
```
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```
Next step is to login to **dockerhub with dockerhub credentials**, as you can see below it failed
```
$ docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: ow
Password: 

Error saving credentials: error storing credentials - err: exit status 1, out: `Cannot autolaunch D-Bus without X11 $DISPLAY`
```
[This solution below worked for me.](https://forums.docker.com/t/docker-login-fails-with-error-message-error-saving-credentials-cannot-autolaunch-d-bus-without-x11-display/77789)

**Install GNU Privacy Guard and Pass (Linux Password Manager)**
```
$ sudo apt-get -V install gnupg2 pass -y
```
Login to **dockerhub** is now successful
```
$ docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: ow
Password: 
WARNING! Your password will be stored unencrypted in /home/ubuntu/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```
