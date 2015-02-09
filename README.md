libnss-ato (Name Service Switch module All-To-One)
==========

The libnss_ato module is a set of C library extensions which allows to map every nss request for unknown user to a single predefined user.

How We Use It
=========

On our system we wanted a secure way to allow ssh tunneling.  We can have many more users than would reasonably fit onto one machine.  We also wanted to avoid trying to sync those users between all of the machines.  

On one hand, systems like gitolite solve the problem by appending to authorized_keys. This is great for a few thousand users.  After that, the cost of linearly searching for the key is time consuming.  And you need to keep all those keys synced.  If the keys change often, that can be quite daunting.

On the other, ssh has AuthorizedKeysCommand which allows you to go look up keys for a username and return just those that should match.

This seemed promising!

The downside was that it looks things up by username, and linux does not let you log in with out a user account on the machine.

Enter - libnss-ato

Libnss-ato lets us resolve every username to the same user-uid, but *still keeping their username*.
So you can ssh into any box with look-up-my-public-keys@server.  libnss-ato will say you are a user.  sshd calls the AuthorizedKeysCommand with that username.  Letting you look up keys specifically for only that user.

In order to keep this more secure, we only allow ssh logins. Only ssh logins with keys.  We allow no commands to be run via ssh (only tunneling).  And the shell and home dir of the ato user is /bin/false and /dev/null.

So far this seems like it's working.


Installation
=========

From source just make and make install.

The only configuration file is `/etc/libnss-ato.conf` which consists of one line in the passwd format. For example:

```console
any_user:x:1000:1000:Test User,:/dev/null:/bin/false
```

Only the first line of the file `/etc/libnss-ato.conf` is parsed.

Here an example of the system file `/etc/nsswitch.conf` to make use of libnss-ato:

```console
passwd:         files ato
group:          files
shadow:         files ato
```

