# r10k Workflow: As-Built Reference

## Overview

r10k manages the production Puppet environment by syncing a self-hosted Gitea repo
to disk on the Puppet primary. The working copy at
/etc/puppet/code/environments/production/ is owned by r10k. Do not edit files there
directly — changes will be overwritten on the next deploy.

## Config

| Property | Value |
|---|---|
| Config file | /etc/puppetlabs/r10k/r10k.yaml |
| Gitea repo | http://gitea.lab.local:3001 |
| Deploy path | /etc/puppet/code/environments/production/ |
| Runs as | root |
| Deploy interval | Every 15 minutes via cron |

## Correct workflow

Every change follows this sequence. No exceptions.

1. Edit files locally in the repo working copy on the Puppet primary
2. Stage and commit

    cd /path/to/homelab-puppet
    sudo git add <file>
    sudo git commit -m "your message"

3. Push to Gitea

    sudo git push origin HEAD:production

4. Deploy via r10k (or wait up to 15 min for cron)

    sudo r10k deploy environment production -p

5. Dry run to verify no unintended changes

    sudo /opt/puppetlabs/bin/puppet agent -t --noop

6. Apply

    sudo /opt/puppetlabs/bin/puppet agent -t

## Why git operations require sudo

r10k runs as root and owns the working copy. The repo was cloned as root, so the
.git directory and all tracked files are root-owned. Running git as a non-root user
will fail or create ownership conflicts.

Always prefix git and r10k commands with sudo on the Puppet primary.

## The working-tree gotcha

If you edit a file directly under /etc/puppet/code/environments/production/ without
committing it to Gitea, the next r10k deploy will overwrite it. r10k does a hard reset
to match the Gitea repo state exactly. There is no merge — local edits are silently lost.

The only safe place to edit Puppet code is in the git working copy before pushing.

## Cron job

r10k deploys automatically every 15 minutes. To check the cron entry:

    sudo crontab -l | grep r10k

## Gitea repo structure

    homelab-puppet/
    manifests/
      site.pp
    modules/
      role/
      profile/
      (component modules)
    data/
      common.yaml
      common.eyaml
      nodes/
        <certname>.yaml
        <certname>.eyaml
    hiera.yaml
    Puppetfile
