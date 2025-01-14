# Guide to access controls for infrastructure

This document provides an overview of the access controls which are applied
at different parts of the project in order to secure the systems and allow
people to have access when required for onboarding and offboarding people. 
Note that this document's scope does not cover the committer/contributor
process for the Adoptium github org which is managed by Eclipse and
controlled via their [normal committer election process](https://www.eclipse.org/membership/become_a_member/committer.php).

## Jenkins build/test agents

(At present, build and test agents have the same access controls on them but
that will potentially change in the future as we look to lock down the build
systems to a higher level)

Administrative access to the build systems is currently granted to all members of the
infrastructure team although in some cases individuals may have more
restricted access (e.g.  to administer certain types of machine).  These
policies are controlled by [Bastillion](https://www.bastillion.io/) which distributes the infrastructure
team's ssh keys to the systems.  Addition and removal of users to profiles
in there can be done when people are onboarded/offboarded. For Windows
systems, the credentials are currently stored in our secrets repostory (See
later).

For Bastillion, the relevant groups are `root`, `root-marist`, `root-aix`,
`root-equinix` and `jenkins`. Each infrastructure team user should generally be added
to all of those.
([Related issue](https://github.com/adoptium/infrastructure/issues/2970))
If we implement stricter controls on the static build systems then it is
likely we would have a `root-build` profile in Bastillion to manage those.

**Onboarding process:** Add a User to Bastillion and add the user to required profiles

**Offboarding process:** Remove the User from the required profiles, remove the account


## Build infrastructure e.g. jenkins/AWX/Bastillion

Jenkins and AWX are controlled by Github teams in the AdoptOpenJDK
organisation. For AWX it is currently using the
[infrastructure-secret](https://github.com/orgs/AdoptOpenJDK/teams/jenkins-admins/members)
team as per the
[AWX setup docs](https://github.com/adoptium/infrastructure/wiki/Ansible-AWX#authentication)
Jenkins administrative access is controlled via the
[jenkins-admins](https://github.com/orgs/AdoptOpenJDK/teams/jenkins-admins/members)
group in GitHub. Shell access to those machines is currently controlled independently
by manually adding keys on both servers.

Nagios access is managed via the `/usr/local/nagios/etc/htpasswd.users` file
(`.htaccess` format) on the server. Shell access is currently manually
controlled, but will be moved to a Bastillion group.

**Onboarding (Normal user)**: Add to infrastructure-secret for AWX access,
and Nagios htpasswd.users file.  Shell access to Nagios, TRSS, and AWX can be
granted if required.
**Offboarding (Normal user)**: Reverse the above

**Onboarding (Superadmin user)**: Add to `AdoptOpenJDK*jenkins-admins` GitHub team
and grant shell access via root's authorized_keys to the jenkins server,
and Bastillion. Note that Bastillion admin access is available through
BitWarden in the `Internal Services` group.

**Offboarding (Superadmin user)**: Remove from the above, change Bastillion
admin password

## Bastillion

Since I've mentioned Bastillion in the earlier section, here are the full
list of groups which we have defined on the server and what they control:

Note that while Bastillion typically controls access to the root account,
there is also a `jenkins` group which the infrastructure team is added to
which allows direct access to the jenkins user. This can, if required, be
granted to people outside the infrastructure team.

Group name | What is in it | Who has access
--- | --- | ---
jenkins | jenkins user on all systems | infra team
root-marist | All marist systems |  infra team + Marist keys
root-aix | Root access to AIX hosts | Infra team + AIX tools + NIM
root-equinix | Root access to Equinix hosts | Infra team + Equinix keys
root | Root access to all other build+test hosts | Infra team
infrahosts | root access to infra services | Infra team
infraadmin | root access to Jenkins/Bastillion | TBC 
root-build | root access to static build machines | (Not yet implemented 

## Temporary test system access

Occasionally non-infrastructure team members require access to the machines
in order to debug problems when they are not able to do so on their local
machines, for example if it's a type of machine they do not otherwise have
access to. In this case we grant people the least amount of access possible.

To request this, a user should create an issue in the infrastructure
repository using [this
template](https://github.com/adoptium/infrastructure/issues/new?assignees=sxa&labels=Temp+Infra+Access&template=machineaccess.md&title=Access+request+for+%3Cyour+username%3E)
which will be labelled as "Temp Infra Access" and allows them to specify and
justify the level of access.  In most cases this will be a normal user
account which will be granted access via their ssh key.  In very rare cases
access to jenkins or root may be provided, although bear in mind this will
be very short lived as Bastillion will typically override that access.  In
either case, the issue should be left open until the access is removed.

**Onboarding**: For normal user, create a new user ID and populate
~user/.ssh/authorized_keys with their ssh key (Often from
https://github.com/user.keys).  Ensure that an issue exists to track that
the access has been granted.  For jenkins/root if needed, add user's key to
the corresponding authorized_keys file.

**Offboarding**: Delete the user or keys that were added, add a comment to
the issue saying it has been done, close the issue. 

See also the [end-user FAQ entry](https://github.com/adoptium/infrastructure/blob/master/FAQ.md#temporary-access-to-a-machine)
on this.

*Note that we don't have a formal process for Windows systems (which require
RDP and a password) at present since users typically have access to those
elsewhere, but a similar process could be followed for those.*


## Infrastructure providers

Access to infrastructure providers to be able to provision and/or reset can
be provided to the infrastructure team. The access to this is generally
provided via BitWarden, so the user needs to have an account in the Adoptium
org, for which we have a limited number of seats. The `Cloud Providers`
folder has the credentials for the providers. There is also a
"Cloud-providers (Chargeable)" sub-folder to make it clear which providers
will incur costs if systems are provisioned in there.

We also have a "Cloud providers (restricted)" which hosts some of the
critical parts of the infrastructure which are likely to be restricted
further which includes our jenkins server. Where possible, all of these
accounts have 2FA enabled using the BitWarden 2FA support.

**Onboarding** An Adoptium BitWarden admin needs to onboard the user into
the Adoptium org, and grant access to the `infrastructure` Group (or
`adopt-infra-admins` for some of the providers)

**Offboarding** Remove access, change relevant passwords.

## secrets / dotgpg

We currently store various credentials in the private secrets repository where the
access to data in there is
([currently](https://github.com/adoptium/infrastructure/issues/2661))
controlled using the [dotgpg](https://github.com/ConradIrwin/dotgpg) tool
which is a wrapper around `gpg`. Access to the repository is via the
[AdoptOpenJDK*infrastructure-secret](https://github.com/orgs/AdoptOpenJDK/teams/infrastructure-secret)
group. The user also requires a GPG key which will give them access to
decrypt the data held in that repository.

Note that the dotgpg tool itself is only required for people who are adding
and removing users' keys. Others can just [use the `gpg` tool
itself](https://github.com/ConradIrwin/dotgpg#use-without-ruby) to
manage the files in the secrets repository, although this can be less convenient.

**Onboarding**: Add to infrastructure-secrets, add GPG key to the repository using `dotgpg add`

**Offboarding**: Remove from infrastructure-secrets. Remove access with `dotgpg rm`
Change any credentials which the user had access to.

## Third party services

Currently access to these are granted on a case by case basis.  These
include dockerhub, CloudFlare, JFrog,  and others.  We are finalising the
migration to BitWarden so this section will be updated later :-)

