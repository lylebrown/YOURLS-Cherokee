What your [Cherokee Web Server](http://cherokee-project.com/) config for [YOURLS](https://github.com/YOURLS/YOURLS), a self-hosted URL shortener, may look like - heavily based off of a guide by Jeff on UtuTech ([Link through Wayback Machine](https://web.archive.org/web/20110210161520/http://www.ututech.com/2010/10/configuring-yourls-to-work-with-cherokee-web-server/))

## Configure YOURLS

Edit the `user/config.php` file, as detailed on the [YOURLS site](http://yourls.org/#Config). 

One suggestion is to add 'admin' to the list of reserved URL keywords, to prevent potential issues with URL rewrite rules. Otherwise, the configuration is standard.

## Cherokee Configuration through `cherokee-admin`

**Note**: These rules assume your YOURLS site is at the document root. If it is located in a subdirectory, you may need to make a few changes.

1. Select or configure the Virtual Server you intend to use YOURLS on.
2. Add your PHP handler to the Virtual Server if you have not already done so.
3. Open the behavior rules management.
4. Add a new behavior with a type of "Regular Expression". The RegEx we're going to match is `^\/admin(*SKIP)(*FAIL)|^\/([0-9a-z]+)(\+)?(all)?\/?$`. (Kudos to @eionrobb for assisting with this.)
5. Select the Handler tab for the Regular Expression just created, and add the following RegEx/Substitution records:

| Type        | Regular Expression           | Substitution  |
| ------------- |:-------------:| -----:|
| Internal      | `^/([0-9a-z]+)/?$` | `/yourls-go.php?id=$1` |
| Internal      | `^/([0-9a-z]+)\+/?$`      |   `/yourls-infos.php?id=$1` |
| Internal | `^/([0-9a-z]+)\+all/?$`      |    `/yourls-infos.php?id=$1&all=1` |
6. Leave the default "List and Send" handler at the bottom.
7. Save and restart Cherokee.

## Cherokee Configuration through `cherokee.conf`

While Cherokee documentation does not recommend directly editing the configuration file, you can also do so by adding the following entries.

**Note**: You will most likely need to change the rule number to be 100 higher than your existing rules for the vServer. This only includes the rewrite rule.

```
vserver!6!rule!600!handler = redir
vserver!6!rule!600!handler!rewrite!10!regex = ^/([0-9a-z]+)/?$
vserver!6!rule!600!handler!rewrite!10!show = 0
vserver!6!rule!600!handler!rewrite!10!substring = /yourls-go.php?id=$1
vserver!6!rule!600!handler!rewrite!20!regex = ^/([0-9a-z]+)\+/?$
vserver!6!rule!600!handler!rewrite!20!show = 0
vserver!6!rule!600!handler!rewrite!20!substring = /yourls-infos.php?id=$1
vserver!6!rule!600!handler!rewrite!30!regex = ^/([0-9a-z]+)\+all/?$
vserver!6!rule!600!handler!rewrite!30!show = 0
vserver!6!rule!600!handler!rewrite!30!substring = /yourls-infos.php?id=$1&all=1
vserver!6!rule!600!match = request
vserver!6!rule!600!match!request = ^\/admin(*SKIP)(*FAIL)|^\/([0-9a-z]+)(\+)?(all)?\/?$
```
