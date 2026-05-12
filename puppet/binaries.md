# Puppet Binaries and Tool Locations: As-Built Reference

## Binary paths

All Puppet binaries require full paths when run with sudo.

| Binary | Full Path | Purpose |
|---|---|---|
| puppet agent | /opt/puppetlabs/bin/puppet | Agent CLI — runs, tests, noop |
| puppetserver | /usr/bin/puppetserver | Server process management |
| puppetserver CA | /usr/bin/puppetserver ca | Cert signing, cleaning, listing |
| r10k | /opt/puppetlabs/puppet/bin/r10k | Environment deployment |
| eyaml | /opt/puppetlabs/puppet/bin/eyaml | Secret encryption/decryption |

## Common commands

### Run puppet agent
    sudo /opt/puppetlabs/bin/puppet agent -t

### Dry run (noop)
    sudo /opt/puppetlabs/bin/puppet agent -t --noop

### Deploy r10k
    sudo /opt/puppetlabs/puppet/bin/r10k deploy environment production -p

### List all certs
    sudo /usr/bin/puppetserver ca list --all

### Clean a cert
    sudo /usr/bin/puppetserver ca clean --certname <certname>

### Encrypt a secret with eyaml
    /opt/puppetlabs/puppet/bin/eyaml encrypt -s 'the secret value'

## Ruby environments

Two separate Ruby environments are active on the Puppet primary. Both matter.

| Environment | Path | Used by |
|---|---|---|
| Agent Ruby | /opt/puppetlabs/puppet/bin/ruby | puppet agent CLI, r10k, eyaml CLI |
| puppetserver JRuby | /opt/puppetlabs/server/bin/ | puppetserver JVM process |

### Installing gems

Gems must be installed in both environments if they need to work at both the
agent CLI level and inside puppetserver catalog compilation.

Agent Ruby:
    sudo /opt/puppetlabs/puppet/bin/gem install <gemname>

puppetserver JRuby:
    sudo /opt/puppetlabs/server/bin/gem install <gemname>

eyaml is installed in both — required for CLI encryption and server-side decryption
during catalog compilation.

## r10k config

| Property | Value |
|---|---|
| Config file | /etc/puppetlabs/r10k/r10k.yaml |
| Binary | /opt/puppetlabs/puppet/bin/r10k |
| Runs as | root (always use sudo) |
| Source repo | Self-hosted Gitea instance |
| Managed environment | production |
| Deploy path | /etc/puppet/code/environments/production/ |
| Cron interval | Every 15 minutes |

Check cron:
    sudo crontab -l | grep r10k

## Service management

| Service | Command |
|---|---|
| Start puppetserver | sudo systemctl start puppetserver |
| Stop puppetserver | sudo systemctl stop puppetserver |
| Restart puppetserver | sudo systemctl restart puppetserver |
| Status puppetserver | sudo systemctl status puppetserver --no-pager |
| Start puppet agent | sudo systemctl start puppet |
| Stop puppet agent | sudo systemctl stop puppet |
| Agent status | sudo systemctl status puppet --no-pager |
