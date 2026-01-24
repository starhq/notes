
```SHELL
#!/bin/bash
# Description: Ubuntu/Debian Mirror Switcher and Environment Setup
# Author: starhq
# Date: $(date +%Y-%m-%d)
set -euo pipefail
# Configuration
MIRROR_URL='https://mirrors.aliyun.com/'
APP_USER="starhq"
APP_NAME="app"
APP_DIR="/opt/$APP_NAME"
APP_BIN_DIR="$APP_DIR/bin"
APP_CONFIG_DIR="$APP_DIR/config"
APP_LIB_DIR="$APP_DIR/lib"
APP_SCRIPTS_DIR="$APP_DIR/scripts"
APP_LOG_DIR="/var/log/$APP_NAME"
APP_PID_DIR="/var/run/$APP_NAME"
DEFAULT_JAR="template-0.0.1-SNAPSHOT.jar"
# APP_PASSWORD=$(openssl rand -base64 32)
APP_PASSWORD="123456"
SSH_PORT="22"
JAVA_VERSION="default-jdk"
JAVA_ENABLED="false"
GOLANG_ENABLED="true"
DOCKER_ENABLED="false"
DOCKER_COMPOSE_ENABLED="false"
PRIVATE_KEY="$HOME/.ssh/$APP_NAME/id_rsa"
PUBLIC_KEY="$HOME/.ssh/$APP_NAME/id_rsa.pub"
SERVER_IP="127.0.0.1"

# Color output
color_output() { printf "\e[${1}m%s\e[0m\n" "${@:2}"; }
red() { color_output "31" "$@"; }
green() { color_output "32" "$@"; }
yellow() { color_output "33" "$@"; }
blue() { color_output "34" "$@"; }

# Detect Linux distribution
detect_distro() {
	if [[ -f /etc/os-release ]]; then
		# shellcheck disable=SC1091
		source /etc/os-release
		case "$ID:$VERSION_ID" in
		ubuntu:*)
			CODENAME="$VERSION_CODENAME"
			return 0
			;;
		debian:12)
			CODENAME="bookworm"
			return 0
			;;
		debian:11)
			CODENAME="bullseye"
			return 0
			;;
		debian:10)
			CODENAME="buster"
			return 0
			;;
		debian:*)
			red "Unsupported Debian version: $VERSION_ID"
			return 1
			;;
		*)
			red "Unsupported distribution: $ID"
			return 1
			;;
		esac
	else
		red "Error: Not a Debian-based system!"
		return 1
	fi
}

# Change apt mirror source
change_mirror() {
	detect_distro || return 1
	green "Current system: ${ID^} $VERSION_ID ($CODENAME)"
	BACKUP_FILE="/etc/apt/sources.list.bak.$(date +%Y%m%d%H%M%S)"
	sudo cp /etc/apt/sources.list "$BACKUP_FILE" && green "Backup created: $BACKUP_FILE"
	local mirror_content
	case "$ID" in
	ubuntu) mirror_content="deb ${MIRROR_URL}${ID}/ $CODENAME main restricted universe multiverse\ndeb ${MIRROR_URL}${ID}/ ${CODENAME}-updates main restricted universe multiverse\ndeb ${MIRROR_URL}${ID}/ ${CODENAME}-backports main restricted universe multiverse\ndeb ${MIRROR_URL}${ID}/ ${CODENAME}-security main restricted universe multiverse" ;;
	debian) mirror_content="deb ${MIRROR_URL}${ID}/ $CODENAME main contrib non-free\ndeb ${MIRROR_URL}${ID}/ ${CODENAME}-updates main contrib non-free\ndeb ${MIRROR_URL}${ID}/ ${CODENAME}-backports main contrib non-free\ndeb ${MIRROR_URL}${ID}-security ${CODENAME}-security main contrib non-free" ;;
	*)
		red "Unsupported distribution: $ID"
		return 1
		;;
	esac
	sudo tee /etc/apt/sources.list <<<"$mirror_content" >/dev/null
	sudo apt-get update
	green "Mirror source replacement completed!\nCurrent source: $MIRROR_URL"
}

# Create application control script
create_app_control_script() {
	local script_path="$APP_SCRIPTS_DIR/appctl.sh"
	blue "Creating application control script..."
	if [[ "$JAVA_ENABLED" == "true" ]]; then
		sudo tee "$script_path" <<EOF >/dev/null
#!/bin/bash
# Optimized Spring Boot Startup Script for ${APP_DIR} structure
set -euo pipefail
### Initialization Section ###
# Basic configuration
readonly APP_BASE_DIR="${APP_DIR}"
readonly JAR_FILE=${JAR_FILE:-"${APP_BIN_DIR}/${DEFAULT_JAR}"}
readonly DEBUG=\${DEBUG:-false}
# Debug mode
if [[ "\$DEBUG" == "true" ]]; then
    set -x
fi
# Working directories
declare WORKING_DIR
WORKING_DIR="${APP_DIR}"
declare SCRIPT_DIR
SCRIPT_DIR="${APP_SCRIPTS_DIR}"
### Function Definitions ###
# Color output function
echo_color() {
    local color_code=\$1
    shift
    echo -e "\033[\${color_code}m\$*\033[0m"
}
echo_red() { echo_color "0;31" "\$@"; }
echo_green() { echo_color "0;32" "\$@"; }
echo_yellow() { echo_color "0;33" "\$@"; }
echo_blue() { echo_color "0;34" "\$@"; }
# Create directory with permissions check
ensure_directory() {
    local dir=\$1
    local owner=\${2:-}
    if [[ ! -d "\$dir" ]]; then
        if ! sudo mkdir -p "\$dir"; then
            echo_red "Failed to create directory: \$dir"
            return 1
        fi
        if [[ -n "\$owner" ]]; then
            sudo chown "\$owner" "\$dir"
        fi
    fi
    if [[ ! -w "\$dir" ]]; then
        echo_red "No write permission to directory: \$dir"
        return 1
    fi
}
# Check if process is running
is_process_running() {
    local pid=\$1
    ps -p "\$pid" > /dev/null 2>&1
}
# Wait for file to appear
await_file() {
    local file=\$1
    local timeout=\${2:-10}
    local end=\$(( \$(date +%s) + timeout ))
    while [[ ! -s "\$file" ]]; do
        sleep 1
        if [[ \$(date +%s) -ge "\$end" ]]; then
            echo_red "Timeout waiting for file: \$file"
            return 1
        fi
    done
    return 0
}
### JAR File Processing ###
# Resolve symbolic links to find real JAR path
resolve_jar_path() {
    local jar_path=\$1
    local init_script
    local config_file
    if [[ ! "\$jar_path" =~ ^\${APP_BASE_DIR} ]]; then
        echo_red "JAR file must be located under \${APP_BASE_DIR}"
        exit 1
    fi
    if ! cd "\$(dirname "\$jar_path")"; then
        exit 1
    fi
    while [[ -L "\$jar_path" ]]; do
        if [[ "\$jar_path" =~ init\.d ]]; then
            init_script=$(basename "\$jar_path")
        else
            config_file="\${jar_path%.*}.conf"
            if [[ -r "\$config_file" ]]; then
                # shellcheck source=/opt/myapp/bin/template.conf
                source "\$config_file"
            fi
        fi
        jar_path=\$(readlink "\$jar_path")
        if ! cd "\$(dirname "\$jar_path")"; then
            exit 1
        fi
        jar_path=\$(pwd)/\$(basename "\$jar_path")
    done
    echo "\$jar_path"
}
# Initialize JAR file path
resolved_jar_file=\$(resolve_jar_path "\${JAR_FILE:-"\$SCRIPT_DIR/\$(basename "$0")"}")
readonly resolved_jar_file
declare jar_folder
jar_folder=\$(cd "$(dirname "\$resolved_jar_file")" && pwd -P)
if ! cd "\$WORKING_DIR"; then
    exit 1
fi
### Configuration Loading ###
# Load configuration file
load_config() {
    local config_file="\${resolved_jar_file%.*}.conf"
    config_file=$(basename "\$config_file")
    local conf_folder=${CONF_FOLDER:-\$jar_folder}
    if [[ -r "\${conf_folder}/\${config_file}" ]]; then
        echo_blue "Loading configuration from \${conf_folder}/\${config_file}"
        # shellcheck source=/opt/myapp/bin/template.conf
        source "\${conf_folder}/\${config_file}"
    fi
}
load_config
### Runtime Configuration ###
# Default values
readonly MODE=\${MODE:-auto}  # auto, service, run
readonly USE_START_STOP_DAEMON=\${USE_START_STOP_DAEMON:-true}
readonly STOP_WAIT_TIME=\${STOP_WAIT_TIME:-60}
# Application identity
# if [[ -z "\${identity:-}" ]]; then
#     if [[ -n "\${init_script:-}" ]]; then
#         identity="\${init_script}"
#     else
#         identity=\$(basename "\${resolved_jar_file%.*}")_\${jar_folder//\//_}
#     fi
# fi
readonly identity=\$(basename "\${resolved_jar_file%.*}")
# File and directory configuration
PID_FOLDER=\${PID_FOLDER:-${APP_PID_DIR}/}
LOG_FOLDER=\${LOG_FOLDER:-${APP_LOG_DIR}/}
LOG_FILENAME=\${LOG_FILENAME:-\${identity}.log}
# Handle relative paths
if [[ "\$PID_FOLDER" != /* ]]; then
    PID_FOLDER="\${jar_folder}/\${PID_FOLDER}"
fi
if [[ "\$LOG_FOLDER" != /* ]]; then
    LOG_FOLDER="\${jar_folder}/\${LOG_FOLDER}"
fi
# Check and create necessary directories
if ! ensure_directory "\${PID_FOLDER}" "\${RUN_AS_USER:-}"; then
    PID_FOLDER="/tmp"
fi
if ! ensure_directory "\${LOG_FOLDER}" "\${RUN_AS_USER:-}"; then
    LOG_FOLDER="/tmp"
fi
# Final paths
readonly PID_FOLDER="\${PID_FOLDER}/\${identity}"
readonly pid_file="\${PID_FOLDER}/\${identity}.pid"
readonly log_file="\${LOG_FOLDER}/\${LOG_FILENAME}"
### User Permission Handling ###
# Determine run user
determine_run_user() {
    if [[ -n "\${RUN_AS_USER:-}" ]]; then
        if ! id -u "\$RUN_AS_USER" > /dev/null 2>&1; then
            echo_red "User \$RUN_AS_USER does not exist"
            exit 2
        fi
        echo "\$RUN_AS_USER"
    elif [[ \$(id -u) -eq 0 ]]; then
        stat -c '%U' "\$resolved_jar_file" 2>/dev/null || echo "root"
    else
        id -un
    fi
}
declare run_user
run_user=\$(determine_run_user)
# Security warning
if [[ \$(id -u "\$run_user") -eq 0 ]]; then
    echo_yellow "WARNING: Application is running as root (UID 0). This is considered insecure."
fi
### Java Environment Detection ###
find_java() {
    if [[ -n "\${JAVA_HOME:-}" ]] && [[ -x "\${JAVA_HOME}/bin/java" ]]; then
        echo "\${JAVA_HOME}/bin/java"
    elif type -p java > /dev/null 2>&1; then
        type -p java
    elif [[ -x "/usr/bin/java" ]]; then
        echo "/usr/bin/java"
    else
        echo_red "Java not found. Please install Java or set JAVA_HOME."
        exit 1
    fi
}
declare java_exe
java_exe=\$(find_java)
### Argument Preparation ###
arguments=(-Dsun.misc.URLClassPath.disableJarChecking=true)
if [[ -n "\${JAVA_OPTS:-}" ]]; then
    arguments+=("\$JAVA_OPTS")
fi
arguments+=(-jar "\$resolved_jar_file")
if [[ -n "\${RUN_ARGS:-}" ]]; then
    arguments+=("\$RUN_ARGS")
fi
arguments+=("\$@")
### Operation Functions ###
start_application() {
    local pid
    if [[ -f "\$pid_file" ]]; then
        pid=\$(cat "\$pid_file")
        if is_process_running "\$pid"; then
            echo_yellow "Application is already running [PID: \$pid]"
            return 0
        fi
    fi
    if ! ensure_directory "\$PID_FOLDER" "\$run_user"; then
        return \$?
    fi
    if ! ensure_directory "\$(dirname "\$log_file")" "\$run_user"; then
        return \$?
    fi
    if ! sudo touch "\$log_file"; then
        echo_red "Failed to create log file: \$log_file"
        return 1
    fi
    if [[ -n "\$run_user" ]]; then
        sudo chown "\$run_user" "\$log_file"
    fi
    local working_dir
    working_dir=\$(dirname "\$resolved_jar_file")
    if ! pushd "\$working_dir" > /dev/null; then
        echo_red "Failed to change to working directory: \$working_dir"
        return 1
    fi
    if [[ "\$USE_START_STOP_DAEMON" == "true" ]] && type start-stop-daemon > /dev/null 2>&1; then
        echo_blue "Starting using start-stop-daemon..."
        if ! sudo start-stop-daemon --start --quiet \
            --chuid "\$run_user" \
            --name "\$identity" \
            --make-pidfile --pidfile "\$pid_file" \
            --background --no-close \
            --startas "\$java_exe" \
            --chdir "\$working_dir" \
            -- "\${arguments[@]}" \
            >> "\$log_file" 2>&1; then
            echo_red "Failed to start application"
            popd > /dev/null
            return 1
        fi
        if await_file "\$pid_file" 10; then
            pid=\$(cat "\$pid_file")
            echo_green "Application started successfully [PID: \$pid]"
        else
            echo_red "Failed to start application (timeout waiting for PID file)"
            popd > /dev/null
            return 1
        fi
    else
        echo_blue "Starting directly..."
        if [[ \$(id -u) -eq 0 ]]; then
            if ! sudo su -s
                echo_red "Failed to start application as user \$run_user"
                popd > /dev/null
                return 1
            fi
        else
            if ! "\$java_exe" "\${arguments[@]}" >> "\$log_file" 2>&1 & then
                echo_red "Failed to start application"
                popd > /dev/null
                return 1
            fi
            echo \$! > "\$pid_file"
        fi

        pid=\$(cat "\$pid_file")
        if ! disown "\$pid"; then
            echo_yellow "Warning: failed to disown process \$pid"
        fi
        echo_green "Application started successfully [PID: \$pid]"
    fi

    popd > /dev/null
}

stop_application() {
    local pid

    if [[ ! -f "\$pid_file" ]]; then
        echo_yellow "Not running (PID file not found)"
        return 0
    fi

    pid=\$(cat "\$pid_file")
    if ! is_process_running "\$pid"; then
        echo_yellow "Not running (process \$pid not found). Removing stale PID file."
        rm -f "\$pid_file"
        return 0
    fi

    echo_blue "Stopping application [PID: \$pid]..."
    if ! kill "\$pid" > /dev/null 2>&1; then
        echo_red "Failed to send termination signal to process \$pid"
        return 1
    fi

    local i
    for ((i=0; i<STOP_WAIT_TIME; i++)); do
        if ! is_process_running "\$pid"; then
            echo_green "Application stopped successfully"
            rm -f "\$pid_file"
            return 0
        fi
        if [[ \$i -eq \$((STOP_WAIT_TIME/2)) ]]; then
            kill "\$pid" > /dev/null 2>&1
        fi
        sleep 1
    done

    echo_red "Failed to stop application [PID: \$pid] within \$STOP_WAIT_TIME seconds"
    return 1
}

force_stop_application() {
    local pid

    if [[ ! -f "\$pid_file" ]]; then
        echo_yellow "Not running (PID file not found)"
        return 0
    fi

    pid=\$(cat "\$pid_file")
    if ! is_process_running "\$pid"; then
        echo_yellow "Not running (process \$pid not found). Removing stale PID file."
        rm -f "\$pid_file"
        return 0
    fi

    echo_blue "Force stopping application [PID: \$pid]..."
    if ! kill -9 "\$pid" > /dev/null 2>&1; then
        echo_red "Failed to send kill signal to process \$pid"
        return 1
    fi

    local i
    for ((i=0; i<STOP_WAIT_TIME; i++)); do
        if ! is_process_running "\$pid"; then
            echo_green "Application force stopped successfully"
            rm -f "\$pid_file"
            return 0
        fi
        if [[ \$i -eq \$((STOP_WAIT_TIME/2)) ]]; then
            kill -9 "\$pid" > /dev/null 2>&1
        fi
        sleep 1
    done

    echo_red "Failed to force stop application [PID: \$pid] within \$STOP_WAIT_TIME seconds"
    return 1
}

restart_application() {
    stop_application && start_application
}

force_reload_application() {
    local pid

    if [[ ! -f "\$pid_file" ]]; then
        echo_red "Not running (PID file not found)"
        return 7
    fi

    pid=\$(cat "\$pid_file")
    if ! is_process_running "\$pid"; then
        echo_red "Not running (process \$pid not found)"
        return 7
    fi

    rm -f "\$pid_file"
    stop_application && start_application
}

check_status() {
    local pid

    if [[ ! -f "\$pid_file" ]]; then
        echo_red "Not running"
        return 3
    fi

    pid=\$(cat "\$pid_file")
    if is_process_running "\$pid"; then
        echo_green "Running [PID: \$pid]"
        return 0
    else
        echo_red "Not running (process \$pid not found)"
        return 1
    fi
}

run_foreground() {
    local working_dir
    local result

    working_dir=$(dirname "\$resolved_jar_file")
    if ! pushd "\$working_dir" > /dev/null; then
        echo_red "Failed to change to working directory: \$working_dir"
        return 1
    fi
    "\$java_exe" "\${arguments[@]}"
    result=\$?
    popd > /dev/null
    return \$result
}

### Main Program ###

action=\${1:-help}
[[ "\$action" == "help" ]] && {
    echo "Usage: $0 {start|stop|force-stop|restart|force-reload|status|run}"
    exit 0
}
shift
case "\$action" in
    start)          start_application "\$@" ;;
    stop)           stop_application "\$@" ;;
    force-stop)     force_stop_application "\$@" ;;
    restart)        restart_application "\$@" ;;
    force-reload)   force_reload_application "\$@" ;;
    status)         check_status "\$@" ;;
    run)            run_foreground "\$@" ;;
esac
exit \$?
EOF
	fi

	if [[ "$GOLANG_ENABLED" == "true" ]]; then
		sudo tee "$script_path" <<EOF >/dev/null
#!/bin/bash
# Go Application Management Script for ${APP_DIR} structure

set -euo pipefail

### Initialization Section ###

# Basic configuration
readonly APP_BASE_DIR="${APP_DIR}"
readonly BIN_FILE="${APP_BIN_DIR}/${APP_NAME}"
readonly CONFIG_FILE="${APP_CONFIG_DIR}/prod.yaml"
readonly DEBUG=\${DEBUG:-false}
readonly STOP_WAIT_TIME=\${STOP_WAIT_TIME:-30} # Default stop wait time in seconds


# Debug mode
if [[ "\$DEBUG" == "true" ]]; then
    set -x
fi

# Working directories
readonly WORKING_DIR="${APP_DIR}"
readonly SCRIPT_DIR="${APP_SCRIPTS_DIR}"

### Color Output Functions ###
echo_color() {
    local color_code=\$1
    shift
    echo -e "\033[\${color_code}m\$*\033[0m"
}

echo_red() { echo_color "0;31" "\$@"; }
echo_green() { echo_color "0;32" "\$@"; }
echo_yellow() { echo_color "0;33" "\$@"; }
echo_blue() { echo_color "0;34" "\$@"; }

### Directory Functions ###
ensure_directory() {
    local dir=\$1
    local owner=\${2:-}

    if [[ ! -d "\$dir" ]]; then
        if ! mkdir -p "\$dir"; then
            echo_red "Failed to create directory: \$dir"
            return 1
        fi
        if [[ -n "\$owner" ]]; then
            chown "\$owner" "\$dir"
        fi
    fi

    if [[ ! -w "\$dir" ]]; then
        echo_red "No write permission to directory: \$dir"
        return 1
    fi
}

### Process Management ###
is_process_running() {
    local pid=\$1
    ps -p "\$pid" > /dev/null 2>&1
}

### Configuration ###
# File and directory setup
PID_FOLDER=\${PID_FOLDER:-"${APP_PID_DIR}"}
LOG_FOLDER=\${LOG_FOLDER:-"${APP_LOG_DIR}"}
LOG_FILENAME=\${LOG_FILENAME:-"${APP_NAME}.log"}

# Final paths
readonly PID_FILE="\${PID_FOLDER}/${APP_NAME}.pid"
readonly LOG_FILE="\${LOG_FOLDER}/\${LOG_FILENAME}"

### User Management ###
determine_run_user() {
    local bin_file="\$1" 
    if [[ -n "\${RUN_AS_USER:-}" ]]; then
        if ! id -u "\$RUN_AS_USER" > /dev/null 2>&1; then
            echo_red "User \$RUN_AS_USER does not exist"
            exit 2
        fi
        echo "\$RUN_AS_USER"
    elif [[ \$(id -u) -eq 0 ]]; then
        stat -c '%U' "\$bin_file" 2>/dev/null || echo "root"
    else
        id -un
    fi
}

# readonly run_user=\$(determine_run_user)

### Operation Functions ###
start_application() {
    local pid
    local run_user

    # Determine the user to run the application as
    run_user=\$(determine_run_user "\$BIN_FILE")

    # Check if already running
    if [[ -f "\$PID_FILE" ]]; then
        pid=\$(cat "\$PID_FILE")
        if is_process_running "\$pid"; then
            echo_yellow "Application is already running [PID: \$pid]"
            return 0
        fi
    fi

    # Prepare directories
    if ! ensure_directory "\$PID_FOLDER" "\$run_user"; then
        return \$?
    fi
    if ! ensure_directory "\$LOG_FOLDER" "\$run_user"; then
        return \$?
    fi

    # Start the application
    echo_blue "Starting ${APP_NAME}..."
    
    if [[ \$(id -u) -eq 0 ]]; then
        # Running as root - use su to switch user
        nohup su -s /bin/sh -c "cd \$WORKING_DIR && \$BIN_FILE -conf \$CONFIG_FILE" "\$run_user" >> "\$LOG_FILE" 2>&1 &
    else
        # Running as regular user
        nohup "\$BIN_FILE" -conf "\$CONFIG_FILE" >> "\$LOG_FILE" 2>&1 &
    fi

    pid=\$!
    echo "\$pid" > "\$PID_FILE"
    echo_green "Application started successfully [PID: \$pid]"
}

stop_application() {
    local pid
    local stop_wait_time=\${STOP_WAIT_TIME:-30}

    if [[ -f "\$PID_FILE" ]]; then
        pid=\$(cat "\$PID_FILE")
        if ! is_process_running "\$pid"; then
            echo_yellow "Not running (process \$pid not found). Removing stale PID file."
            rm -f "\$PID_FILE"
            return 0
        fi

        echo_blue "Stopping application [PID: \$pid]..."
        kill "\$pid" || { echo_red "Failed to send TERM signal"; return 1; }

        # Wait for graceful shutdown
        for ((i=0; i<stop_wait_time; i++)); do
            if ! is_process_running "\$pid"; then
                echo_green "Application stopped successfully"
                rm -f "\$PID_FILE"
                return 0
            fi
            sleep 1
        done

        echo_red "Failed to stop within \$stop_wait_time seconds"
        return 1
    else
        echo_yellow "Not running (PID file not found)"
        return 0
    fi
}

force_stop_application() {
    local pid

    if [[ -f "\$PID_FILE" ]]; then
        pid=\$(cat "\$PID_FILE")
        if ! is_process_running "\$pid"; then
            echo_yellow "Not running (process \$pid not found). Removing stale PID file."
            rm -f "\$PID_FILE"
            return 0
        fi

        echo_blue "Force stopping application [PID: \$pid]..."
        kill -9 "\$pid" || { echo_red "Failed to send KILL signal"; return 1; }

        # Verify process is gone
        sleep 1
        if is_process_running "\$pid"; then
            echo_red "Failed to force stop application"
            return 1
        else
            echo_green "Application force stopped"
            rm -f "\$PID_FILE"
            return 0
        fi
    else
        echo_yellow "Not running (PID file not found)"
        return 0
    fi
}


restart_application() {
    stop_application && start_application
}

check_status() {
    if [[ -f "\$PID_FILE" ]]; then
        local pid
        pid=\$(cat "\$PID_FILE")
        if is_process_running "\$pid"; then
            local user=\$(ps -p "\$pid" -o user= | tr -d ' ')
            echo_green "Running [PID: \$pid] as user: \$user"
            return 0
        else
            echo_red "Not running (stale PID file)"
            return 1
        fi
    else
        echo_red "Not running"
        return 3
    fi
}

run_foreground() {
    cd "\$WORKING_DIR"
    exec "\$BIN_FILE" -conf "\$CONFIG_FILE" "\$@"
}

### Main Program ###
action=\${1:-help}
shift

case "\$action" in
    start)          start_application "\$@" ;;
    stop)           stop_application "\$@" ;;
    force-stop)     force_stop_application "\$@" ;;
    restart)        restart_application "\$@" ;;
    status)         check_status "\$@" ;;
    run)            run_foreground "\$@" ;;
    help|*)         echo "Usage: \$0 {start|stop|force-stop|restart|status|run|help}"
                    exit 0 ;;
esac

exit \$?
EOF
	fi

	chmod +x "$script_path"
	sudo chown "$APP_USER:$APP_USER" "$script_path"
	green "Control script created: $script_path"
}

init_server() {
	blue "Preparing server's application workspace..."
	sudo id -u "$APP_USER" >/dev/null 2>&1 || sudo useradd -m -s /bin/bash -d "/home/$APP_USER" "$APP_USER"
	sudo id -u "$APP_USER" >/dev/null 2>&1 && echo "$APP_USER:$APP_PASSWORD" | sudo chpasswd && green "Created user: $APP_USER:$APP_PASSWORD, please remember it not to share it with others!"
	sudo mkdir -p "$APP_BIN_DIR" "$APP_CONFIG_DIR" "$APP_LIB_DIR" "$APP_SCRIPTS_DIR" "$APP_LOG_DIR" "$APP_PID_DIR"
	sudo touch "$APP_LOG_DIR/example.log"
	sudo chown -R "$APP_USER:$APP_USER" "$APP_DIR" "$APP_LOG_DIR" "$APP_PID_DIR"
	sudo chmod 750 "$APP_DIR"
	sudo chmod 640 "$APP_LOG_DIR"/*.log 2>/dev/null || true
	create_app_control_script
	open_firewall_port
	green "server's workspace preparation completed!"
}

# Open firewall port
open_firewall_port() {
	sudo sudo ufw allow ssh
	sudo sudo ufw allow 80
	sudo sudo ufw enable
}

# Prepare client's SSH configuration
init_client() {
	[[ -f "$PRIVATE_KEY" ]] || {
		red "Generating SSH key pair..."
		ssh-keygen -t rsa -b 4096 -N "" -C "50849806@qq.com" -f "$PRIVATE_KEY"
	}
	ssh-copy-id -i "$PUBLIC_KEY" -p "$SSH_PORT" "$APP_USER@$SERVER_IP"
}

# Install software
install_software() {
	wait_for_apt_lock() {
		blue "Waiting for the APT system to be ready .."
		while fuser /var/lib/dpkg/lock /var/lib/apt/lists/lock /var/cache/apt/archives/lock >/dev/null 2>&1; do
			sleep 0.5
		done
		green "APT ready"
	}

	install_basic_tools() {
		blue "Install basic tools ..."
		wait_for_apt_lock
		if ! sudo apt-get install -y sudo vim curl lsof; then
			red "Basic tool installation failed"
			return 1
		fi
		green "Basic tool installation completed"
	}

	install_java() {
		if [[ "$JAVA_ENABLED" == "true" ]]; then
			blue "Install Java ($JAVA_VERSION)..."
			wait_for_apt_lock
			if ! sudo apt-get install -y "$JAVA_VERSION"; then
				red "Java installation failed"
				return 1
			fi
			green "Java installation completed"
		fi
	}

	install_docker() {
		if [[ "$DOCKER_ENABLED" == "true" ]]; then
			blue "Install Docker..."
			wait_for_apt_lock

			if ! sudo apt-get install -y docker.io; then
				red "Docker installation failed"
				return 1
			fi

			if ! sudo usermod -aG docker "$APP_USER"; then
				red "Unable to add user to Docker group"
				return 1
			fi

			blue "Start Docker service..."
			if ! sudo systemctl restart docker; then
				red "Docker service start failed"
				return 1
			fi

			# 等待Docker服务完全就绪
			local timeout=10
			blue "Wait for Docker to be ready (maximum ${timeout} seconds) .."
			if ! timeout $timeout bash -c 'until docker info >/dev/null 2>&1; do sleep 1; done'; then
				red "Docker service not ready in a timely manner"
				return 1
			fi

			green "Docker installation and configuration completed"
		fi
	}

	install_docker_compose() {
		if [[ "$DOCKER_COMPOSE_ENABLED" == "true" ]]; then
			blue "Install Docker Compose..."
			wait_for_apt_lock
			if ! sudo apt-get install -y docker-compose; then
				red "Docker Compose installation failed"
				return 1
			fi
			green "Docker Compose installation completed"
		fi
	}

	install_basic_tools &&
		install_java &&
		install_docker &&
		install_docker_compose
}

main() {
	local action="${1:-help}"
	handleAction "$action"
}

handleAction() {
	case "$1" in
	help)
		usage
		exit 0
		;;
	server)
		detect_distro
		change_mirror
		install_software
		init_server
		;;
	client)
		init_client
		;;
	*)
		echo "Usage: $0 {server|client}"
		exit 1
		;;
	esac
}

usage() {
	echo "Usage: $0 {server|client}"
}

main "$@
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