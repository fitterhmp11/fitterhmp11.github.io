

I have 2 user accounts in my GCP Ubuntu server, one that was created with the GCP web Console - fitterhmp_plab and another that I used from my local machine to ssh - fitterhmp11.
I noticed I could run some commands differently depending which user was logged in.  Had to use Sudo for some commands

Used chatGPT to fix issue and learn more about user linux management

how to show all users on system?
`cat /etc/passwd`

Each line shows a user's details: username:x:UID:GID:comment:home_directory:shell.
System accounts have UIDs below 1000 (usually).

how to list groups and memberships?
`cat /etc/group`
this shows all groups and memberships
`groups <username>`
this shows which groups a user is member of.

How to change group membership?

to add a user to a group:
`sudo usermod -aG <group> <username>`
the -a appends, thus not overwriting existing users, the G allows you to specify the group(s) to add (can list multiple id group1,group2,groupr3)

to remove a user from a group:

`sudo gpasswed -d <username> <group>`


User needs to log out and log back in for the group changes to take effect.
