# macOS Login Item Shell Script (No Terminal)

It is easy to add a Login Item in macOS.
Let's imagine we want to test a simple infinite-loop shell script like this one:

```sh
while true
do
  date -Iseconds
  sleep 1
done
```

- Go to Apple logo > System Settings > General > Login Items > Open at Login
- Press + to locate your shell script, use Cmd+Shift+. to show hidden paths!
- Sign out of your macOS user session using Apple logo > Log Out `whoami`
- Sign back in to macOS and run `ps` in Terminal to see the shell script process
  (Identified by the path of the shell script selected in the previous steps.)

However, we don't see a Terminal window with the outputs of the script process.
How can we be sure it is actually running?

Well, a crude way to test it is to make the shell script have an observable side
effect, like dropping a file somewhere.
Let's use a script like this:

```sh
while true
do
  date -Iseconds >> ~/Desktop/login-item-shell-script.log
  pwd >> ~/Desktop/login-item-shell-script.log
  sleep 1
done
```

We use a full path of the ouput file because we can't be sure what it's relative 
to.
We'll find that out as well thanks to the `pwd` line.

So - that didn't work and the script no longer appears even in `ps`.
Most likely this means the script crashed and chances are the reason why is
because the shell it is run in doesn't resolve `~` to your user directory the
same way Terminal does.
We will need to hardcode this path.
We can either run `cd ~` followed by `pwd` in Terminal to know what to replace
`~` with or if we don't want to hardcode the user name into the script, we can
instead use `/Users/$(whoami)` instead of `~`.

The script now looks like this:

```sh
while true
do
  date -Iseconds >> /Users/$(whoami)/Desktop/login-item-shell-script.log
  pwd >> /Users/$(whoami)/Desktop/login-item-shell-script.log
  sleep 1
done
```

Let's sign out and back in again and see if `login-item-shell-script.log` will
contain any new entries and `ps` will show the infinite loop script is running.

Well, it is not.
Let's hardcode the username and take the L here.

…And that didn't work either.
I actually don't know what the problem is with the script at this point, but
that only helps drive home the point of what I am going to present next, which
should be much simpler.

Let's replace the startup script with this:

```sh
screen -d -m /Users/…/Desktop/…/long-running-script.sh -S my-long-running-script
```

This should make the login item shell script start a default-detached `screen`
session which will run in the background, but will allow us to drop in on it
and check its outputs through the Terminal as well as drop back out if we like
the status.
Let's see if this works as a login item shell script.

…And again, it didn't work.
I don't know why it worked the first time but not any later time.

Anyway, there is this thing where you can name the login item file `.command`
and it will run it in Terminal.
The upside is that it actually runs the scripts, but the obvious downside is
that the Terminal is shown with the scripts output and when closed, terminates
the script.

Let's put the screen call into the `.command` file and see if we can start the
screen session detached and exit right away.

```sh
screen -d -m /Users/…/Desktop/…/long-running-script.sh -S my-long-running-script
exit
```
