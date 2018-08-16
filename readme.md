# Install debian on a laptop

### TODO: cursor;

if you're using a atheros WiFi card, you have to use a non-free debian netiso, which can be found at https://cdimage.debian.org/cdimage/unofficial/non-free/cd-including-firmware/, recommendations are testing or sid.

here my set-up is XPS15(9570), debian non-free xfce sid, windows10 1803.

## 0. Works on the windows side

1. change BIOS setting to no"secure boot" and sata option on "ACHI"(not RAID) [1]

   a. Right-click the Windows Start Menu. Choose **Command Prompt (Admin)**.

   b. Type this command and press ENTER: **bcdedit /set {current} safeboot minimal**, If this command does not work for you, try **bcdedit /set safeboot minimal**

   c. Restart the computer and enter BIOS Setup, Change the SATA Operation mode to AHCI from either IDE or RAID, Save changes and exit Setup and Windows will automatically boot to Safe Mode

   d. Right-click the Windows Start Menu once more. Choose **Command Prompt** **(Admin)**.

   e. Type this command and press ENTER: **bcdedit /deletevalue {current} safeboot** or **bcdedit /deletevalue safeboot**

   f. Reboot once more and Windows will automatically start with AHCI drivers enabled

2. create a partion for linux using compress partition in disk management.

#### Reference

[1]. https://triplescomputers.com/blog/uncategorized/solution-switch-windows-10-from-raidide-to-ahci-operation/

## I.Install the system

1. not much to say, use English as default language ,select Xfce as desktop.

2. add own account into sudoers:

   ```bash
   su
   nano /etc/sudoers
   ```

3. change sudo to keep HOME variables[1]

   add this line to /etc/sudoers

   ```bash
   Defaults	env_keep+="HOME"
   ```

4. add useful tools:

   ```bash
   sudo apt install psmisc # this allows you to use killall
   sudo apt install chromium # personal perfers
   ```

5. After bootintg into GUI when asked about if use default configuration, select yes.

#### Reference

[1]. https://unix.stackexchange.com/questions/91384/how-is-sudo-set-to-not-change-home-in-ubuntu-and-how-to-disable-this-behavior/91572#91572?newreg=8fbbf9777edf477a831ee3e6aebc204c

## II.Add bluetooth support

1. install (bluetooth(the driver) -> pulseaudio(bluetooth GUI) -> pavcontrol(audio volume handler)), see[1]\[2].

   ```bash
   sudo apt install pulseaudio pulseaudio-utils pulseaudio-module-bluetooth bluetooth pavcontrol
   ```

2. right click on the top panel, in the pop-out menu select panel->add New Items, and add pulseaudio plugin.

3. enable bluetooth[1]

   ```
   sudo systemctl enable bluetooth
   sudo systemctl start bluetooth
   ```

4. edit configuration of pulseaudio:

   ```bash
   sudo nano /etc/pulse/default.pa
   ```

   add these lines:[3]

   ```bash
   # automatically switch to newly-connected devices
   load-module module-switch-on-connect
   ```

5. connect and trust audio handset, note to connect to "audio sinks" and in "audio profile" select "High Fidelity Playback(A2DP Sink)"[4], you can make this as default by: [5]

   ```bash
   sudo nano /etc/bluetooth/main.conf
   ```

   add these lines to [General] section:

   ```bash
   [General]
   # this will make device automatical connect in A2DP mode
   Disable=Headset
   ```

6. now, in pavcontrol (can be accessed by icon on right-top corner), you can select bluetooth device as output.

#### Reference

[1]. https://wiki.archlinux.org/index.php/Bluetooth

[2]. https://wiki.debian.org/BluetoothUser

[3]. https://wiki.archlinux.org/index.php/Bluetooth_headset#Headset_via_Bluez5.2FPulseAudio

[4]. https://askubuntu.com/questions/824404/bluetooth-speaker-connected-but-not-listed-in-sound-output

[5]. https://askubuntu.com/questions/319871/bluetooth-speaker-preferred-mode-high-fidelity-playback-a2dp-is-not-getting # not the accepted answer but the one from _Caumons_

## III.Configure trackpad

#### TODO: A macbook-rival mtrack configuration

#### Minimal set-up:

1. install the drive:

   ```bash
   sudo apt install xserver-xorg-input-libinput
   ```

