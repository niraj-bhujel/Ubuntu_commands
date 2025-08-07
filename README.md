## File Transfer
### SCP 
```
# From local host to dgx
scp -r /home/niraj/Desktop/multi-target-tracking/appearance_model/ stunb@10.218.13.21:/home/i2r/stunb/scratch/multi-target-tracking@dgx/
```
### Rsync
```
# Mirror directory tree (local to remote)
rsync -avHz -e ssh /source/mydirectory user@remotehost:/where/this/should/go

# Synchronize projects between machines
rsync -avHz -e ssh /home/ubuntu/Projects/DIP/src ubuntu@172.16.109.249:/home/ubuntu/Projects/DIP/
rsync -avHz -e ssh wtl81681@130.246.213.106:/home/pdq39273/dip/DIP/data/casp /home/ubuntu/Projects/DIP/data/

# Sync to other machines
rsync -avHz -e ssh /home/ubuntu/Projects ubuntu@172.16.114.151:/home/ubuntu
rsync -avHz -e ssh /home/ubuntu/Projects ubuntu@172.16.101.179:/home/ubuntu
```

## SSH Connections
```
# Deep server with port number
ssh -X niraj@10.217.131.115 -p 33

# ADA Machine A100
ssh ubuntu@172.16.100.181

# ADA Machine V100x2
ssh ubuntu@172.16.101.179

# STFC Cloud V100 Large
ssh ubuntu@172.16.114.151

# BioEM Server (Nicholas)
ssh wtl81681@130.246.213.106
```

## Mounting Remote Directories
### Mount using SSHFS
```
# Create mount directories
mkdir /home/niraj/Desktop/mnt
mkdir /home/niraj/Desktop/mnt/niraj_deep

# Mount remote directories
sshfs niraj@10.217.128.217:/home/niraj/Desktop/ /home/niraj/mnt/niraj_GPU/
sshfs dl-asoro@10.217.131.177:/home/dl-asoro/Desktop/multi-target-tracking@dl-asoro /home/niraj/mnt/dl_asoro
```

#### Advanced option for sshfs
```
# Mount with reconnection and keep-alive
sshfs -o reconnect,ServerAliveInterval=60,ServerAliveCountMax=10 dl-asoro@10.2.3.219:/home/dl-asoro/Desktop/Recon_GCN /mnt/dl-asoro/

# Mount with custom port
sshfs -o port=32900 niraj@10.2.56.220:/home/niraj/dsacstar /mnt/niraj/
```

#### Troubleshooting sshfs
```
# Fix permissions
sudo chmod 777 /mnt/dl-asoro

# Fix transport endpoint issues
fusermount -uz /mnt/dl-asoro
killall sshfs
```

### Windows Network Mounting using net use
```
# Mount Linux to Windows
net use Z: \\sshfs\dl-asoro@10.2.3.219\Desktop\Recon_GCN /persistent:yes

# Mount using SSH keys
net use Z: \\sshfs.k\ubuntu@host-172-16-101-186.nubes.stfc.ac.uk
net use Z: \\sshfs.k\ubuntu@172.16.101.179
```

### CIFS/Samba Mounting
```
# Mount Samba share
sudo mount -t cifs -o "domain=ares,username=bhujeln1,password=B********5,vers=3.0" //access.serc.acrces2.shared-svc.local/i2r/home/bhujeln1 /mnt/local_share/
```

**Note**: For SMB mounting, if client cannot resolve URL, add `nameserver 10.10.49.253` to `/etc/resolv.conf`

### Bind Mounting
```
# Mount one directory to another
sudo mount --bind /src /dst

# Unmount directory
sudo umount -l /mnt/dir
```

### S3FS Mounting (for S3 Storage )
```
# Configure credentials
echo ACCESS_KEY_ID:SECRET_ACCESS_KEY > ${HOME}/.passwd-s3fs
chmod 600 ${HOME}/.passwd-s3fs

# Mount S3 bucket
s3fs sciml-ccpem-dpi-score /mnt/sciml-ccpem-dpi-score/ -o passwd_file=${HOME}/.passwd-s3fs
```

### Fix permissions
```
# Check owner
ls -ld /mnt/<mountpoint>

# Change owner if needed
sudo chown $USER:$USER /mnt/sciml-ccpem-dpi-score

# Unmount bucket
umount /mountpoint
```

