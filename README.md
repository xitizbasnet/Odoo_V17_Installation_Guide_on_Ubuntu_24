# Odoo Ver.17 Installation Guide on Ubuntu 24

This document provides a step-by-step guide to install and configure Odoo on Ubuntu 24. The instructions include setting up the system dependencies, installing PostgreSQL, configuring the Odoo service, and setting up a virtual environment for Python dependencies.

## Prerequisites

Before proceeding with the installation, ensure that you have:

- Ubuntu 24 or a compatible version installed.
- A user with `sudo` privileges to execute commands.
- A stable internet connection for downloading packages.

---

## 1. Update the System

Update the system to ensure all packages are up-to-date:

```bash
sudo apt-get update
```

```bash
sudo apt-get upgrade
```

---

## 2. Install System Dependencies

Install the required system packages for Odoo:

```bash
sudo apt install python3-minimal python3-dev python3-pip python3-venv python3-setuptools build-essential libzip-dev libxslt1-dev libldap2-dev python3-wheel libsasl2-dev node-less libjpeg-dev xfonts-utils libpq-dev libffi-dev fontconfig git wget
```

---

## 3. Install PostgreSQL Database

Install PostgreSQL, which is required for Odoo's database backend:

```bash
sudo apt-get install postgresql
```

Start and enable the PostgreSQL service:

```bash
systemctl start postgresql
```
```bash
systemctl enable postgresql
```

Check the status of PostgreSQL:

```bash
systemctl status postgresql
```

Create a PostgreSQL user for Odoo:

```bash
su - postgres -c "createuser -s odoo"
```

---

## 4. Install Node.js and npm

Install Node.js and npm to handle JavaScript dependencies:

```bash
apt install nodejs npm
```

```bash
npm install -g rtlcss
```

Install additional fonts for Odoo:

```bash
apt-get install xfonts-75dpi xfonts-base
```

---

## 5. Install wkhtmltopdf

Odoo requires `wkhtmltopdf` to generate PDF reports. Download and install the `wkhtmltopdf` package:

```bash
wget http://security.ubuntu.com/ubuntu/pool/universe/w/wkhtmltopdf/wkhtmltopdf_0.12.6-2build2_amd64.deb
```

```bash
dpkg -i wkhtmltopdf_0.12.6-2build2_amd64.deb
```

```bash
apt-get install -f
```

```bash
dpkg -i wkhtmltopdf_0.12.6-2build2_amd64.deb
```

Verify the installation:

```bash
wkhtmltopdf --version
```

---

## 6. Create Odoo User

Create a system user for Odoo:

```bash
adduser --system --group --home=/opt/odoo --shell=/bin/bash odoo
```

Switch to the Odoo user:

```bash
su - odoo
```

---

## 7. Install Odoo

Clone the Odoo repository from GitHub:

```bash
git clone https://www.github.com/odoo/odoo --depth 1 --branch 17.0 /opt/odoo/odoo
```

Create a virtual environment for Python:

```bash
python3 -m venv odoo-env
```

Activate the virtual environment:

```bash
source odoo-env/bin/activate
```

Install required Python dependencies:

```bash
pip3 install wheel
```

```bash
pip3 install -r /opt/odoo/odoo/requirements.txt
```

Deactivate the virtual environment after installation:

```bash
clear
```

```bash
deactivate
```

---

## 8. Configure Custom Addons

Create a directory for custom addons:

```bash
mkdir /opt/odoo/custom-addons
```

```bash
exit
```

---

## 9. Set Up Log Directory

Create and set proper ownership for Odoo log files:

```bash
mkdir /var/log/odoo17
```

```bash
chown odoo:odoo /var/log/odoo17
```

---

## 10. Configure Odoo

Create and configure the Odoo configuration file:

```bash
nano /etc/odoo.conf
```

Add the following configuration:

