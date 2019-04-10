********************************
Ansible Network Dummy Collection
********************************

This is a dummy collection to show the documentation needs for a full Ansible collection that includes roles, modules, and plugins.

.. contents::
   :local:

Requirements
============

* Ansible 2.8 or later

Dependencies
============

A list of any other dependencies that might need to be installed on a system before this collection can be operated. This is not a repeat of dependencies listed already in the galaxy.yml file etc, which is already part of the details tab in the Galaxy UI.

Included roles
==============

This collection includes the following top-level roles:

* config_manager - This role provides a network platform agnostic approach to managing the active (running) configuration file on a remote device. This role requires one (or more) platform provider roles to execute properly.

* backup_config - This roll collects the current device configurations and store it on the Ansible controller with a user specific backup filename and path. It copies running-config from remote device via SSH so it works only with network_cli connections.

* network_engine - This role provides the foundation for building network roles by providing modules and plugins that are common to all Ansible Network roles. Typically this role should not be directly invoked in a playbook.

Provider roles
--------------

The roles in this collection depend on provider roles to support the network agnostic roles on different network providers. The following provider roles are supported:

* arista_eos - This role provides a set of network functions that are designed to work with Arista EOS network devices. The functions included in this role include gathering facts from EOS devices, performing declarative configuration tasks and handling various operational tasks on the device.

* cisco_asa - This role provides a set of platform dependent functions that are designed to work with Cisco ASA network devices. The functions included int his role inlcuding both configuration and fact collection.


* cisco_ios - This role provides a set of platform dependent functions that are designed to work with Cisco IOS network devices. The functions included in this role including both configuration and fact collection.

* ciso_iosxr - This role provides a set of platform dependent functions that are designed to work with Cisco IOS-XR network devices. The functions included int his role including both configuration and fact collection.

* cisco_nxos - This role  provides a set of platform dependent functions that are designed to work with Cisco NX-OS network devices. The functions in this role include both configuration and fact collection.

* juniper_junos - This role role provides a set of network functions that are designed to work with Juniper JUNOS network devices. The functions included in this role include gathering facts from JUNOS devices, performing declarative configuration tasks and handling various operational tasks on the device.

* vyos - This role provides a set of platform dependent functions that are designed to work with VyOS network devices. The functions included in this role include both configuration and fact collection.

How to use the Ansible Network Dummy Collection
===============================================

This section briefly describes how to use this collection and includes a series of example playbooks.

How to get configuration from a network device
----------------------------------------------

The Config Manager role provides the ``get`` function that will connect to the remote network device and retrieve, by default, the current device active (running) configuration. The configuration will be returned to the calling playbook as a fact in the device host facts. The returned text based configuration can be accessed via config_manager.config fact.

Below is an example of calling the get function from the playbook.

.. code-block:: yaml

  ---
  - hosts: network
    connection: network_cli
    gather_facts: false
    roles:
      - name: ansible-network.config_manager
        function: get

The example playbook above will return the current active configuration for each device in the inventory group network.

How to get the device configuration from tasks
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The ``get`` function can also be includes in the playbook Tasks: section and executed as part of the list of tasks. Below is an example of how to retrieve the configuration using tasks.

.. code-block:: yaml

  ---
  - hosts: network
    gather_facts: false

    tasks:
      - name: get active configuration from device
        include_role:
          name: ansible-network.config_manager
          tasks_from: get

      - name: display the device configuration to stdout
        debug:
          msg: "{{ configuration.split('\n') }}"

The example playbook above will retrieve the current running configuration and then display the configuration contents to stdout.

How to load configuration to a network device
---------------------------------------------

The Config Manager role provides a function that will load configuration onto a remote network device. The load function will accept the device configuration as either text or a source file.

The load function provides some configurable options when pushing the configuration to the remote device. See the How to use this function section for different example of how to push configurations to network devices.

How to load and merge a configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Loading a configuration onto a target device is fairly simple and straightforward. By default, the load function will merge the contents of the provided configuration file with the configuration running on the target device.

Below is an example of how to call the load function.

.. code-block:: yaml

  - hosts: network
    gather_facts: false

    roles:
      - name: ansible-network.config_manager
        function: load
        config_manager_file: device.cfg

