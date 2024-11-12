FROM ubuntu:22.04

# Non-interactive installation mode
ENV DEBIAN_FRONTEND=noninteractive

# Update apt database
RUN apt update

# Install essentials
RUN apt install -y apt-utils software-properties-common apt-transport-https sudo \
    psmisc tmux nano wget curl telnet gnupg gdb git gitk autoconf locales gdebi \
    terminator meld dos2unix meshlab

RUN wget https://packages.microsoft.com/config/ubuntu/22.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb && \
    sudo dpkg -i packages-microsoft-prod.deb && \
    rm -f packages-microsoft-prod.deb

# install something
RUN apt install htop neofetch vim apt-transport-https -y && apt update && apt-get upgrade -y && \
    sudo apt-get install -y dotnet-sdk-8.0

# install Golang
RUN wget https://go.dev/dl/go1.23.3.linux-amd64.tar.gz && \
    sudo tar -xvf go1.23.3.linux-amd64.tar.gz && \
    sudo mv go /usr/local && \
    rm -rf *.tar.gz

# Set the locale
RUN locale-gen en_US.UTF-8

# Install graphics
RUN apt install -y xfce4 xfce4-goodies xserver-xorg-video-dummy xserver-xorg-legacy x11vnc && \
    apt remove -y xfce4-power-manager xfce4-screensaver light-locker && \
    apt autoremove -y && \
    sed -i 's/allowed_users=console/allowed_users=anybody/' /etc/X11/Xwrapper.config
COPY xorg.conf /etc/X11/xorg.conf
RUN dos2unix /etc/X11/xorg.conf

# Install browser
RUN wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb && \
    gdebi -n google-chrome-stable_current_amd64.deb && \
    rm google-chrome-stable_current_amd64.deb

# Install python
RUN apt install -y python3 python3-dev python3-pip python3-setuptools && \
    if [ ! -f "/usr/bin/python" ]; then ln -s /usr/bin/python3 /usr/bin/python; fi

# Install magic-wormhole to get things from one computer to another safely
RUN apt install -y magic-wormhole

# Install glog
RUN apt install -y libgoogle-glog-dev

# Install noVNC
RUN git clone https://github.com/novnc/noVNC.git /opt/novnc && \
    git clone https://github.com/novnc/websockify /opt/novnc/utils/websockify && \
    echo "<html><head><meta http-equiv=\"Refresh\" content=\"0; url=vnc.html?autoconnect=true&reconnect=true&reconnect_delay=1000&resize=scale&quality=9\"></head></html>" > /opt/novnc/index.html

# Set environmental variables
ENV DISPLAY=:1

# Create user gitpod
RUN useradd -l -u 33333 -G sudo -md /home/gitpod -s /bin/bash -p gitpod gitpod && \
    # passwordless sudo for users in the 'sudo' group
    sed -i.bkp -e 's/%sudo\s\+ALL=(ALL\(:ALL\)\?)\s\+ALL/%sudo ALL=NOPASSWD:ALL/g' /etc/sudoers

# Switch to gitpod user
USER gitpod

# Install informative git for bash
RUN git clone https://github.com/magicmonty/bash-git-prompt.git ~/.bash-git-prompt --depth=1

# Set up .bashrc and PATH for Go
WORKDIR /home/gitpod
RUN echo "GIT_PROMPT_ONLY_IN_REPO=1" >> ~/.bashrc && \
    echo "source \${HOME}/.bash-git-prompt/gitprompt.sh" >> ~/.bashrc && \
    echo "export GOROOT=/usr/local/go" >> ~/.bashrc && \
    echo "export GOPATH=\$HOME/go" >> ~/.bashrc && \
    echo "export PATH=\$GOPATH/bin:\$GOROOT/bin:\$HOME/.local/bin:\$PATH" >> ~/.bashrc && \
    echo "LD_LIBRARY_PATH=/usr/local/lib:\${LD_LIBRARY_PATH}" >> ~/.bashrc && \
    curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- --default-toolchain nightly -y

# Create the Desktop dir
RUN mkdir -p /home/gitpod/Desktop

# Switch back to root
USER root

# Set up script to launch graphics and vnc
COPY start-vnc-session.sh /usr/bin/start-vnc-session.sh
RUN chmod +x /usr/bin/start-vnc-session.sh && \
    dos2unix /usr/bin/start-vnc-session.sh

# Make sure specific dirs are owned by gitpod user
RUN mkdir /workspace && \
    chown -R gitpod.gitpod /workspace

# Manage ports
EXPOSE 5901 6080 10000/tcp 10000/udp

# Workaround for mesa-utils (https://launchpad.net/~kisak/+archive/ubuntu/kisak-mesa)
RUN add-apt-repository -y ppa:kisak/kisak-mesa && \
    apt update && \
    apt install -y aptitude && \
    aptitude full-upgrade -y

# Clean up unnecessary installation products
RUN rm -Rf /var/lib/apt/lists/*

# Ignore all files by creating a .gitignore
RUN echo "*" > /workspace/.gitignore

# Launch bash from /workspace
WORKDIR /workspace
CMD ["bash"]