2. copy configuration file to /etc:

   ```bash
   sudo cp /usr/share/X11/xorg.conf.d/40-libinput.conf /etc/X11/xorg.conf.d/
   ```

   edit it, in the "libinput touchpad catchall"section, add these lines:

   ```bash
   Section "InputClass"
   	...
   	Option "Tapping" "on" # use tapping as click
   	Option "NaturalScrolling" "True" # use natural scrolling
   	...
   EndSection
   ```

   for more detailed options see[1-3].

#### Best set-up:

1. install xf86-inputs-mtrack:[4]

   ```bash
   cd ~
   git clone https://github.com/p2rkw/xf86-input-mtrack.git
   cd xf86-input-mtrack
   sudo apt build-dep xserver-xorg-input-mtrack
   ./configure --with-xorg-module-dir=/usr/lib/xorg/modules
   make && sudo make install
   ```

2. use configuration file in repos examples folder, here I use dell-xps13-9333.conf

   ```bash
   sudo cp ~/xf86-input-mtrack/example/dell-xps13-9333.conf /etc/X11/xorg.conf.d/40-libinput.conf
   ```

3. some fine tune on the configuration file.

#### Reference

[1]. https://wiki.debian.org/SynapticsTouchpad

[2]. https://wiki.archlinux.org/index.php/Libinput

[3]. https://wiki.archlinux.org/index.php/Touchpad_Synaptics

[4]. https://github.com/p2rkw/xf86-input-mtrack

## IV. Install shadowsocks

0. see [1] for instructions of server configuration.

1. install shadowsocks :[1]

   ```bash
   sudo apt install shadowsocks-libev
   ```

2. install proxychains:[3]

   ```bash
   sudo apt install proxychains
   ```

3. config shadowsocks:[2]

   ```bash
   sudo nano /etc/shadowsocks-libev/config.json
   ```

   to

   ```bash
   {
       "server":"server ip",
       "server_port":443, 
       "local_port":1080,
       "password":"passwd",
       "timeout":60,
       "method":"aes-256-cfb",
       "mode":"tcp_and_udp",
       "fast_open":false,
       "plugin":"obfs-local",
       "plugin_opts":"obfs=http;obfs-host=www.baidu.com"
   }
   
   ```

   Then:

   ```bash
   sudo systemctl enable shadowsocks-libev-local@config
   sudo systemctl start shadowsocks-libev-local@config
   sudo systemctl status shadowsocks-libev-local@config
   ```

4. config proxychain:[3]

   ```bash
   sudo nano /etc/proxychains.conf
   ```

   At the ProxyList add these lines:

   ```bash
   socks5    127.0.0.1    1080
   ```

   these transform socks5 proxy to a http/https proxy.

   you can test these settings by:

   ```bash
   proxychain wget www.google.com
   ```

   This will download front-page of google for you.

5. config chromium to use shadowsocks[4]

   run chromium in proxy:

   ```bash
   proxychains chromium
   ```

   install **SwitchyOmega** from app store, or on GitHub:https://github.com/FelisCatus/SwitchyOmega/releases/, and you can use configuration file from wiki:https://github.com/FelisCatus/SwitchyOmega/wiki/GFWList. Then close chromium and run it from menu, remember to use SwitchyOmege at auto profile.

#### Reference

[1]. https://github.com/li012589/debianShadowsocksSetup

[2]. https://dreamcreator108.com/dreams/debian-stretch-shadowsocks-libev-client/

[3]. https://www.jianshu.com/p/589af998dde0

[4]. https://github.com/FelisCatus/SwitchyOmega/wiki/

## V. Emacs and Zsh

1. install

   ```bash
   sudo apt install emacs zsh
   ```

2. install oh-my-zsh, follow instructions from [1]

3. install powerline-fonts:

   ```bash
   sudo apt-get install fonts-powerline
   ```

4. edit ~/.zsh to use "agnoster" theme:

   change this line:

   ```bash
   ZSH_THEME="robbyrussell"
   ```

   to

   ```bash
   ZSH_THEME="agnoster"
   ```

   and add this line to stop zsh from showing your username:

   ```bash
   DEFAULT_USER="your username"
   ```

   Note, you have to change terminal setting to allow font to work.

5. install emacs configure file, follow instructions from [2]

#### Reference

