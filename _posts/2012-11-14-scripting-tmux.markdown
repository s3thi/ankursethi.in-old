---
author: ankur
date: 2012-11-14 14:42:56+00:00
layout: post
slug: scripting-tmux
title: Scripting tmux
categories:
    - Programming
---

[tmux](http://tmux.sourceforge.net/) ranks highly on the list of programs that I cannot live without. I consider its split-screen and terminal multiplexing capabilities absolutely essential for day-to-day hacking. It belongs to that rare breed of software that has _measurably_ improved my development productivity, software that makes me _genuinely happy_.

This is what my tmux pane layout usually looks like when I'm working on a project:

         ---------------------------------------------------------------
        |                               |                               |
        |                               |                               |
        |                               |                               |
        |                               |             repl              |
        |                               |                               |
        |       directory operations    |                               |
        |       git interactions        |-------------------------------|
        |       etc.                    |                               |
        |                               |        directory watcher      |
        |                               |        server                 |
        |                               |        logger                 |
        |                               |        etc.                   |
        |                               |                               |
         ---------------------------------------------------------------

Since I use this layout for nearly every project that I work on, it makes sense to have tmux automatically set it up for me so that I don't have to type a bunch of keyboard shortcuts every time I start working on a new project. Luckily, tmux is highly scriptable. Here's a BASH function that automatically sets up the three-pane layout from above:

{% highlight bash %}
setup_tmux_layout() {
    # Create a new window.
    tmux new-window -a -n "$1" -c "$2"

    # Now split it twice, first horizontally and then vertically.
    tmux split-window -h
    tmux split-window -v
}
{% endhighlight %}

Now I can run the following from my shell:

{% highlight bash %}setup_tmux_layout <window_name> <starting directory>{% endhighlight %}

(Note that at least one tmux session must be active for this to work. This function affects the most recently active tmux session.)

This is good, but we can go a step further. I have several Django projects, and whenever I start working on one of them there are a number of additional Python/Django-specific actions that I take.
	
  1. Switch to the project directory in all three panes.	
  2. Activate the project's virtualenv in all three panes.
  3. Run a `git status` in the large pane on the left.
  4. Start a Django shell (`python manage.py shell`) in the top right pane.
  5. Run the Django development server (`python manage.py runserver`) in the bottom right pane.

Once again, it makes sense to automate these actions. Here is the actual code I have in my `~/.bash_profile` to do that.

{% highlight bash %}    
workon_project() {
    if [ $# -lt 2 ]
    then
        echo "Usage:"
        echo -e "tworkon_project <project directory> <virtualenv name>"
        return 1
    fi

    # Create a new window.
    tmux new-window -a -n "$2" -c "$1"

    # Send keys to the large pane on the left.
    tmux send-keys "workon $2" C-m
    tmux send-keys "git status" C-m

    # Split the window horizontally.
    tmux split-window -h -c "$1"

    # Send keys to the top right pane.
    tmux send-keys "workon $2" C-m
    tmux send-keys "python manage.py shell" C-m

    # Split the window again, this time vertically.
    tmux split-window -v -c "$1"

    # Send keys to the bottom right pane.
    tmux send-keys "workon $2" C-m
    tmux send-keys "python manage.py runserver" C-m
}
{% endhighlight %}

Brilliant! Now I can perform all those tedious actions in one fell swoop by running this in my shell:
 
{% highlight bash %}$ workon_project <project directory> <virtualenv name>{% endhighlight %}

(In case you are wondering, the `workon` command above comes from [virtualenvwrapper](http://www.doughellmann.com/projects/virtualenvwrapper/).)

This little function has saved me hundreds of keystrokes in the last few months, and it only scratches the surface of what tmux is capable of. If you are a regular tmux user, I highly recommend skimming the tmux manpage at least once.