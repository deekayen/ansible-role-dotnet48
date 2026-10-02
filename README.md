# deekayen.dotnet48

[![CI](https://github.com/deekayen/ansible-role-dotnet48/actions/workflows/ci.yml/badge.svg)](https://github.com/deekayen/ansible-role-dotnet48/actions/workflows/ci.yml) [![Ansible Galaxy](https://img.shields.io/badge/galaxy-deekayen.dotnet48-blue.svg)](https://galaxy.ansible.com/ui/standalone/roles/deekayen/dotnet48/) [![Project Status: WIP – Initial development is in progress, but there has not yet been a stable, usable release suitable for the public.](https://www.repostatus.org/badges/latest/wip.svg)](https://www.repostatus.org/#wip) ![BSD 3-Clause license](https://img.shields.io/badge/license-BSD%203--Clause-blue)

An Ansible role that installs Microsoft .NET Framework 4.8 on a Windows host and reboots it, or uninstalls the framework when `dotnet48_uninstall` is `true`.

The install task passes the `ndp48-x86-x64-allos-enu.exe` offline installer URL on `download.visualstudio.microsoft.com` to `ansible.windows.win_package`, with product ID `{16735AF7-1D8D-3681-94A5-C578A61EC832}` as the installed-state check. The uninstall task is a `raw` PowerShell command that runs `msiexec.exe /x` against the same product ID with `/qn` and waits for it to exit. Microsoft describes the installer on the [.NET Framework 4.8 offline installer](https://support.microsoft.com/en-us/topic/microsoft-net-framework-4-8-offline-installer-for-windows-9d23f658-3b97-68ab-d013-aa3c3e7495e0) page.

The role installs 4.8, not 4.8.1. A comment in `vars/main.yml` says the role stays on 4.8 until the 4.8.1 product code is known. The Microsoft Update Catalog lists the 4.8.1 packages under [KB5011048](https://www.catalog.update.microsoft.com/Search.aspx?q=KB5011048).

## Requirements

- ansible-core 2.15 or newer on the controller.
- The `ansible.windows` collection: `ansible-galaxy collection install ansible.windows`.
- A WinRM or SSH connection to the target with administrative rights.
- Outbound HTTPS from the target to `download.visualstudio.microsoft.com` for installs.

## Supported platforms

| Platform | Versions |
| --- | --- |
| Windows | 2016, 2019, 2022, 2025 |

CI lints the role and runs `ansible-playbook --syntax-check`; it does not apply the role to a Windows host.

## Installation

From Ansible Galaxy:

```bash
ansible-galaxy role install deekayen.dotnet48
ansible-galaxy collection install ansible.windows
```

Or pin it in `requirements.yml`:

```yaml
---
roles:
  - name: deekayen.dotnet48
    src: https://github.com/deekayen/ansible-role-dotnet48.git
    scm: git
    version: main

collections:
  - name: ansible.windows
```

```bash
ansible-galaxy install -r requirements.yml
```

## Role variables

| Variable | Default | Description |
| --- | --- | --- |
| `dotnet48_uninstall` | `false` | Boolean. `false` installs the framework; `true` uninstalls it. `meta/argument_specs.yml` validates the type. |

The installer URL is the internal `ndp_install_url` value in `vars/main.yml`.

## Behavior

- When the install task reports `changed`, it notifies a handler that reboots the host with `ansible.windows.win_reboot`. No variable turns that reboot off.
- Every uninstall run reports `changed`, since the `raw` task sets `changed_when: true`. The task does not check the `msiexec` exit code or whether the product is still installed afterward, and it does not reboot.

## Dependencies

None. The `ansible.windows` collection is a requirement, not a role dependency.

## Example playbook

```yaml
---
- name: Install .NET Framework 4.8.
  hosts: windows_app_servers

  roles:
    - deekayen.dotnet48
```

## Known issues

- `tasks/main.yml:6` passes `arguments: '/qn'` to the installer. Microsoft's [.NET Framework deployment guide](https://learn.microsoft.com/en-us/dotnet/framework/deployment/deployment-guide-for-developers) lists `/q` for quiet mode and `/norestart` to suppress the installer's own reboot; `/qn` is not in that list.
- The install and uninstall tasks both register `dotnet48_installer` (`tasks/main.yml:7` and `tasks/main.yml:15`). Ansible registers a result for skipped tasks too, so in install mode the skipped uninstall task overwrites the install result and the debug task prints a skip.

## Development

CI runs on every push to `main` and every pull request (see `.github/workflows/ci.yml`). It installs `ansible.windows`, runs `ansible-lint --profile production`, and syntax-checks `tests/test.yml`. To run the same checks locally:

```bash
pip3 install ansible-lint
ansible-galaxy collection install ansible.windows
ansible-lint --profile production
mkdir -p .ansible/roles && ln -sfn "$PWD" .ansible/roles/deekayen.dotnet48
ANSIBLE_ROLES_PATH=.ansible/roles:~/.ansible/roles ansible-playbook --syntax-check tests/test.yml -i tests/inventory
```

The repository also has a `.pre-commit-config.yaml`; run `pre-commit run --all-files` before pushing.

### Repository layout

| Path | Purpose |
| --- | --- |
| `tasks/main.yml` | Install and uninstall tasks, selected by `dotnet48_uninstall`. |
| `handlers/main.yml` | Reboot after an install. |
| `vars/main.yml` | Installer URL. |
| `defaults/main.yml` | The one user-facing variable. |
| `meta/argument_specs.yml` | Type validation for the variable. |
| `tests/` | Syntax-check playbook and inventory used by CI. |

## Releases

Pushing a git tag runs `.github/workflows/release.yml`, which imports the tagged commit into Ansible Galaxy as `deekayen.dotnet48`. The import needs a `GALAXY_API_KEY` repository or organization secret.

## License

BSD 3-Clause. See [LICENSE](LICENSE).

## Author

[David Norman](https://github.com/deekayen). Sponsorship links are in [.github/FUNDING.yml](.github/FUNDING.yml).
