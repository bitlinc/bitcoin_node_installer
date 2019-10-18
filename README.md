#!/usr/bin/env bash

    ## This script is the first step for running your Bitcoin / LND / Electrum node
    ########################################################################################################################################
        ## It will configure the following - for both mainnet and testnet
        ## Change root password - create  user 'bitcoin' for daemon services to run under 
        ## Set the visible name your Pi on the network - Set boot to GUI - Expand Filesystem - Set Memory Split to 16
        ## Update system and install necessary software packages [apt-get install htop git curl bash-completion jq dphys-swapfile dirmngr]
        ## Provision external drive - format with ext4 - edit fstab for mounting the drive to the file system - mount the drive and set owner 
        ## Create Swap File on external drive to prolong life of the SD card 
        ## Configure Uncomplicated Firewall for Bitcoin - LND - Electrum Personal Server - Electrum Wallet for both mainnet and testnet
        ## Insall fail2ban 
        ## Increate open files limit
    ########################################################################################################################################

## User Functions

# Hides the output of shell command 
        function suppress () { 
            /bin/rm --force /tmp/suppress.out 2> /dev/null; ${1+"$@"} > /tmp/suppress.out 2>&1 || cat /tmp/suppress.out; /bin/rm /tmp/suppress.out
        }

    # Script to run 'su root -c' command in bash function script
        function su_Root {
            local firstArg=$1
            if [ $(type -t $firstArg) = function ]
            then
                    shift && command sudo bash -c "$(declare -f $firstArg);$firstArg $*"
            else
                    command su root -c "$@"
            fi
        } 

    # Script to run 'sudo' command in bash function script
        function sudo_Root {
            local firstArg=$1
            if [ $(type -t $firstArg) = function ]
            then
                    shift && command sudo bash -c "$(declare -f $firstArg);$firstArg $*"
            else
                    command sudo "$@"
            fi
        }


