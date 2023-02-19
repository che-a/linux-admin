## Неразобранное

```sh
apt-get update && apt-get upgrade -y

apt-get install -y \
curl \
exa \
fonts-ubuntu \
htop \
lm-sensors \
mc \
remmina \
screenfetch \
tmux tmux-themepack-jimeh \
tree \
vlc
```

### Установка шрифтов ubuntu в debian
```sh
# В репо добавить non-free
# apt install fonts-ubuntu
```


```
ss -lntp
```


### Create MikroTik CHR on Proxmox
```sh


#!/usr/bin/env bash                                                                                                                                                                                                                           
                                                                                                                                                                                                                                              
#vars                                                                                                                                                                                                                                         
version="nil"                                                                                                                                                                                                                                 
vmID="nil"                                                                                                                                                                                                                                    
                                                                                                                                                                                                                                              

echo "############## Start of Script ##############
                                                
## Checking if temp dir is available..."            
if [ -d /root/temp ]                             
then                                              
    echo "-- Directory exists!"                   
else                                                          
    echo "-- Creating temp dir!"
    mkdir /root/temp                        
fi                                                                
# Ask user for version
echo "## Preparing for image download and VM creation!"    
read -p "Please input CHR version to deploy (6.38.2, 6.40.1, etc):" version
# Check if image is available and download if needed
if [ -f /root/temp/chr-$version.img ]               
then                                                 
    echo "-- CHR image is available."                 
else                                                
    echo "-- Downloading CHR $version image file."
    cd  /root/temp                                     
    echo "---------------------------------------------------------------------------"
    wget https://download.mikrotik.com/routeros/$version/chr-$version.img.zip
    unzip chr-$version.img.zip                
    echo "---------------------------------------------------------------------------"
fi                                              
# List already existing VM's and ask for vmID
echo "== Printing list of VM's on this hypervisor!"
qm list                                                  
echo ""                                                      
read -p "Please Enter free vm ID to use:" vmID
echo ""                                                        
# Create storage dir for VM if needed.                      
if [ -d /var/lib/vz/images/$vmID ]           
then
    echo "-- VM Directory exists! Ideally try another vm ID!"
    read -p "Please Enter free vm ID to use:" vmID
else                                            
    echo "-- Creating VM image dir!"                         
    mkdir /var/lib/vz/images/$vmID
fi                                          
# Creating qcow2 image for CHR.                                       
echo "-- Converting image to qcow2 format "                         
qemu-img convert \                                                  
    -f raw \
    -O qcow2 \                    
    /root/temp/chr-$version.img \
    /var/lib/vz/images/$vmID/vm-$vmID-disk-1.qcow2
# Creating VM                      
echo "-- Creating new CHR VM"
qm create $vmID \                          
  --name chr-$version \              
  --net0 virtio,bridge=vmbr0 \       
  --bootdisk virtio0 \               
  --ostype l26 \                                              
  --memory 256 \   
  --onboot no \
  --sockets 1 \
  --cores 1 \
  --virtio0 local:$vmID/vm-$vmID-disk-1.qcow2
echo "############## End of Script ##############"
```

### Winbox Install
```sh
#!/usr/bin/env bash

# dpkg --add-architecture i386 && apt update && apt install wine32

# mv winbox.exe netinstall.exe /bin

# Добавляем в ~/.bashrc

### MikroTik ###
#alias dude='wine "/home/user/.wine/dosdevices/c:/Program Files/Dude/dude.exe"'
#alias netinstall='wine "/home/user/.wine/dosdevices/c:/Program Files/MikroTik/netinstall.exe"'
#alias winbox='wine "/home/user/.wine/dosdevices/c:/Program Files/MikroTik/winbox.exe"'
```

#### Resolution_install
```
#!/usr/bin/env bash

SCRIPT_PATH='/usr/share/resolution_set.sh'
LIGHTDM_CONF_FILE='/etc/lightdm/lightdm.conf'

cp ./resolution_set.sh $SCRIPT_PATH
chmod +x $SCRIPT_PATH

cp $LIGHTDM_CONF_FILE $LIGHTDM_CONF_FILE.bak
sed -i "s|#display-setup-script=|display-setup-script=$SCRIPT_PATH|" $LIGHTDM_CONF_FILE
```
#### Resolution_set
```
#!/usr/bin/env bash

X1=1280     # Ширина экрана
Y1=1024     # Высота экрана
F1=60       # Частота обновления, Гц
M1=VGA-1    # 

X2=1280
Y2=1024
F2=75
M2=VGA-1

MODE1=`cvt $X1 $Y1 $F1 | grep Modeline | sed 's/Modeline //'`
MODE2=`cvt $X2 $Y2 $F2 | grep Modeline | sed 's/Modeline //'`
MODE1_NAME=`echo $MODE1 | awk '{print $1}'`
MODE2_NAME=`echo $MODE2 | awk '{print $1}'`

xrandr --newmode $MODE1
xrandr --newmode $MODE2

xrandr --addmode $M1 $MODE1_NAME
xrandr --addmode $M2 $MODE2_NAME

xrandr --output $M1 --mode $MODE1_NAME
# xrandr --output $M2 --mode $MODE2_NAME
```

### Запись RTSP-потока в файл
```
openRTSP    -4 \
            -D 1 \
            -B 10000000 -b 10000000 \
            -w 1920 -h 1080 \
            -f $FRAME_RATE \
            -Q \
            -d $RECORDING_TIME \
            -t \
            -u $LOGIN $PASSWORD \
            $RTSP_URL > $FILE_RECORD

```
