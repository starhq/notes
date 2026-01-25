
```SHELL
#!/usr/bin/env bash
# Description: Ubuntu/Debian Mirror Switcher and Environment Setup
# Author: starhq
# Date: $(date +%Y-%m-%d)

# ============================================================================== # STRICT MODE (Safety First) # ============================================================================== 
# -e: Exit immediately if a command fails 
# -u: Treat unset variables as an error 
# -o pipefail: Pipeline returns the status of the last command to fail
set -euo pipefail

#==========================================
# Configuration Section
#==========================================

readonly APP_USER="${APP_USER:-starhq}" 
readonly JAVA_ENABLED="${JAVA_ENABLED:-true}" 
readonly GOLANG_ENABLED="${GOLANG_ENABLED:-false}" 
readonly DOCKER_ENABLED="${DOCKER_ENABLED:-false}"
readonly DOCKER_COMPOSE_ENABLED="${DOCKER_COMPOSE_ENABLED:-false}"
readonly APP_PASSWORD="${APP_PASSWORD:-123456}"
readonly SSH_PORT="${SSH_PORT:-22}"
readonly JAVA_VERSION="${JAVA_VERSION:-default-jdk}"

readonly MIRROR_URL='https://mirrors.aliyun.com/'
readonly APP_NAME="app"
readonly APP_DIR="/opt/$APP_NAME"
readonly APP_BIN_DIR="$APP_DIR/bin"
readonly APP_CONFIG_DIR="$APP_DIR/config"
readonly APP_LIB_DIR="$APP_DIR/lib"
readonly APP_SCRIPTS_DIR="$APP_DIR/scripts"
readonly APP_LOG_DIR="/var/log/$APP_NAME"
readonly APP_PID_DIR="/var/run/$APP_NAME"
readonly APP_WORKING_DIR="/var/lib/$APP_NAME"
readonly PRIVATE_KEY="$HOME/.ssh/$APP_NAME/id_rsa"
readonly PUBLIC_KEY="$HOME/.ssh/$APP_NAME/id_rsa.pub"
readonly SERVER_IP="127.0.0.1"
readonly DEFAULT_JAR="template-0.0.1-SNAPSHOT.jar"

#==========================================
# Color Output Functions
#==========================================

color_output() { printf "\e[${1}m%s\e[0m\n" "${@:2}"; }
red() { color_output "31" "$@"; }
green() { color_output "32" "$@"; }
yellow() { color_output "33" "$@"; }
blue() { color_output "34" "$@"; }

#==========================================
# Error Handling
#==========================================

error_exit() {
    red "Error: $1" >&2
    exit "${2:-1}"
}

#==========================================
# Distribution Detection
#==========================================

detect_distro() {
	[[ -f /etc/os-release ]] || error_exit "Not a Debian-based system!"
    
    # shellcheck disable=SC1091
    source /etc/os-release
    
    case "$ID:$VERSION_ID" in
        ubuntu:*)
            CODENAME="$VERSION_CODENAME"
            ;;
        debian:12)
            CODENAME="bookworm"
            ;;
        debian:11)
            CODENAME="bullseye"
            ;;
        debian:10)
            CODENAME="buster"
            ;;
        debian:*)
            error_exit "Unsupported Debian version: $VERSION_ID"
            ;;
        *)
            error_exit "Unsupported distribution: $ID"
            ;;
    esac
    
    green "Detected system: ${ID^} $VERSION_ID ($CODENAME)"

}

#==========================================
# APT Mirror Configuration
#==========================================

change_mirror() {
	detect_distro

	local backup_file="/etc/apt/sources.list.bak.$(date +%Y%m%d%H%M%S)"
	yellow "Backing up the original /etc/apt/sources.list to $backup_file"
	sudo cp /etc/apt/sources.list "$backup_file" || error_exit "Failed to backup sources.list"
	green "Backup created: $backup_file"

	local mirror_content
	case "$ID" in
        ubuntu)
            mirror_content="deb ${MIRROR_URL}${ID}/ $CODENAME main restricted universe multiverse
deb ${MIRROR_URL}${ID}/ ${CODENAME}-updates main restricted universe multiverse
deb ${MIRROR_URL}${ID}/ ${CODENAME}-backports main restricted universe multiverse
deb ${MIRROR_URL}${ID}/ ${CODENAME}-security main restricted universe multiverse"
            ;;
        debian)
            mirror_content="deb ${MIRROR_URL}${ID}/ $CODENAME main contrib non-free
deb ${MIRROR_URL}${ID}/ ${CODENAME}-updates main contrib non-free
deb ${MIRROR_URL}${ID}/ ${CODENAME}-backports main contrib non-free
deb ${MIRROR_URL}${ID}-security ${CODENAME}-security main contrib non-free"
            ;;
        *)
            error_exit "Unsupported distribution: $ID"
            ;;
    esac
	
	
	echo "$mirror_content" | sudo tee /etc/apt/sources.list >/dev/null
	sudo apt-get update || error_exit "Failed to update package lists"
	green "Mirror source replacement completed! Current source: $MIRROR_URL"
}

#==========================================
# Systemd Service Creation
#==========================================

create_java_systemd_service() {
    local service_name="${APP_NAME}.service"
    local service_path="/etc/systemd/system/${service_name}"
    
    blue "Creating Java systemd service..."
    
    sudo tee "$service_path" <<EOF >/dev/null
[Unit]
Description=${APP_NAME} Spring Boot Application
After=network.target

[Service]
Type=simple
User=${APP_USER}
Group=${APP_USER}
WorkingDirectory=/var/lib/${APP_NAME}

# Java executable and JAR file
ExecStart=/usr/bin/java \
    -Djava.security.egd=file:/dev/./urandom \
    -Dspring.pid.file=${APP_PID_DIR}/${APP_NAME}.pid \
    -jar ${APP_BIN_DIR}/${DEFAULT_JAR}

# Environment variables
EnvironmentFile=${APP_BIN_DIR}/${DEFAULT_JAR}.conf

# Logging
StandardOutput=file:${APP_LOG_DIR}/${APP_NAME}.log
StandardError=file:${APP_LOG_DIR}/${APP_NAME}-error.log
SyslogIdentifier=${APP_NAME}

# Process management
PIDFile=${APP_PID_DIR}/${APP_NAME}.pid
Restart=on-failure
RestartSec=10
StartLimitInterval=60
StartLimitBurst=3
KillMode=mixed
KillSignal=SIGTERM
TimeoutStopSec=60

# Security settings
NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=strict
ProtectHome=true
ProtectKernelTunables=true
ProtectKernelModules=true
ProtectControlGroups=true
DynamicUser=false
CapabilityBoundingSet=CAP_NET_BIND_SERVICE
AmbientCapabilities=CAP_NET_BIND_SERVICE

# Resource limits
LimitNOFILE=65536
LimitNPROC=4096

# Writable paths
ReadWritePaths=/var/lib/${APP_NAME} ${APP_LOG_DIR} ${APP_PID_DIR} ${APP_DIR}

[Install]
WantedBy=multi-user.target
EOF

    # sudo systemctl daemon-reload
    green "Java systemd service created: $service_path"
}

create_golang_systemd_service() {
    local service_name="${APP_NAME}.service"
    local service_path="/etc/systemd/system/${service_name}"
    
    blue "Creating Go systemd service..."
    
    sudo tee "$service_path" <<EOF >/dev/null
[Unit]
Description=${APP_NAME} Go Application
After=network.target

[Service]
Type=simple
User=${APP_USER}
Group=${APP_USER}
WorkingDirectory=/var/lib/${APP_NAME}

# Go executable with config
ExecStart=${APP_BIN_DIR}/${APP_NAME} -conf ${APP_CONFIG_DIR}/prod.yaml

# Environment variables (optional)
Environment="GOPROXY='https://goproxy.cn,direct'"
Environment="GO111MODULE='on'"

# Logging
StandardOutput=file:${APP_LOG_DIR}/${APP_NAME}.log
StandardError=file:${APP_LOG_DIR}/${APP_NAME}-error.log
SyslogIdentifier=${APP_NAME}

# Process management
PIDFile=${APP_PID_DIR}/${APP_NAME}.pid
Restart=on-failure
RestartSec=10
StartLimitInterval=60
StartLimitBurst=3
KillMode=mixed
KillSignal=SIGTERM
TimeoutStopSec=60

# Security settings
NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=strict
ProtectHome=true
ProtectKernelTunables=true
ProtectKernelModules=true
ProtectControlGroups=true
DynamicUser=false
CapabilityBoundingSet=CAP_NET_BIND_SERVICE
AmbientCapabilities=CAP_NET_BIND_SERVICE

# Resource limits
LimitNOFILE=65536
LimitNPROC=4096

# Writable paths
ReadWritePaths=/var/lib/${APP_NAME} ${APP_LOG_DIR} ${APP_PID_DIR} ${APP_DIR}

[Install]
WantedBy=multi-user.target
EOF

    # sudo systemctl daemon-reload
    green "Go systemd service created: $service_path"
}

create_systemd_service() {
    if [[ "$JAVA_ENABLED" == "true" ]]; then
        create_java_systemd_service
    elif [[ "$GOLANG_ENABLED" == "true" ]]; then
        create_golang_systemd_service
    else
        error_exit "No application type enabled (JAVA_ENABLED or GOLANG_ENABLED)"
    fi
}

#==========================================
# Helper Script Creation (Optional)
#==========================================

create_helper_script() {
    local script_path="$APP_SCRIPTS_DIR/appctl.sh"
    blue "Creating helper script for systemd service..."
    
    sudo tee "$script_path" <<'EOF' >/dev/null
#!/bin/bash
# Helper script for systemd service management

set -euo pipefail

readonly SERVICE_NAME="__APP_NAME__.service"

# Color output
echo_color() { echo -e "\033[${1}m${*:2}\033[0m"; }
echo_red() { echo_color "0;31" "$@"; }
echo_green() { echo_color "0;32" "$@"; }
echo_yellow() { echo_color "0;33" "$@"; }
echo_blue() { echo_color "0;34" "$@"; }

# Commands
start_service() {
    echo_blue "Starting $SERVICE_NAME..."
    sudo systemctl start "$SERVICE_NAME"
    echo_green "Service started"
}

stop_service() {
    echo_blue "Stopping $SERVICE_NAME..."
    sudo systemctl stop "$SERVICE_NAME"
    echo_green "Service stopped"
}

restart_service() {
    echo_blue "Restarting $SERVICE_NAME..."
    sudo systemctl restart "$SERVICE_NAME"
    echo_green "Service restarted"
}

status_service() {
    sudo systemctl status "$SERVICE_NAME"
}

enable_service() {
    echo_blue "Enabling $SERVICE_NAME to start on boot..."
    sudo systemctl enable "$SERVICE_NAME"
    echo_green "Service enabled"
}

disable_service() {
    echo_blue "Disabling $SERVICE_NAME from starting on boot..."
    sudo systemctl disable "$SERVICE_NAME"
    echo_green "Service disabled"
}

logs_service() {
    local lines="${1:-50}"
    sudo journalctl -u "$SERVICE_NAME" -n "$lines" --no-pager
}

follow_logs() {
    echo_blue "Following logs for $SERVICE_NAME (Ctrl+C to exit)..."
    sudo journalctl -u "$SERVICE_NAME" -f
}

reload_service() {
    echo_blue "Reloading systemd configuration..."
    sudo systemctl daemon-reload
    echo_green "Configuration reloaded"
}

# Main
case "${1:-help}" in
    start)
        start_service
        ;;
    stop)
        stop_service
        ;;
    restart)
        restart_service
        ;;
    status)
        status_service
        ;;
    enable)
        enable_service
        ;;
    disable)
        disable_service
        ;;
    logs)
        logs_service "${2:-50}"
        ;;
    follow)
        follow_logs
        ;;
    reload)
        reload_service
        ;;
    help|*)
        cat <<HELP
Usage: $0 {start|stop|restart|status|enable|disable|logs|follow|reload|help}

Commands:
    start       - Start the service
    stop        - Stop the service
    restart     - Restart the service
    status      - Show service status
    enable      - Enable service to start on boot
    disable     - Disable service from starting on boot
    logs [N]    - Show last N lines of logs (default: 50)
    follow      - Follow logs in real-time
    reload      - Reload systemd configuration
    help        - Show this help message

Examples:
    $0 start
    $0 status
    $0 logs 100
    $0 follow
HELP
        exit 0
        ;;
esac
EOF

    sudo sed -i "s|__APP_NAME__|$(printf '%s' "$APP_NAME")|g" "$script_path"
    sudo chmod +x "$script_path"
    sudo chown "$APP_USER:$APP_USER" "$script_path"
    green "Helper script created: $script_path"
}

#==========================================
# Server Initialization
#==========================================

init_server() {
	blue "Preparing server's application workspace..."
	
	# Create user if not exists
    if ! id "$APP_USER" &>/dev/null; then
	    sudo useradd -m -s /bin/bash -d "/home/$APP_USER" "$APP_USER" || error_exit "Failed to create user"
	    echo "$APP_USER:$APP_PASSWORD" | sudo chpasswd || error_exit "Failed to set password"
	    green "Created user: $APP_USER (password set)"
	fi

	
	# Create directory structure
	sudo mkdir -p "$APP_BIN_DIR" "$APP_CONFIG_DIR" "$APP_LIB_DIR" \
                  "$APP_SCRIPTS_DIR" "$APP_LOG_DIR" "$APP_PID_DIR" "/var/lib/${APP_NAME}" || error_exit "Failed to create directories"
    
    sudo touch "$APP_LOG_DIR/example.log"
    sudo touch "${APP_BIN_DIR}/${DEFAULT_JAR}.conf"
	
	sudo chown -R "$APP_USER:$APP_USER" "$APP_DIR" "$APP_LOG_DIR" "$APP_PID_DIR" "/var/lib/${APP_NAME}"
	sudo chmod 750 "$APP_DIR"
	sudo chmod 640 "$APP_LOG_DIR"/*.log 2>/dev/null || true
	
	# Create systemd service
	create_app_control_script
	
	# Create helper script
	create_helper_script
	
	open_firewall_port
	
	green "Server workspace preparation completed!"
    yellow "To manage the service, use:"
    yellow "  sudo systemctl start ${APP_NAME}"
    yellow "  sudo systemctl stop ${APP_NAME}"
    yellow "  sudo systemctl status ${APP_NAME}"
    yellow "  sudo systemctl enable ${APP_NAME}  # Start on boot"
    yellow "Or use the helper script:"
    yellow "  ${APP_SCRIPTS_DIR}/appctl.sh start"
    yellow "  ${APP_SCRIPTS_DIR}/appctl.sh status"
    yellow "  ${APP_SCRIPTS_DIR}/appctl.sh logs"
}

#==========================================
# Firewall Configuration
#==========================================

open_firewall_port() {
	sudo sudo ufw allow ssh
	sudo sudo ufw allow 80
	sudo sudo ufw enable
}

#==========================================
# Client Initialization
#==========================================

init_client() {
	blue "Preparing client SSH configuration..."
    
    if [[ ! -f "$PRIVATE_KEY" ]]; then
        local key_dir
        key_dir=$(dirname "$PRIVATE_KEY")
        mkdir -p "$key_dir"
        
        green "Generating SSH key pair..."
        ssh-keygen -t rsa -b 4096 -N "" -C "50849806@qq.com" -f "$PRIVATE_KEY" || error_exit "Failed to generate SSH key"
    fi
    
    blue "Copying public key to server..."
    if ! ssh-copy-id -i "$PUBLIC_KEY" -p "$SSH_PORT" "$APP_USER@$SERVER_IP"; then
	    error_exit "Failed to copy SSH key to $APP_USER@$SERVER_IP"
	fi
    
    green "Client SSH configuration completed!"
}

#==========================================
# Software Installation
#==========================================

wait_for_apt_lock() {
    blue "Waiting for APT lock..."
    local timeout=300 elapsed=0
    
    while fuser /var/lib/dpkg/lock /var/lib/apt/lists/lock /var/cache/apt/archives/lock >/dev/null 2>&1; do
        sleep 1
        ((elapsed++))
        [[ $elapsed -ge $timeout ]] && error_exit "APT lock timeout"
    done
    
    green "APT ready"
}

install_basic_tools() {
    blue "Installing basic tools..."
    wait_for_apt_lock
    sudo apt-get install -y sudo vim curl lsof || error_exit "Failed to install basic tools"
    green "Basic tools installed"
}

install_java() {
    [[ "$JAVA_ENABLED" != "true" ]] && return 0
    
    blue "Installing Java ($JAVA_VERSION)..."
    wait_for_apt_lock
    sudo apt-get install -y "$JAVA_VERSION" || error_exit "Failed to install Java"
    green "Java installed"
}

install_docker() {
    [[ "$DOCKER_ENABLED" != "true" ]] && return 0
    
    blue "Installing Docker..."
    wait_for_apt_lock
    sudo apt-get install -y docker.io || error_exit "Failed to install Docker"
    
    sudo usermod -aG docker "$APP_USER" || error_exit "Failed to add user to docker group"
    sudo systemctl restart docker || error_exit "Failed to start Docker"
    
    # Wait for Docker to be ready
    local timeout=30
    blue "Waiting for Docker (max ${timeout}s)..."
    if ! timeout $timeout bash -c 'until docker info >/dev/null 2>&1; do sleep 1; done'; then
        error_exit "Docker not ready"
    fi
    
    green "Docker installed and configured"
}

install_docker_compose() {
    [[ "$DOCKER_COMPOSE_ENABLED" != "true" ]] && return 0
    
    blue "Installing Docker Compose..."
    wait_for_apt_lock
    sudo apt-get install -y docker-compose || error_exit "Failed to install Docker Compose"
    green "Docker Compose installed"
}

install_software() {
    install_basic_tools
    install_java
    install_docker
    install_docker_compose
}

#==========================================
# Main Function
#==========================================

usage() {
    cat <<EOF
Usage: $0 {server|client|help}

Commands:
    server  - Setup server environment (mirror, software, workspace, systemd service)
    client  - Setup client SSH configuration
    help    - Show this help message

Configuration:
    APP_USER: $APP_USER
    APP_DIR: $APP_DIR
    JAVA_ENABLED: $JAVA_ENABLED
    GOLANG_ENABLED: $GOLANG_ENABLED
    DOCKER_ENABLED: $DOCKER_ENABLED
EOF
}

main() {
    local action="${1:-help}"
    
    case "$action" in
        server)
            detect_distro
            change_mirror
            install_software
            init_server
            green "Server setup completed successfully!"
            ;;
        client)
            init_client
            green "Client setup completed successfully!"
            ;;
        help|--help|-h)
            usage
            ;;
        *)
            red "Unknown command: $action"
            usage
            exit 1
            ;;
    esac
}

# Run main function
main "$@"
```

