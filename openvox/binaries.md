# Binaries and Common Commands

Paths are unchanged from Open Source Puppet. Use full paths with sudo.

| Tool | Path |
|---|---|
| puppet agent | /opt/puppetlabs/bin/puppet |
| r10k | /opt/puppetlabs/puppet/bin/r10k |
| eyaml | /opt/puppetlabs/puppet/bin/eyaml |

## Common commands

Agent run:

    sudo /opt/puppetlabs/bin/puppet agent -t

Dry run:

    sudo /opt/puppetlabs/bin/puppet agent -t --noop

Validate a manifest:

    sudo /opt/puppetlabs/bin/puppet parser validate <file>.pp

Deploy:

    sudo /opt/puppetlabs/puppet/bin/r10k deploy environment production -pv

List certs on the primary:

    sudo puppetserver ca list --all

Clean a cert on the primary (`puppet cert` no longer exists in version 8):

    sudo puppetserver ca clean --certname <certname>

Check root cron on any host:

    sudo crontab -l -u root

## Logs

Pipe journal output through cat so long lines are not truncated:

    sudo journalctl -u <service> --no-pager | cat
