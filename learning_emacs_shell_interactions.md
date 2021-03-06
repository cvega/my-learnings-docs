---
path: "/learnings/dmacs_shell_interactions"
title: "Learnings: Emacs: Shell Interactions"
---


# EShell

Emacs shell, with corebins reimplemented in Lisp, so it runs anywhere, even in non POSIX environments (looking at you, Windows versions without WSL installed...)

Because this ALL happens in emacs you can do emacs things to edit your text.

Like:

  1. Turn line numbers on M-x linum-mode , for me, still on emacs 24 because distro). Set it globally by putting this in init.el, but this may be stupid ((global-linum-mode) .
  2. use emacs built in find, navigation

This also does the thing I want natively: being able to go into the output of a command, edit it, and run the (previously outputted!) line like another command.

## Neat workflow thing: Plan 9 smart display

enable by putting this in init.el

       (require 'eshell)
       (require 'em-smart)

       (add-hook 'eshell-mode-hook
       		 (lambda ()
    		 (eshell-smart-initialize)))

What this does is doesn't move the cursor to the next, blank, line, but assumes you want to adjust the output of the previous command.

This actually is useful, because it ALSO means that you don't lose track of "where is the start of this command's output?"

Except when you scroll down with your scrollwheel you do lose that a bit. Work around: use SPACE to go down and BACKSPACE to go up. It will stop you at the top of the executed command, so you don't go up into the previous command's output.

## Implementing things I miss from BBEdit

### Text Filters on selected text: running them through an arbritrary shell command

	(defun bb/filter-region-through-shell-command ()
	  "Like BBEdit's Text Filters, but the command is selected at runtime. Also I can never remember the universal modifier"
	  (interactive)
	  (shell-command-on-region (region-beginning) (region-end) (read-string "Command to execute: ") "current buffer" t))

This lets us select say the following text

list
i
want
alphabetized

and run it through the sort shell command.

Note: this only runs it through the shell shell, not eshell

### Running an arbitrary (eshell) command

	(defun bb/script-eval-selected-in-eshell ()
	  "Like BBEdit's script functionality, but (a) command is selected at runtime by looking at the region. Either switch to the eshell buffer OR use the fancy eshell redirection properties to get the output out."
	  (interactive)
	  (let ((oldbuff (current-buffer))) 
	    (run-this-in-eshell
	     (buffer-substring-no-properties (region-beginning) (region-end)))
	  ))

	(defun run-this-in-eshell (cmd)
	  "Runs the command 'cmd' in eshell."
	  (with-current-buffer "*eshell*"
	    (eshell-kill-input)
	    (end-of-buffer)
	    (insert cmd)
	    (eshell-send-input)
	    ))

Examples:

Now, this is pretty bare bones, but this is not bad because we can use clever eshell redirections...

     { uptime && whoami } > /dev/kill

(the {s are a sh/bash thing that groups the output of the commands together)

/dev/kill sends it to the kill ring (aka: the clipboard)

    { echo && uptime && whoami } >>> #<buffer *scratch*>

This sends the output to the buffer named.... (well, I was using the scratch buffer)
	
## Windows support

Because it's lisp reimplementations of corebins, AND it doesn't do interactive TTY mode very well, it should actually work well on Windows!

### Calling Windows commands (aka Powershell)

You can call Powershell from eshell with this eshell alias:

    alias p pwsh --NonInteractive -Command "$*"

This will allow you to do the following in eshell

    p Get-Help Get-Date

or

    p Get-Date -Format \"dddd MM/dd/yyy\"

Note because it's super trivial and I have no idea what I'm doing, you need to pre-escpae those quote characters.

Interestingly enough: calls to `p` seem to be passed the `cwd`, so I can `cd` around all I want and something like `p Get-ChildItems` works just fine.

## See also

  * https://masteringemacs.org/article/complete-guide-mastering-eshell
  * https://www.reddit.com/r/emacs/comments/6y3q4k/yes_eshell_is_my_main_shell/


# ansi-term

Note: doesn't seem to play well with zsh, or at least my copy of zsh...

## you want to turn on line mode

which lets you edit the output

In Emacs GUI it's in the Terminal menu once you go into ansi-term or maybe term.

Then to run your editing lines do - just click the lines you want to edit, then hit return to run the entire line

### alternatives

https://www.emacswiki.org/emacs/essh may take care of that, being able to send arbitrary regions in the buffer to the shell.

## Windows support

term says no windows support (but eshell would work technically, although once you need to do a Powershekl command your may be messed)