The example playbook above will simple load the contents of device.cfg onto the target network devices and merge the configurations.

How to load and replace a configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Similar to the merge capabilities, this role also supports replacing the current device configuration on the remote target. In order to tell the load function to replace the entire configure on the remote device with the provided configuration, set the config_manager_replace value to True.

.. code-block:: language

  - hosts: network
    gather_facts: false

    roles:
      - name: ansible-network.config_manager
        function: load
        config_manager_file: device.cfg
        config_manager_replace: True

The example playbook above will load the file specified by config_manager_file and replace the configuration on the remote device.

How to save active configuration to startup configuration on a device
---------------------------------------------------------------------

For network platforms that support saving the current active (running) configuration to non-volatile storage, the Config Manager save function can be invoked. This function will issue the save command on the target platform regardless of whether or not the active configuration has changed.

If the target platform does not support the save function, then it should simply return a NOOP.
How to save the active configuration

To save the current active configuration to the startup configuration simply invoke the save function on the target device. There are no additional configuration options for this function.

Below is an example of calling the save_config function from the playbook.

.. code-block:: yaml

  - hosts: network
    gather_facts: false

    roles:
      - name ansible-network.arista_eos
        function: save

How to back up a configuration
------------------------------

Use the ``backup_config`` role to back up a device configuration. This role collects the current device configurations and store it on the Ansible controller with a user specific backup filename and path. It copies running-config from remote device via SSH so it works only with network_cli connections. Features of this role are:

* Role supports idempotent behaviour so if there is no difference between current configurations and configurations present in destination file then it will return changed=0 and will not overwrite destination filename.

* If there is a change detected between current configurations and configurations present in destination file, it can backup last configurations before overwriting to new config file.

* By using above features, one can run this role periodically with backup option as "yes" to create devices configurations change history on local disk of ansible controller.

The following example shows how to backup the configuration.

.. code-block:: yaml

  ---
  - hosts: iosxr01 csr01
  roles:
    - backup_config
  vars:
        {
          "backup_config": {
            "filename" : "config_{{ ansible_host }}.cfg",
            "path" : "~/network_configs_1/",
            "backup" : "yes"
          }
        }

How to use the network_engine role
----------------------------------

The ``network_engine`` role includes the ``cli`` task (or function). The ``cli`` task provides an implementation for running CLI commands on network devices that is platform agnostic. The ``cli`` task accepts a command and will attempt to execute that command on the remote device returning the command output.

If the parser argument is provided, the output from the command will be passed through the parser and returned as JSON facts using the engine argument.

The following example runs CLI command on the network node.

.. code-block:: yaml

  ---
  - hosts: ios01
    connection: network_cli

    tasks:
    - name: run cli command with cli task
      import_role:
        name: ansible-network.network-engine
        tasks_from: cli
      vars:
        ansible_network_os: ios
        command: show version

When run with verbose mode, the output returned is as follows:

.. code-block:: JSON

  ok: [ios01] => {
    "changed": false,
    "json": null,
    "stdout": "Cisco IOS Software, IOSv Software (VIOS-ADVENTERPRISEK9-M), Version 15.6(2)T, RELEASE SOFTWARE (fc2)\nTechnical Support: http://www.cisco.com/techsupport\nCopyright (c) 1986-2016 by Cisco Systems, Inc.\nCompiled Tue 22-Mar-16 16:19 by prod_rel_team\n\n\nROM: Bootstrap program is IOSv\n\nan-ios-01 uptime is 19 weeks, 5 days, 19 hours, 14 minutes\nSystem returned to ROM by reload\nSystem image file is \"flash0:/vios-adventerprisek9-m\"\nLast reload reason: Unknown reason\n\n\n\nThis product contains cryptographic features and is subject to United\nStates and local country laws governing import, export, transfer and\nuse. Delivery of Cisco cryptographic products does not imply\nthird-party authority to import, export, distribute or use encryption.\nImporters, exporters, distributors and users are responsible for\ncompliance with U.S. and local country laws. By using this product you\nagree to comply with applicable laws and regulations. If you are unable\nto comply with U.S. and local laws, return this product immediately.\n\nA summary of U.S. laws governing Cisco cryptographic products may be found at:\nhttp://www.cisco.com/wwl/export/crypto/tool/stqrg.html\n\nIf you require further assistance please contact us by sending email to\nexport@cisco.com.\n\nCisco IOSv (revision 1.0) with  with 460033K/62464K bytes of memory.\nProcessor board ID 92O0KON393UV5P77JRKZ5\n4 Gigabit Ethernet interfaces\nDRAM configuration is 72 bits wide with parity disabled.\n256K bytes of non-volatile configuration memory.\n2097152K bytes of ATA System CompactFlash 0 (Read/Write)\n0K bytes of ATA CompactFlash 1 (Read/Write)\n0K bytes of ATA CompactFlash 2 (Read/Write)\n10080K bytes of ATA CompactFlash 3 (Read/Write)\n\n\n\nConfiguration register is 0x0"
}

