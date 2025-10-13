# Download CRC from [CRC](https://console.redhat.com/openshift/create/local)
# Step 1: 
Latest two minor releases.
The host running CRC is registered with the Red Hat Customer Portal.
libvirt and NetworkManager packages are installed.
```bash
sudo dnf install libvirt NetworkManager
```
# Step 2:
Extract the contents of the archive:
```bash
tar xvf crc-linux-amd64.tar.xz
```
# Step 3:
Create the ~/bin directory if it does not exist and copy the crc executable to it:
```bash
mkdir -p ~/bin
cp ~/Downloads/crc-linux-*-amd64/crc ~/bin
```
# Step 4:
Add the ~/bin directory to your $PATH:
```bash
export PATH=$PATH:$HOME/bin
echo 'export PATH=$PATH:$HOME/bin' >> ~/.bashrc
```
