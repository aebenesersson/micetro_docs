.. meta::
   :description: Installing the DNS Agent for Micetro by Men&Mice
   :keywords: DNS, DNS Agent, Micetro, BIND, Unbound, AuthServe

.. _install-dns-controllers:

Micetro DNS Agents
==================

Micetro comes with two types of DNS agents: 

   * the Micetro :ref:`dns-server`
   * the Men&Mice :ref:`authserve` 

.. _dns-server:

DNS Server Controller
---------------------

By default, the installer attempts to automatically detect the installed DNS service (e.g., BIND) and install the appropriate controller. If it can't detect the service, it provides hints and additional information.

.. note::
  If you're running BIND DNS, ensure that the DNS Agents run as the same user as BIND (by default, ``named``.)

  If BIND is running as a different user, or files are updated, ensure that the ``mmremote`` service is run as the same user and has sufficient access to files and directories.

For machines with multiple services (for example, ISC DHCP and ISC BIND DNS), explicitly specify the desired controllers during installation.

To view available controller options and parameters, run the installer script with the --help parameter:

  .. code-block:: bash

    cd archive-name
    ./install --help

    Micetro server controller installer.
    --help:  Print help.
    --quiet:  Suppress output during install.
    --auto:  Automatically determine what controllers to install. Default if no other option is given.
    --bind-dns-controller:  Install a DNS server controller for BIND.
    --unbound-dns-controller:  Install a DNS server controller for Unbound.
    --generic-dns-controller:  Install a Generic DNS server controller.
    --isc-dhcp-controller:  Install a DHCP server controller for ISC dhcpd.
    --kea-dhcp-controller:  Install a DHCP server controller for Kea dhcp4.
    --update-controller:  Install update controller. Always installed, if another Men&Mice service is installed.

Running the Installer
^^^^^^^^^^^^^^^^^^^^^
* To install controllers automatically (recommended when you have a single service like BIND or Unbound):

.. code-block:: bash

  ./install --auto

* For a specific set of controllers, run the installer as follows (example with ISC BIND and Generic DNS controller):

.. code-block:: bash

  ./install --generic-dns-controller --bind-dns-controller --isc-dhcp-controller

* For quiet/unattended installation with no output:

.. code-block:: bash

  ./install --generic-dns-controller --bind-dns-controller --quiet

.. note::
  The Micetro Update Controller is automatically added when another Men&Mice service is installed.

If you plan to use the Generic DNS Controller, refer to the :ref:`generic-dns-controller` for more details.

In case of issues with the new installer, the old Perl-based installer is still available in the same archive as ``deprecated_installer.pl``. Run it as follows:

.. code-block:: bash

  cd archive-name
  ./deprecated_installer

The installer will ask a series of questions. Be prepared to answer them, as described for each component.

Micetro Controllers Running on Linux
------------------------------------

Preliminary Checks
^^^^^^^^^^^^^^^^^^
Before installing the Micetro DNS Controller on a Linux system, ensure that you have thoroughly examined your system's configuration. Pay close attention to the following aspects:

  * **Configuration File:** Check if there is a valid starting configuration file, typically located at ``/etc/named.conf``. If one doesn't exist, you will need to create it.
    
  * **Content of named.conf:** Verify that your ``named.conf`` file contains all the necessary statements as detailed below.

  * **Ownership of Named Data Directory:** Determine if the named init script changes the ownership of the named data directory. This is crucial, especially for certain Red Hat Linux versions and derivatives that may modify the ownership (check for the ``ENABLE_ZONE_WRITE`` setting).

  * **Chroot Environment:** Check if named runs within a chroot environment. If it does, be aware of specific issues that may arise and consult the knowledge base for solutions. Pay attention to the following:

    * Does the named init script copy anything into the chroot jail when starting the service (relevant for SUSE Linux)?

    * Consider potential problems that might occur when the installer rearranges the data directory listed in ``named.conf`` (relevant for SUSE Linux).

    * When installing agents in a chrooted environment, note that the only way to accomplish this is by using the ``deprecated_installer.pl`` script.

  * **User Account for Named:** Identify the user account that owns the named process. Typically, the Micetro DNS Controller should run under the same user account. However, it is occasionally possible to use group membership instead.

Installation Steps
^^^^^^^^^^^^^^^^^^