The following example runs ``cli`` command and parse output to JSON facts.

.. code-block:: yaml

  ---
  - hosts: ios01
    connection: network_cli

    tasks:
    - name: run cli command and parse output to JSON facts
      import_role:
        name: ansible-network.network-engine
        tasks_from: cli
      vars:
        ansible_network_os: ios
        command: show version
        parser: parser_templates/ios/show_version.yaml
        engine: command_parser

When run with verbose mode, the output returned is as follows:

.. code-block:: yaml


  ok: [ios01] => {
      "ansible_facts": {
          "system_facts": {
              "image_file": "\"flash0:/vios-adventerprisek9-m\"",
              "memory": {
                  "free": "62464K",
                  "total": "460033K"
              },
              "model": "IOSv",
              "uptime": "19 weeks, 5 days, 19 hours, 34 minutes",
              "version": "15.6(2)T"
          }
      },
      "changed": false,
      "included": [
          "parser_templates/ios/show_version.yaml"
      ],
      "json": null,
      "stdout": "Cisco IOS Software, IOSv Software (VIOS-ADVENTERPRISEK9-M), Version 15.6(2)T, RELEASE SOFTWARE (fc2)\nTechnical Support: http://www.cisco.com/techsupport\nCopyright (c) 1986-2016 by Cisco Systems, Inc.\nCompiled Tue 22-Mar-16 16:19 by prod_rel_team\n\n\nROM: Bootstrap program is IOSv\n\nan-ios-01 uptime is 19 weeks, 5 days, 19 hours, 34 minutes\nSystem returned to ROM by reload\nSystem image file is \"flash0:/vios-adventerprisek9-m\"\nLast reload reason: Unknown reason\n\n\n\nThis product contains cryptographic features and is subject to United\nStates and local country laws governing import, export, transfer and\nuse. Delivery of Cisco cryptographic products does not imply\nthird-party authority to import, export, distribute or use encryption.\nImporters, exporters, distributors and users are responsible for\ncompliance with U.S. and local country laws. By using this product you\nagree to comply with applicable laws and regulations. If you are unable\nto comply with U.S. and local laws, return this product immediately.\n\nA summary of U.S. laws governing Cisco cryptographic products may be found at:\nhttp://www.cisco.com/wwl/export/crypto/tool/stqrg.html\n\nIf you require further assistance please contact us by sending email to\nexport@cisco.com.\n\nCisco IOSv (revision 1.0) with  with 460033K/62464K bytes of memory.\nProcessor board ID 92O0KON393UV5P77JRKZ5\n4 Gigabit Ethernet interfaces\nDRAM configuration is 72 bits wide with parity disabled.\n256K bytes of non-volatile configuration memory.\n2097152K bytes of ATA System CompactFlash 0 (Read/Write)\n0K bytes of ATA CompactFlash 1 (Read/Write)\n0K bytes of ATA CompactFlash 2 (Read/Write)\n10080K bytes of ATA CompactFlash 3 (Read/Write)\n\n\n\nConfiguration register is 0x0"
  }

To know how to write a parser for command_parser or textfsm_parser engine, please follow the user guide at https://github.com/ansible-network/network-engine/blob/devel/docs/user_guide/README.md.

To develop your own collections that use the ``network_engine`` role, refer to the user guide content for:

* Parser Directives (insert url)
* Filter Plugins (insert url)
* How to test (insert url)
