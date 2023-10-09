.NET Framework 4.8
====================
[![CI](https://github.com/deekayen/ansible-role-dotnet48/actions/workflows/ci.yml/badge.svg)](https://github.com/deekayen/ansible-role-dotnet48/actions/workflows/ci.yml) [![Project Status: Inactive – The project has reached a stable, usable state but is no longer being actively developed; support/maintenance will be provided as time allows.](https://www.repostatus.org/badges/latest/inactive.svg)](https://www.repostatus.org/#inactive)

Install (or uninstall) Microsoft .NET Framework 4.8 on Windows.

Requirements
------------

The target Windows machines must have whitelisted Internet access to download the .NET installer from [download.microsoft.com]().

Role Variables
--------------

By default, this role installs the .NET Framework. Toggling the `dotnet48_uninstall` variable from `false` to `true` will uninstall the framework if it exists.

Dependencies
------------

None.

Example Playbook
----------------

    - hosts: servers
      roles:
         - { role: deekayen.dotnet48, dotnet48_uninstall: false }

Example Install
---------------

    TASK [deekayen.dotnet48 : Install Microsoft .NET Framework 4.8.] ************
    ok: [10.0.0.100] => {"changed": false, "name": "https://download.microsoft.com/download/E/2/1/E21644B5-2DF2-47C2-91BD-63C560427900/NDP452-KB2901907-x86-x64-AllOS-ENU.exe"}

    TASK [deekayen.dotnet48 : debug] **********************************************
    ok: [10.0.0.100] => {
        "dotnet48_exe": {
            "changed": false,
            "name": "https://download.visualstudio.microsoft.com/download/pr/2d6bb6b2-226a-4baa-bdec-798822606ff1/8494001c276a4b96804cde7829c04d7f/ndp48-x86-x64-allos-enu.exe"
        }
    }

Because the uninstall task uses Ansible's `raw` module, the play output will always report `ok` status instead of `changed`. The playbook may also complete before the msiexec process has completely finished uninstalling the framework.

### Windows 2008R2

Microsoft .NET Framework must exist already on Windows 2008R2 for Ansible to connect and invoke Powershell. This module will confirm that the desired version is installed.

Example Uninstall
-----------------

    TASK [deekayen.dotnet48 : Uninstall Microsoft .NET Framework 4.8.] **********
    ok: [10.0.0.100] => {"changed": false, "rc": 0, "stderr": "", "stdout": "", "stdout_lines": []}

    TASK [deekayen.dotnet48 : debug] **********************************************
    ok: [10.0.0.100] => {
        "dotnet48_removed": {
            "changed": false,
            "rc": 0,
            "stderr": "",
            "stdout": "",
            "stdout_lines": []
        }
    }

### Caveat

Uninstalling .NET Framework on Windows 2008R2 will break Ansible's ability to invoke Powershell. You won't be able to reconnect with Ansible to the remote host until you re-install .NET Framework by some other means than this role.


License
-------

BSD
