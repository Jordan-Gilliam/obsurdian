# irc from the command-line

Install irssi:

    $ sudo apt-get install irssi

Then create a screen for irc:

    $ screen -S irc

---
Then in a screen, you can run `irssi` to use irc from the
command line.

* `/nick robowizard` - set your nickname on the server
* `/connect irc.freenode.net` - connect to irc.freenode.net
* `/join #cyberwizard` - join the channel called cyberwizard
* ESC+N or `/win N` - to jump to the window at number N

Once you're in a channel, type text like usual.
`CTRL+A d` to detach and `screen -x irc` to resume.

---

#unix #bash #irc