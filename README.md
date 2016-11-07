# pi-distortion-product
An experiment involving a custom spatialized audio system, inner-ear distortion products, and mechatronics.

# setup

## Raspberry-Pi Setup
Install a fresh [Raspian](https://www.raspberrypi.org/downloads/noobs/), and then
do some basic configuration. I named my pis Agnes and Ethel, and set the keyboard
to US.

### Initial WIFI Setup

Next, we'll need to enable the WIFI; I'm using Raspberry Pi 3s, so I'll be using
the onboard WIFI for now. To add an existing network, edit your `wpa-supplicant.conf`
file using the `nano` file editor.

    sudo nano /etc/wpa_supplicant/wpa_supplicant.conf

And then add this block of text to the bottom of the `.conf` file.

    network={
        ssid="your_network"
        psk="your_passkey"
    }

You can test that this is working by either rebooting `sudo reboot` or by disabling and
enabling your WIFI.

    sudo ifdown wlan0
    sudo ifup wlan0

Then you can using `ifconfig` to see if your pi has an IP and a network it is connected to.

Next, and this is only pertinent to the Raspberry Pi3, we have the option to disable
the WIFI power saving feature. This will ensure that our pis do not disconnect from the
internet during a period of inactivity, to do this we'll modify the `interfaces` file.

    sudo nano /etc/network/interfaces

And then underneath the `iface wlan0 inet manual` line add the following.

    post-up iw dev $IFACE set power_save off

Altogether, the interfaces file should look like this.

    allow-hotplug wlan0
    iface wlan0 inet manual
        post-up iw dev $IFACE set power_save off
        wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf

At this point I recommend installing `ChucK` and it's dependencies and cloning this
repository onto your pi. Which is shown in the next section.

### ChucK/Chugins Setup

To install ChucK, we'll first need a few dependencies, this section requires that
your pi is connected to the internet, which is outlined in the above section.

You can use `apt-get` to install these dependencies.

    sudo apt-get install bison flex
    sudo apt-get install alsa-base libasound2-dev libasndfile1-dev

Use `git` (which comes installed by default on Raspbian) to clone ChucK. Choose a
directory to install into, I use `~/git/`, and then clone ChucK.

    git clone https://github.com/ccrma/chuck

Then we'll have to navigate into the `chuck/src` and `make` the ChucK build.

    cd chuck/src
    make linux-alsa

And install using the following `make` command while in the same directory.

    sudo make install

This project is using a custom Chugin (think ChucK plugin) which did not install
with your ChucK build, so you'll have to clone that repo to. Change back to your
specified `git` directory, (again, I used `~/git`) and clone the Chugins repo.

    git clone https://github.com/ccrma/chugins

Then navigate into the `chugins/winfuncenv/` directory and build the Chugin.

    cd ~/git/chugins/winfuncenv
    make linux-alsa

And if alls well, install it.

    sudo make install

ChucK should be ready to go!

### HiFiBerry Amp+ Setup

This project is using the amp shields from [HiFiBerry](https://www.hifiberry.com/),
so we'll need to disable the default soundcard and instead use the DAC on the amp.

We'll need to edit our `/boot/config.txt` on our pi. So `nano` into it.

    sudo nano /boot/config.txt

And remove or comment out the default soundcard.

    dtparam=audio=on

Then replace it with this HiFiBerry

    dtoverlay=hifiberry-amp

And that's it! ChucK will automatically use the amp now.

### Putting Your Pi On a Private Network

This project uses a dedicated router for its communication, this ensures that
and network traffic doesn't have to compete with competing signals, and also enables
each pi to have a static IP address.

To set a static IP, we'll have to edit the `/etc/dhcpcd.conf` file, but first we'll
need to know the router's IP address. To find it, type in `netstat -nr`, the router's
IP will be listed under `Gateway`, it should look something like the following.

    Destination     Gateway         Genmaks         Flag    MSS Window  irtt Iface
    0.0.0.0         192.168.X.X     0.0.0.0         UG      0 0         0 wlan0
    192.168.1.0     0.0.0.0         255.255.255.0   U       0 0         0 wlan0

Then type on `sudo nano /etc/dhcpcd.conf` to edit the `dhcpdc.conf` file, at the bottom, add the following.

    interface wlan0

    static ip_address=192.168.1.10/24       #put your desired IP address here, with the /24 after it
    static routers=192.168.X.X              #put your router's IP address here, in place of the 192.168.X.X
    static domain_name_servers=192.168.X.X  #same IP address as the above line

Reboot to make sure that your IP is to what you set it, and you're still on the network, then you should be good to go.

This project is using two pis, `agnes` & `ethel`.

`agnes` will be set to 192.168.1.10, and `ethel` will be set to 192.168.1.20.

### rc.local

The last step, is to tell your pi to boot the ChucK script on boot, and this can
but done by editing your `etc/rc.local` file.

This is the scariest part, because if you forget to tell your program to run in the
background, then your pi will be locked out. Generally on a command line, you tell
a program to run in the background by placing an `&` after the command. We'll add the
following line to our `rc.local`.

    chuck /pi/home/git/pi-distortion-product/pi/run-pi.ck &

Now the pis should start the script on boot.
