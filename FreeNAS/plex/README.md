### Create an SSH tunnel to the Plex Jail (only necessary for the initial setup)
Source: https://www.ixsystems.com/community/threads/plex-cannot-find-a-server.58954/page-2

Open a shell for the jail through the FreeNAS Web GUI

Edit `/etc/rc.conf` (you can use ee (easy editor) or vi if you're a psychopath), and add the following line anywhere in the file:
`sshd_enable="YES"`

Start SSH with the command: `service sshd start`

Add a user by typing `adduser` and following the prompts

When you get to the prompt to add your new user to any additional groups, add it to wheel:
Login group is NewUserThatYouJustCreated. Invite NewUserThatYouJustCreated into other groups? [ ]: wheel

Set the root password so that the new user will be able to use the su command to gain superuser privilege. To set the password, type in passwd and entire your desired password.

Create your tunnel
7.a) On OSX or Linux, run the command `ssh IP.address.of.server -L 8888:localhost:32400`