1. Extract the Micetro Controller installation package (as root):

  .. code-block:: bash

    tar -xzvf micetro-controllers-10.0.linux.x64.tgz

2. In the newly created ``micetro-controllers-10.0.linux.x64`` directory, run the installer script to install the Micetro Controller (as root):

  .. code-block:: bash

    cd micetro-controllers-10.1.linux.x64 && ./install

Installer Questions
^^^^^^^^^^^^^^^^^^^

During the installation process, the installer will prompt you with questions related to the Micetro DNS Server Controller. Be prepared to answer the following:

  * Do you want to install the Micetro DNS Server Controller?
  * Are you running named in a chroot() environment?
  * What is the chroot() directory?
  * Where is the BIND configuration file?
  * Would you like the DNS Server Controller to run ``name-checkconf`` to verify changes when editing advanced server and zone options?
  * Where is ``named-checkconf`` located?
  * The installer needs to rearrange the files in ``<directory>`` and restart the name server. A backup will be created. Is this OK?
  * Enter the user and group names under which you want to run the Micetro DNS Server Controller. This must be the user that is running named.
  * Where would you like to install the Men&Mice external static zone handling utilities?
  * Where do you want to install the Micetro Server Controller binaries?
  * BIND needs to be restarted. Would you like to restart it now?

Ensure the ``named-checkconf`` file is readable:

  .. code-block:: bash

    chmod a+s /usr/sbin/named-checkconf

Required named.conf Statements
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The Micetro DNS Server Controller requires specific settings within the ``named.conf`` file (including any files listed in ``include`` statements in ``named.conf``). Ensure the following statements are present:

* ``directory``: The ``directory`` substatement of the ``options`` statement must be present and must point to a directory that the installer can replace. It should not refer to ``/``, ``/etc``, the root of a chroot jail, or any partition mount point. If you need to change or add the ``directory`` statement, be prepared to move files or update paths used elsewhere in your ``named.conf``.

* ``key``: For BIND, there must be an explicitly defined key in ``named.conf`` to enable control of named using ``rndc`` commands. Copy the contents of the key file, such as ``rnds.key``, into ``named.conf`` if it's not explicitly defined.

  To generate a key, consider using the following command (adjust the path if needed):

  .. code-block:: bash

    rndc-confgen > /etc/rndc.conf

  This creates the ``rndc.conf`` file, which contains configuration for local use and key and controls statements that can be copied into ``named.conf``.

* ``controls``: The Micetro DNS Server Controller uses a ``controls`` statement for BIND. There must be a ``controls`` statement with an ``inet`` substatement that references an explicitly defined key (as mentioned above). The ``inet`` statement should allow connections from the loopback address, ``127.0.0.1``. If no ``controls`` statement is defined, the installer will prompt you to create one manually.

Changes in named.conf
^^^^^^^^^^^^^^^^^^^^^

Note that the installation of the Micetro DNS Server Controller will rearrange your named configuration data, including rewriting ``named.conf`` and reorganizing the data directory. The new configuration is functionally equivalent to the old one, except that the logging statement may be added or modified to include new channels.

Common Files
""""""""""""

The file layout differs slightly between instances with and without BIND views, but there are some common parts:

.. csv-table::
  :header: "Description", "File(s) or directory"
  :widths: 40, 60

  "Micetro DNS Server Controller daemon", "mmremoted, usually in /usr/sbin or /usr/local/sbin"
  "Micetro external static zone handling utilities", "mmedit and mmlock, usually in /usr/bin or /usr/local/bin"
  "Data directory for Micetro DNS Server Controller", "Usually /var/named, /etc/namedb, /var/lib/named, or something within a chroot jail; the same location as before the DNS Server Controller was installed"
  "Backup of original data directory", "Same as above, with '.bak' appended to the path"
  "New starting configuration file", "Usually either /etc/named.conf or /etc/namedb/named.conf; possibly located within a chroot jail"
  "Backup of original starting configuration file", "Same as above, with '.bak' appended to the path"
  "logging statement from named.conf", "conf/logging, relative to the data directory"
  "key and acl statements from named.conf", "conf/user_before, relative to the data directory"
  "options statement from named.conf", "conf/options, relative to the data directory"
  "controls, server, and trusted-keys statements from named.conf; also, if present and if not using views, the root hints zone statement", "conf/user_after, relative to the data directory"
  "Preferences file", "mmsuite/preferences.cfg, located in the data directory"
  "init script, the shell script that can be used to control the service; used by init during system startup", "/etc/init.d/mmremote"
  "settings file used by the init script (Ubuntu Linux only)", "/etc/default/mmremote"

