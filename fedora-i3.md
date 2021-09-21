## Minimal Fedora Instalation with i3wm and Ansible
 

This is the same tutorial for sway but using i3wm, rembember that i3wm uses X11 so this playbook handle all x11 dependencies instead wayland.

Download the fedora Anywhere ISO and burn into a CD rom or USB.

In the installation steps select minimal installation and Common NetworkManager Submodules group for network wireless support.

After installation install ansible and check the following playbook:

~~~yaml
setup-playbook.yaml
# ansible-playbook ~/.config/setup-playbook.yaml -b --ask-become-pass
---
- name: Setup laptop config
  hosts: localhost
  tasks:
  - name: Enable Fusion repositories
    dnf:
      name:
        - 'https://download1.rpmfusion.org/free/fedora/rpmfusion-free-release-34.noarch.rpm'
      state: latest
      disable_gpg_check: true
  - name: Upgrade all packages
    dnf:
      name: "*"
      state: latest
  - name: Install UI software
    dnf:
      name: 
        - setxkbmap
        - i3
        - xorg-x11-server-Xorg
        - dbus-x11
        - notification-daemon
        - xss-lock
        - numlockx
        - picom
        - py3status
        - rofi
        - clipman
        - blueberry
        - lxpolkit
        - htop
        - ranger
        - bashmount
        - pavucontrol
        - zip
        - unzip
        - p7zip
        - p7zip-plugins
        - flatpak
        - mediawriter
        - neovim
        - liberation-fonts
        - liberation-sans-fonts
        - light
        - dunst
        - arandr
        - gnome-themes-extra
        - candy-icon-theme
        - pipewire
        - pipewire0.2-libs
        - pulseaudio-utils
        - alsa-utils
        - firefox
        - iwl7260-firmware
        - NetworkManager-wifi
        - NetworkManager-tui
        - terminus-fonts
        - rbanffy-3270-fonts
        - bash-completion
        - lxterminal
        - ffmpeg
        - flameshot
        - ncdu
        - atool
        - cronie
        - imv
        - libva-intel-driver
        - upower
        - feh
        - wget
      state: latest
  - name: Install Development software
    dnf:
      name:
        - podman
        - skopeo
        - buildah
        - toolbox
        - '@virtualization'
      state: latest
  - name: flatpak add flathub repo
    become: yes
    ansible.builtin.shell: flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
  - name: flatpak install software
    become: yes
    become_user: "user"
    ansible.builtin.shell: flatpak install -y --noninteractive --or-update flathub {{item}}
    with_items:
      - org.libreoffice.LibreOffice 
      - com.vscodium.codium 
      - io.mpv.Mpv
      - com.jetbrains.IntelliJ-IDEA-Community 
      - org.gimp.GIMP 
      - org.filezillaproject.Filezilla
      - com.github.tchx84.Flatseal
      - net.cozic.joplin_desktop
  - name: flatpak install superuser dependencies
    become: yes
    ansible.builtin.shell: flatpak install -y --noninteractive --or-update flathub {{item}}
    with_items:
      - org.gtk.Gtk3theme.Adwaita-dark
  - name: install ubuntu font family
    become: yes
    become_user: "user"
    ansible.builtin.unarchive:
      src: https://assets.ubuntu.com/v1/0cef8205-ubuntu-font-family-0.83.zip
      dest: ~/.fonts
      remote_src: yes
  - name: install oh my bash
    become: yes
    become_user: "user"
    ansible.builtin.shell: bash -c "$(curl -fsSL https://raw.githubusercontent.com/ohmybash/oh-my-bash/master/tools/install.sh)"
---
~~~

you can customize according your needs, this playbook manages flatpak software, container tools and several fonts and themes as well.
run the playbook with the user permissions to install software

    ansible-playbook ~/.config/setup-playbook.yaml -b --ask-become-pass
