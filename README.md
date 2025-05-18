# Linux-Dev-Setup

Linux Setup Guide for Development & Security

This comprehensive guide covers setting up Linux environments for various development and security purposes:

    Mobile App Development
    Web Development
    Database Management
    Cybersecurity

Table of Contents

    General System Setup
    Best Linux Kernels for Development
    Mobile App Development
    Web Development
    Database Management
    Apache2 Web Server Setup
    Laravel Installation & Setup
    IDE and Text Editors
    Cybersecurity Tools

General System Setup
Update and Upgrade System

bash

sudo apt update
sudo apt upgrade -y

Essential Development Tools

bash

sudo apt install build-essential git curl wget unzip npm nodejs vim -y

SSH Setup

bash

sudo apt install openssh-server -y
sudo systemctl enable ssh
sudo systemctl start ssh

Terminal Improvements

bash

sudo apt install zsh -y
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

Best Linux Kernels for Development

Selecting the right kernel can significantly improve performance for development and multitasking. Here are some recommended kernels for development work:
Liquorix Kernel

A desktop-focused kernel with optimizations for responsiveness and real-time performance, ideal for development workstations.

bash

# Add PPA repository
sudo add-apt-repository ppa:damentz/liquorix
sudo apt update
sudo apt install linux-image-liquorix-amd64 linux-headers-liquorix-amd64 -y

Xanmod Kernel

Offers improved system responsiveness and reduced latency, optimized for multitasking and demanding workloads.

bash

# Add repository
echo 'deb http://deb.xanmod.org releases main' | sudo tee /etc/apt/sources.list.d/xanmod-kernel.list
wget -qO - https://dl.xanmod.org/gpg.key | sudo apt-key add -
sudo apt update
sudo apt install linux-xanmod -y

Zen Kernel

A kernel that prioritizes desktop and workstation performance, with optimizations for multitasking and responsiveness.

bash

# For Ubuntu-based systems
sudo add-apt-repository ppa:damentz/zen-kernel
sudo apt update
sudo apt install linux-image-zen linux-headers-zen -y

Ubuntu Low-Latency Kernel

An official Ubuntu kernel variant optimized for low-latency operations.

bash

sudo apt install linux-image-lowlatency linux-headers-lowlatency -y

Installing a Custom Kernel on Arch-based Systems

For Arch Linux users, you can use the package manager:

bash

# For Zen kernel
sudo pacman -S linux-zen linux-zen-headers

# For Xanmod kernel
yay -S linux-xanmod linux-xanmod-headers

After Installing a New Kernel

Update GRUB to include the new kernel in the boot menu:

bash

sudo update-grub

Reboot your system to use the new kernel:

bash

sudo reboot

Check which kernel you're currently using:

bash

uname -r

Kernel Tweaking for Development

Adjust swappiness for better performance:

bash

# Temporarily set swappiness to 10 (less swapping, better for systems with good RAM)
sudo sysctl vm.swappiness=10

# To make it permanent
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf

Increase file watching limits (useful for Node.js development):

bash

echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

Laravel Installation & Setup

Laravel is a popular PHP framework for web application development. Here's how to set it up on your Linux system:
Prerequisites

Ensure PHP and Composer are installed:

bash

# Install PHP and required extensions
sudo apt update
sudo apt install php php-cli php-common php-mbstring php-xml php-zip php-curl php-mysql php-pgsql php-sqlite3 php-gd unzip -y

# Install Composer globally if not already installed
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer
sudo chmod +x /usr/local/bin/composer

Install Laravel via Composer

bash

# Install Laravel installer globally
composer global require laravel/installer

# Add Composer bin directory to PATH (add this to your .bashrc or .zshrc)
echo 'export PATH="$PATH:$HOME/.config/composer/vendor/bin"' >> ~/.bashrc
source ~/.bashrc

# OR if you use ZSH
# echo 'export PATH="$PATH:$HOME/.config/composer/vendor/bin"' >> ~/.zshrc
# source ~/.zshrc

Create a New Laravel Project

bash

# Using Laravel installer
laravel new my-project

# OR using Composer directly
composer create-project --prefer-dist laravel/laravel my-project

Configure Environment

bash

# Navigate to project directory
cd my-project

