#!/bin/bash

# This script automates the installation of ODAT and its dependencies,
# making it easy to run from anywhere on the system.

# --- Let's set some variables first ---
ORACLE_VERSION="21.4.0.0.0dbru"
ORACLE_DIR="/opt/oracle"
INSTANT_CLIENT_DIR="${ORACLE_DIR}/instantclient_21_4"
ODAT_DIR="/opt/odat"
VENV_DIR="${ODAT_DIR}/venv"

# --- Gotta make sure we're root to do this stuff ---
if [ "$(id -u)" != "0" ]; then
   echo "Hey, you need to run this script as root (or with sudo) to get it to work." 1>&2
   exit 1
fi

# --- 1. Download and install the Oracle Instant Client ---
echo "Alright, starting the installation! First up, downloading and installing the Oracle Instant Client..."
wget -q "https://download.oracle.com/otn_software/linux/instantclient/214000/instantclient-basic-linux.x64-${ORACLE_VERSION}.zip"
wget -q "https://download.oracle.com/otn_software/linux/instantclient/214000/instantclient-sqlplus-linux.x64-${ORACLE_VERSION}.zip"
mkdir -p "${INSTANT_CLIENT_DIR}"
unzip -q -o "instantclient-basic-linux.x64-${ORACLE_VERSION}.zip" -d "${ORACLE_DIR}"
unzip -q -o "instantclient-sqlplus-linux.x64-${ORACLE_VERSION}.zip" -d "${ORACLE_DIR}"
rm "instantclient-basic-linux.x64-${ORACLE_VERSION}.zip" "instantclient-sqlplus-linux.x64-${ORACLE_VERSION}.zip"

# --- 2. Now let's grab ODAT itself ---
echo "Next, we're cloning the ODAT repository into /opt/odat."
git clone https://github.com/quentinhardy/odat.git "${ODAT_DIR}"
chown -R root:root "${ODAT_DIR}"

# --- 3. Set up the virtual environment so nothing breaks ---
echo "Creating a dedicated Python virtual environment for ODAT. This keeps things tidy!"
dpkg -s python3-venv >/dev/null 2>&1 || {
    echo "Looks like python3-venv isn't installed. Installing it for you..."
    apt-get update
    apt-get install python3-venv -y
}
python3 -m venv "${VENV_DIR}"

# --- 4. Install all the Python libraries ODAT needs ---
echo "Installing ODAT's Python dependencies. This might take a minute..."
source "${VENV_DIR}/bin/activate"
pip install python-libnmap cx_Oracle pycryptodome passlib colorlog termcolor
git -C "${ODAT_DIR}" submodule init
git -C "${ODAT_DIR}" submodule update
deactivate

# --- 5. Make a system-wide command for ODAT ---
echo "Creating a simple 'odat' command so you don't have to type the whole path every time."
cat << 'EOF' > /usr/local/bin/odat
#!/bin/bash
source /opt/odat/venv/bin/activate
/opt/odat/venv/bin/python3 /opt/odat/odat.py "$@"
EOF

chmod +x /usr/local/bin/odat

# --- 6. Set up the environment paths for the Oracle client ---
echo "Setting up the environment paths so your system can find the Oracle libraries."
cat << 'EOF' > /etc/profile.d/oracle.sh
#!/bin/bash
export LD_LIBRARY_PATH="/opt/oracle/instantclient_21_4:$LD_LIBRARY_PATH"
export PATH="$LD_LIBRARY_PATH:$PATH"
EOF
chmod +x /etc/profile.d/oracle.sh

echo ""
echo "Done! ODAT is now installed and ready to go for all users. 🎉"
echo "To use it, just open a new terminal and type: sudo odat"
echo "You might need to log out and log back in for the changes to fully kick in."