**Without Views**

If views are not defined, the following files are created inside the data directory:

.. csv-table:: Without BIND views
  :header: "Description", "File(s) or directory"
  :widths: 40, 60

  "List of include statements, one for each zone statement file", "conf/zones"
  "Directory of zone statement files", "conf/zoneopt"
  "A sample zone statement file, for the zone 'localhost'.", "conf/zoneopt/localhost.opt"
  "Directory of primary master zone files", "hosts/masters"
  "Directory of secondary zone files", "hosts/slaves"
  "A sample zone file, for the primary master zone 'localhost.'", "hosts/masters/localhost-hosts"

**With views**

If views are defined, the following files are created inside the data directory:

.. csv-table:: With BIND views
  :header: "Description", "File(s) or directory"
  :widths: 40, 60

  "View statements, not including zone statements within each view", "conf/zones"
  "List of include statements for a particular view, one for each zone statement file", "conf/zones_viewname"
  "Directory of zone statement files for a particular view", "conf/zo_viewname"
  "A sample zone statement file, for the zone 'localhost'. in the view 'internal'", "conf/zo_internal/localhost.opt"
  "Directory of primary master zone files for a particular view", "hosts/view_viewname/masters"
  "Directory of secondary zone files for a particular view", "hosts/view_viewname/slaves"
  "A sample zone file, for the primary master zone 'localhost.' in the view 'internal'", "hosts/view_internal/masters/localhost-hosts"

Removing the DNS Server Controller and Reverting to Original Data
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Stopping the Service
""""""""""""""""""""
Use the init script to stop the DNS Server Controller service. You can achieve this by providing the *stop* argument to the init script. For example:

.. code-block:: bash

   sudo /etc/init.d/dns-controller stop

or

.. code-block:: bash

   sudo systemctl stop dns-controller

Replace ``/etc/init.d/dns-controller`` and ``dns-controller`` with the appropriate paths and service names for your system.

Removing Controller Files
"""""""""""""""""""""""""
Once the service is stopped, you can proceed to remove the DNS Server Controller files:

   * Delete the daemon binary file associated with the DNS Server Controller.
   * Delete the init script used to start the DNS Server Controller service.
   * If the init script was registered as part of the boot system, remove any references to it. This may involve using system-specific tools or manually editing boot configuration files.

Reverting to Original Data
""""""""""""""""""""""""""
If you wish to revert to your original DNS configuration and data, follow these additional steps:

1. Stop the BIND or named service, which might have been managed by the DNS Server Controller, using its respective init script. For example:

.. code-block:: bash

   sudo /etc/init.d/named stop

or

.. code-block:: bash

   sudo systemctl stop named

2. With the BIND or named service stopped, you can proceed to restore your original DNS configuration and data:
   * Delete the initial configuration file (``named.conf``) created by the DNS Server Controller. 
   * Delete the data directory created by the DNS Server Controller.
   * If you created backup files by renaming the originals with a ".bak" extension, restore the original files by removing the ".bak" extension from their names.

These steps will effectively remove the DNS Server Controller and revert your DNS setup to its original state. Be cautious when performing these actions, as they may impact your DNS service.

SELinux
^^^^^^^

.. note::
  The following commands apply to Linux distributions based on RedHat EL 8 or higher. Your distribution may differ.

After installing the DNS Server Controller, run the following commands as root:

.. code-block:: bash

  semanage fcontext -a -t named_cache_t --ftype f "/var/named(/.*)?"
  semanage fcontext -a -t named_cache_t --ftype d "/var/named(/.*)?"
  semanage fcontext -a -t named_conf_t --ftype f "/var/named/conf(/.*)?"
  semanage fcontext -a -t named_conf_t --ftype d "/var/named/conf(/.*)?"
  semanage fcontext -a -t named_zone_t --ftype f "/var/named/hosts(/.*)?"
  semanage fcontext -a -t named_zone_t --ftype d "/var/named/hosts(/.*)?"
  restorecon -rv /var/named

These will adjust the SELinux security label for the BIND 9 configuration and zone files.