### Rclone S3 Mounting

Create a configuration file `/home/ubuntu/rclone.txt` and add the following
```ini
[rclone]
type = s3
provider = Ceph
env_auth = false
access_key_id = 72EKHEXCF2A71KP2ZPM3
secret_access_key = 6bLJpEQgG2DOTFy6VDrCUSlgoDeUlpDMmATDCroI
endpoint = https://s3.echo.stfc.ac.uk
region =
```
Mount command
```
# Mount with daemon mode
rclone mount rclone:sciml-ccpem-dpi-score/dpi/data /mnt/sciml-ccpem-dpi-score --config="/home/ubuntu/rclone.txt" --daemon --allow-other -vv

# Add --read_only for read-only access
```

## Docker Management and Container operations
```
# View all docker containers
docker ps -a

# View running containers
docker ps

# Start a container
docker start inspiring_wright

# Execute container with 
docker exec -it inspiring_wright 
```

## Port management
```
# Free up TCP port
fuser -k 6006/tcp

# Check if port is in use
sudo netstat -tulpn | grep :7007
```

## PBS Job Scheduling

#### Create job script
Create a .sh file  test.sh file and add the following 
```
#!/bin/sh
#PBS -l select=1:ncpus=5:ngpus=2:host=dgx04
#PBS -N test
#PBS -l software=nvidia-docker-images
#PBS -l walltime=24:00:00
```

#### Submit jobs
```
# Submit interactive job with script
qsub -I test.sh

# Direct interactive job submission
qsub -I -l select=1:ncpus=1:ngpus=1:host=dgx05 -N bhujeln1 -l software=nvidia-docker-images -l walltime=12:00:00

# Interactive job with memory specification
qsub -I -l select=1:ncpus=1:ngpus=1:mem=128g -l walltime=01:00:00 -P personal-bhujeln1 -q normal
```

## S3CMD Operations
```
# Configure s3cmd
s3cmd --configure

# Create bucket
s3cmd mb s3://datanb

# List buckets
s3cmd ls s3://<bucket name>

# Get bucket info
s3cmd info s3://sciml-epac-incoming

# Copy files to S3 bucket
s3cmd put -rp /home/ubuntu/Projects/DPI/dpi/data s3://sciml-ccpem-dpi-score/dpi/ --debug

# Download from S3
s3cmd get --recursive s3://sciml-epac-incoming/ ./
s3cmd get --recursive s3://sciml-epac-incoming/FS_LIDT_April22/ ./FS_LIDT_April22
```

### Kill Python multiprocessing
```
kill -9 `ps -ef | grep train.py | grep -v grep | awk '{print $2}'`
```

### System Services
```
# Start/stop lightdm
sudo service lightdm start
```

## Python Environment Management

### Conda Operations
```
# Download Conda
curl -O https://repo.anaconda.com/archive/Anaconda3-2024.06-1-Linux-x86_64.sh
 Anaconda3-2024.06-1-Linux-x86_64.sh

# Create environment
conda create -n myenv python=3.8

# Remove environment
conda env remove -n ENV_NAME

# Disable auto-activation
conda config --set auto_activate_base false

# Isolate .local from conda env
conda env config vars set PYTHONNOUSERSITE=1 -n my_environment
```

### Jupyter Integration
```
# Add conda environment to Jupyter
conda activate menv
pip install ipykernel
python -m ipykernel install --user --name=myenv
```

### Package Management
```
# Create requirements.txt
pip install pipreqs
pipreqs /path/to/project

# Fix Spyder matplotlib issues
pip install opencv-python==4.3.0.36
# or
pip uninstall opencv-python
pip install opencv-python-headless
```

## Jupyter Notebook Remote Access

### Server setup
```
# Run Jupyter remotely
jupyter notebook --no-browser --port=50100

# Run in Docker
jupyter notebook --ip 0.0.0.0 --port 8888 --allow-root
```

### Local port forwarding
```
# Setup port forwarding
ssh -L 50100:localhost:50100 niraj@10.2.56.220 -p 32900

# Access via browser
http://127.0.0.1:50100/?token=565188c6c78cb04d3eb40e9c0a8fcb42926053d54a4d7830
```

## NVIDIA Driver Management

