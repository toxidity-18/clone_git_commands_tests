# GITHUBGIST[LINK](https://gist.github.com/alejandro-martin/aabe88cf15871121e076f66b65306610)
= Use Multiple SSH Keys for _Git_ host websites (Github, Gitlab)
:icons: font

This is guide about how to configure multiple _SSH_ keys for some _Git_ host
websites such as _Github_, _Gitlab_, among others.

== Creating SSH keys

. Create _SSH_ directory:

 mkdir ~/.ssh

. Move to created directory:

 cd ~/.ssh

. To create a _SSH_ key, type:
+
 ssh-keygen -t rsa -C "EMAIL@HOST.com"
+
a message will be displayed:
+
  Generating public/private rsa key pair.
  Enter file in which to save the key (/home/USER/.ssh/id_rsa):
+
You should type someething of the default name of the file to distinguish service,
such as: `id_rsa_myaccount_github`, `id_rsa_myaccount_gitlab`,
`id_rsa_mycompanyaccount_gitlab`.
+
After this step a _passphrase_ is needed for security, which can be empty.

. Repeat previous step for every required account.

. To see if the keys were successful created:
+
 ls ~/.ssh
+
which it is going to print all key files, for example:
+
```
  id_rsa_myaccount_github         id_rsa_myaccount_gitlab.pub
  id_rsa_myaccount_github.pub     id_rsa_mycompanyaccount_gitlab
  id_rsa_myaccount_gitlab         id_rsa_mycompanyaccount_gitlab.pub
```

== Creating _config_ file for manage _SSH_ keys

. To create `config` file:

 touch ~/.ssh/config

. Edit the file to configure _domains_ for the keys:
+
 nano ~/.ssh/config
+
for example, if three accounts were added, should look like this:
+
```
# github account
Host github.com
HostName github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa_myaccount_github

# gitlab account
Host gitlab.com
HostName gitlab.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa_myaccount_gitlab

# gitlab company account
Host gitlab.my_company.com
HostName gitlab.my_company.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa_mycompanyaccount_gitlab
```

. Save file and exit.

=== Using two accounts from the same server (website) [Optional]

A new host has to be created.

 . Create a new entry on the `~/.ssh/config` file(example with _Github_):

```
Host other.github.com
HostName github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa_otheraccount_github
```

where `other.github.com` is the alias for the host, although the server(`HostName`) is
`github.com`.
The `IdentityFile` option points to a (previously) created key file configured with
the required account.

After this, a custom url can be used to clone the project.

 git clone git@other.github.com:USER/REPOSITORY.git

where `other.github.com` is the previously created domain.

== Configure _SSH_ on Repository site

To configure ssh on each repository website:

. Copy the content of the `id_rsa_X.pub` with `xclip` command(may not be installed)
to the clipboard(where `id_rsa_X.pub` is the wanted key file):
+
 xclip -sel clip < ~/.ssh/id_rsa_X.pub
+
(content also can also be manually copied from the `*.pub` file)

. Paste the content on the repository site, check next section.

=== Configure _SSH_ on Github

. Go to https://github.com.

. Go to Profile _Settings_ > _SSH and GPG Keys_ > click on button _New SSH Key_.

. On _Title_ add a descriptive label, such as the _hostname_ of the device...

. On the _Key_ field past the clip content with the key.

. Finally click on _Add SSH key_ and after that the site ask for the user password.

=== Configure _SSH_ on Gitlab

. Go to https://gitlab.com.

. Go to Profile _Settings_ > _SSH Keys_.

. On the _Key_ field past the clip content with the key.

. On _Title_ add a descriptive label, such as the _hostname_ of the device...

. Finally click on _Add key_.


== Testing SSH Keys

. Type(substitute `HOST` with the desired one(_github_, _gitlab_, ...)):
+
 ssh -T git@HOST.com
