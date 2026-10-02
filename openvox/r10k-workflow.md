# r10k Workflow: As Built Reference

## Config

| Property | Value |
|---|---|
| Binary | /opt/puppetlabs/puppet/bin/r10k |
| Config | /etc/puppetlabs/r10k/r10k.yaml |
| Cache | /var/cache/r10k/ (bare mirror of the Gitea repo) |
| Source | homelab-puppet on self hosted Gitea, production branch |
| Deploy path | /etc/puppet/code/environments/production/ |
| Working clone | ~/repos/homelab-puppet on openvox-cli |

## Deploys are manual on purpose

r10k has no cron or timer. Every deploy is run by hand so each change gets a noop review before it reaches the fleet. Drift correction does not depend on r10k: agents still enforce the deployed code every 30 minutes.

## Change workflow

Every change follows the same path. Output is reviewed at each step before moving on.

1. Edit in the working clone. Small in place edits are often done with a short patch script so the change is exact and repeatable.

2. Validate syntax.

       sudo /opt/puppetlabs/bin/puppet parser validate <file>.pp

3. Review the diff.

       git diff | cat

4. Commit and push to Gitea.

       git add <file>
       git commit -m "message"
       git push origin production

5. Deploy.

       sudo /opt/puppetlabs/puppet/bin/r10k deploy environment production -pv

6. Apply on the target node.

       sudo /opt/puppetlabs/bin/puppet agent -t

7. Verify directly on the node. A clean agent run is not proof the service is in the state you wanted.

## The working tree gotcha

r10k does a hard reset of the deploy path to match the repo. Edits made directly under /etc/puppet/code are silently lost on the next deploy. The working clone is the only safe place to edit.

## Repo layout

    homelab-puppet/
      manifests/site.pp
      modules/
        role/manifests/
        profile/manifests/
        profile/templates/
        base/manifests/
      data/
        common.yaml
        common.eyaml
        nodes/<certname>.yaml
        nodes/<certname>.eyaml
      hiera.yaml
      Puppetfile
      docs/