### Download specific drivers
```
version=550.144.03
distro=ubuntu2404
arch=amd64
wget https://us.download.nvidia.com/tesla/${version}/nvidia-driver-local-repo-${distro}-${version}_1.0-1_${arch}.deb

# Network installation
wget https://developer.download.nvidia.com/compute/cuda/repos/$distro/$arch/cuda-keyring_1.1-1_all.deb
```

### Verify installation
```
# Check driver version
cat /proc/driver/nvidia/version
```

### Complete removal
```
sudo apt remove --autoremove --purge -V nvidia-driver\* libxnvctrl\* -y
sudo apt remove --purge '^nvidia-.*' -y
sudo apt-get remove --purge '^libnvidia-.*' -y
sudo apt-get remove --purge '^cuda-.*' -y
sudo apt autoremove -y
sudo apt autoclean
sudo reboot

# Restore to nouveau
sudo apt install ubuntu-desktop
sudo rm /etc/X11/xorg.conf
echo 'nouveau' | sudo tee -a /etc/modules
```

### Environment setup
```
# Export CUDA paths
export PATH=/usr/local/cuda-12.8/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda-12.8/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
```

### Library issues
```
# Fix GUI app loading issues
sudo ldconfig /lib/x86_64-linux-gnu/
export LD_LIBRARY_PATH=/home/niraj/miniconda3/lib/

# Find specific libraries
find / -name "libnvrtc.so*" 2>/dev/null

# Fix CuPy library issues
sudo ln -s /home/ubuntu/miniconda3/lib/libnvrtc.so.11.2 /usr/lib/libnvrtc.so.11.2
sudo ln -s /home/ubuntu/miniconda3/lib/libnvrtc-builtins.so.11.8 /usr/lib/libnvrtc-builtins.so.11.8
```

## System Utilities

### Disk usage analysis
```
# List folder sizes and sort
du -chd 1 | sort -h
du -shc ./datasets/* | sort -h
du -sh * | sort -n

# Count files in directory
ls -1 /usr/bin/ | wc -l
```

### Session management
```
# Create tmux session
tmux new -s niraj
```

## TensorBoard
```
# Start TensorBoard on specific port
tensorboard --logdir logs/cocostuff27/ --bind_all --port 50100
```

## Manila Share Management

### Prerequisites
```
# Install OpenStack client
sudo apt update && sudo apt upgrade
sudo apt install python3-openstackclient

# Install Manila client
pip install python-manilaclient==2.0.0

# Install Ceph
sudo apt install ceph-common

# Source authentication (download from OpenStack dashboard)
source SciML-New-openrc.sh
```

### Manila operations
```
# Add access rule
manila access-allow <share_name> cephx <access_name>

# Check access rules
manila access-rules <share_name>

# View share details
manila show <share_name>

# View messages/errors
manila message-list
```

### Mount Manila share
```
# Create mount point
sudo mkdir /mnt/manila

# Mount command (example)
sudo mount -t ceph 130.246.208.244:6789,130.246.209.22:6789,130.246.209.142:6789:/volumes/_nogroup/cfcbe6c3-1330-4470-9492-553f810ee83f/127a8bea-bdfb-40f3-a1b5-cfe68baab2fa -o name=dpi,secret=AQDqlEFok6tlFRAAmfRTXwo1KmJ5fWbmB8y5xA== /mnt/manila
```

### Persistent configuration

#### `/etc/ceph/ceph.conf`
```ini
[client]
    client quota = True
    mon host = MON_HOST_IP_ADDRESSES
```

#### `/etc/ceph/ceph.keyring`
```ini
[client.<access_name>]
    key = CEPHX_KEY
```

## FFmpeg Video Processing

### Create video from images
```
# Create video from sequentially numbered images
ffmpeg -framerate 25 -i image-%05d.jpg -c:v libx264 -profile:v high -crf 20 -pix_fmt yuv420p output.mp4

# Convert images to video with custom settings
ffmpeg -framerate 15 -start_number 1030 -i %06d.jpg -c:v libx264 -b:v 2M output.mp4
```

### Search text in PDF files
```
# Search for specific word in multiple PDF files
pdfgrep 'Gaussian' *.pdf
```

## Shell Configuration

### Add alias to 
```
echo "alias python='python3.6'">> ~/.rc && source ~/.rc
```