# Copy environment file
cp .env.example .env

# Generate application key
php artisan key:generate

Configure Database Connection

Edit the .env file to set up database connection:

bash

nano .env

Change the following settings:

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=your_db_username
DB_PASSWORD=your_db_password

Run Migrations

bash

php artisan migrate

Start Development Server

bash

php artisan serve

This will start the development server at http://localhost:8000
Configure Apache for Laravel

Create a virtual host configuration:

bash

sudo nano /etc/apache2/sites-available/laravel.conf

Add the following:

apache

<VirtualHost *:80>
    ServerName laravel.local
    ServerAlias www.laravel.local
    DocumentRoot /path/to/your/laravel/public
    
    <Directory /path/to/your/laravel/public>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
    
    ErrorLog ${APACHE_LOG_DIR}/laravel_error.log
    CustomLog ${APACHE_LOG_DIR}/laravel_access.log combined
</VirtualHost>

Enable the site and reload Apache:

bash

sudo a2ensite laravel.conf
sudo systemctl reload apache2

Add domain to hosts file:

bash

sudo nano /etc/hosts

Add this line:

127.0.0.1   laravel.local www.laravel.local

Setting Proper Permissions

bash

# Set ownership
sudo chown -R $USER:www-data /path/to/your/laravel

# Set directory permissions
sudo find /path/to/your/laravel -type d -exec chmod 755 {} \;

# Set file permissions
sudo find /path/to/your/laravel -type f -exec chmod 644 {} \;

# Make storage and bootstrap/cache writable
sudo chmod -R 775 /path/to/your/laravel/storage
sudo chmod -R 775 /path/to/your/laravel/bootstrap/cache

Laravel with Nginx

If you prefer Nginx over Apache:

bash

# Install Nginx
sudo apt install nginx -y

# Configure site
sudo nano /etc/nginx/sites-available/laravel

Add the following configuration:

nginx

server {
    listen 80;
    server_name laravel.local www.laravel.local;
    root /path/to/your/laravel/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";

    index index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}

Enable the site and reload Nginx:

bash

sudo ln -s /etc/nginx/sites-available/laravel /etc/nginx/sites-enabled/
sudo systemctl reload nginx

Laravel Scheduler Setup

Add Laravel's scheduler to crontab:

bash

crontab -e

Add this line:

* * * * * cd /path/to/your/laravel && php artisan schedule:run >> /dev/null 2>&1

Install Laravel Valet (Optional)

Laravel Valet provides a fast development environment:

bash

# Install required packages
sudo apt install network-manager libnss3-tools jq xsel -y

# Install Valet
composer global require cpriego/valet-linux
valet install

Common Laravel Commands

bash

# Create a controller
php artisan make:controller UserController

# Create a model with migration
php artisan make:model Post -m

# Create a migration
php artisan make:migration create_articles_table

# Create a resource controller
php artisan make:controller ProductController --resource

# Create a middleware
php artisan make:middleware CheckAge

# Run tests
php artisan test

# Clear various caches
php artisan cache:clear
php artisan config:clear
php artisan route:clear
php artisan view:clear

# Optimize for production
php artisan optimize

bash

sudo apt update
sudo apt install apache2 -y

Enable Required Modules

bash

sudo a2enmod rewrite
sudo a2enmod ssl
sudo a2enmod headers
sudo a2enmod proxy
sudo a2enmod proxy_http

Configure Apache2 Security

Edit the main configuration file:

bash

sudo nano /etc/apache2/conf-available/security.conf

Recommended security settings:

apache

# Hide Apache version
ServerTokens Prod
ServerSignature Off

# Disable directory listing
<Directory />
    Options -Indexes
</Directory>

# Disable CGI execution
<Directory /usr/lib/cgi-bin>
    Options -ExecCGI
    AddHandler cgi-script .cgi .pl .py
</Directory>

Create a Virtual Host

bash

sudo nano /etc/apache2/sites-available/myproject.conf

Basic virtual host configuration:

apache

<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    ServerName myproject.local
    ServerAlias www.myproject.local
    DocumentRoot /var/www/myproject/public_html
    
    <Directory /var/www/myproject/public_html>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
    
    ErrorLog ${APACHE_LOG_DIR}/myproject_error.log
    CustomLog ${APACHE_LOG_DIR}/myproject_access.log combined