[1]. https://github.com/robbyrussell/oh-my-zsh

[2]. https://github.com/redguardtoo/emacs.d

## VI. i3

### TODO: state bar, sound bar; dmenu; lock screen and desktop background

1. install

   ```bash
   sudo apt install i3 feh playerctl xbacklight
   ```

2. at login screen select i3, when logged in and when asked if create a default configuration file, select yes, and then select use win key as mod key.

3. add multimedia key:

   edit ~/.config/i3/config, and add these lines:[3,4]

   ```bash
   # Pulse Audio controls
   bindsym XF86AudioRaiseVolume exec --no-startup-id amixer -q set Master +5% #increase sound volume
   bindsym XF86AudioLowerVolume exec --no-startup-id amixer -q set Master -5% #decrease sound volume
   bindsym XF86AudioMute exec --no-startup-id amixer -q set Master toggle # mute sound
   
   # Sreen brightness controls
   bindsym XF86MonBrightnessUp exec xbacklight -inc 20 # increase screen brightness
   bindsym XF86MonBrightnessDown exec xbacklight -dec 20 # decrease screen brightness
   
   # Media player controls
   bindsym XF86AudioPlay exec playerctl play
   bindsym XF86AudioPause exec playerctl pause
   bindsym XF86AudioNext exec playerctl next
   bindsym XF86AudioPrev exec playerctl previous
   ```

   restart i3 to see the effect.

   if backlighting control not working, see if you have a `/sys/class/backlight`, if not, 

   ```bash
   sudo find /sys/ -type f -iname '*brightness*'
   ```

   you should see something like this:

   ```bash
   ....
   /sys/devices/pci0000:00/0000:00:02.0/drm/card0/card0-LVDS-1/intel_backlight/brightness
   ....
   ```

   link this file to `/sys/class/backlight`, restart x backlight should work now.

   if you have a `/sys/class/backlight` and it's a folder, or it's a file and not work. You have to edit `/etc/X11/xorg.conf`, above we create a folder to store separate file, so we create another file:

   ```bash
   sudo nano /etc/X11/xorg.conf.d/10-internalscreen.conf
   ```

   and put these inside:[1,2]

   ```bash
   Section "Device"
           Identifier  "Intel Graphics" 
           Driver      "intel"
           Option      "Backlight"  "intel_backlight"
   EndSection
   ```

4. install fonts from Apple

   ```bash
   git clone https://github.com/AppleDesignResources/SanFranciscoFont.git
   mkdir -p ~/.fonts/
   cp ./SanFranciscoFont/*.otf ~/.fonts/
   ```

   edit ~/.conf/i3/conf, change font

   ```bash
   font pango:monospace 8 
   ```

   to 

   ```bash
   font pango:SanFranciscoDisplay 10 # last number stays for font size
   ```

   and change GTK font using `xfce4-appearance-settings`

5. other fonts to install:

   https://github.com/supermarin/YosemiteSanFranciscoFont

   https://github.com/belluzj/fantasque-sans

   https://github.com/FortAwesome/Font-Awesome
   
6. install icon theme: https://github.com/PapirusDevelopmentTeam/papirus-icon-theme, 

```bash
sudo sh -c "echo 'deb http://ppa.launchpad.net/papirus/papirus/ubuntu xenial main' > /etc/apt/sources.list.d/papirus-ppa.list"

sudo apt-key adv --recv-keys --keyserver keyserver.ubuntu.com E58A9D36647CAE7F
sudo apt-get update
sudo apt-get install papirus-icon-theme
```

7. install GTK theme: https://github.com/nana-4/materia-theme
```bash
sudo apt install materia-gtk-theme
```

8. open `lxappearance`, select this icon theme and GTK theme.