```
[options]
admin_passwd = P@ssw0rd
db_host = False
db_port = False
db_user = odoo
db_password = False
logfile = /var/log/odoo17/odoo-server.log
addons_path = /opt/odoo/odoo/addons,/opt/odoo/custom-addons
xmlrpc_port = 8069
```

---

## 11. Create Odoo Systemd Service

Create a systemd service to manage Odoo:

```bash
nano /etc/systemd/system/odoo.service
```

Add the following content:

```
[Unit]
Description=Odoo
Requires=postgresql.service
After=network.target postgresql.service

[Service]
Type=simple
SyslogIdentifier=odoo
PermissionsStartOnly=true
User=odoo
Group=odoo
ExecStart=/opt/odoo/odoo-env/bin/python3 /opt/odoo/odoo/odoo-bin -c /etc/odoo.conf
StandardOutput=journal+console

[Install]
WantedBy=multi-user.target
```

---

## 12. Start and Enable Odoo Service

Reload the systemd daemon, start Odoo, and enable it to start on boot:

```bash
systemctl daemon-reload
```

```bash
systemctl start odoo
```

```bash
systemctl enable odoo
```

Check the status of the Odoo service:

```bash
systemctl status odoo
```

---

## 13. Access Odoo

Once the Odoo service is running, you can access the Odoo web interface by navigating to the following URL in your web browser:

```
http://<your-server-ip>:8069
```

OR

```
http://localhost:8069/
```

Use the admin password (`P@ssw0rd`) for initial setup.

---

## Conclusion

You have successfully installed and configured Odoo on Ubuntu 24. If you encounter any issues during the installation, check the Odoo logs located in `/var/log/odoo17/odoo-server.log` for troubleshooting.

For further customization, you can refer to the official [Odoo documentation](https://www.odoo.com/documentation) for more information.

---
---




# Database creation error: connection to server on socket "/var/run/postgresql/.s.PGSQL.5432" failed: FATAL: role "odoo" does not exist 
---
 
# Odoo Setup: Resolving "FATAL: role 'odoo' does not exist" Error

When setting up Odoo on Ubuntu, you may encounter the error:

FATAL: role "odoo" does not exist

---

This error occurs when PostgreSQL cannot find the `odoo` user required for database operations. To resolve this issue, follow the steps below to create the `odoo` user in PostgreSQL.


## Steps to Resolve the Issue

### 1. Log into PostgreSQL as the `postgres` user:

Open a terminal and log into PostgreSQL using the `postgres` system user:


```bash
sudo -u postgres psql
```

### 2. Create the `odoo` role:

Once you're inside the PostgreSQL prompt (`psql`), run the following command to create the `odoo` user with login credentials:

```sql
CREATE ROLE odoo WITH LOGIN PASSWORD 'odoo';
```

This creates a PostgreSQL role named `odoo`, with the ability to log in using the password `'odoo'`.

### 3. Grant the necessary permissions to the `odoo` user:

You need to grant the `odoo` user the necessary permission to create databases. Run the following command:

```sql
ALTER ROLE odoo CREATEDB;
```

This gives the `odoo` user the permission to create databases, which is essential for Odoo.

### 4. Exit the PostgreSQL prompt:

After creating the role and assigning permissions, exit the PostgreSQL prompt by typing:

```sql
\q
```

### 5. Verify the role setup:

To check if the `odoo` role was created successfully, run the following command:

```bash
sudo -u postgres psql -c "\du"
```

This will list all PostgreSQL roles, and you should see the `odoo` role listed.

## Additional Tips:

- If you are running Odoo under a specific user, make sure the PostgreSQL role name (`odoo`) matches the username configured for Odoo in your `odoo.conf` file.
- After creating the PostgreSQL user, restart the Odoo service to apply the changes:

```bash
sudo systemctl restart odoo
```

## Conclusion:

By creating the `odoo` role in PostgreSQL and granting it the `CREATEDB` permission, you should be able to resolve the error and successfully set up your Odoo instance.

---