</VirtualHost>

Create the project directory:

bash

sudo mkdir -p /var/www/myproject/public_html
sudo chown -R $USER:$USER /var/www/myproject
sudo chmod -R 755 /var/www/myproject

Create a test index file:

bash

echo '<html><head><title>Test Site</title></head><body><h1>Success! Your virtual host is working!</h1></body></html>' > /var/www/myproject/public_html/index.html

Enable the Virtual Host

bash

sudo a2ensite myproject.conf
sudo systemctl reload apache2

Setup Local Domain (Optional)

Add the domain to your hosts file:

bash

sudo nano /etc/hosts

Add this line:

127.0.0.1   myproject.local www.myproject.local

SSL Configuration (Optional)

Create a self-signed certificate for development:

bash

sudo mkdir -p /etc/apache2/ssl
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/apache2/ssl/myproject.key -out /etc/apache2/ssl/myproject.crt

Create an SSL virtual host:

bash

sudo nano /etc/apache2/sites-available/myproject-ssl.conf

Add the following configuration:

apache

<VirtualHost *:443>
    ServerAdmin webmaster@localhost
    ServerName myproject.local
    ServerAlias www.myproject.local
    DocumentRoot /var/www/myproject/public_html
    
    <Directory /var/www/myproject/public_html>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
    
    ErrorLog ${APACHE_LOG_DIR}/myproject_error.log
    CustomLog ${APACHE_LOG_DIR}/myproject_access.log combined
    
    SSLEngine on
    SSLCertificateFile /etc/apache2/ssl/myproject.crt
    SSLCertificateKeyFile /etc/apache2/ssl/myproject.key
    
    <FilesMatch "\.(cgi|shtml|phtml|php)$">
        SSLOptions +StdEnvVars
    </FilesMatch>
    
    BrowserMatch "MSIE [2-6]" nokeepalive ssl-unclean-shutdown downgrade-1.0 force-response-1.0
</VirtualHost>

Enable SSL site:

bash

sudo a2ensite myproject-ssl.conf
sudo systemctl reload apache2

Apache Performance Tuning

Edit the mpm_prefork.conf file for better performance:

bash

sudo nano /etc/apache2/mods-available/mpm_prefork.conf

Optimized configuration for development server:

apache

<IfModule mpm_prefork_module>
    StartServers             5
    MinSpareServers          5
    MaxSpareServers         10
    MaxRequestWorkers       150
    MaxConnectionsPerChild   0
</IfModule>

Common Apache Commands

bash

# Start Apache
sudo systemctl start apache2

# Stop Apache
sudo systemctl stop apache2

# Restart Apache
sudo systemctl restart apache2

# Reload configuration (without stopping)
sudo systemctl reload apache2

# Check Apache status
sudo systemctl status apache2

# Check configuration syntax
sudo apache2ctl configtest

# Enable site
sudo a2ensite site_name.conf

# Disable site
sudo a2dissite site_name.conf

# Enable module
sudo a2enmod module_name

# Disable module
sudo a2dismod module_name

Troubleshooting Apache

Check error logs:

bash

sudo tail -f /var/log/apache2/error.log

Check access logs:

bash

sudo tail -f /var/log/apache2/access.log

Fix permissions issues:

bash

sudo chown -R www-data:www-data /var/www/myproject
sudo find /var/www/myproject -type d -exec chmod 755 {} \;
sudo find /var/www/myproject -type f -exec chmod 644 {} \;

Android Development
Install JDK

bash

sudo apt install openjdk-17-jdk -y

Install Android Studio

bash

sudo snap install android-studio --classic

Setting up Environment Variables (add to ~/.bashrc or ~/.zshrc)

bash

export ANDROID_HOME=$HOME/Android/Sdk
export PATH=$PATH:$ANDROID_HOME/emulator
export PATH=$PATH:$ANDROID_HOME/tools
export PATH=$PATH:$ANDROID_HOME/tools/bin
export PATH=$PATH:$ANDROID_HOME/platform-tools

Flutter Development
Install Flutter

bash

sudo snap install flutter --classic
flutter doctor

Install Additional Dependencies

bash