# k8s

```SHELL
#!/bin/bash

MASTER_PORTS=(6443 2379 2380 10250 10259 10257)
WORKER_PORT=10250
KUBERNETES_VERSION=v1.31
CODE_NAME=noble

function init_ssh() {
	if ! command -v ssh &>/dev/null; then
		sudo apt-get -qq install -y openssh-server
		echo "root:root" | sudo chpasswd
		sudo sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
		sudo systemctl ssh start
	fi
}

function configure_hosts() {
	# todo don't forget edit
	sudo hostnamectl set-hostname k8s-master
	cat <<EOF | sudo tee -a /etc/hosts >/dev/null
172.16.4.90 master
172.16.4.91 node1
172.16.4.92 node2
EOF
}

function check_ports() {
	if ! command -v ufw &>/dev/null; then
		sudo apt-get -qq install -y ufw
	fi
	sudo ufw --force enable
	local ports=("${MASTER_PORTS[@]}")
	[ "$1" == "worker" ] && ports=("$WORKER_PORT")
	for port in "${ports[@]}"; do
		if ! sudo ufw status | grep -q "$port/tcp"; then
			echo "Opening port: $port"
			sudo ufw allow "$port/tcp"
		fi
	done
}

function time_sync() {
	sudo apt-get -qq install -y systemd-timesyncd
	sudo timedatectl set-ntp true
	sudo timedatectl set-timezone Asia/Shanghai
	sudo tee -a /etc/systemd/timesyncd.conf >/dev/null <<EOF
NTP=ntp.aliyun.com ntp1.aliyun.com
FallbackNTP=0.pool.ntp.org 1.pool.ntp.org
EOF
	sudo systemctl systemd-timesyncd restart
}

function disable_swap() {
	sudo swapoff -a
	sudo sed -i.bak -r 's/(.+ swap .+)/#\1/' /etc/fstab
}

function load_modules() {
	sudo cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf >/dev/null
overlay
br_netfilter
EOF
	sudo modprobe overlay
	sudo modprobe br_netfilter
}

function configure_network() {
	sudo cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf >/dev/null
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
	sudo sysctl --system
}

function install_kubernetes() {
	sudo apt-get install -y apt-transport-https ca-certificates curl gpg
	curl -fsSL https://pkgs.k8s.io/core:/stable:/$KUBERNETES_VERSION/deb/Release.key | gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
	echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/$KUBERNETES_VERSION/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
	sudo apt-get update
	sudo apt-get -qq install -y kubelet kubeadm kubectl
	sudo apt-mark hold kubelet kubeadm kubectl
}

function install_container() {
	curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo tee /etc/apt/keyrings/docker.asc >/dev/null
	sudo chmod a+r /etc/apt/keyrings/docker.asc
	echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list >/dev/null
	sudo apt-get update
	sudo apt-get -qq install -y containerd.io
	sudo mkdir -p /etc/containerd
	containerd config default | sudo tee /etc/containerd/config.toml >/dev/null
	sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
	# watch out version
	sudo sed -i 's|k8s.gcr.io/pause|registry.aliyuncs.com/google_containers/pause/g' /etc/containerd/config.toml
	sudo service containerd restart
	sudo systemctl enable containerd
}

function install_container_binary() {
  sudo apt-get -qq install -y wget
  sudo wget https://www.ghproxy.cn/https://github.com/containerd/containerd/releases/download/v2.0.2/containerd-2.0.2-linux-amd64.tar.gz
  sudo tar Cxzvf /usr/local containerd-2.0.2-linux-amd64.tar.gz
  sudo wget https://www.ghproxy.cn/https://github.com/opencontainers/runc/releases/download/v1.2.4/runc.amd64
  sudo install -m 755 runc.amd64 /usr/local/sbin/runc
  sudo wget https://www.ghproxy.cn/https://github.com/containernetworking/plugins/releases/download/v1.6.2/cni-plugins-linux-amd64-v1.6.2.tgz
  sudo mkdir -p /opt/cni/bin
  sudo tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.1.1.tgz
  sudo mkdir -p /etc/containerd
  sudo containerd config default | sudo tee /etc/containerd/config.toml >/dev/null
  sudo sed -i "/\[plugins.'io.containerd.cri.v1.runtime'.containerd.runtimes.runc.options\]/a \ \ \ \ \ \ \ \ \ \ \ \ SystemdCgroup = true" /etc/containerd/config.toml
  sudo sed -i 's|registry.k8s.io/pause|registry.aliyuncs.com/google_containers/pause/g' /etc/containerd/config.toml
  sudo sed -i "s|config_path = ''|config_path = '/etc/containerd/certs.d'|" /etc/containerd/config.toml
  sudo cat <<EOF | sudo tee /usr/local/lib/systemd/system/containerd.service >/dev/null
[Unit]
Description=containerd container runtime
Documentation=https://containerd.io
After=network.target local-fs.target dbus.service

[Service]
ExecStartPre=-/sbin/modprobe overlay
ExecStart=/usr/local/bin/containerd

Type=notify
Delegate=yes
KillMode=process
Restart=always
RestartSec=5

# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNPROC=infinity
LimitCORE=infinity

# Comment TasksMax if your systemd version does not supports it.
# Only systemd 226 and above support this version.
TasksMax=infinity
OOMScoreAdjust=-999

[Install]
WantedBy=multi-user.target
EOF

  sudo mkdir -p /etc/containerd/certs.d/docker.io
  sudo cat <<EOF | sudo tee /etc/containerd/certs.d/docker.io/hosts.toml >/dev/null
server = "https://docker.io"

[host."https://registry.dockermirror.com"]
  capabilities = ["pull", "resolve"]
[host."https://n2eudwtr.mirror.aliyuncs.com"]
  capabilities = ["pull", "resolve"]
[host."https://docker.1ms.run"]
  capabilities = ["pull", "resolve"]
[host."https://docker.xuanyuan.me"]
  capabilities = ["pull", "resolve"]
EOF

  sudo systemctl daemon-reload
  sudo systemctl enable --now containerd
}

function configure_crictl() {
	cat <<EOF | sudo tee /etc/crictl.yaml >/dev/null
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 2
debug: true
EOF
}

function init_kubernetes() {
	sudo systemctl enable kubelet

  if [ "$1" == "master" ]; then
    echo "Initializing Kubernetes Master..."
    sudo kubeadm config images pull --cri-socket unix:///var/run/containerd/containerd.sock
    sudo kubeadm init --image-repository registry.aliyuncs.com/google_containers --pod-network-cidr=10.244.0.0/16 --cri-socket=/var/run/containerd/containerd.sock --v=5

    # Set up kubeconfig for the root user
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config

    # Install Flannel network plugin
    sudo kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

    # Install weave network plugin
    # sudo kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml

    # Install calico network plugin
    # sudo kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/tigera-operator.yaml

    echo "Master node initialized. You can now join worker nodes using kubeadm token create --print-join-command to generate the join command ."
  fi
}

function main() {
	sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak

	sudo echo "deb http://mirrors.aliyun.com/ubuntu/ $CODE_NAME main restricted universe multiverse
  deb http://mirrors.aliyun.com/ubuntu/ $CODE_NAME-updates main restricted universe multiverse
  deb http://mirrors.aliyun.com/ubuntu/ $CODE_NAME-backports main restricted universe multiverse
  deb http://mirrors.aliyun.com/ubuntu/ $CODE_NAME-security main restricted universe multiverse" >>/etc/apt/sources.list

	sudo apt update

#	init_ssh
	configure_hosts
	check_ports "$1"
	time_sync
	disable_swap
	load_modules
	configure_network
	install_kubernetes
	install_container
#	configure_crictl
	init_kubernetes "$1"
}

main "$1"

```