6. install

   ```bash
   sudo apt install compton
   ```

   add this line to i3 config file, this enable transition and other effects,[5]

   ```bash
   exec compton
   ```
   And then create a config file for compton at `~/.config/compton.conf`[7]

   ```bash
   backend = "glx";
   vsync = "opengl-swc";
   
   shadow = true;
   no-dock-shadow = true;
   no-dnd-shadow = true;
   clear-shadow = true;
   
   shadow-radius = 10;
   shadow-offset-x = -5;
   shadow-offset-y = 0;
   shadow-opacity = 0.8;
   shadow-red = 0.11;
   shadow-green = 0.12;
   shadow-blue = 0.13;
   shadow-exclude = [
     "name = 'Notification'",
     "_GTK_FRAME_EXTENTS@:c",
     "class_g = 'i3-frame'",
     "_NET_WM_STATE@:32a *= '_NET_WM_STATE_HIDDEN'",
     "_NET_WM_STATE@:32a *= '_NET_WM_STATE_STICKY'",
     "!I3_FLOATING_WINDOW@:c"
   ];
   shadow-ignore-shaped = true;
   
   alpha-step = 0.06;
   blur-background = false;
   blur-background-fixed = true;
   blur-kern = "7x7box";
   blur-background-exclude = [
     "class_g = 'i3-frame'",
     "window_type = 'dock'",
     "window_type = 'desktop'",
     "_GTK_FRAME_EXTENTS@:c"
   ];
   
   # Duplicating the _NET_WM_STATE entries because compton cannot deal with atom arrays :-/
   opacity-rule = [
     "97:class_g = 'Termite' && !_NET_WM_STATE@:32a",
   
     "0:_NET_WM_STATE@[0]:32a = '_NET_WM_STATE_HIDDEN'",
     "0:_NET_WM_STATE@[1]:32a = '_NET_WM_STATE_HIDDEN'",
     "0:_NET_WM_STATE@[2]:32a = '_NET_WM_STATE_HIDDEN'",
     "0:_NET_WM_STATE@[3]:32a = '_NET_WM_STATE_HIDDEN'",
     "0:_NET_WM_STATE@[4]:32a = '_NET_WM_STATE_HIDDEN'",
   
     "90:_NET_WM_STATE@[0]:32a = '_NET_WM_STATE_STICKY'",
     "90:_NET_WM_STATE@[1]:32a = '_NET_WM_STATE_STICKY'",
     "90:_NET_WM_STATE@[2]:32a = '_NET_WM_STATE_STICKY'",
     "90:_NET_WM_STATE@[3]:32a = '_NET_WM_STATE_STICKY'",
     "90:_NET_WM_STATE@[4]:32a = '_NET_WM_STATE_STICKY'"
   ];
   
   fading = false;
   fade-delta = 7;
   fade-in-step = 0.05;
   fade-out-step = 0.05;
   fade-exclude = [];
   
   mark-wmwin-focused = true;
   mark-ovredir-focused = true;
   use-ewmh-active-win = true;
   detect-rounded-corners = true;
   detect-client-opacity = true;
   refresh-rate = 0;
   dbe = false;
   paint-on-overlay = true;
   glx-no-stencil = true;
   glx-copy-from-front = false;
   glx-swap-method = "undefined";
   sw-opti = true;
   unredir-if-possible = false;
   focus-exclude = [];
   detect-transient = true;
   detect-client-leader = true;
   invert-color-include = [];
   
   wintypes: {
       tooltip = { fade = true; shadow = false; opacity = 1.00; focus = true; };
   };
   ```

### Minimal settig: using xfce4's notification

7. test if notification works:[6]

   ```bash
   notify-send --icon=gtk-info Test "This is a test"
   ```

### Better setting: using dunst

7. run:
```bash
sudo apt remove xfce4-notifyd
sudo apt install dunst
```
test dunst:
```bash
notify-send 'Hello world!' 'This is an example notification.' --icon=dialog-information
notify-send 'Hello world!' 'This is an example notification.' -u low
notify-send 'Hello world!' 'This is an example notification.' -u normal
notify-send 'Hello world!' 'This is an example notification.' -u critical
```
configuration file can be found at `~/.config/dunst/dunstrc`, first
```bash
cp /usr/share/dunst/dunstrc ~/.config/dunst/dunstrc
```

8. add this line to i3 config to enable applet for network manager, bluetooth etc.

   ```bash
   exec --no-startup-id nm-applet
   exec --no-startup-id blueman-applet
   ```

#### Reference 

[1]. https://askubuntu.com/questions/715306/xbacklight-no-outputs-have-backlight-property-no-sys-class-backlight-folder

[2]. https://wiki.archlinux.org/index.php/backlight#xbacklight

[3]. https://faq.i3wm.org/question/3747/enabling-multimedia-keys.1.html