sudo apt install clang cmake ninja-build pkg-config libgtk-3-dev -y

React Native Development
Install Node.js and npm

bash

curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs

Install React Native CLI

bash

sudo npm install -g react-native-cli

Web Development
Frontend Development Tools
Install Node Version Manager (NVM)

bash

curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
source ~/.bashrc  # or source ~/.zshrc if using zsh
nvm install node  # installs latest version
nvm install --lts  # installs LTS version

Install Yarn

bash

npm install -g yarn

Install Frontend Frameworks/Tools

bash

# Angular
npm install -g @angular/cli

# Vue
npm install -g @vue/cli

# React (Create React App)
npm install -g create-react-app

Backend Development Tools
Install Python and pip

bash

sudo apt install python3 python3-pip python3-venv -y

Install Django and Flask

bash

pip3 install django flask

Install Docker

bash

sudo apt install docker.io docker-compose -y
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker $USER

Install PHP

bash

sudo apt install php php-cli php-fpm php-json php-common php-mysql php-zip php-gd php-mbstring php-curl php-xml php-pear php-bcmath -y

Install Composer (PHP)

bash

curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer

Database Management
MySQL/MariaDB

bash

sudo apt install mysql-server -y
sudo systemctl enable mysql
sudo systemctl start mysql
sudo mysql_secure_installation

PostgreSQL

bash

sudo apt install postgresql postgresql-contrib -y
sudo systemctl enable postgresql
sudo systemctl start postgresql

MongoDB

bash

# Import the public key
curl -fsSL https://www.mongodb.org/static/pgp/server-6.0.asc | sudo apt-key add -

# Create list file for MongoDB
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu $(lsb_release -cs)/mongodb-org/6.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list

# Update package lists
sudo apt update

# Install MongoDB
sudo apt install mongodb-org -y
sudo systemctl enable mongod
sudo systemctl start mongod

Redis

bash

sudo apt install redis-server -y
sudo systemctl enable redis-server
sudo systemctl start redis-server

Database Management Tools

bash

# MySQL Workbench
sudo apt install mysql-workbench -y

# pgAdmin for PostgreSQL
sudo apt install pgadmin4 -y

# DBeaver (Universal DB tool)
sudo snap install dbeaver-ce

Cybersecurity Tools
Basic Security Tools

bash

sudo apt install nmap wireshark tcpdump aircrack-ng netcat-openbsd -y

Penetration Testing Tools
Install Metasploit Framework

bash

curl https://raw.githubusercontent.com/rapid7/metasploit-omnibus/master/config/templates/metasploit-framework-wrappers/msfupdate.erb > msfinstall
chmod 755 msfinstall
./msfinstall

Install OWASP ZAP

bash

sudo snap install zaproxy --classic

Install Burp Suite Community Edition

bash

sudo apt install burpsuite -y

Forensics Tools

bash

sudo apt install autopsy sleuthkit foremost scalpel testdisk ddrescue -y

Networking and Monitoring

bash

sudo apt install tshark tcpdump iftop htop netstat-nat iptraf -y

Vulnerability Scanning

bash

# Install OpenVAS
sudo apt install openvas -y
sudo gvm-setup
sudo gvm-start

Additional Security Tools

bash

# Install John the Ripper
sudo apt install john -y

# Install Hydra
sudo apt install hydra -y

# Install Nikto
sudo apt install nikto -y

# Install SQLMap
sudo apt install sqlmap -y

Configure Firewall (UFW)

bash

sudo apt install ufw -y
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw allow http
sudo ufw allow https
sudo ufw enable

Virtualization
VirtualBox

bash

sudo apt install virtualbox -y

KVM/QEMU

bash

sudo apt install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virt-manager -y
sudo systemctl enable libvirtd
sudo systemctl start libvirtd
sudo usermod -aG libvirt $USER
sudo usermod -aG kvm $USER

Version Control
Git Configuration

bash

git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

Generate SSH Key for GitHub/GitLab

bash

ssh-keygen -t ed25519 -C "your.email@example.com"
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
cat ~/.ssh/id_ed25519.pub

IDE and Text Editors
Visual Studio Code

bash

sudo snap install code --classic

Cursor IDE

Cursor is an AI-first code editor built on top of VS Code:

