## Minimal Fedora Instalation with sway and Ansible
 

If you want to install fedora with a minimal desktop configuration, swaywm is a desktop environment light and useful.

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
        - sway
        - waybar
        - wofi
        - clipman
        - blueberry
        - lxpolkit
        - htop
        - ranger
        - bashmount
        - swaylock
        - pavucontrol
        - zip
        - unzip
        - p7zip
        - p7zip-plugins  
        - flatpak
        - mediawriter
        - vim
        - liberation-fonts
        - liberation-sans-fonts
        - light
        - mako
        - wdisplays
        - wlogout
        - gnome-themes-extra
        - candy-icon-theme
        - pipewire
        - pulseaudio-utils
        - alsa-utils
        - firefox
        - iwl7260-firmware
        - NetworkManager-wifi
        - NetworkManager-tui
        - terminus-fonts
        - rbanffy-3270-fonts
        - bash-completion
        - kitty
        - ffmpeg
        - swappy
        - ncdu
        - atool
        - libva-intel-driver
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
    become_user: "jet"
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
    ansible.builtin.unarchive:
      src: https://assets.ubuntu.com/v1/0cef8205-ubuntu-font-family-0.83.zip
      dest: ~/.fonts
      remote_src: yes
~~~