# This script will download and install only Bitcoin-QT
## The purpose of this script is NOT To create a bitcoin wallet address but to download and install bitcoin just to SYNC the blockchain 
########################################################################################################################################
########################################################################################################################################  

 function download_Bitcoind_Linux {
        cd /home/"$USERNAME"/Downloads
        wget https://bitcoincore.org/bin/bitcoin-core-0.18.1/bitcoin-0.18.1-x86_64-linux-gnu.tar.gz
        wget https://bitcoincore.org/bin/bitcoin-core-0.18.1/SHA256SUMS.asc
        wget https://bitcoin.org/laanwj-releases.asc
            echo ""
            echo ""
            echo ""
            echo "Bitcoin downloaded successfully - you will have to verify signatures to install"
            echo "*******************************************************************************"
            echo ""
            echo "Verifing the checksum of bitcoin-core-0.18.1/bitcoin-0.18.1-x86_64-linux-gnu.tar.gz"
            echo ""
            sleep 3.0
            echo ""
            echo "Ensure the output lists "OK" after bitcoin-0.18.1..."
            echo "--------------- Right below this line here |"
            echo "----------------------------------------------------"
            sha256sum --ignore-missing --check SHA256SUMS.asc
            echo ""
            echo "----------------------------------------------------"
            echo "You can safely ignore any warnings and failures"
            echo ""
            read -p "Press any key to continue as long as the output says "OK" above - or ctrl + z to exit if not"
            echo ""
            echo ""
            echo ""
            echo "Importing the public key of Wladimir J. van der Laan and verify the signed checksum file..."
            echo ""
            sleep 3.0
            echo ""
            gpg --import ./laanwj-releases.asc
            echo ""
            gpg --refresh-keys
            echo ""
            echo ""
            echo "Verifing that the checksums file is PGP signed by the release signing key of Wladimir J. van der Laan"
            echo ""
            sleep 3.0
            gpg --verify SHA256SUMS.asc
            echo ""
            echo "----------------------------------------------------------------------------"
            echo "Primary key fingerprint: 01EA 5486 DE18 A882 D4C2  6845 90C8 019E 36C2 E964 <------ The fingerprint directly above MUST match this one!" 
            echo ""
            echo ""
            echo "Press any key to proceed ONLY if the fingerprint matches and gpg says 'Good signature from Wladimir J. van der Laan'"
            read -p "Type 'ctrl + z' to exit if not"
            echo ""
            echo ""
            echo "We have successfully downloaded Bitcoin Core for Linux and verified the integrity of the the download - next we will install Bitcoin Core"
            echo ""
            echo ""
            sleep 3.0
            echo "--------------------------------"
            echo "Extracting Bitcoin Core binaries"
            echo "--------------------------------"
            echo ""
            echo ""
            sleep 2.0
            cd /home/"$USERNAME"/Downloads
            tar -xvf bitcoin-0.18.1-x86_64-linux-gnu.tar.gz 
            sudo install -m 0755 -o root -g root -t /usr/local/bin bitcoin-0.18.1/bin/*
            echo ""
            echo "    Verify version below as v0.18.1"
            sleep 3.0
            echo ""
            bitcoind --version
            echo ""
            sleep 2.0
            echo "First we will SYNC Bitcoin for testnet - we do this by simply just running the bitcoin core graphical wallet"
            echo ""
            sleep 3.0
            echo ""
            echo "Allow bitcoin core wallet to run 24/7 until the wallet says '100% synced' - in the bottom left hand corner"
            echo ""
            sleep 2.0
            echo "Once bitcoin core wallet is fully synced you must close the application so that bitcoin core wallet for mainnet can sync too"
            echo ""
            sleep 2.0
            echo "Mainnet is a different blockchain then testnet and is ten times the size so this download can take a bit of time"
            echo ""
            sleep 2.0
            echo ""
            echo ""
            sleep 6.0
    }

function download_Bitcoind_macOS {
        cd ~/Downloads
        curl -O https://bitcoincore.org/bin/bitcoin-core-0.18.1/bitcoin-0.18.1-osx.dmg
        curl -O https://bitcoincore.org/bin/bitcoin-core-0.18.1/SHA256SUMS.asc
        curl -O https://bitcoin.org/laanwj-releases.asc
            echo ""
            echo ""
            echo ""
            echo "Bitcoin downloaded successfully - you will have to verify signatures to install"
            echo "*******************************************************************************"
            echo ""
            echo ""
            echo "Installing Homebrew and GPG to verify the integrity of the files we just downnloaded"
            echo ""
            sleep 3.0
            ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
            brew install gnupg
            echo "Verifing the checksum of bitcoin-core-0.18.1/bitcoin-0.18.1-osx.dmg"
            echo ""
            sleep 3.0
            echo ""
            echo "Ensure the output lists "OK" after bitcoin-0.18.1-osx.dmg"
            echo "--------------- Right below this line here |"
            echo "----------------------------------------------------"
            shasum -a 256 --check ~/Downloads.SHA256SUMS.asc
            echo ""
            echo "----------------------------------------------------"
            echo "You can safely ignore any warnings and failures"
            echo ""
            read -p "Press any key to continue as long as the output says "OK" above - or ctrl + z to exit if not"
            echo ""
            echo ""
            echo ""
            echo "Importing the public key of Wladimir J. van der Laan and verify the signed checksum file..."
            echo ""
            sleep 3.0
            echo ""
            gpg --import ./laanwj-releases.asc
            echo ""
            gpg --refresh-keys
            echo ""
            echo ""
            echo "Verifing that the checksums file is PGP signed by the release signing key of Wladimir J. van der Laan"
            echo ""
            sleep 3.0
            gpg --verify SHA256SUMS.asc
            echo ""
            echo "----------------------------------------------------------------------------"
            echo "Primary key fingerprint: 01EA 5486 DE18 A882 D4C2  6845 90C8 019E 36C2 E964 <------ The fingerprint directly above MUST match this one!" 
            echo ""
            echo ""
            echo "Press any key to proceed ONLY if the fingerprint matches and gpg says 'Good signature from Wladimir J. van der Laan'"
            read -p "Type 'ctrl + z' to exit if not"
            echo ""
            echo ""
            echo "We have successfully downloaded Bitcoin Core for macOS and verified the integrity of the the download - next we will install Bitcoin Core"
            echo ""
            echo ""
            sleep 3.0
            echo "---------------------"
            echo "Installing Bitcoin-Qt"
            echo "---------------------"
            echo ""
            echo ""
            sleep 2.0
            cd ~/Downloads
            sudo hdiutil attach bitcoin-0.18.1-osx.dmg
            cp -a /Volumes/Bitcoin-Core/Bitcoin-Qt.app /Applications/Bitcoin-Qt.app
            sudo diskutil umount /Volumes/Bitcoin-Core
            sleep 2.0
            echo "First we will SYNC Bitcoin for testnet - we do this by simply just running the bitcoin core graphical wallet"
            echo ""
            sleep 3.0
            echo ""
            echo "Allow bitcoin core wallet to run 24/7 until the wallet says '100% synced' - in the bottom left hand corner"
            echo ""
            sleep 2.0
            echo "Once bitcoin core wallet is fully synced you must close the application so that bitcoin core wallet for mainnet can sync too"
            echo ""
            sleep 2.0
            echo "Mainnet is a different blockchain then testnet and is ten times the size so this download can take a bit of time"
            echo ""
            sleep 2.0
            echo ""
            echo ""
            sleep 6.0 
    }

# Creates Mount Point for External Drive 
    function make_Mount_Point {
        sudo mount /dev/"$LINUXDRIVE" ~/btc_drive
    }

# Creates Mount Point for External Drive Parent function 
    function make_Mount_Point_Parent {
        make_Mount_Point
    }

# Creating directory to add the hard disk and set the correct owner - verify the drive is mounted at /home/hdd and set owner as the user 'bitcoin' 
    function mount_Filesystem {
        mkdir /home/"$USERNAME"/btc_drive/hdd  
    }

# Create directory 'bitcoin' on external drive and set the ownership to user 'bitcoin' 
    function make_Bitcoin_Directory {
        mkdir /home/"$USERNAME"/btc_drive/hdd/bitcoin
    } 

# Verify Filesystem is mounted at /home/hdd
    function verify_Mount_AND_Set_Ownership {
        echo ""
        echo "Verify 'Filesystem' on the far left is mounted at '/home/hdd' on the the far right"
        echo "___________________________________________________________________________________"
        df /home/"$USERNAME"/btc_drive
        echo ""
        read -p "Press any key to proceed or 'ctrl + z to exit'..."
    }                  

# Unmount external drive 
    function un_Mount {
        sudo umount -a 
    } 

#  Unmount external drive 
    function un_Mount_Suppress {
        suppress un_Mount 
    } 


# Install Bitcoind and provision drive for a Linux OS
    function install_Bitcoin_Linux_OS {
        echo ""
        echo "Confirmed Linux OS"
        echo "Confirmed No Bitcoin directory found on external drive"
        sleep 3.0
        echo ""
        echo ""
        echo "Provisioning your external drive for Bitcoin and downloading Bitcoin Core Wallet so you can sync the full blockchain"
        echo ""
        sleep 2.0
        read -r -p "Enter 'yes' to confirm formatting of your external drive /dev/"$LINUXDRIVE" or ctrl + z to exit `echo $'\n> '`" CONFIRMFORMAT
                    if [[ "$CONFIRMFORMAT" = 'yes' || "$CONFIRMFORMAT" = 'Yes' ]]; then
                        echo ""
                        echo "Formatting your drive"
                        echo ""
                        un_Mount_Suppress
                        echo ""
                        sleep 2.0 
                        echo ""
                        echo ""
                        sudo mkfs.exfat /dev/"$LINUXDRIVE"
                        echo ""
                        echo ""
                        echo "Your external drive was created successfully"
                        echo ""
                        sleep 2.0
                    else
                        echo ""
                        echo "Exiting formatting your drive and exiting install"
                        exit
                        echo ""
                        sleep 2.0
                    fi   
        mkdir /home/"$USERNAME"/btc_drive    
        sudo mount /dev/"$LINUXDRIVE" /home/"$USERNAME"/btc_drive
        echo ""
        echo "--------------------------------------------------------------------------------------"
        echo "Please enter your password to mount your external drive to /home/"$USERNAME"/btc_drive"
        echo " This is the location of where the Bitcoins blockchain will be downloaded to"
        echo ""
        sleep 3.5
        su_Root "chown -R "$USERNAME":"$USERNAME" /home/"$USERNAME"/btc_drive"         
        mkdir /home/"$USERNAME"/btc_drive/hdd  
        mkdir /home/"$USERNAME"/btc_drive/hdd/bitcoin
        download_Bitcoind_Linux
        bitcoin-qt -datadir=/home/"$USERNAME"/btc_drive/hdd/bitcoin
        bitcoin-qt -testnet -datadir=/home/"$USERNAME"/btc_drive/hdd/bitcoin
        sudo umount -a 
        echo ""
        echo ""
        echo ""
        echo "**************************************************************************"
        echo "Bitcoin's blockchain has been downloaded completly for testnet and mainnet"
        echo "**************************************************************************"
        echo ""
        echo ""
        exit
       
        #sudo reboot                
    }

# Install Bitcoind and provision drive for a  macOS
    function install_Bitcoin_macOS {
        echo ""
        echo "Confirmed macOS"
        echo "Confirmed No Bitcoin directory found on external drive"
        sleep 3.0
        echo ""
        echo ""
        sleep 2.0 
        diskutil list
        echo ""
        read -r -p "Please enter the disk "Identifier" located on the far right that itentifes your external drive - this drive WILL be formatted for Bitcoin `echo $'\n> '`" MACOSDRIVE
        echo ""
        echo ""
        sleep 2.0
        echo ""
        echo "You'll need to enter your password to temporarily mount external drive and download the blockchain"
        echo ""
        sleep 2.0
        echo "Provisioning your external drive for Bitcoin and downloading Bitcoin Core Wallet so you can sync the full blockchain"
        echo ""
        sleep 2.0
        read -r -p "Enter 'yes' to confirm formatting of your external drive /dev/"$MACOSDRIVE" or ctrl + z to exit `echo $'\n> '`" CONFIRMFORMAT
                    if [[ "$CONFIRMFORMAT" = 'yes' || "$CONFIRMFORMAT" = 'Yes' ]]; then
                        echo ""
                        echo "Formatting your drive"
                        echo ""
                        sudo diskutil umount /dev/"$MACOSDRIVE"
                        echo ""
                        echo "Confirm format one more time by entering "y" - make sure you have only ONE external drive connected to your computer"
                        echo ""
                        diskutil eraseDisk ExFAT btc_drive /dev/"$MACOSDRIVE"
                        echo ""
                        echo ""
                        echo "Your external drive was created successfully"
                        echo ""
                        sleep 2.0
                    else
                        echo ""
                        echo "Exiting formatting your drive and exiting install"
                        exit
                        echo ""
                        sleep 2.0
                    fi   
        cd /Volumes/btc_drive
        mkdir /Volumes/btc_drive/hdd
        mkdir /Volumes/btc_drive/hdd/bitcoin             
        echo ""
        echo "----------------------------------------------------------------------------------------------------"
        echo "Mounting Bitcoin-Qt application and setting the blockchain download directory to your external drive"
        echo "----------------------------------------------------------------------------------------------------"
        echo ""
        sleep 3.5
        download_Bitcoind_macOS
        #open -a Bitcoin-Qt.app --args -datadir=/Volumes/btc_drive/hdd/bitcoin -testnet -zmqpubrawblock=tcp://127.0.0.1:28332 -zmqpubrawtx=tcp://127.0.0.1:28333
        #open -a Bitcoin-Qt.app --args -datadir=/Volumes/btc_drive/hdd/bitcoin -zmqpubrawblock=tcp://127.0.0.1:29000 -zmqpubrawtx=tcp://127.0.0.1:29001
        /Applications/Bitcoin-Qt.app/Contents/MacOS/Bitcoin-Qt -datadir=/Volumes/btc_drive/hdd/bitcoin -testnet 
        /Applications/Bitcoin-Qt.app/Contents/MacOS/Bitcoin-Qt -datadir=/Volumes/btc_drive/hdd/bitcoin
        sudo diskutil umount /Volumes/btc_drive 
        echo ""
        echo ""
        echo ""
        echo "*****************************************************************************"
        echo "Bitcoin's blockchain has been downloaded successfully for testnet and mainnet"
        echo "*****************************************************************************"
        echo ""
        echo ""
        exit       
    }

    ######################################################################################################################
    # Script Functions  
        function setup_Bash { 
            mkdir /home/pi/download && cd /home/pi/download
            rm -rf /home/admin/.bashrc
            wget https://raw.githubusercontent.com/bitlinc/custom_command_line/master/README.md
            cp -avr /home/pi/download/README.md /home/pi/.bashrc
            source /home/pi/.bashrc && source /home/pi/.bash_aliases
        }

    # Setp 1: Sets user 'root' and 'pi' and creates 'bitcoin user and password'
        function create_Passwords_Users {
            echo "Set your root user password to password [A] as explained in the directions"
                sudo passwd root
                sleep 1.0
                echo ""
            echo "Set your user 'pi' password to password [A] also"
                sudo passwd pi
                echo "" 
            echo ""
                sleep 3.0    
            echo ""
            echo "Users 'root' and 'pi' were both configured successfully "
        }  

    # Step 2: Creates a text doc on desktop with the hostname provided by the user - GLOBAL VARABLE -> USERGIVENHOSTNAME    
        function create_Hostname () {
            cd /home/pi/Desktop
            touch hostname
            echo "$USERGIVENHOSTNAME" >> /home/pi/Desktop/hostname
            sudo rm -rf /etc/hostname
            sudo mv /home/pi/Desktop/hostname /etc/hostname
            source /home/pi/.bashrc
            source ~/.profile        
        }  

    # Step 2: Creates a text doc on desktop with the hostname provided by the user - GLOBAL VARABLE -> USERGIVENHOSTNAME    
        function create_Hosts () {
            sudo rm -rf /etc/hosts
            cd ~/Desktop
            sudo wget https://raw.githubusercontent.com/bitlinc/pi_hosts/master/README.md
            sudo mv /home/pi/Desktop/README.md /etc/hosts
            source /home/pi/.bashrc
            source ~/.profile     
        }  

    # Step 2: Removes default /etc/hostname file and replaces it with one creaded above - edits the /etc/hosts file - sets hostname - reboots bash / .profile / avahi-deamon      
        function change_Hosts {
            sudo sed -i -e "s/raspberrypi/$USERGIVENHOSTNAME/" /etc/hosts
            sudo hostnamectl set-hostname "$USERGIVENHOSTNAME"
            sudo systemctl restart avahi-daemon
        }     
    # Step 2: Removes default /etc/hostname Parent Function      
        function change_Hostname_Hosts_Parent {
            create_Hostname
            create_Hosts
            change_Hosts  
        }          

    #  Step 3: Install the necessary software packages: [apt-get install htop git curl bash-completion jq dphys-swapfile dirmngr] then apt-get update / upgrade   
        function install_Packges {
            apt-get install htop git curl bash-completion jq dphys-swapfile dirmngr --install-recommends -y    
            apt-get update -y
            apt-get upgrade -y
            apt autoremove -y
            sleep 2.0
        }   

    # Step 3: Install the necessary software packages: START
        function install_Packages_Start {
            echo ""
            echo "Download necessary software packages"
            echo ""
            echo "This will take about ~10 minutes to complete depending on your internet connection"
            echo ""
            sleep 1.25
            echo "Installing packages ..."
            echo ""
        }
        
    # Step 3: Install the necessary software packages: SUCCESSFULLY COMPLETD
        function install_Packages_Successfull {
            echo ""
            echo "Packages installed and updated the system successfully"
            sleep 2.0
        }

    # Step 3: Install software packages encapsuling function 
        function install_Packages_Parent {
            sudo_Root install_Packges
        }  

    # Step 4: Provision external drive - format with ext4 - edit fstab for symbolic link pointing to external drive at /media/hdd  
        function make_Drive {
            UUIDEXTERNALDRIVE="$(sudo blkid -s UUID -o value /dev/sda)"
            echo UUID="$UUIDEXTERNALDRIVE" /media/hdd ext4 rw,nosuid,dev,noexec,noatime,nodiratime,auto,nouser,async,nofail 0 2 >> /etc/fstab
        }

    # Step 4: Provision external drive Parent
        function make_Drive_Parent {
            su_Root "make_Drive"
        }

    # Step 5: Creating directory to add the hard disk and set the correct owner - verify the drive is mounted at /media/hdd and set owner as the user 'bitcoin' 
        function mount_Filesystem {
            echo ""
            mkdir /media/hdd
            sudo chown -R pi:pi /media/hdd
            sudo mount -a   
        }

    # Step 5: Creating directory to add the hard disk.. Parent
        function mount_Filesystem_Parent {
            su_Root "mount_Filesystem"
        }

    # Step 5: Verify Filesystem is mounted at /media/hdd
        function verify_Mount_AND_Set_Ownership {
            echo ""
            echo "Verify 'Filesystem' on the far left is mounted at '/media/hdd' on the the far right"
            echo "___________________________________________________________________________________"
            df /media/hdd
            echo ""
            read -p "Press any key to proceed or 'ctrl + z to exit'..."
        }  

    # Step 5: Create directory 'bitcoin' on external drive and set the ownership to user 'bitcoin' 
        function make_Bitcoin_Directory {
            mkdir /media/hdd/bitcoin
        }                 

    # Step 6: Moving system swap file from SD card to external drive
        function move_Swap_File_Start {
            echo ""
            echo "Moving system Swap File to external drive, this will preserve SD card life"
            echo "This will take about ~5 minutes to complete ..."
        }  

    # Step 6: Moving system swap file from SD card to external drive
        function move_Swap_File { 
            sudo dphys-swapfile swapoff
            sudo dphys-swapfile uninstall
            sudo sed -i "s/#CONF_SWAPFILE=\/var\/swap/CONF_SWAPFILE=\/mnt\/swapfile/g" /etc/dphys-swapfile
            sudo sed -i "s/CONF_SWAPSIZE=100/#CONF_SWAPSIZE=/g" /etc/dphys-swapfile
            sudo dd if=/dev/zero of=/media/swapfile count=1000 bs=1MiB
            sudo chmod 600 /media/swapfile
            sudo mkswap /media/swapfile
            sudo /sbin/dphys-swapfile setup
            sudo /sbin/dphys-swapfile swapon
        }
    
    # Step 6: Moving system swap file from SD card to external drive Encapsulated
        function move_Swap_File_Parent {
            su_Root "move_Swap_File"
        }

    # Step 7: Start the install of Uncomplicated Firewall - configure for Bitcoin, Lightning, and Electrum, - install fail2ban
        function ufw_Install_Start () {
            echo "IMPORTANT - You must forward ports on your router and / or modem as explained in the instructions"
            echo ""
            sleep 5.0
            echo "ONLY SSH and LND will be accessible from outside your home network over the WAN port and VPN tunnel"
            echo ""
            sleep 5.0
            echo "This is the prefered security model for Bitcoin"
            echo ""
            sleep 5.0
            echo ""
            echo "-----------------------------------------------------------------------------------------------"
            echo "Installing and configuring Uncompicated Firewall and fail2ban - this will take about ~5 minutes"
            echo ""

        }  
    # Step 7: Install and Configure Uncomplicated Firewall
        function ufw_Install {
            sudo killall apt apt-get
            sudo apt-get install ufw
            sudo ufw default deny incoming
            sudo ufw default allow outgoing
            sudo ufw allow 22 comment 'allow SSH from anywhere over WAN'
            sudo ufw allow 23 comment 'allow SSH from anywhere over WAN Pi-3'
            sudo ufw allow 24 comment 'allow SSH from anywhere over WAN Pi-4'
            sudo ufw allow proto udp from 192.168.2.0/24 port 1900 to any comment 'allow local LAN SSDP for UPnP discovery'
            sudo ufw allow proto udp from 192.168.1.0/24 port 1900 to any comment 'allow local LAN SSDP for UPnP discovery'
            sudo ufw allow 9735  comment 'allow Lightning mainnet'
            sudo ufw allow 19735  comment 'allow Lightning testnet'
            sudo ufw allow 8333  comment 'allow Bitcoin mainnet'
            sudo ufw allow 18333  comment 'allow Bitcoin testnet' 
            sudo ufw allow 10009 comment 'allow LND mainnet wallet from anywhere over WAN'
            sudo ufw allow 10010 comment 'allow LND testnet wallet from anywhere over WAN'
            sudo ufw allow from 192.168.2.0/24 to any port 50002 comment 'allow eps mainnet from local lan'
            sudo ufw allow from 192.168.1.0/24 to any port 50002 comment 'allow eps mainnet from local lan'
            sudo ufw allow from 192.168.2.0/24 to any port 50003 comment 'allow eps testnet from local lan'
            sudo ufw allow from 192.168.1.0/24 to any port 50003 comment 'allow eps testnet from local lan'
            sudo apt-get install fail2ban -y
        }
    # Step 7: Finished installing and configuring Uncomplicated Firewall and fail2ban
        function ufw_Enable {
            echo ""
            echo "Enabling Uncomplicated Firewall - enter 'y' below"
            ufw enable
            echo ""
            echo "" 
        }
    # Step 7: Enable systemctl enable Uncomplicated Firewall 
        function ufw_Systemctrl_Enable {
            systemctl enable ufw 
        }    
    # Step 7: Encapsulated Uncomplicated Firewall install and configure function 
        function ufw_Install_Parent_Function {
            sudo_Root "ufw_Install"  
        }
    # Step 7: Uncomplicated Firewall enable function 
        function ufw_Enable_Parent_Function {
            sudo_Root "ufw_Enable"           
        }    

    # Step 7: Encapsulated Uncomplicated Firewall systemctrl enable 
        function ufw_Systemctrl_Parent_Function {    
            sudo_Root "ufw_Systemctrl_Enable"     
        }    

    # Step 8: Increase File Limit - handles the six 'su root -c' commands
        function increase_File_Limit () {
            echo *    soft nofile 128000 >> /etc/security/limits.conf
            echo *    hard nofile 128000 >> /etc/security/limits.conf
            echo root soft nofile 128000 >> /etc/security/limits.conf
            echo root hard nofile 128000 >> /etc/security/limits.conf
            sed -i '/^# and here are more*/i session required pam_limits.so' /etc/pam.d/common-session
            sed -i '/^# end of pam-auth-update*/i session required pam_limits.so' /etc/pam.d/common-session-noninteractive
        
        }    
    # Step 9: Make a directory, '/home/pi/download' - download Bitcoin Core v0.18.1 32 bit ARM Linux 
    # Step 9: verify fingerprints and chucksum on-screen - then install Bitcoin Core
        function download_AND_Install_Bitcoin {
            mkdir /home/pi/download
            cd /home/pi/download
            rm -rf /home/pi/download/*.*
            wget https://bitcoincore.org/bin/bitcoin-core-0.18.1/bitcoin-0.18.1-arm-linux-gnueabihf.tar.gz
            wget https://bitcoincore.org/bin/bitcoin-core-0.18.1/SHA256SUMS.asc
            wget https://bitcoin.org/laanwj-releases.asc
            echo ""
            echo ""
            echo "Bitcoin downloaded successfully - you will have to verify signatures to install"
            echo "*******************************************************************************"
            echo ""
            echo ""
            echo "Ensure the output lists "OK" after bitcoin-0.18.1-arm"
            echo "--------------- Right below this line here |"
            echo "----------------------------------------------------"
            sha256sum --ignore-missing --check SHA256SUMS.asc
            echo ""
            echo "----------------------------------------------------"
            echo "You can safely ignore any warnings and failures"
            echo ""
            read -p "Press any key to continue as long as the output says "OK" or ctrl + z to exit if not"
            gpg --verify SHA256SUMS.asc
            echo ""
            echo "Import the public key of Wladimir van der Laan and verify the signed checksum file"
            gpg --import ./laanwj-releases.asc
            echo ""
            gpg --refresh-keys
            echo ""
            gpg --verify SHA256SUMS.asc
            echo ""
            echo "----------------------------------------------------------------------------"
            echo "Primary key fingerprint:  01EA 5486 DE18 A882 D4C2 6845 90C8 019E 36C2 E964 <------ The fingerprint directly above MUST match this one!" 
            echo ""
            echo ""
            echo "Press any key to proceed ONLY if the fingerprint matches and gpg says 'Good signature from Wladimir J. van der Laan'"
            read -p "Type 'ctrl + z' to exit if not"
            echo ""
            echo "Extracting Bitcoin Core binaries"
            tar -xvf bitcoin-0.18.1-arm-linux-gnueabihf.tar.gz  
            sudo install -m 0755 -o root -g root -t /usr/local/bin bitcoin-0.18.1/bin/*
            echo ""
            echo "    Verify version below as v0.18.1"
            sleep 3.0
            echo ""
            bitcoind --version

        }
    # 
    # Step 10: Create Bitcoin Core Directory and add a symbolic link that points to the external drive
        function create_Bitcoin_Core_Directory {
            mkdir /media/hdd/bitcoin
            unlink /home/pi/.bitcoin
            ln -s /media/hdd/bitcoin /home/pi/.bitcoin

        }   
    # Step 11: Start configuration download
        function download_Bitcoin_Conf_File_Start {
            echo ""
            echo "Configuring Bitcoin Core for mainnet and testnet, this will take about ~25 seconds..."
            echo ""
        }    
    # Step 11: Download already configured Bitcoin configuration files for mainnet and testnet and copy to bitcoin and testnet3 dir
        function download_Bitcoin_Conf_File {
            rm -rf /home/pi/download/bitcoin_config
            rm -rf /home/pi/.bitcoin/bitcoin.conf
            rm -rf /home/pi/.bitcoin/testnet3/bitcoin.conf
            cd /home/pi/download
            git clone https://github.com/bitlinc/bitcoin_config.git
            cp -avr /home/pi/download/bitcoin_config/README.md /home/pi/.bitcoin/bitcoin.conf
            cp -avr /home/pi/download/bitcoin_config/README.md /home/pi/.bitcoin/testnet3/bitcoin.conf
        }
    # Step 11: Parent function for download_Bitcoin_Conf_File function
        function download_Bitcoin_Conf_File_Complete_Parent {
            download_Bitcoin_Conf_File
        }

    # Step 12: Get bitcoind RPC user names and passwords for mainnet and testnet
        function set_RPC_Creds {
            echo ""
            read -r -p "Enter the TESTNET 'rpc' username `echo $'\n> '`" USERGIVENRPCTESTNETUSERNAME
            read -r -p "Enter the TESTNET 'rpc' password `echo $'\n> '`" USERGIVENRPCTESTNETPASSWORD
            echo ""
            echo ""
            read -r -p "Enter the MAINNET 'rpc' username `echo $'\n> '`" USERGIVENRPCMAINNETUSERNAME
            read -r -p "Enter the MAINNET 'rpc' password `echo $'\n> '`" USERGIVENRPCMAINNETPASSWORD
            echo ""
            echo ""   
        
        }

    # Step 12: Get bitcoind RPC user names and passwords for mainnet and testnet Parent
        function set_RPC_Creds_Bitcoin_Conf {  
            sed -i "s/bitcointestnetusername/$USERGIVENRPCTESTNETUSERNAME/g" /home/pi/.bitcoin/bitcoin.conf
            sed -i "s/bitcointestnetpassword/$USERGIVENRPCTESTNETPASSWORD/g" /home/pi/.bitcoin/bitcoin.conf
            sed -i "s/bitcoinmainnetusername/$USERGIVENRPCMAINNETUSERNAME/g" /home/pi/.bitcoin/bitcoin.conf
            sed -i "s/bitcoinmainnetpassword/$USERGIVENRPCMAINNETPASSWORD/g" /home/pi/.bitcoin/bitcoin.conf
        }     

    # Step 12: Copy bitcoin.conf to Pi user and testnet3 folder location
        function copy_Bitcoin_Conf_Testnet3 {
            rm -rf /home/pi/.bitcoin/testnet3/bitcoin.conf
            #rm -rf /home/pi/.bitcoin/bitcoin.conf
            cp -avr /home/pi/.bitcoin/bitcoin.conf /home/pi/.bitcoin/testnet3/bitcoin.conf
            #cp -avr /home/pi/.bitcoin/bitcoin.conf /home/pi/.bitcoin/bitcoin.conf
        }

    # Step 13: Use systemd to create a daemon that controls the startup process of Bitcoin mainnet and testnet
        function set_Bitcoind_Auto {
            cd /home/pi/download
            rm -rf /home/pi/download/bitcoin_testnet_auto
            rm -rf /home/pi/download/bitcoin_mainnet_auto      
            git clone https://github.com/bitlinc/bitcoin_testnet_auto
            git clone https://github.com/bitlinc/bitcoin_mainnet_auto
            sudo mv /home/pi/download/bitcoin_testnet_auto/README.md /etc/systemd/system/bitcoind_testnet.service
            sudo mv /home/pi/download/bitcoin_mainnet_auto/README.md /etc/systemd/system/bitcoind_mainnet.service
        }  

    # Step 13: Enable bitcoin_mainnet and bitcoin_testnet service configuration file downloaded from GitHub
        function enable_Bitcoind_Auto {
            sudo systemctl daemon-reload
            sudo chown -R root:root /etc/systemd/system
            sudo systemctl daemon-reload
            sudo systemctl enable bitcoind_testnet.service
            sudo systemctl enable bitcoind_mainnet.service
            sudo systemctl start bitcoind_mainnet.service
            sudo systemctl start bitcoind_testnet.service 
            #/run/user/1000
        }     

    # Step 13: Enable bitcoin_mainnet and bitcoin_testnet service configuration file downloaded from GitHub Parent
        function enable_Bitcoind_Auto_Parent {
            su_Root "enable_Bitcoind_Auto"  
        }

    # Step 14: Install Electrum Wallet - download package
        function download_Electrum_Wallet {
            cd /home/pi/download
            wget https://download.electrum.org/3.3.8/Electrum-3.3.8.tar.gz.asc
            wget https://download.electrum.org/3.3.8/Electrum-3.3.8.tar.gz
            wget https://raw.githubusercontent.com/spesmilo/electrum/master/pubkeys/ThomasV.asc
        }

    # Step 14: Import Thomas Voegtlin's Primary key and fingerprints 
        function import_Release_Electrum_Wallet {
            gpg --import /home/pi/download/ThomasV.asc
        }

    # Step 14: Verify Thomas Voegtlin's Primary key and fingerprints 
        function verify_Release_Electrum_Wallet {
            gpg --verify /home/pi/download/Electrum-3.3.8.tar.gz.asc
        }    

    # Step 14: Install Electrum Wallet - dependencies
        function install_Electrum_Wallet_Dependencies {
            apt-get install python3-pyqt5
            echo "PATH=$PATH:~/root/.local/bin" >> /home/pi/.bashrc
            echo "alias ls='ls -la --color=always'" >> /home/pi/.bashrc
        }

    # Step 14: Suppress Install Electrum Wallet - dependencies
            function install_Electrum_Wallet_Dependencies_Parent {
            su_Root "install_Electrum_Wallet_Dependencies"   
        }

    # Step 14: Set ownership of external drive directory 'pi' for user 'pi'
            function set_Pi_As_Owner_Drive {
                chown -R pi:pi /media/pi
            }

    # Step 14: Set ownership of external drive directory 'pi' for user 'pi' Parent
            function set_Pi_As_Owner_Drive_Parent {
                su_Root "set_Pi_As_Owner_Drive"
            }

    # Step 14: Install Electrum Wallet - install Electrum Wallet for mainnet and testnet
        function install_Electrum_Wallet {
            unlink /home/pi/.electrum
            cd /home/pi/download
            sudo apt-get install python3-setuptools python3-pip -y
            sudo pip3 install setuptools
            python3 -m pip install Electrum-3.3.8.tar.gz[fast]
            sudo mkdir /media/pi/Electrum
            sudo chown -R pi:pi /media/pi
            ln -s /media/pi/Electrum /home/pi/.electrum
            cd /home/pi/.electrum
            wget http://icons.iconarchive.com/icons/alecive/flatwoken/256/Apps-Electrum-icon.png
            mv Apps-Electrum-icon.png electrum-icon.png
            mkdir testnet 
            cp -avr /home/pi/.electrum/electrum-icon.png /home/pi/.electrum/testnet/electrum-icon.png
            cd /home/pi/download
            rm -rf /home/pi/download/electrum_config
            git clone https://github.com/bitlinc/electrum_config
            mv /home/pi/download/electrum_config/README.md /home/pi/.electrum/testnet/config  
            cp -avr ~/.electrum/testnet/config ~/.electrum/config
        }

    # Step 14: Set ownership of Electrum wallet's config file to user 'bitcoin' and then back to user 'pi'
        function set_Electrum_Config_Ownership {
            chown -R pi:pi /media/pi/Electrum
            
        }
    # Step 14: Set ownership of Electrum wallet's config file to user 'bitcoin' and then back to user 'pi'
        function set_Electrum_Config_Ownership_Parent {
            su_Root "set_Electrum_Config_Ownership" 
        }

    # Step 14: Edit Electrum wallet config file 
        function edit_Electrum_Config {
            sed -i "s/var2/$USERGIVENRPCTESTNETUSERNAME/g" /home/pi/.electrum/testnet/config
            sed -i "s/bin1/$USERGIVENRPCTESTNETPASSWORD/g" /home/pi/.electrum/testnet/config
            sed -i "s/var2/$USERGIVENRPCMAINNETUSERNAME/g" /home/pi/.electrum/config
            sed -i "s/bin1/$USERGIVENRPCMAINNETPASSWORD/g" /home/pi/.electrum/config
        }

    # Step 14: Creates a desktop shortcut to Electrum by double-clicking the icon    
    function electrum_Wallet_Desktop_Shortcut_Testnet {
            cd ~/Desktop
            rm -rf Electrum_Testnet.desktop
            touch Electrum_Testnet.desktop
            echo "[Desktop Entry]" >> Electrum_Testnet.desktop
            echo "Comment=Lightweight Bitcoin Client" >> Electrum_Testnet.desktop
            echo "Name=Electrum Testnet" >> Electrum_Testnet.desktop
            echo "Exec=sh -c \"~/.local/bin/electrum --testnet --oneserver --server 127.0.0.1:50003:s\"" >> Electrum_Testnet.desktop
            echo "Icon=/home/pi/.electrum/testnet/electrum-icon.png" >> Electrum_Testnet.desktop
            echo "Type=Application" >> Electrum_Testnet.desktop
            echo "Name[en_US]=Electrum_Testnet" >> Electrum_Testnet.desktop
            echo "StartupNotify=true" >> Electrum_Testnet.desktop
            #echo "Electrum_Testnet.desktop" >> Electrum_Testnet.desktop
            echo "StartupWMClass=electrum" >> Electrum_Testnet.desktop
            echo "Terminal=false" >> Electrum_Testnet.desktop
            
            chmod a+x Electrum_Testnet.desktop
        }

    # Step 14: Creates a desktop shortcut to Electrum by double-clicking the icon    
    function electrum_Wallet_Desktop_Shortcut_Mainnet {
            cd ~/Desktop
            rm -rf Electrum_Mainnet.desktop
            rm -rf Electrum_mainnet.desktop
            touch Electrum_Mainnet.desktop
            echo "[Desktop Entry]" >> Electrum_Mainnet.desktop
            echo "Comment=Lightweight Bitcoin Client" >> Electrum_Mainnet.desktop
            echo "Name=Electrum Mainnet" >> Electrum_Mainnet.desktop
            echo "Exec=sh -c \"~/.local/bin/electrum --oneserver --server 127.0.0.1:50002:s\"" >> Electrum_Mainnet.desktop
            echo "Icon=/home/pi/.electrum/electrum-icon.png" >> Electrum_Mainnet.desktop
            echo "Type=Application" >> Electrum_Mainnet.desktop
            echo "Name[en_US]=Electrum_Mainnet" >> Electrum_Mainnet.desktop
            echo "StartupNotify=true" >> Electrum_Mainnet.desktop
            #echo "Electrum_mainnet.desktop" >> Electrum_mainnet.desktop
            echo "StartupWMClass=electrum" >> Electrum_Mainnet.desktop
            echo "Terminal=false" >> Electrum_Mainnet.desktop

            chmod a+x Electrum_Mainnet.desktop
        }


    # Step 14: Getting electrum wallet master keys from your Electrum Wallet
        function get_Electrum_MPK {
            USERMASTERKEYMAINNET="$(~/.local/bin/./electrum getmpk)"
            USERMASTERKEYTESTNET="$(~/.local/bin/./electrum --testnet getmpk)"
            #ELECTRUMPID="$(ps -e | pgrep electrum)"
            #kill -INT $ELECTRUMPID
        }
    # Step 15: Downloadn and verify Electrum Personal Server - EPS
        function download_EPS {
            unlink /home/pi/eps_mainnet
            unlink /home/pi/eps_testnet
            mkdir /media/hdd/eps_testnet
            mkdir /media/hdd/eps_mainnet
            ln -s /media/hdd/eps_mainnet /home/pi/eps_mainnet
            ln -s /media/hdd/eps_testnet /home/pi/eps_testnet
            cd /home/pi/download
            wget https://github.com/chris-belcher/electrum-personal-server/archive/electrum-personal-server-v0.1.7.tar.gz
            wget https://github.com/chris-belcher/electrum-personal-server/releases/download/electrum-personal-server-v0.1.7/electrum-personal-server-v0.1.7.tar.gz.asc
            wget https://raw.githubusercontent.com/chris-belcher/electrum-personal-server/master/docs/pubkeys/belcher.asc        
        }

    # Step 15: Extract Electrum Personal Server - EPS
        function extract_EPS {
            cd /home/pi/download
            tar -xvf electrum-personal-server-v0.1.7.tar.gz
        } 

    # Step 15: Install Electrum Personal Server Parent function - EPS
        function extract_EPS_Parent {
            "extract_EPS" 
        }

    # Step 15: Download EPS config.ini for mainnet and append user given rps creds
        function copy_EPS_Mainnet {
            cp -avr /home/pi/download/electrum-personal-server-electrum-personal-server-v0.1.7 /home/pi/eps_mainnet/electrum-personal-server
            cd /home/pi/eps_mainnet/electrum-personal-server
            pip3 install wheel
            pip3 install --user .
            wget https://raw.githubusercontent.com/bitlinc/eps_mainnet/master/README.md
            mv README.md.1 config.ini
        }

    # Step 15: Download EPS config.ini for mainnet and edit append user given rps creds parent 
        function copy_EPS_Mainnet_Parent {
            "copy_EPS_Mainnet" 
        }

    # Step 15: Download EPS config.ini for testnet and append user given rps creds 
        function copy_EPS_Testnet {
            cp -avr /home/pi/download/electrum-personal-server-electrum-personal-server-v0.1.7 /home/pi/eps_testnet/electrum-personal-server
            cd /home/pi/eps_testnet/electrum-personal-server
            pip3 install wheel
            pip3 install --user .
            wget https://raw.githubusercontent.com/bitlinc/eps_testnet/master/README.md
            mv README.md.1 config.ini
        }

    # Step 15: Download EPS config.ini for testnet and append user given rps creds parent 
        function eps_MPK_Mainnet_Testnet_Config {
            sed -i "s/var2/$USERGIVENRPCMAINNETUSERNAME/g" /home/pi/eps_mainnet/electrum-personal-server/config.ini
            sed -i "s/bin1/$USERGIVENRPCMAINNETPASSWORD/g" /home/pi/eps_mainnet/electrum-personal-server/config.ini
            sed -i "s/replacempkmainnet/$USERMASTERKEYMAINNET/g" /home/pi/eps_mainnet/electrum-personal-server/config.ini
            sed -i "s/var2/$USERGIVENRPCTESTNETUSERNAME/g" /home/pi/eps_testnet/electrum-personal-server/config.ini
            sed -i "s/bin1/$USERGIVENRPCTESTNETPASSWORD/g" /home/pi/eps_testnet/electrum-personal-server/config.ini
            sed -i "s/replacempktestnet/$USERMASTERKEYTESTNET/g" /home/pi/eps_testnet/electrum-personal-server/config.ini
        }

 
    # Step 15: Download EPS config.ini for testnet and append user given rps creds parent 
        function copy_EPS_Testnet_Parent {
            "copy_EPS_Testnet" 
        }    

    # Step 15: Start EPS 
        function start_EPS {
            /home/pi/.local/bin/electrum-personal-server /home/pi/eps_mainnet/electrum-personal-server/config.ini
            /home/pi/.local/bin/electrum-personal-server /home/pi/eps_mainnet/electrum-personal-server/config.ini
        }  

    # Step 15: Create system daemon to auto start EPS after Bitcoind synced  
        function auto_Start_EPS {
            cd /etc/systemd/system
            sudo rm -rf eps_*
            mkdir /home/pi/download/eps_auto
            cd /home/pi/download/eps_auto
            wget https://raw.githubusercontent.com/bitlinc/eps_mainnet_auto/master/README.md
            sudo mv README.md /etc/systemd/system/eps_mainnet.service
            wget https://raw.githubusercontent.com/bitlinc/eps_testnet_auto/master/README.md
            sudo mv README.md /etc/systemd/system/eps_testnet.service
            sudo systemctl daemon-reload
            systemctl enable eps_mainnet.service
            systemctl enable eps_testnet.service
            systemctl start eps_mainnet.service
            systemctl start eps_testnet.service
        } 

    # Step 15: Create system daemon to auto start EPS after Bitcoind is synced
        function auto_Start_EPS_Parent {
            su_Root "auto_Start_EPS" 
        } 

    # 15 Run EPS Mainnet 
        function run_EPS_Mainnet {
            /home/pi/.local/bin/electrum-personal-server /home/pi/eps_mainnet/electrum-personal-server/config.ini
        }

    # 15 Run EPS Testnet 
        function run_EPS_Testnet {
            /home/pi/.local/bin/electrum-personal-server /home/pi/eps_testnet/electrum-personal-server/config.ini
        }


    # Step 16 install hardware wallet support for Trezor for Electrum Wallet
        function install_Hardware_Wallet_Support_Trezor {
            sudo apt update
            sudo apt install libhidapi-hidraw0 libhidapi-libusb0
            sudo apt install libusb-devel systemd-devel
            python3 -m pip install trezor[hidapi]
            cd  /etc/udev/rules.d
            sudo wget https://raw.githubusercontent.com/trezor/trezor-common/master/udev/51-trezor.rules
            sudo udevadm control --reload-rules && sudo udevadm trigger
        }  

    # Step 17 install hardware wallet support for Ledger Nano S for Electrum Wallet
        function install_Hardware_Wallet_Support_Trezor {
            sudo groupadd plugdev


            sudo udevadm control --reload-rules && sudo udevadm trigger
        }    


    # Step 16 install hardware wallet support
        function install_Hardware_Wallet_Support_Trezor {
            su_Root "install_Hardware_Wallet_Support"
        }    


    # Download Bitcoin - LND - Electrum preconfigured aliases, save to the external drive and then create a symbolic link for all users
        function get_Bitlinc_Aliases {
            cd /home/pi/download
            mkdir bitlinc_aliases
            cd /home/pi/download/bitlinc_aliases
            rm -rf README.md
            sudo wget https://raw.githubusercontent.com/bitlinc/btc_lnd_alises/master/README.md
            rm -rf /home/pi/.bash_aliases
            sudo chown -R pi:pi /home/pi/download
            cp -avr /home/pi/download/bitlinc_aliases/README.md /home/pi/.bash_aliases
            source /home/pi/.bash_aliases 

        }

    # Download Bitcoin - LND - Electrum preconfigured aliases, save to the external drive and then create a symbolic link for all users
        function install_Team_Viewer {
            cd /home/pi/download
            sudo apt-get update -y
            sudo apt-get upgrade -y
            wget https://download.teamviewer.com/download/linux/teamviewer-host_armhf.deb
            sudo dpkg -i teamviewer-host_armhf.deb
            sudo apt --fix-broken install -y
        }

    ## Body of code
echo ""
echo ""
echo "**********************************************" 
echo "*  Bitcoin - Electrum Wallet Install Script  *"
echo "**********************************************"

sleep 2.0
# First step - looking for the bitcoin director on external drive, if not found provsioning drive for full install and start initial blockchain download
echo "------------------------------------------------------------------------------------------------------------"
echo "Pre-Raspberry Pi Install: Downloading Bitcoin Blockchain for Mainnet and Testnet - Prepairing External Drive"
echo "------------------------------------------------------------------------------------------------------------"
    echo ""
    echo "Verifing the operating system you are using..."
    sleep 2.0
    echo ""
    echo "Verifing your external drive does NOT contain a Bitcoin directory..."
    sleep 2.0
    echo ""
    echo "You may be promted to enter your password to mount your external drive"
    echo ""
    echo ""
    echo "Here is a list of all drives connect to your Linux computer"
    USERNAME=$(whoami)
    lsblk
    echo ""
    read -r -p "Please enter the disk "Identifier" located on the far left that itentifes your external drive - this drive WILL be formatted for Bitcoin `echo $'\n> '`" LINUXDRIVE
    echo "$LINUXDRIVE"
    echo ""
    echo ""
    sleep 3.0
    UUIDEXTERNALDRIVE="$(sudo blkid -s UUID -o value /dev/"$LINUXDRIVE")"
        if  [  -d /home/"$USERNAME"/btc_drive ] || [  -d /media/"$USERNAME"/hdd/bitcoin ] || [  -d /media/"$USERNAME"/"$UUIDEXTERNALDRIVE"/hdd ] || [ -d /Volumes/btc_drive ] || [ -d /media/"$USERNAME"/btc_drive/hdd ]; then
            echo ""
            echo "Your 'bitcoin' directory already exists - running install on your Raspeberry Pi" 
            echo ""
            echo ""
            sleep 3.0
        else
            case "$OSTYPE" in
                darwin*)  install_Bitcoin_macOS ;;
                linux*)   install_Bitcoin_Linux_OS ;;
                msys*)    echo "This Bitcoin install script isn't configured for use with Windows yet" ;;
                *)        echo "unknown: $OSTYPE" ;;
            esac
        fi 

# - 1: set users 'root' & 'pi' to password [A] and create user 'bitcoin' and assign it password [A]
echo "--------------------------------------------------------------------------"
echo "Step 1: Set your password [A] to 'pi' and 'root' users"
echo "--------------------------------------------------------------------------"
    echo ""  
        create_Passwords_Users 
        sleep 1.5
    echo ""
    echo ""
# set system host name     
echo "--------------------------------------------------------------------------"
echo "Step 2: Set system host name used to identify your Pi over a network" 
echo "--------------------------------------------------------------------------"
    echo ""
    echo "Create the network name used to identify your node when online"
    echo ""    
        sleep 2.0 
    read -r -p "Enter the host name for your Pi `echo $'\n> '`" USERGIVENHOSTNAME
        suppress change_Hostname_Hosts_Parent
    echo ""    
    echo "Your Pi host name updated was successfully set to "$USERGIVENHOSTNAME""
        sleep 1.5
    echo ""
    echo ""
# installing necessary software packages then update and upgrade the system [apt-get install htop git curl bash-completion jq dphys-swapfile dirmngr] then apt-get update / upgrade   
echo "--------------------------------------------------------------------------"
echo "Step 3: Install the necessary software packages for your Pi and updating"
echo "--------------------------------------------------------------------------"
    echo "" 
        install_Packages_Start
        suppress install_Packages_Parent
        install_Packages_Successfull
    echo ""
    echo ""
# Provision external drive - format with ext4 - edit fstab for symbolic link pointing to external drive at /media/hdd - mount the drive and set owner as the user 'bitcoin'  
echo "--------------------------------------------------------------------------"
echo "Step 4: Provisioning your external drive"
echo "--------------------------------------------------------------------------"
    echo ""
    echo "VERY IMPORTANT - PLEASE READ"
        sleep 4.0
    echo ""         
    echo "This step will require that you have only ONE external drive connected to your Pi"
    echo ""
        sleep 4.0
    echo "The drive will be wiped clean and formatted as ext4 unless you already have the bitcoin directory on your drive - then it will skip this step"
    echo "---------------------------------------------------------------------------------------------------------------------------------------------"
    echo "---------------------------------------------------------------------------------------------------------------------------------------------"
    echo ""
    sleep 4.0
    read -p "Press any key to continue - again the script will check to make sure it does NOT delete your bitcoin directory..."
    echo ""
        sleep 5.0
    echo ""
        UUIDEXTERNALDRIVE="$(sudo blkid -s UUID -o value /dev/sda)"
        if [ ! -d /media/pi/"$UUIDEXTERNALDRIVE"/bitcoin ]; then
                read -r -p "Enter 'yes' to confirm formatting of your external drive or ctrl + c to exit `echo $'\n> '`" CONFIRMFORMAT
                    if [[ "$CONFIRMFORMAT" = 'yes' || "$CONFIRMFORMAT" = 'Yes' ]]; then
                        echo "Formatting your drive"
                        echo "Confirm one more time by entering "y" "
                        sudo mkfs.ext4 /dev/sda
                        echo "Your external drive was created successfully"
                        echo ""
                        sleep 2.0
                     else
                        echo ""
                        echo "Skipping formating your external drive since your 'bitcoin' dirctory already exists"
                        echo ""
                        sleep 2.0
                    fi
            else
                echo ""
                echo "Skipping formatting your drive since your 'bitcoin' directory already exists" 
                echo ""
                echo ""
                sleep 3.0
            fi 
        if [ ! -d /media/hdd ]; then
                echo "------------------------------------------------------"
                echo "Step 5: Verify 'Filesystem' has been mounted to drive "
                echo "------------------------------------------------------"
                    echo ""
                    echo "Mounting external drive to your 'pi's file system"
                    echo ""
                    sleep 2.0
                        suppress make_Drive_Parent
                        suppress mount_Filesystem_Parent
                        verify_Mount_AND_Set_Ownership
                        suppress make_Bitcoin_Directory
                    echo ""    
                    echo "Your external hard drive is then attached to the file system and can be accessed as a regular folder now"    
                        sleep 4.0   
                    echo ""
                    echo ""
            else
                echo ""
                echo "Skipping mounting your drive since the link exists already" 
                echo ""
                echo ""
                sleep 3.0
            fi             


# Moving system swap file from SD card to external drive
echo "--------------------------------------------------------------------------"
echo "Step 6: Moving location of system 'Swap File'"
echo "--------------------------------------------------------------------------"
    echo ""
         if [ ! -f /media/swapfile ]; then
                move_Swap_File_Start
                suppress move_Swap_File_Parent
                echo ""
                echo "System Swap File has been moved successfully"
                echo ""
                echo ""
                sleep 2.5
        else
                echo ""
                echo "System swapfile has already been moved to your external drive - skipping this step"
                echo ""
                echo ""
                sleep 3.5
        fi          
# Installing Uncomplicated Firewall and configuring it for Bitcoin, LND and Electrum
echo "--------------------------------------------------------------------------"
echo "Step 7: Installing and configuring Uncomplicated Firewall & fail2ban"
echo "--------------------------------------------------------------------------"
    echo ""
        ufw_Install_Start
        suppress ufw_Install_Parent_Function
        ufw_Enable_Parent_Function
        suppress ufw_Systemctrl_Parent_Function
    echo ""
    echo "Uncomplicated Firewall and fail2ban have been install and configured successfully"
        sleep 3.0
    echo ""
    echo ""
# Increasing your open files limit 
echo "--------------------------------------------------------------------------"
echo "Step 8: Increasing your open files limit"
echo "--------------------------------------------------------------------------"
    echo ""
    echo "Increaing TCP connection limits"
        sudo_Root "increase_File_Limit"
    echo ""
    echo "Completed successfully"
        sleep 3.0
    echo ""
    echo ""    
# Increasing your open files limit 
echo "--------------------------------------------------------------------------"
echo "Step 9: Downloading, Install and verify Bitcoin Core"
echo "--------------------------------------------------------------------------"
    echo ""
    echo "Downloading and installing Bitcoin version 0.18.1 32 bit ARM for Linux"
    echo ""  
    sleep 4.0
        download_AND_Install_Bitcoin   
    echo ""
    echo "Bitcoin Core has been verified and installed successfully"
        sleep 6.0
    echo ""
    echo ""
# Creating Bitcoin Core directory on the external drive and linking it to your /home/pi/.bitcoin
echo "--------------------------------------------------------------------------"
echo "Step 10: Creating Bitcoin Core Directory"
echo "--------------------------------------------------------------------------"
    echo ""
    echo "Creating Bitcoin core home directory and linking it to the external drive"
    echo ""
        sleep 3.0      
            suppress create_Bitcoin_Core_Directory   
    echo ""
    echo "Successfully linked bitcoin core installed on the external drive, to your home directory"
        sleep 3.0
    echo ""
    echo ""
# Downloading pre-configured bitcoin core mainnet and testnet conf files 
echo "--------------------------------------------------------------------------"
echo "Step 11: Download Bitcoin Core for Mainnet and Testnet config file"
echo "--------------------------------------------------------------------------"
    echo ""
    echo "Configuring Bitcoin Core for mainnet and testnet, this will take a few moments" 
    suppress download_Bitcoin_Conf_File_Complete_Parent        
    echo ""
    echo ""
    echo "Bitcoin Core mainnet and testnet config files downloaded successfully"
        sleep 3.0
    echo ""
    echo ""
# Set bitcoind rpc username and password and assign it to a global varable
echo "--------------------------------------------------------------------------"
echo "Step 12: Set RPC Username and Password"
echo "--------------------------------------------------------------------------"
    echo ""  
    echo "Please set your RPC username and password [B] for Bitcoin mainnet and testnet"
    echo ""
    echo ""
        sleep 2.0    
        set_RPC_Creds
        suppress set_RPC_Creds_Bitcoin_Conf
        suppress copy_Bitcoin_Conf_Testnet3
    echo ""
    echo "RPC username and password for mainnet and testnet were configured successfully"
        sleep 3.0
    echo ""
    echo ""
# Create a system daemon to run / auto-start Bitcoind in the background     
echo "------------------------------------------------------------------------------"
echo "Step 13: Create a system daemon to run / auto-start Bitcoind in the background"
echo "------------------------------------------------------------------------------"
    echo ""
    echo "Creating a system deamon that will auto load Bitcoin Core mainnet and testnet on boot"
    echo ""
    echo ""
        if [ ! -f /etc/systemd/system/bitcoin_testnet.service ]; then
                suppress set_Bitcoind_Auto
                suppress enable_Bitcoind_Auto_Parent
                echo "Bitcoind was successfully configured to auto boot in the backgroud"
                echo ""
                sleep 3.0
                echo ""
                echo "Mainnet and testnet will now auto boot simultaneously when pi is powered on or rebooted"
                echo ""
                echo ""
                sleep 3.0
        else
                echo "Bitcoin daemon already exists - skipping this step"
                sleep 3.0
                echo ""
                echo ""
        fi

# Install Electrum Wallet and configure it to auto connect to node open on boot and reboot   
echo "------------------------------------------------------------------------------"
echo "Step 14: Installing and configuring Electrum Wallet for Mainnet and Testnet"
echo "------------------------------------------------------------------------------"
    echo ""
    echo "Electrum Wallet will be configured to connect to your Bitcoin node automatically at boot" 
    echo ""
    echo ""
        sleep 3.0
    echo "Downloading Electrum - you must veryify signatures below to proceed"
    echo ""
    echo ""
        sleep 4.0 
        download_Electrum_Wallet
    echo ""
    echo ""
    echo "Electrum Wallet downloaded successfully - you will have to verify signatures to install"
    echo "***************************************************************************************"
    echo ""
    echo ""
        sleep 3.0
        gpg --import /home/pi/download/ThomasV.asc
    echo ""
    echo ""
        gpg --verify /home/pi/download/Electrum-3.3.8.tar.gz.asc
    echo ""    
    echo "Verify 'Good signature from Thomas Voegtlin' is shown and the Primary key fingerprints match"
        sleep 4.0
    echo ""    
echo "---------------------------------------------------------------------------"
echo "Primary key fingerprint: 6694 D8DE 7BE8 EE56 31BE  D950 2BD5 824B 7F94 70E6 <------ The fingerprint above MUST match this one!"
echo "---------------------------------------------------------------------------"
        sleep 2.0
    read -p "Press any key if fingerprint matches and signature reads 'Good' from Thomas Voegtlin or 'ctrl + z' to exit"
    echo ""
        sleep 2.0
    echo ""
    echo "Installing Electrum Wallet for mainnet and testnet - configuring to auto connect to your node at boot"
    echo "*****************************************************************************************************"
    echo ""
    echo "Configuring..."
        suppress install_Electrum_Wallet_Dependencies_Parent
        suppress install_Electrum_Wallet
        suppress set_Electrum_Config_Ownership_Parent
        suppress sed -i "s/var2/$USERGIVENRPCTESTNETUSERNAME/g" /home/pi/.electrum/testnet/config
        suppress sed -i "s/bin1/$USERGIVENRPCTESTNETPASSWORD/g" /home/pi/.electrum/testnet/config
        suppress sed -i "s/var2/$USERGIVENRPCMAINNETUSERNAME/g" /home/pi/.electrum/config
        suppress sed -i "s/bin1/$USERGIVENRPCMAINNETPASSWORD/g" /home/pi/.electrum/config
        sudo rm -rf /etc/xdg/autostart/Elec*
        suppress electrum_Wallet_Desktop_Shortcut_Testnet
        sudo cp -avr /home/pi/Desktop/Electrum_Testnet.desktop /etc/xdg/autostart/Electrum_Testnet.desktop
        suppress electrum_Wallet_Desktop_Shortcut_Mainnet
        sudo cp -avr /home/pi/Desktop/Electrum_Mainnet.desktop /etc/xdg/autostart/Electrum_Mainnet.desktop
    echo ""
    echo ""
    echo ""
echo "********************************************************************************************"
echo "Electrum Wallet for mainnet and testnet is starting - You must setup both wallets to proceed"
echo "********************************************************************************************"
    echo ""
    echo ""
    echo ""
    echo "Setup NEW Electrum MAINNET wallet - once Electrum loads, close Electrum to continue"
    echo "-----------------------------------------------------------------------------------"
    sleep 4.0
    echo ""
        ~/.local/bin/electrum --oneserver --server 127.0.0.1:50002:s
    echo ""
    echo "Enter the name of the wallet you just created with Electrum for bitcoin mainnet"
    echo ""
    sleep 3.0
    echo ""
    read -r -p "If you didn't change the name type without quotes, 'default_wallet' `echo $'\n> '`" USERGIVENMAINNETWALLETNAME
    echo "" 
        USERMASTERKEYMAINNET="$(~/.local/bin/./electrum getmpk -w ~/.electrum/wallets/$USERGIVENMAINNETWALLETNAME)"
    echo ""
    echo ""
        sleep 2.0
    echo ""    
    echo ""
    echo "Setup NEW Electrum TESTNET wallet - once Electrum loads, close Electrum to continue"
    echo "-----------------------------------------------------------------------------------"
        sleep 5.0
    echo ""    
        ~/.local/bin/electrum --testnet --oneserver --server 127.0.0.1:50003:s
    echo ""
    echo "Enter the name of the wallet you just created with Electrum for bitcoin testnet"
    echo ""
    sleep 3.0
    echo ""
    read -r -p "If you didn't change the name type without quotes, 'default_wallet' `echo $'\n> '`" USERGIVENTESTNETWALLETNAME
    echo ""
        USERMASTERKEYTESTNET="$(~/.local/bin/./electrum --testnet getmpk -w ~/.electrum/testnet/wallets/$USERGIVENTESTNETWALLETNAME)"
        
    echo ""
    echo ""
        sleep 2.0
    echo ""
    echo ""
echo "##########################################################################################################"
echo "You MUST read about the basics of wallet security - read the 'Wallet Security' link in the instructions!!!"
echo "##########################################################################################################"
    echo ""
    echo ""
        sleep 4.0
    echo "-----------------------------------------------------------------------------"
    echo "Your Electrum Wallet has been configured for mainnet and testnet successfully"
    echo ""
        sleep 4.0
    echo "There is an Electrum application shortcut on your desktop one for testnet and one mainnet"
    echo ""
        sleep 4.0
    echo "Both Electrum Wallets will open when the Pi is booted or restarted - connecting only to your bitcoin node"
        sleep 4.0
    echo ""
    echo ""

# Install Electrum Personal Server - EPS     
echo "------------------------------------------------------------------"
echo "Step 15: Downloading and Installing Electrum Personal Server - EPS"
echo "------------------------------------------------------------------"
    echo ""
    echo "Installing Electrum Personal Server (EPS) for both mainnet and testnet"
    echo ""
        sleep 4.0
    echo "EPS connects your Electrum Wallets - both mainnet and testnet - to your bitcoin node"
    echo ""
        sleep 4.0
    echo "EPS runs as a deamon in the backgroud and therefore you won't see or interact with it"               
    echo "-------------------------------------------------------------------------------------"
    echo ""
        sleep 4.0
        download_EPS
    echo "EPS downloaded successfully - you will have to verify signatures to install"
    echo "***************************************************************************"
    echo ""
        cd /home/pi/download
        gpg --import belcher.asc
        gpg --verify electrum-personal-server-v0.1.7.tar.gz.asc
    echo "Verify that the release is signed by Chris Belcher and Primary key fingerprints match"
        sleep 2.0           
echo "--------------------------------------------------------------------------"
echo "Primary key fingerprint: 0A8B 038F 5E10 CC27 89BF  CFFF EF73 4EA6 77F3 1129 <------ The fingerprint above MUST match this one"
echo "--------------------------------------------------------------------------"
    read -p "Press any key if everything matches or 'ctrl + z' to exit"
    echo ""
    echo "Installing Electrum Personal Server..."
    echo ""
        extract_EPS_Parent
        copy_EPS_Mainnet
        copy_EPS_Testnet
        eps_MPK_Mainnet_Testnet_Config
        suppress get_Bitlinc_Aliases
        read -r -p "STOPPING BEFORE WE RUN EPS"
    echo ""
    echo ""
    echo "Electrum Personal Server will now SYNC with the blockchain - this can take upwards of 30 minutes since there are two chains to sync"
    echo ""
    sleep 3.0
    echo "After EPS has configured all the address it will reload - please just let it work"
    echo ""
    sleep 3.0
        #lxterminal --title="EPS_Mainnet" -e "bash -c /home/pi/.local/bin/electrum-personal-server /home/pi/eps_mainnet/electrum-personal-server/config.ini;bash"
        #lxterminal --title="EPS_Testnet" -e "bash -c /home/pi/.local/bin/electrum-personal-server /home/pi/eps_testnet/electrum-personal-server/config.ini;bash"
        run_EPS_Mainnet
        run_EPS_Testnet
        auto_Start_EPS_Parent
    echo ""
    echo ""    
    echo "Electrum Personal Server was installed and configured for mainnet and testnet successfully"
    echo ""
    echo ""
    # Install Team Viewer 
echo "--------------------------------------------"
echo "Step 16: Downloading and Install Team Viewer"
echo "--------------------------------------------"
echo ""
    echo "Installing Team Viewer for simple remote access to your Node - this is only suggested for Bitcoin testnet"
    echo ""
        sleep 4.0
        read -r -p "Enter 'yes' to confirm installing Team Viewer or anything else to skip install `echo $'\n> '`" CONFIRMTEAMVIEWER
                    if [[ "$CONFIRMTEAMVIEWER" = 'yes' || "$CONFIRMTEAMVIEWER" = 'Yes' ]]; then
                        echo "Installing Team Viewer - this will take about 3 minutes..."
                            install_Team_Viewer
                            echo ""    
                            echo "Team Viewer has been successfully installed"    
                                sleep 2.0   
                            echo ""
                            echo ""
                     else
                        echo ""
                        echo "Skipping Team Viewer install"
                        echo ""
                        sleep 2.5
                    fi
    echo "END OF SCIPT" 
    sleep 3.0
    sudo reboot
        