.. note::
  Due to the complexity of and variation between SELinux configuration files, we are currently unable to officially support SELinux configuration, as SELinux settings can interfere with the normal operation of named after its configuration has been rewritten by the installer for Micetro DNS Server Controller. It is possible to make ``named``, Micetro, and SELinux all work together, but we cannot currently offer official support for this.

The $INCLUDE and $GENERATE Directives
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Refer to the following articles for information about how these directives are handled in Micetro.

* :ref:`dns-controller-include`

* :ref:`dns-controller-generate`

Installation with Dynamic Zones
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Micetro expects dynamic zones to be made dynamic by allowing signed updates. Any dynamic zone must have an allow-update statement whose ACL contains a key. If you do not otherwise need signed updates, add the rndc key (or any other key) to the list.

Furthermore, after installation, be sure that your server allows zone transfers of dynamic zones to the loopback address, 127.0.0.1, or users will be unable to open dynamic zones from this server. Zone transfer restrictions can be set or changed in the server's and in each zone's **Options** window in the Men&Mice Management Console.

Verify the DNS Server Controller is running
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Verify the Controller application is running:

.. code-block:: bash

  systemctl status mmremote

Micetrol Server Controller running on Windows
---------------------------------------------

Active Directory Integrated Zones and Other Dynamic Zones
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To open a dynamic zone, Micetro must read it from the DNS service rather than from a file. The way this is done is via *zone transfer*. On Windows Server 2003 and later, the zone transfer restriction setting in the zone's options window must be set to allow transfers to an explicit list of IP addresses that includes the server's address. The default setting of allowing zone transfers to any server listed in the zone's NS records will not suffice.

In some cases, Micetro DNS Server Controller will also need to be told specifically which interface to use when requesting zone transfers. If you have trouble opening a dynamic zone after setting the zone's transfer restrictions appropriately, check the Event Log / Application Log for messages from Micetro DNS Server Controller. If there is a message indicating that it was unable to get a zone transfer, note the address it tried to use; you can either add that IP address to the transfer restrictions list or else edit a configuration file for Micetro DNS Server Controller.

To configure the DNS Server Controller to use a different address, edit the service's preferences.cfg file on the DNS server computer. The file is located in one of the following two locations, where {Windows} is probably C:\\Windows:

* {Windows}\\System32\\dns\\mmsuite\\preferences.cfg
* C:\\Documents and Settings\\All Users\\Application Data\\Men and Mice\\DNS Server Controller\\preferences.cfg
* C:\\ProgramData\\Men and Mice\\DNS Server Controller\\preferences.cfg

If the file does not exist, create it. The file is a text file in a simple XML-based format. Add the following element, replacing the dummy address here with the server's correct network address:

.. code-block::

  <DNSServerAddress value="192.0.2.1"/>

Save the file, and then restart Micetro DNS Server Controller using :menuselection:`Administrative Tools --> Services` in Windows. Then also restart Men&Mice Central, so that it can cache the zone's contents.

.. note::
  For Active Directory-integrated zones, other domain controllers running Microsoft DNS do not need to get zone transfers. This is because the zone data is replicated through LDAP, rather than through zone transfers. Thus, for an AD-integrated zone, the zone transfer restriction list might need only the server's own address.

Running Micetro DNS Server Controller under a privileged user account / Server type: "Microsoft Server Controller-Free"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Normally, the Micetro DNS Server Controller is installed on only *one* host in an Active Directory forest, or one copy per site. That installation can then manage all MS DNS servers in the forest, or in the site, using Microsoft's own DNS management API. In order for this to work, the service needs to run as a user that has DNS management privileges (i.e. the AD service account must be a member of the DNSAdmins group of the domain).

To configure Micetro DNS Server Controller to access DNS servers on remote computers, do the following:

 1. Start the Windows 'Services' program and open the properties dialog box for Micetro DNS Server Controller.
 2. Click the :guilabel:`Log On` tab. The :guilabel:`Local System account` radio button is most likely selected.
 3. Select the :guilabel:`This account` radio button and enter the name and password of a Windows user that is a member of the Administrators group.
 4. Close the dialog box and restart the Micetro DNS Server Controller service.

If Micetro DNS Server Controller is run as a local system service (the default), it will only be able to manage the MS DNS service on the same host.

Enable the Generic DNS Server Controller functionality
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If the Controller should be configured to run a connector script in order to interface with other DNS servers than the natively supported Windows DNS/Unix BIND DNS, the script interpreter and the connector script must be configured in the controllers ``preferences.cfg`` file.