bash

# Method 1: Using AppImage
# Download the AppImage
wget https://download.cursor.sh/linux/appImage/x64
mv x64 cursor-linux.AppImage
chmod +x cursor-linux.AppImage

# Create desktop entry
mkdir -p ~/.local/share/applications
cat > ~/.local/share/applications/cursor.desktop << EOL
[Desktop Entry]
Name=Cursor
Comment=AI-first code editor
Exec=/path/to/cursor-linux.AppImage
Icon=cursor
Terminal=false
Type=Application
Categories=Development;IDE;
EOL

# Method 2: Using .deb package
# Download the .deb package
wget https://cursor.sh/download/linux-deb
sudo dpkg -i linux-deb
sudo apt install -f

# Method 3: Using the snap package
sudo snap install cursor --classic

Configuration for Cursor IDE

Create a configuration directory and file:

bash

mkdir -p ~/.config/cursor
nano ~/.config/cursor/settings.json

Basic settings to start with:

json

{
    "editor.fontFamily": "JetBrains Mono, 'Droid Sans Mono', monospace",
    "editor.fontSize": 14,
    "editor.lineHeight": 1.5,
    "workbench.colorTheme": "Default Dark+",
    "terminal.integrated.shell.linux": "/bin/bash",
    "editor.formatOnSave": true,
    "workbench.startupEditor": "newUntitledFile",
    "explorer.confirmDelete": false,
    "editor.tabSize": 2,
    "terminal.integrated.fontFamily": "monospace"
}

Install Recommended Extensions for Development

Using the command line with Cursor:

bash

# For JavaScript/TypeScript development
cursor --install-extension dbaeumer.vscode-eslint
cursor --install-extension esbenp.prettier-vscode

# For Python development
cursor --install-extension ms-python.python
cursor --install-extension ms-python.vscode-pylance

# For PHP/Laravel development
cursor --install-extension bmewburn.vscode-intelephense-client
cursor --install-extension onecentlin.laravel-blade

# For Docker support
cursor --install-extension ms-azuretools.vscode-docker

# For Git integration
cursor --install-extension eamodio.gitlens

# For Database management
cursor --install-extension mtxr.sqltools

JetBrains Toolbox

bash

curl -fsSL https://raw.githubusercontent.com/nagygergo/jetbrains-toolbox-install/master/jetbrains-toolbox.sh | bash

Sublime Text

bash

wget -qO - https://download.sublimetext.com/sublimehq-pub.gpg | sudo apt-key add -
echo "deb https://download.sublimetext.com/ apt/stable/" | sudo tee /etc/apt/sources.list.d/sublime-text.list
sudo apt update
sudo apt install sublime-text -y

Vim Configuration for Development

Install Vim with enhanced features:

bash

sudo apt install vim-gtk3 -y

Create a basic .vimrc configuration:

bash

cat > ~/.vimrc << EOL
syntax on
set number
set relativenumber
set tabstop=4
set shiftwidth=4
set expandtab
set smartindent
set incsearch
set hlsearch
set showmatch
set ignorecase
set smartcase
set cursorline
set colorcolumn=80
set scrolloff=8
set signcolumn=yes

