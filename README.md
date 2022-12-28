# Automated backups with `restic`

This is an [Ansible][ansible] role for deploying and configuring `restic` as a
backup solution.

[ansible]: https://ansible.com

## Required variables

| **Name** | **Type** | **Description**
| -------- | -------- | ---------------
| `backup_restic_password` | string |  The encryption passphrase Restic will use to secure the backup keys. |
| `backup_env` | array | list of dicts with `key` and `value`: environment variables to set to configure Restic, generally to set config and credentials for your backup target (cloud endpoints, tokens, etc.). Refer to [the Restic documentation][resticenv] for details on what you need to set. |
| `backup_restic_repository` | string | The Restic repository identifier/path. |

### Example

```yaml
backup_restic_password: "swordfish123"
backup_env:
  - key: AZURE_ACCOUNT_KEY
    value: "asdfasdfasdf"
  - key: AZURE_ACCOUNT_NAME
   value: "mybackupstorageaccount"
backup_restic_repository: "azure:mycontainer:/"
```

[resticenv]: https://restic.readthedocs.io/en/stable/040_backup.html#environment-variables

## Initialisation

Restic needs to "initialise" a backup repository before you can use it as a
backup target. This role supports this through the usage of a
`backup_initialize=true` variable.

Set this temporarily (with all other vars set) to have Ansible initialise your
backup target for you.

For example:

```
$ ansible-playbook -e backup_initialize=true myplaybook.yml
```

## Usage

Install the role by adding either the Ansible Galaxy package or Git remote to
your `requirements.yml`:

```yaml
# via Galaxy
- src: blackieops.backup

# or via Git
- src: https://github.com/blackieops/ansible-role-backup.git
```

Then install it with `ansible-galaxy`:

```
$ ansible-galaxy install -r requirements.yml
```

Finally, you can reference the role in your playbooks:

```yaml
- hosts: all
  roles:
    - { role: blackieops.backup }
```

## Optional Configuration

### File inclusion and exclusion

Two variables control which folders and files get backed up:

* `backup_include` (Default: `["/srv", "/home", "/root"]`) List of paths to be backed up.
* `backup_exclude` (Default: `[]`) List of paths/patterns to exclude from the backup.

### Scheduling

This role will configure a cron job to regularly run the backup script. By
default, it runs every day at 02:15.

* `backup_schedule_day` (Default: `*`) What days to run the backup.
* `backup_schedule_hour` (Default: `2`) What hours to run the backup.
* `backup_schedule_minute` (Default: `15`) What minutes to run the backup.

* `backup_user` (Default: `root`) Sets the user whose crontab we will configure
  to run the backup.

### Preparation Commands

If you want to do something before the backup starts, such as flush some caches
to disk or export something, you can do so by providing a list of commands to
run.

* `backup_prepare_commands` (Default: `[]`)

### Alternative Architectures / Supply Chains

Restic is downloaded from the official GitHub releases on each machine. If you
need a system architecture other than `amd64`, or need to provide a custom
download URL, you can override a couple variables to do so.

* `backup_restic_arch` (Default: `amd64`) The architecture to support.
* `backup_restic_release_checksum` (Default: `sha25:...`) The checksum to verify the downloaded file is legitimate and intact. (You will need to update this if you change any other variable in this section.)
* `backup_restic_release_url` (Default: `"https://github.com/restic/restic/releases/download/v{{ backup_restic_version }}/{{ backup_restic_artifact }}.bz2"`) The URL from which Restic will be downloaded.
* `backup_restic_version` (Default: `"0.14.0"`) The version of Restic to download.
