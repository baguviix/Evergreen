# Installing the Evergreen Server

## Table of Contents
- [Preamble: Referenced User Accounts](#preamble-referenced-user-accounts)
- [Preamble: Developer Instructions](#preamble-developer-instructions)
- [Installing Prerequisites](#installing-prerequisites)
  - [Optional: Developer Additions](#optional-developer-additions)
- [Extra Steps for Web Staff Client](#extra-steps-for-web-staff-client)
  - [Install AngularJS Files for Web Staff Client](#install-angularjs-files-for-web-staff-client)
  - [Install Angular Files for Web Staff Client](#install-angular-files-for-web-staff-client)
  - [Install OPAC Skin Dependencies](#install-opac-skin-dependencies)
- [Configuration and Compilation Instructions](#configuration-and-compilation-instructions)
- [Installation Instructions](#installation-instructions)
- [Change Ownership of the Evergreen Files](#change-ownership-of-the-evergreen-files)
- [Run ldconfig](#run-ldconfig)
- [Additional Instructions for Developers](#additional-instructions-for-developers)
- [Configure the Apache Web Server](#configure-the-apache-web-server)
- [Configure OpenSRF for the Evergreen Application](#configure-opensrf-for-the-evergreen-application)
  - [Optional: Configure Evergreen for OpenSRF + Redis](#optional-configure-evergreen-for-opensrf--redis)
  - [Configure Action Triggers for the Evergreen Application](#configure-action-triggers-for-the-evergreen-application)
- [Creating the Evergreen Database](#creating-the-evergreen-database)
  - [Setting Up the PostgreSQL Server](#setting-up-the-postgresql-server)
  - [Creating the Evergreen Database and Schema](#creating-the-evergreen-database-and-schema)
  - [Loading Sample Data](#loading-sample-data)
  - [Creating the Database on a Remote Server](#creating-the-database-on-a-remote-server)
- [Starting Evergreen](#starting-evergreen)
- [Testing Connections to Evergreen](#testing-connections-to-evergreen)
- [Getting Help](#getting-help)
- [License](#license)

---

## Preamble: Referenced User Accounts

In subsequent sections, we will refer to a number of different accounts, as follows:

### Linux User Accounts
* **The `user` Linux account:** The account that you use to log onto the Linux system as a regular user.
* **The `root` Linux account:** An account that has system administrator privileges.
  * On **Debian**, you can switch to this account from your `user` account by issuing `su -` and entering the password for the `root` account when prompted.
  * On **Ubuntu**, you can switch to this account from your `user` account using `sudo su -` and entering the password for your `user` account when prompted.
* **The `opensrf` Linux account:** An account that you create when installing OpenSRF. You can switch to this account from the `root` account by issuing `su - opensrf`.
* **The `postgres` Linux account:** Created automatically when you install the PostgreSQL database server. You can switch to this account from the `root` account by issuing `su - postgres`.

### PostgreSQL User Accounts
* **The `evergreen` PostgreSQL account:** A superuser account that you will create to connect to the PostgreSQL database server.

### Evergreen Administrator Account
* **The `egadmin` Evergreen account:** An administrator account for Evergreen that you will use to test connectivity and configure your Evergreen instance.

---

## Preamble: Developer Instructions

> [!NOTE]
> Skip this section if you are using an official release tarball downloaded from [evergreen-ils.org/egdownloads](http://evergreen-ils.org/egdownloads).

Developers working directly with the source code from the Git repository, rather than an official release tarball, must perform one step before they can proceed with the `./configure` step.

As the **`user`** Linux account, issue the following command in the Evergreen source directory to generate the configure script and Makefiles:

```bash
autoreconf -i
```

---

## Installing Prerequisites

* **PostgreSQL:** The minimum supported version is **14**.
* **Linux:** Evergreen has been tested on:
  * Debian Trixie (13)
  * Debian Bookworm (12)
  * Debian Bullseye (11)
  * Debian Buster (10)
  * Ubuntu Noble Numbat (24.04)
  * Ubuntu Jammy Jellyfish (22.04)
  *(If you are running an older version of these distributions, you may want to upgrade before upgrading Evergreen. For instructions, visit the Debian or Ubuntu websites.)*
* **OpenSRF:** The minimum supported version of OpenSRF is **3.3.0**.

Evergreen has a number of prerequisite packages that must be installed before you can successfully configure, compile, and install Evergreen.

1. Begin by installing the most recent version of OpenSRF (3.3.0 or later). You can download OpenSRF releases from [OpenSRF Downloads](http://evergreen-ils.org/opensrf-downloads/).

2. Issue the following commands as the **`root`** Linux account to install prerequisites using the `Makefile.install` prerequisite installer, substituting `debian-bookworm`, `debian-bullseye`, `debian-buster`, `ubuntu-jammy`, or `ubuntu-noble` for `<osname>` below:

   ```bash
   make -f Open-ILS/src/extras/Makefile.install <osname>
   ```

   > [!TIP]
   > To save time while installing prerequisites, set the environment variable `NO_CPAN_TEST` to `1`:
   > ```bash
   > NO_CPAN_TEST=1 make -f Open-ILS/src/extras/Makefile.install <osname>
   > ```

### Optional: Developer Additions

To perform certain developer tasks from a Git source code checkout, additional packages are required. As the **`root`** Linux account:

* **Web dependencies:** To install packages needed for retrieving and managing web dependencies, use the `<osname>-developer` Makefile.install target *(currently only needed for building and installing the web staff client)*.
* **Testing:** To optionally run Angular and AngularJS tests, you will need to manually install Firefox and your choice of Chromium or Chrome.

```bash
make -f Open-ILS/src/extras/Makefile.install <osname>-developer
```

* **Translations:** To install packages required for building Evergreen translations, use the `<osname>-translator` Makefile.install target:

```bash
make -f Open-ILS/src/extras/Makefile.install <osname>-translator
```

* **Release bundles:** To install packages required for building Evergreen release bundles, use the `<osname>-packager` Makefile.install target:

```bash
make -f Open-ILS/src/extras/Makefile.install <osname>-packager
```

---

## Extra Steps for Web Staff Client

> [!NOTE]
> Skip this entire section if you are using an official release tarball downloaded from [evergreen-ils.org/downloads](http://evergreen-ils.org/downloads). Otherwise, ensure you have installed the [Optional Developer Additions](#optional-developer-additions) before proceeding.

### Install AngularJS Files for Web Staff Client

1. **Navigate to the staff JS web root:**
   ```bash
   cd $EVERGREEN_ROOT/Open-ILS/web/js/ui/default/staff/
   ```

2. **Install project-local dependencies** via `npm`:
   ```bash
   npm ci   # fetch JS dependencies
   ```

3. **Run the production build script:**
   ```bash
   npm run build-prod
   ```

4. **OPTIONAL: Test web client code** (requires `<osname>-developer` packages and browsers installed):
   ```bash
   CHROME_BIN=/path/to/chrome npm run test
   ```

---

### Install Angular Files for Web Staff Client

1. **Navigate to the Angular staff root:**
   ```bash
   cd $EVERGREEN_ROOT/Open-ILS/src/eg2/
   ```

2. **Install project-local dependencies:**
   ```bash
   npm ci   # fetch JS dependencies
   ```

3. **Run the build script:**
   ```bash
   npx ng build --configuration=production
   ```

   > [!NOTE]
   > This can be a memory-intensive build. If the process does not finish and you get the message `"Killed"` in the console, try running it with an explicit [max-old-space-size option](https://nodejs.org/api/cli.html#cli_max_old_space_size_size_in_megabytes) to encourage more garbage collection. For example, on a machine with 4GB of memory, limit it to 3GB with:
   > ```bash
   > NODE_OPTIONS=--max-old-space-size=3072 npx ng build --configuration=production
   > ```

4. **OPTIONAL: Test eg2 web client code:**
   ```bash
   CHROME_BIN=/path/to/chrome npm run test
   MOZ_HEADLESS=1 npx ng e2e
   ```

---

### Install OPAC Skin Dependencies

1. **Navigate to the OPAC dependencies root:**
   ```bash
   cd $EVERGREEN_ROOT/Open-ILS/web/opac/deps
   ```

2. **Install project-local dependencies:**
   ```bash
   npm ci   # fetch JS and CSS dependencies
   ```
   *(Note: There is no build step.)*

3. **OPTIONAL: Test OPAC JavaScript code:**
   ```bash
   npm run test
   ```

---

## Configuration and Compilation Instructions

For the time being, we are still installing everything in the `/openils/` directory. From the Evergreen source directory, issue the following commands as the **`user`** Linux account to configure and build Evergreen:

```bash
PATH=/openils/bin:$PATH ./configure --prefix=/openils --sysconfdir=/openils/conf
make
```

> [!NOTE]
> These instructions assume that you have also installed OpenSRF under `/openils/`. If not, please adjust `PATH` as needed so that the Evergreen `configure` script can find `osrf_config`.

---

## Installation Instructions

Once you have configured and compiled Evergreen, issue the following command as the **`root`** Linux account to install Evergreen and copy example configuration files to `/openils/conf`:

```bash
make install
```

---

## Change Ownership of the Evergreen Files

All files in the `/openils/` directory and subdirectories must be owned by the `opensrf` user. Issue the following command as the **`root`** Linux account:

```bash
chown -R opensrf:opensrf /openils
```

---

## Run ldconfig

On Ubuntu or Debian, run the following command as the **`root`** user:

```bash
ldconfig
```

---

## Additional Instructions for Developers

> [!NOTE]
> Skip this section if you are using an official release tarball downloaded from [evergreen-ils.org/egdownloads](http://evergreen-ils.org/egdownloads).

Developers working directly with the source code from the Git repository, rather than an official release tarball, need to install the Dojo Toolkit set of JavaScript libraries. The appropriate version of Dojo is included in Evergreen release tarballs. Developers should install Dojo version 1.3.3 by issuing the following commands as the **`opensrf`** Linux account:

```bash
wget https://download.dojotoolkit.org/release-1.3.3/dojo-release-1.3.3.tar.gz
tar -C /openils/var/web/js -xzf dojo-release-1.3.3.tar.gz
cp -r /openils/var/web/js/dojo-release-1.3.3/* /openils/var/web/js/dojo/.
```

---

## Configure the Apache Web Server

1. **Copy example configuration files** for the Evergreen catalog, web staff client, web services, and administration interfaces. Issue the following commands as the **`root`** Linux account:

   ```bash
   cp Open-ILS/examples/apache_24/eg_24.conf       /etc/apache2/sites-available/eg.conf
   cp Open-ILS/examples/apache_24/eg_vhost_24.conf /etc/apache2/eg_vhost.conf
   cp Open-ILS/examples/apache_24/eg_startup      /etc/apache2/

   # Set up SSL
   mkdir /etc/apache2/ssl
   cd /etc/apache2/ssl
   ```

2. **Generate SSL Key & Certificate:**
   The `openssl` command cuts a new SSL key for your Apache server. For a production server, purchase a signed SSL certificate; for testing/development, a self-signed certificate can be used:

   ```bash
   openssl req -new -x509 -days 365 -nodes -out server.crt -keyout server.key
   ```

3. **Edit `eg.conf`:**
   As the **`root`** Linux account, edit the `/etc/apache2/sites-available/eg.conf` file:
   * To enable access to the offline upload / execute interface from any workstation on any network (ensure this is secured for production!):
     * Replace `Require host 10.0.0.0/8` with `Require all granted`

4. **Change the Apache user:**
   * Edit `/etc/apache2/envvars` as **`root`**.
   * Change `export APACHE_RUN_USER=www-data` to:
     ```bash
     export APACHE_RUN_USER=opensrf
     ```

5. **Configure Apache KeepAlive settings:**
   Higher values can improve client performance by allowing multiple requests over the same TCP connection, but increase the risk of using up Apache child processes and memory.
   * Edit `/etc/apache2/apache2.conf` as **`root`**:
     * Change `KeepAliveTimeout` to `1`
     * Change `MaxKeepAliveRequests` to `100`

6. **Configure the prefork module:**
   Set up `/etc/apache2/mods-available/mpm_prefork.conf` with values suitable for a default Evergreen site:

   ```apache
   <IfModule mpm_prefork_module>
      StartServers            15
      MinSpareServers          5
      MaxSpareServers         15
      MaxRequestWorkers       75
      MaxConnectionsPerChild 500
   </IfModule>
   ```

7. **Enable the `mpm_prefork` module** as **`root`**:
   ```bash
   a2dismod mpm_event
   a2enmod mpm_prefork
   ```

8. **Enable the Evergreen site:**
   ```bash
   a2dissite 000-default  # OPTIONAL: disable the default "It Works" page
   a2ensite eg.conf
   ```

9. **Allow Apache to write to the lock directory** *(skip this step on Debian 13)*:
   ```bash
   chown opensrf /var/lock/apache2
   ```

---

## Configure OpenSRF for the Evergreen Application

There are example OpenSRF configuration files in `/openils/conf/` that serve as templates. Issue the following commands as the **`opensrf`** Linux account:

```bash
cp -b /openils/conf/opensrf_core.xml.example /openils/conf/opensrf_core.xml
cp -b /openils/conf/opensrf.xml.example /openils/conf/opensrf.xml
```

> [!NOTE]
> The `-b` flag tells `cp` to create a backup file with a tilde (`~`) appended.
> 
> When installing OpenSRF, you created four Jabber users on two separate domains. Edit `/openils/conf/opensrf_core.xml` using those exact Jabber users and domains.
> 
> `eg_db_config` (detailed in [Creating the Evergreen Database](#creating-the-evergreen-database)) will automatically configure database connection parameters in `opensrf.xml`.

### Optional: Configure Evergreen for OpenSRF + Redis

If using the Redis variant of OpenSRF, modify `/openils/conf/opensrf_core.xml` to use the Redis settings instead of Ejabberd.

For each configuration block that contains both Ejabberd and Redis sections, comment out Ejabberd and un-comment Redis:

**Before:**
```xml
<!-- Ejabberd -->                                                          
<passwd>password</passwd>                                                  
<port>5222</port>                                                          
<!-- ===  -->                                                              
                                                                           
<!-- Redis -->                                                             
<!--                                                                       
<passwd>456fc340-beba-4489-9070-0d6b49e9952b</passwd>                      
<port>6379</port>                                                          
-->                                                                        
<!-- ===  -->
```

**After:**
```xml
<!-- Ejabberd -->                                                          
<!--
<passwd>password</passwd>                                                  
<port>5222</port>                                                          
-->
<!-- ===  -->                                                              
                                                                           
<!-- Redis -->                                                             
<passwd>456fc340-beba-4489-9070-0d6b49e9952b</passwd>                      
<port>6379</port>                                                          
<!-- ===  -->
```

### Configure Action Triggers for the Evergreen Application

*Action Triggers* provide hooks for the system to perform actions when a given event occurs (e.g., generating reminder or overdue notices via `checkout.due`).

To enable the default set of hooks, issue the following command as the **`opensrf`** Linux account:

```bash
cp -b /openils/conf/action_trigger_filters.json.example /openils/conf/action_trigger_filters.json
```

---

## Creating the Evergreen Database

### Setting Up the PostgreSQL Server

For production use, most libraries install PostgreSQL on a dedicated machine. Therefore, by default, `Makefile.install` does **not** install the PostgreSQL server package.

Install the required PostgreSQL packages as the **`root`** Linux account using the make target matching your distribution and target PostgreSQL version:

> [!WARNING]
> **PostgreSQL 12+ JIT (Just-in-Time compilation):**
> Do **not** turn on PostgreSQL's JIT capabilities. Evergreen's queries, especially complex ones used for search, are intentionally tuned for non-JIT execution, and JIT has been shown to be harmful in some circumstances.
> 
> Recommended minimum tweak to `postgresql.conf`:
> ```ini
> jit_above_cost = -1
> ```

#### PostgreSQL Version Options:

<details>
<summary><strong>PostgreSQL 14</strong></summary>

```bash
make -f Open-ILS/src/extras/Makefile.install postgres-server-debian-trixie-14
make -f Open-ILS/src/extras/Makefile.install postgres-server-debian-bookworm-14
make -f Open-ILS/src/extras/Makefile.install postgres-server-debian-bullseye-14
make -f Open-ILS/src/extras/Makefile.install postgres-server-debian-buster-14
make -f Open-ILS/src/extras/Makefile.install postgres-server-ubuntu-jammy-14
make -f Open-ILS/src/extras/Makefile.install postgres-server-ubuntu-noble-14
```
</details>

<details>
<summary><strong>PostgreSQL 15</strong></summary>

```bash
make -f Open-ILS/src/extras/Makefile.install postgres-server-debian-trixie-15
make -f Open-ILS/src/extras/Makefile.install postgres-server-debian-bookworm-15
make -f Open-ILS/src/extras/Makefile.install postgres-server-debian-bullseye-15
make -f Open-ILS/src/extras/Makefile.install postgres-server-debian-buster-15
make -f Open-ILS/src/extras/Makefile.install postgres-server-ubuntu-jammy-15
make -f Open-ILS/src/extras/Makefile.install postgres-server-ubuntu-noble-15
```
</details>

<details>
<summary><strong>PostgreSQL 16</strong></summary>

```bash
make -f Open-ILS/src/extras/Makefile.install postgres-server-debian-trixie-16
make -f Open-ILS/src/extras/Makefile.install postgres-server-debian-bookworm-16
make -f Open-ILS/src/extras/Makefile.install postgres-server-debian-bullseye-16
make -f Open-ILS/src/extras/Makefile.install postgres-server-debian-buster-16
make -f Open-ILS/src/extras/Makefile.install postgres-server-ubuntu-jammy-16
make -f Open-ILS/src/extras/Makefile.install postgres-server-ubuntu-noble-16
```
</details>

<details>
<summary><strong>PostgreSQL 17</strong></summary>

```bash
make -f Open-ILS/src/extras/Makefile.install postgres-server-debian-trixie-17
make -f Open-ILS/src/extras/Makefile.install postgres-server-debian-bookworm-17
make -f Open-ILS/src/extras/Makefile.install postgres-server-debian-bullseye-17
make -f Open-ILS/src/extras/Makefile.install postgres-server-debian-buster-17
make -f Open-ILS/src/extras/Makefile.install postgres-server-ubuntu-jammy-17
make -f Open-ILS/src/extras/Makefile.install postgres-server-ubuntu-noble-17
```
</details>

<details>
<summary><strong>PostgreSQL 18</strong></summary>

```bash
make -f Open-ILS/src/extras/Makefile.install postgres-server-debian-trixie-18
make -f Open-ILS/src/extras/Makefile.install postgres-server-debian-bookworm-18
make -f Open-ILS/src/extras/Makefile.install postgres-server-debian-bullseye-18
make -f Open-ILS/src/extras/Makefile.install postgres-server-debian-buster-18
make -f Open-ILS/src/extras/Makefile.install postgres-server-ubuntu-jammy-18
make -f Open-ILS/src/extras/Makefile.install postgres-server-ubuntu-noble-18
```
</details>

---

#### Create the Evergreen PostgreSQL User

Issue the following command as the **`postgres`** Linux account to create a new superuser named `evergreen`. When prompted, enter a secure password:

```bash
createuser -s -P evergreen
```

#### Enabling Connections to the PostgreSQL Database

Check your `pg_hba.conf` file (located under `/etc/postgresql/` on Debian and Ubuntu). To allow TCP/IP connections from `localhost` with password authentication for testing, ensure the following lines appear **before** any `host ... ident` entries:

```text
host    all             all             ::1/128                 md5
host    all             all             127.0.0.1/32            md5
```

Reload PostgreSQL after updating `pg_hba.conf` for changes to take effect.

---

### Creating the Evergreen Database and Schema

From inside the Evergreen source directory, run `eg_db_config` as the **`root`** Linux account:

```bash
perl Open-ILS/src/support-scripts/eg_db_config --update-config \
       --service all --create-database --create-schema --create-offline \
       --user <user> --password <password> --hostname <hostname> --port <port> \
       --database <dbname> --admin-user <admin-user> --admin-pass <admin-pass>
```

* `<user>` & `<password>`: Credentials for the `evergreen` PostgreSQL account.
* `<admin-user>` & `<admin-pass>`: Desired credentials for the `egadmin` Evergreen administrator account.

*(Run with `--help` to inspect all available options.)*

---

### Loading Sample Data

* **Standard *Concerto* Dataset:**
  Add `--load-all-sample` to `eg_db_config` to load authority/bibliographic records, call numbers, copies, staff/regular users, and sample transactions.
* **Enhanced *Concerto* Dataset:**
  Use `--load-concerto-enhanced` for additional testing data, including:
  * Friendly organization unit names
  * Acquisitions and billing scenarios
  * Shelving locations & settings
  * Multilingual bibliographic records (Japanese, Spanish, French, Czech)
  * Metarecord holds & item stat categories
  * Bookings, pre-created OPAC carousels, and serials data

---

### Creating the Database on a Remote Server

To provision the database on a remote dedicated server, include the `--create-database` flag when running `eg_db_config`.

---

## Starting Evergreen

1. **Start background services** as **`root`**:
   ```bash
   /etc/init.d/ejabberd start
   /etc/init.d/memcached start
   ```

2. **Start OpenSRF / Evergreen services** as **`opensrf`**:
   ```bash
   osrf_control -l --start-all
   ```
   *(Note: The `-l` flag forces `localhost`. Omit it if you configured `opensrf.xml` using your machine's fully qualified domain name.)*

   > [!TIP]
   > If you see `bash: osrf_control: command not found`, add `/openils/bin` to your `PATH` in `~/.bashrc`:
   > ```bash
   > export PATH=$PATH:/openils/bin
   > ```

3. **Generate web files and org unit proximity** as **`opensrf`**:
   ```bash
   autogen.sh
   ```
   *(Run this whenever library organization unit configurations change.)*

4. **Restart Apache** as **`root`**:
   ```bash
   /etc/init.d/apache2 restart
   ```

---

## Testing Connections to Evergreen

Test system connectivity using `srfsh` as the **`opensrf`** Linux account:

```bash
/openils/bin/srfsh
srfsh% login <admin-user> <admin-pass>
```

**Expected output:**
```text
Received Data: "250bf1518c7527a03249858687714376"
------------------------------------
Request Completed Successfully
Request Time in seconds: 0.045286
------------------------------------

Received Data: {
   "ilsevent":0,
   "textcode":"SUCCESS",
   "desc":" ",
   "pid":21616,
   "stacktrace":"oils_auth.c:304",
   "payload":{
      "authtoken":"e5f9827cc0f93b503a1cc66bee6bdd1a",
      "authtime":420
   }
}

------------------------------------
Request Completed Successfully
Request Time in seconds: 1.336568
------------------------------------
```

### Troubleshooting
* Run `Open-ILS/src/support-scripts/settings-tester.pl` as `opensrf` to identify configuration issues.
* Consult the [Evergreen Troubleshooting Guide](http://evergreen-ils.org/dokuwiki/doku.php?id=troubleshooting:checking_for_errors).
* Contact the [Evergreen Development Mailing List](http://evergreen-ils.org/communicate/mailing-lists/) with your configuration and log files.

---

## Getting Help

* **Mailing Lists:** [evergreen-ils.org/communicate/mailing-lists](http://evergreen-ils.org/communicate/mailing-lists/)
* **IRC:** `#evergreen` on the Libera.Chat / Freenode network

---

## License

This work is licensed under the [Creative Commons Attribution-ShareAlike 3.0 Unported License](http://creativecommons.org/licenses/by-sa/3.0/).
*(Creative Commons, 444 Castro Street, Suite 900, Mountain View, California, 94041, USA)*

2. **Install project-local dependencies:**
   ```bash
   npm ci   # fetch JS and CSS dependencies
   ```
   *(Note: There is no build step.)*

3. **OPTIONAL: Test OPAC JavaScript code:**
   ```bash
   npm run test
   ```

---

## Configuration and Compilation Instructions

For the time being, we are still installing everything in the `/openils/` directory. From the Evergreen source directory, issue the following commands as the **`user`** Linux account to configure and build Evergreen:

```bash
PATH=/openils/bin:$PATH ./configure --prefix=/openils --sysconfdir=/openils/conf
make
```

> [!NOTE]
> These instructions assume that you have also installed OpenSRF under `/openils/`. If not, please adjust `PATH` as needed so that the Evergreen `configure` script can find `osrf_config`.

---

## Installation Instructions

Once you have configured and compiled Evergreen, issue the following command as the **`root`** Linux account to install Evergreen and copy example configuration files to `/openils/conf`:

```bash
make install
```

---

## Change Ownership of the Evergreen Files

All files in the `/openils/` directory and subdirectories must be owned by the `opensrf` user. Issue the following command as the **`root`** Linux account:

```bash
chown -R opensrf:opensrf /openils
```

---

## Run ldconfig

On Ubuntu or Debian, run the following command as the **`root`** user:

```bash
ldconfig
```

---

## Additional Instructions for Developers

> [!NOTE]
> Skip this section if you are using an official release tarball downloaded from [evergreen-ils.org/egdownloads](http://evergreen-ils.org/egdownloads).

Developers working directly with the source code from the Git repository, rather than an official release tarball, need to install the Dojo Toolkit set of JavaScript libraries. The appropriate version of Dojo is included in Evergreen release tarballs. Developers should install Dojo version 1.3.3 by issuing the following commands as the **`opensrf`** Linux account:

```bash
wget https://download.dojotoolkit.org/release-1.3.3/dojo-release-1.3.3.tar.gz
tar -C /openils/var/web/js -xzf dojo-release-1.3.3.tar.gz
cp -r /openils/var/web/js/dojo-release-1.3.3/* /openils/var/web/js/dojo/.
```

---

## Configure the Apache Web Server

1. **Copy example configuration files** for the Evergreen catalog, web staff client, web services, and administration interfaces. Issue the following commands as the **`root`** Linux account:

   ```bash
   cp Open-ILS/examples/apache_24/eg_24.conf       /etc/apache2/sites-available/eg.conf
   cp Open-ILS/examples/apache_24/eg_vhost_24.conf /etc/apache2/eg_vhost.conf
   cp Open-ILS/examples/apache_24/eg_startup      /etc/apache2/

   # Set up SSL
   mkdir /etc/apache2/ssl
   cd /etc/apache2/ssl
   ```

2. **Generate SSL Key & Certificate:**
   The `openssl` command cuts a new SSL key for your Apache server. For a production server, purchase a signed SSL certificate; for testing/development, a self-signed certificate can be used:

   ```bash
   openssl req -new -x509 -days 365 -nodes -out server.crt -keyout server.key
   ```

3. **Edit `eg.conf`:**
   As the **`root`** Linux account, edit the `/etc/apache2/sites-available/eg.conf` file:
   * To enable access to the offline upload / execute interface from any workstation on any network (ensure this is secured for production!):
     * Replace `Require host 10.0.0.0/8` with `Require all granted`

4. **Change the Apache user:**
   * Edit `/etc/apache2/envvars` as **`root`**.
   * Change `export APACHE_RUN_USER=www-data` to:
     ```bash
     export APACHE_RUN_USER=opensrf
     ```

5. **Configure Apache KeepAlive settings:**
   Higher values can improve client performance by allowing multiple requests over the same TCP connection, but increase the risk of using up Apache child processes and memory.
   * Edit `/etc/apache2/apache2.conf` as **`root`**:
     * Change `KeepAliveTimeout` to `1`
     * Change `MaxKeepAliveRequests` to `100`

6. **Configure the prefork module:**
   Set up `/etc/apache2/mods-available/mpm_prefork.conf` with values suitable for a default Evergreen site:

   ```apache
   <IfModule mpm_prefork_module>
      StartServers            15
      MinSpareServers          5
      MaxSpareServers         15
      MaxRequestWorkers       75
      MaxConnectionsPerChild 500
   </IfModule>
   ```

7. **Enable the `mpm_prefork` module** as **`root`**:
   ```bash
   a2dismod mpm_event
   a2enmod mpm_prefork
   ```

8. **Enable the Evergreen site:**
   ```bash
   a2dissite 000-default  # OPTIONAL: disable the default "It Works" page
   a2ensite eg.conf
   ```

9. **Allow Apache to write to the lock directory** *(skip this step on Debian 13)*:
   ```bash
   chown opensrf /var/lock/apache2
   ```

---

## Configure OpenSRF for the Evergreen Application

There are example OpenSRF configuration files in `/openils/conf/` that serve as templates. Issue the following commands as the **`opensrf`** Linux account:

```bash
cp -b /openils/conf/opensrf_core.xml.example /openils/conf/opensrf_core.xml
cp -b /openils/conf/opensrf.xml.example /openils/conf/opensrf.xml
```

> [!NOTE]
> The `-b` flag tells `cp` to create a backup file with a tilde (`~`) appended.
> 
> When installing OpenSRF, you created four Jabber users on two separate domains. Edit `/openils/conf/opensrf_core.xml` using those exact Jabber users and domains.
> 
> `eg_db_config` (detailed in [Creating the Evergreen Database](#creating-the-evergreen-database)) will automatically configure database connection parameters in `opensrf.xml`.

### Optional: Configure Evergreen for OpenSRF + Redis

If using the Redis variant of OpenSRF, modify `/openils/conf/opensrf_core.xml` to use the Redis settings instead of Ejabberd.

For each configuration block that contains both Ejabberd and Redis sections, comment out Ejabberd and un-comment Redis:

**Before:**
```xml
<!-- Ejabberd -->                                                          
<passwd>password</passwd>                                                  
<port>5222</port>                                                          
<!-- ===  -->                                                              
                                                                           
<!-- Redis -->                                                             
<!--                                                                       
<passwd>456fc340-beba-4489-9070-0d6b49e9952b</passwd>                      
<port>6379</port>                                                          
-->                                                                        
<!-- ===  -->
```

**After:**
```xml
<!-- Ejabberd -->                                                          
<!--
<passwd>password</passwd>                                                  
<port>5222</port>                                                          
-->
<!-- ===  -->                                                              
                                                                           
<!-- Redis -->                                                             
<passwd>456fc340-beba-4489-9070-0d6b49e9952b</passwd>                      
<port>6379</port>                                                          
<!-- ===  -->
```

### Configure Action Triggers for the Evergreen Application

*Action Triggers* provide hooks for the system to perform actions when a given event occurs (e.g., generating reminder or overdue notices via `checkout.due`).

To enable the default set of hooks, issue the following command as the **`opensrf`** Linux account:

```bash
cp -b /openils/conf/action_trigger_filters.json.example /openils/conf/action_trigger_filters.json
```

---

## Creating the Evergreen Database

### Setting Up the PostgreSQL Server

For production use, most libraries install PostgreSQL on a dedicated machine. Therefore, by default, `Makefile.install` does **not** install the PostgreSQL server package.

Install the required PostgreSQL packages as the **`root`** Linux account using the make target matching your distribution and target PostgreSQL version:

> [!WARNING]
> **PostgreSQL 12+ JIT (Just-in-Time compilation):**
> Do **not** turn on PostgreSQL's JIT capabilities. Evergreen's queries, especially complex ones used for search, are intentionally tuned for non-JIT execution, and JIT has been shown to be harmful in some circumstances.
> 
> Recommended minimum tweak to `postgresql.conf`:
> ```ini
> jit_above_cost = -1
> ```

#### PostgreSQL Version Options:

<details>
<summary><strong>PostgreSQL 14</strong></summary>

```bash
make -f Open-ILS/src/extras/Makefile.install postgres-server-debian-trixie-14
make -f Open-ILS/src/extras/Makefile.install postgres-server-debian-bookworm-14
make -f Open-ILS/src/extras/Makefile.install postgres-server-debian-bullseye-14
make -f Open-ILS/src/extras/Makefile.install postgres-server-debian-buster-14
make -f Open-ILS/src/extras/Makefile.install postgres-server-ubuntu-jammy-14
make -f Open-ILS/src/extras/Makefile.install postgres-server-ubuntu-noble-14
```
</details>

<details>
<summary><strong>PostgreSQL 15</strong></summary>

```bash
make -f Open-ILS/src/extras/Makefile.install postgres-server-debian-trixie-15
make -f Open-ILS/src/extras/Makefile.install postgres-server-debian-bookworm-15
make -f Open-ILS/src/extras/Makefile.install postgres-server-debian-bullseye-15
make -f Open-ILS/src/extras/Makefile.install postgres-server-debian-buster-15
make -f Open-ILS/src/extras/Makefile.install postgres-server-ubuntu-jammy-15
make -f Open-ILS/src/extras/Makefile.install postgres-server-ubuntu-noble-15
```
</details>

<details>
<summary><strong>PostgreSQL 16</strong></summary>

```bash
make -f Open-ILS/src/extras/Makefile.install postgres-server-debian-trixie-16
make -f Open-ILS/src/extras/Makefile.install postgres-server-debian-bookworm-16
make -f Open-ILS/src/extras/Makefile.install postgres-server-debian-bullseye-16
make -f Open-ILS/src/extras/Makefile.install postgres-server-debian-buster-16
make -f Open-ILS/src/extras/Makefile.install postgres-server-ubuntu-jammy-16
make -f Open-ILS/src/extras/Makefile.install postgres-server-ubuntu-noble-16
```
</details>

<details>
<summary><strong>PostgreSQL 17</strong></summary>

```bash
make -f Open-ILS/src/extras/Makefile.install postgres-server-debian-trixie-17
make -f Open-ILS/src/extras/Makefile.install postgres-server-debian-bookworm-17
make -f Open-ILS/src/extras/Makefile.install postgres-server-debian-bullseye-17
make -f Open-ILS/src/extras/Makefile.install postgres-server-debian-buster-17
make -f Open-ILS/src/extras/Makefile.install postgres-server-ubuntu-jammy-17
make -f Open-ILS/src/extras/Makefile.install postgres-server-ubuntu-noble-17
```
</details>

<details>
<summary><strong>PostgreSQL 18</strong></summary>

```bash
make -f Open-ILS/src/extras/Makefile.install postgres-server-debian-trixie-18
make -f Open-ILS/src/extras/Makefile.install postgres-server-debian-bookworm-18
make -f Open-ILS/src/extras/Makefile.install postgres-server-debian-bullseye-18
make -f Open-ILS/src/extras/Makefile.install postgres-server-debian-buster-18
make -f Open-ILS/src/extras/Makefile.install postgres-server-ubuntu-jammy-18
make -f Open-ILS/src/extras/Makefile.install postgres-server-ubuntu-noble-18
```
</details>

---

#### Create the Evergreen PostgreSQL User

Issue the following command as the **`postgres`** Linux account to create a new superuser named `evergreen`. When prompted, enter a secure password:

```bash
createuser -s -P evergreen
```

#### Enabling Connections to the PostgreSQL Database

Check your `pg_hba.conf` file (located under `/etc/postgresql/` on Debian and Ubuntu). To allow TCP/IP connections from `localhost` with password authentication for testing, ensure the following lines appear **before** any `host ... ident` entries:

```text
host    all             all             ::1/128                 md5
host    all             all             127.0.0.1/32            md5
```

Reload PostgreSQL after updating `pg_hba.conf` for changes to take effect.

---

### Creating the Evergreen Database and Schema

From inside the Evergreen source directory, run `eg_db_config` as the **`root`** Linux account:

```bash
perl Open-ILS/src/support-scripts/eg_db_config --update-config \
       --service all --create-database --create-schema --create-offline \
       --user <user> --password <password> --hostname <hostname> --port <port> \
       --database <dbname> --admin-user <admin-user> --admin-pass <admin-pass>
```

* `<user>` & `<password>`: Credentials for the `evergreen` PostgreSQL account.
* `<admin-user>` & `<admin-pass>`: Desired credentials for the `egadmin` Evergreen administrator account.

*(Run with `--help` to inspect all available options.)*

---

### Loading Sample Data

* **Standard *Concerto* Dataset:**
  Add `--load-all-sample` to `eg_db_config` to load authority/bibliographic records, call numbers, copies, staff/regular users, and sample transactions.
* **Enhanced *Concerto* Dataset:**
  Use `--load-concerto-enhanced` for additional testing data, including:
  * Friendly organization unit names
  * Acquisitions and billing scenarios
  * Shelving locations & settings
  * Multilingual bibliographic records (Japanese, Spanish, French, Czech)
  * Metarecord holds & item stat categories
  * Bookings, pre-created OPAC carousels, and serials data

---

### Creating the Database on a Remote Server

To provision the database on a remote dedicated server, include the `--create-database` flag when running `eg_db_config`.

---

## Starting Evergreen

1. **Start background services** as **`root`**:
   ```bash
   /etc/init.d/ejabberd start
   /etc/init.d/memcached start
   ```

2. **Start OpenSRF / Evergreen services** as **`opensrf`**:
   ```bash
   osrf_control -l --start-all
   ```
   *(Note: The `-l` flag forces `localhost`. Omit it if you configured `opensrf.xml` using your machine's fully qualified domain name.)*

   > [!TIP]
   > If you see `bash: osrf_control: command not found`, add `/openils/bin` to your `PATH` in `~/.bashrc`:
   > ```bash
   > export PATH=$PATH:/openils/bin
   > ```

3. **Generate web files and org unit proximity** as **`opensrf`**:
   ```bash
   autogen.sh
   ```
   *(Run this whenever library organization unit configurations change.)*

4. **Restart Apache** as **`root`**:
   ```bash
   /etc/init.d/apache2 restart
   ```

---

## Testing Connections to Evergreen

Test system connectivity using `srfsh` as the **`opensrf`** Linux account:

```bash
/openils/bin/srfsh
srfsh% login <admin-user> <admin-pass>
```

**Expected output:**
```text
Received Data: "250bf1518c7527a03249858687714376"
------------------------------------
Request Completed Successfully
Request Time in seconds: 0.045286
------------------------------------

Received Data: {
   "ilsevent":0,
   "textcode":"SUCCESS",
   "desc":" ",
   "pid":21616,
   "stacktrace":"oils_auth.c:304",
   "payload":{
      "authtoken":"e5f9827cc0f93b503a1cc66bee6bdd1a",
      "authtime":420
   }
}

------------------------------------
Request Completed Successfully
Request Time in seconds: 1.336568
------------------------------------
```

### Troubleshooting
* Run `Open-ILS/src/support-scripts/settings-tester.pl` as `opensrf` to identify configuration issues.
* Consult the [Evergreen Troubleshooting Guide](http://evergreen-ils.org/dokuwiki/doku.php?id=troubleshooting:checking_for_errors).
* Contact the [Evergreen Development Mailing List](http://evergreen-ils.org/communicate/mailing-lists/) with your configuration and log files.

---

## Getting Help

* **Mailing Lists:** [evergreen-ils.org/communicate/mailing-lists](http://evergreen-ils.org/communicate/mailing-lists/)
* **IRC:** `#evergreen` on the Libera.Chat / Freenode network

---

## License

This work is licensed under the [Creative Commons Attribution-ShareAlike 3.0 Unported License](http://creativecommons.org/licenses/by-sa/3.0/).
*(Creative Commons, 444 Castro Street, Suite 900, Mountain View, California, 94041, USA)*