+
a warning will appear, accept it with `yes`:
+
```
The authenticity of host 'HOST.com (IP ADDRESS)' can't be established.
RSA key fingerprint is 16:27:ac:a5:76:28:2d:36:63:1b:56:4d:eb:df:a6:48.
Are you sure you want to continue connecting (yes/no)?
```
+
A successful message will appear:
+
.. For _Github_:
+
```
Hi USERNAME! You've successfully authenticated, but GitHub does not
provide shell access.
```
+
.. For _Gitlab_:
+
```
Welcome to GitLab, USERNAME!
```

=== Delete _SSH_ Cache and add Keys

If the _SSH_ does not work, maybe the keys need to be added with `ssh-add` command

. First delete keys cache:

 ssh-add -D
+
if a message appear:
+
 Could not open a connection to your authentication agent.
+
use this command and after that retry:
+
 eval `ssh-agent -s`

. Add key file with `ssh-add` command:

 ssh-add ~/.ssh/id_rsa_file

. To see added keys, type:
+
 ssh-add -ld
+
and something such as this will be displayed:
+
```
2048 SHA256:DXlgYQo1o/65JQCSYQo/L4RRP4i+wTouyEetkOIcn/o EMAIL_1 (RSA)
2048 SHA256:4FPtZYDtHipZeHqP9KNB3Wslz9L5q/JoAGT3g/NW3O8 EMAIL_2 (RSA)
2048 SHA256:tXCoBI2dMtTFhUhE5oBT+XwwkrhkorkOHbSc1J22urQ EMAIL_3 (RSA)
```

. Retry testing connection.

== Using SSH keys

To use a git repository with the _SSH_, this url style has to be used for the
repository:

 git@HOST:USERNAME/REPOSITORY.git

where `HOST` is the configured domain, which can be _github_, _gitlab_ or a
personalized one.

If project origin is already configured with HTTPS, it has to be changed to the
SSH url style (check next section).

WARNING: If you want to use the HTTPS url, other steps will be required.

=== Change HTTPS url to SSH url [Optional]

. List existing remotes in order to get the name of the repository:

 git remote -v

. Change remote url, substitute HOST for server domain or a previously create
custom HOST:

 git remote set-url origin git@HOST:USERNAME/REPOSITORY.git

NOTE: It can be used the same method to change from SSH to HTTPS.

== Important: About `git config` user _name_ and _email_

In spite _SSH_ keys were configured for the access, the _Git_ user _name_
and _email_ need to be configured, because these will be associated to the
_commits_.

To see actual configuration, type:

 git config --list

If global user _name_ and _email_ were configured will be displayed at the beginning,
if not, these values will not appear.

NOTE: If the command was run on a repository and this one has a local
user _name_ and _email_ configured, these values will be displayed at the end of
configuration.

=== Configure user _name_ and _email_ on all repositories (globally)

IMPORTANT: You may want to configure the most used user _name_ and _email_ globally,
but watch out since every commit with not _local configuration_ will use these
values.

. Go to the root directory of the repository.

. Type to configure user _name_:

 git config --global user.name "YOUR NAME"

. Type to configure user _email_:

 git config --global user.email "email@HOST.com"

. To see if the *global* fields were correctly configured, use the
`git config --list` command or check global _Git_ file.
+
 nano ~/.gitconfig
+
this will display a section like:
+
----
[user]
	name = YOUR NAME
	email = email@HOST.com
----
+
NOTE: If the _user name_ or the _user email_ were not configured this section
will not appear.

=== Configure user _name_ and _email_ on a unique repository (locally)

. Go to the root directory of the repository.

. Type to configure user _name_:

 git config user.name "YOUR NAME"

. Type to configure user _email_:

 git config user.email "email@HOST.com"

 . To see if the *global* fields were correctly configured, use the
 `git config --list` command or check local _Git_ file:
+
 nano ./.git/config
+
this will display a section like:
+
----
[user]
	name = YOUR NAME
	email = email@HOST.com
----
+
NOTE: If the _user name_ or the _user email_ were not configured this section
will not appear.

== References

* https://docs.gitlab.com/ee/ssh/
* https://gist.github.com/jexchan/2351996
* https://help.github.com/articles/setting-your-commit-email-address-in-git/