" Plugin management with vim-plug
call plug#begin('~/.vim/plugged')
Plug 'tpope/vim-sensible'
Plug 'tpope/vim-commentary'
Plug 'tpope/vim-surround'
Plug 'tpope/vim-fugitive'
Plug 'junegunn/fzf', { 'do': { -> fzf#install() } }
Plug 'junegunn/fzf.vim'
Plug 'preservim/nerdtree'
Plug 'vim-airline/vim-airline'
Plug 'vim-airline/vim-airline-themes'
Plug 'neoclide/coc.nvim', {'branch': 'release'}
Plug 'morhetz/gruvbox'
Plug 'jiangmiao/auto-pairs'
call plug#end()

" Color scheme
colorscheme gruvbox
set background=dark

" Key mappings
let mapleader = " "
nnoremap <leader>fe :NERDTreeToggle<CR>
nnoremap <leader>ff :Files<CR>
nnoremap <leader>fg :Rg<CR>
nnoremap <leader>fb :Buffers<CR>
nnoremap <C-h> <C-w>h
nnoremap <C-j> <C-w>j
nnoremap <C-k> <C-w>k
nnoremap <C-l> <C-w>l
EOL

Install vim-plug (plugin manager):

bash

curl -fLo ~/.vim/autoload/plug.vim --create-dirs \
    https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim

Install plugins:

bash

vim +PlugInstall +qall

Neovim for Development

bash

# Install Neovim
sudo apt install neovim -y

# Create configuration directory
mkdir -p ~/.config/nvim

# Create init.vim
cat > ~/.config/nvim/init.vim << EOL
" Basic settings
set number
set relativenumber
set tabstop=4
set shiftwidth=4
set expandtab
set smartindent
set termguicolors
set incsearch
set hlsearch
set ignorecase
set smartcase
set cursorline
set colorcolumn=80
set scrolloff=8
set signcolumn=yes

" Plugins with vim-plug
call plug#begin('~/.local/share/nvim/plugged')
Plug 'neovim/nvim-lspconfig'
Plug 'hrsh7th/nvim-cmp'
Plug 'hrsh7th/cmp-nvim-lsp'
Plug 'hrsh7th/cmp-buffer'
Plug 'hrsh7th/cmp-path'
Plug 'L3MON4D3/LuaSnip'
Plug 'saadparwaiz1/cmp_luasnip'
Plug 'nvim-lua/plenary.nvim'
Plug 'nvim-telescope/telescope.nvim'
Plug 'nvim-treesitter/nvim-treesitter', {'do': ':TSUpdate'}
Plug 'kyazdani42/nvim-web-devicons'
Plug 'kyazdani42/nvim-tree.lua'
Plug 'folke/tokyonight.nvim'
Plug 'windwp/nvim-autopairs'
Plug 'numToStr/Comment.nvim'
Plug 'nvim-lualine/lualine.nvim'
Plug 'lewis6991/gitsigns.nvim'
call plug#end()

" Color scheme
colorscheme tokyonight

" Key mappings
let mapleader = " "
nnoremap <leader>ff <cmd>Telescope find_files<cr>
nnoremap <leader>fg <cmd>Telescope live_grep<cr>
nnoremap <leader>fb <cmd>Telescope buffers<cr>
nnoremap <leader>fh <cmd>Telescope help_tags<cr>
nnoremap <leader>e <cmd>NvimTreeToggle<cr>
EOL

Install Neovim plugin manager:

bash

sh -c 'curl -fLo "${XDG_DATA_HOME:-$HOME/.local/share}"/nvim/site/autoload/plug.vim --create-dirs \
       https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim'

Install plugins:

bash

nvim +PlugInstall +qall

Containerization and Orchestration
Install Docker (if not already installed)

bash

sudo apt install docker.io -y
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker $USER

Install Docker Compose

bash

sudo apt install docker-compose -y

Install Kubernetes Tools

bash

sudo snap install kubectl --classic
sudo snap install helm --classic

Install Minikube (Local Kubernetes)

bash

curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

Troubleshooting
Common Issues and Solutions

    Package Installation Failures
        Try: sudo apt clean && sudo apt update
    Permission Issues
        For Docker: sudo usermod -aG docker $USER (log out and log back in)
        For Android development: sudo chown -R $USER:$USER ~/Android
    Java Version Conflicts
        Install alternatives: sudo update-alternatives --config java
    Node.js Version Issues
        Use NVM to switch versions: nvm use 14 or nvm use 16
    Database Connection Issues
        Check service status: sudo systemctl status mysql
        Allow remote connections: Edit configuration files in /etc/mysql/mysql.conf.d/
    Missing Dependencies
        For common development libraries: sudo apt install libssl-dev libffi-dev

Maintenance
System Cleanup

bash

sudo apt autoremove -y
sudo apt autoclean

Update Development Tools

bash

# Node.js packages
npm update -g

# Python packages
pip3 list --outdated | cut -d ' ' -f1 | xargs -n1 pip3 install -U

# Flutter
flutter upgrade

Feel free to customize this guide according to your specific needs and Linux distribution. This setup is primarily based on Ubuntu/Debian-based systems, but most commands can be adapted for other distributions by replacing package managers (e.g., apt with dnf for Fedora or pacman for Arch Linux).