[4]. https://faq.i3wm.org/question/125/how-to-change-the-systems-volume.1.html

[5]. https://wiki.archlinux.org/index.php/compton

[6]. https://faq.i3wm.org/question/121/whats-a-good-notification-daemon-for-i3.1.html

[7]. https://github.com/Airblader/dotfiles-manjaro/blob/master/.compton.conf

## VII. Power saving

1. install 

   ```bash
   sudo apt install tlp tlp-rdw powertop powerstat
   sudo powertop --auto-tune
   ```

2. start powertop auto tune at boot:[1,2]

   create a file name `/etc/systemd/system/powertop.service`

   ```bash
   [Unit]
   Description=PowerTOP auto tune
   
   [Service]
   Type=idle
   Environment="TERM=dumb"
   ExecStart=/usr/sbin/powertop --auto-tune
   
   [Install]
   WantedBy=multi-user.target
   ```

   Then, 

   ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable powertop.service
   sudo systemctl start powertop.service
   ```

3. use  `systemctl`  to suspend or hibernate:

   ```bash
   systemctl suspend
   systemctl hibernate
   ```

4. test if `systemctl` support hybrid-sleep[3]

   ```bash
   systemctl hybrid-sleep
   ```

   (this will put debian to sleep at near zero power consumption, while almost instant start)

   now we purge default power manager from xfce4:

   ```bash
   sudo apt purge xfce4-power-manager
   ```

   edit `/etc/systemd/logind.conf`, and add these lines:

   ```bash
   HandleSuspendKey=hybrid-sleep
   HandleLidSwitch=hybrid-sleep
   ```

5. add screen lock function at resume

   add file at `/etc/systemd/system/screenlock.service`

   ```bash
   [Unit]
   Description=Lock the screen on resume from suspend
   
   [Service]
   User=YOURUSERNAME
   Type=forking
   Environment=DISPLAY=:0
   ExecStart=/usr/bin/xflock4 # or i3lock
   
   [Install]
   WantedBy=sleep.target
   WantedBy=suspend.target
   ```

   then 

   ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable screenlock.service
   ```
6. fix chromium slow down on newer CPUs, add file at `/etc/X11/xorg.conf.d/20-intel.conf` [4]
   ```bash
   Section "Device"
     Identifier  "Intel Graphics"
     Driver      "intel"
     Option      "TearFree" "true"
     Option      "AccelMethod"  "uxa"
   EndSection
   ```

#### Reference

[1]. https://blog.sleeplessbeastie.eu/2015/08/10/how-to-set-all-tunable-powertop-options-at-system-boot/

[2]. https://wiki.archlinux.org/index.php/powertop#Calibration_to_prevent_inaccurate_measurement

[3]. https://askubuntu.com/questions/145443/how-do-i-use-pm-suspend-hybrid-by-default-instead-of-pm-suspend/781957#781957

[4]. https://wiki.archlinux.org/index.php/intel_graphics#Disable_Vertical_Synchronization_.28VSYNC.29

## VIII. Chinese input
1. First, add Chinese to `/etc/locale.gen`, edit it and uncomment "zh_CN GB2312, zh_CN_GBK, zh_CN.UTF-8", then run:
```bash
sudo locale-gen
```

2. Change default CJK preference to first Simplified Chinese then whatever[3], this resolve the problem where some Chinese charaters being rendered as Japanese charaters,
create file at `/etc/fonts/conf.avail/64-language-selector-prefer.conf`, enter this:
```bash
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "fonts.dtd">
<fontconfig>
    <alias>
        <family>sans-serif</family>
        <prefer>
            <family>Noto Sans CJK SC</family>
            <family>Noto Sans CJK TC</family>
            <family>Noto Sans CJK JP</family>
        </prefer>
    </alias>
    <alias>
        <family>monospace</family>
        <prefer>
            <family>Noto Sans Mono CJK SC</family>
            <family>Noto Sans Mono CJK TC</family>
            <family>Noto Sans Mono CJK JP</family>
        </prefer>
    </alias>
</fontconfig>
```
Then
```bash
# if you have a /etc/fonts/conf.d folder
sudo ln -s /etc/fonts/conf.avail/64-language-selector-prefer.conf /etc/fonts/conf.d/64-language-selector-prefer.conf
sudo fc-cache -fv
sudo fc-match -s |grep 'Noto Sans CJK'
# last command should give you this:
# NotoSansCJK-Regular.ttc: "Noto Sans CJK SC" "Regular"
# (here SC means Simplified Chinese)
```