The file is a text file in a simple XML-based format. Add the following element, replacing the dummy script interpreter and script:

.. code-block:: XML

  <GenericDNSScript value="python /scripts/genericDNS.py" />

Configure the DNS Server Controller to work with Microsoft Azure DNS
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

For information on configuring Microsoft Azure DNS, see :ref:`configure-azure-dns`.

Where to install Micetro DNS Server Controller
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If Men&Mice Central is installed on a Windows host, one option is to install Micetro DNS Server Controller on the same host. If this is not done, the system will need to be told where to find the DNS Server Controller when adding a new DNS server to the system. This will be presented as connecting via proxy.

.. note::
  The Men&Mice communication protocol used to control a DNS server is more efficient than the Microsoft protocol. This means that if a DNS server is separated from Men&Mice Central by a slow network link, it is more efficient to install a copy of the Micetro DNS Server Controller in the same local network (the same site, typically) as the DNS server.
   
.. _authserve:   

AuthServe Agent
---------------
  
Agent Setup
^^^^^^^^^^^^
  
Download
""""""""
Download the latest package from https://download.menandmice.com/ and extract the installer into ``/var/mmsuite``. A different location for the agent can also be chosen if preferred.
  
.. code-block::
 
   mkdir -p /var/mmsuite && cd /var/mmsuite

   # Assuming the package is in local directory
   tar oxzf ./mm-authserve-agent.tar.gz

   # Ensure that the user running the service owns the agent files 
   chown ${SUDO_USER:-$USER}: -R mm-authserve-agent

   # Enter the extracted directory and proceed to configure the agent
   cd mm-authserve-agent

Installing the Agent
""""""""""""""""""""

   1. Install the agent as a service with ``sudo ./install``. Note that the install script requires Python. Make sure that the user that runs the install script is the same user that owns the ``mm-authserve-agent`` folder.
   2. Copy the agent setup key that the install script prints out. The Men&Mice AuthServe Agent should now be up and running but you need to connect it to Central to be able to manage it through Micetro.

.. note::
   The Men&Mice AuthServe Agent runs on port 50051 and Central runs on port 1231. Ensure that no firewall settings prevent connection from Central to the agent.
   
Adding the Agent to Central
"""""""""""""""""""""""""""
   1. Select :guilabel:`Admin` on the top navigation bar.
   2. Click :guilabel:`Service Management` on the menu bar at the top of the admin workspace.
   3. Click :guilabel:`Add Service` above the list of services.
   4. On the list of services, select **AuthServe**.
   5. Click the :guilabel:`New Agent` tab, and fill in the information.
   
       .. image:: ../../images/add-authserve.png
          :width: 65%
      
      *  **Agent host**: the hostname or IP address of the machine where the agent is located. Note that the Central machine must be able to communicate with the agent machine. 
      * **Agent display name**: this box is optional and should be filled in if you want your agent to be displayed in the UI under some other name than the hostname/IP address.
      * **Agent setup key**: enter the setup key for the agent that you copied earlier from the agent installation script. If you forgot to copy it, you can also find it in the ssl directory which can be found under the agent directory on the agent machine. The agent also prints it out on startup if it hasn’t been added to a Central server yet. The setup key is used to encrypt certificates that Central sends over to the agent. These certificates are then used to allow for a secure encrypted connection to be created between Central and the agent.

      .. note::
         If the agent you are adding to Central has been previously added to a Central server, you will have to remove the SSL directory and restart the agent before adding it. The restart will generate a new setup key that you should use when adding the agent.


   6. When you are finished, click :guilabel:`Next`.
   7. Enter :guilabel:`Service name` and the Nominum Command Channel used to connect to ANS in the :guilabel:`Channel` box. If you have some custom properties defined for DNS servers in your Micetro setup, you can fill in values for them as well in this panel. 
   8. Click :guilabel:`Add`. Micetro should now have a secure connection to the Men&Mice AuthServe Agent and you should be able to manage your AuthServe DNS server.

Updating the Agent
"""""""""""""""""""
Currently, the  ``mmupdater`` service is not capable of updating the AuthServe agent, so the update process must be done manually. To update the agent, an Administrator must unzip the latest agent package and run the ``update.sh`` script. 

**Related Topic:**

.. toctree::
  :maxdepth: 1

  generic_dns_controller