3. install

   ```bash
    sudo apt install ibus-rime librime-data-double-pinyin
    mkdir ~/.config/ibus/rime/ -p
    cp /usr/share/rime-data/luna_pinyin_simp.yaml ~/.config/ibus/rime/
    cp /usr/share/rime-data/double_pinyin_flypy.yaml ~/.config/ibus/rime/
   ```

4. use `im-config` to select `ibus` as IM, and use `ibus-config` to select font and size:

   ```bash
   im-config
   ibus-setup
   ```

5. edit `./.config/ibus/rime/default.custom.yaml`, add these lines:

   ```bash
   patch:
   	schema_list:
   		- schema: luna_pinyin_simp
   		- schema: double_pinyin_flypy
   	menu:
   		page_size: 9
   ```

6. reboot

7. win+space to select rime, to change this, use menu key on IM icon, in pop-up menu, select preference.

8. install `fontconfig-infinality` [2]

   ```bash
   echo "deb http://ppa.launchpad.net/no1wantdthisname/ppa/ubuntu precise main" | sudo tee /etc/apt/sources.list.d/infinality.list
   sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E985B27B
   sudo apt-get update
   sudo apt-get install fontconfig-infinality
   ```

   run

   ```bash
   sudo bash /etc/fonts/infinality/infctl.sh setstyle
   ```

   In `xfce4-appearance-settings` in `Font` tab,in `hinting` select Full.

#### Reference

[1]. https://github.com/rime/home/wiki/CustomizationGuide

[2]. https://gist.github.com/TheWaWaR/a0e7c6b8d6d149490a4a

[3]. https://wiki.archlinux.org/index.php/Localization/Simplified_Chinese_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#.E4.B8.AD.E6.96.87.E5.AD.97.E4.BD.93

## IX. Keyboard remap

### Globally

1. edit `/etc/default/keyboard`

   change `XKBOPTIONS` to 

   ```bash
   XKBOPTIONS="altwin:swap_lalt_lwin,ctrl:swapcaps"
   ```

   this exchange ctrl and caps lock,[2] and exchange left alt and left Win.[3]
   
### Match different devices

1. create `/etc/X11/xorg.conf.d/15-keyboard.conf`, enter this:[4-6]

```bash
Section "InputClass"
        Identifier "system-keyboard"
        MatchIsKeyboard "on"
        Option "XkbLayout" "us"
        Option "XkbModel" "pc104"
        Option "XkbOptions" "altwin:swap_lalt_lwin,ctrl:swapcaps"
EndSection

Section "InputClass"
        Identifier "Happy Hacking Keyboard"
        MatchIsKeyboard "on"
        MatchVendor "Topre_Corporation"

        Option "XkbLayout" "us"
EndSection
```

First one is common keyboard including the one internal in laptop, we swip ctrl and caps lock, left alt and left win. The second one is for HHKB, it use normal settings.

#### Reference

[1]. https://medium.com/@damko/a-simple-humble-but-comprehensive-guide-to-xkb-for-linux-6f1ad5e13450

[2]. https://askubuntu.com/questions/33774/how-do-i-remap-the-caps-lock-and-ctrl-keys

[3]. https://askubuntu.com/questions/29731/rebind-alt-key-to-win-using-setxkbmap

[4]. https://superuser.com/questions/75817/two-keyboards-on-one-computer-when-i-write-with-a-i-want-a-us-keyboard-layout

[5]. https://wiki.archlinux.org/index.php/Keyboard_configuration_in_Xorg

[6]. ftp://www.x.org/pub/X11R7.7-RC1/doc/xorg-docs/input/XKB-Config.html

## X. windows integration 



## ETC

```bash
sudo apt install tree dnsutils htop xclip
```
let firefox support Emacs-like hotkeys[1], edit `~/.config/gtk-3.0/settings.ini`, add this line:
```bash
# enable emacs key theme on gtk
gtk-key-theme-name=Emacs
```
And in terminal inter this:
```bash
gsettings set org.gnome.desktop.interface gtk-key-theme "Emacs"
```

#### Reference
[1]. https://www.emacswiki.org/emacs/Mozilla