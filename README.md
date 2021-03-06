mailaprop: modern autofill behavior for email addresses in GNU Emacs.
=====================================================================

Overview
--------

Mailaprop provides modern, popup-style autofill of email addresses
when you're composing messages in Emacs:

<p align="center">
  <a href="https://player.vimeo.com/video/250746180">
    <img alt="Mailaprop Example Session (raw video file is here: mailaprop-example-session.webm)" src="mailaprop-example-session.png" width="75%" />
  </a>
</p>

As you start typing a name or an address, a popup window offers the
possible matches so far.  The matches are ordered based on how often
and how recently those addresses occur in all your email, with sending
to an address counting for more than receiving from it.

To the right of each address, mailaprop shows the date you last
interacted with that address, that is, the most recent date you sent
mail to or received mail from it.  Just for fun, the address's raw
mailaprop score is shown on the farthest right in parentheses.

Usually you only need to type a few letters for the top candidate to
be the one you want, at which point you just hit Return to choose it.
Otherwise, you can use the arrow keys to navigate in the list, or type
more letters to narrow down to fewer candidates.

This is basically the same autofill feature you get in your browser
when you interact with the sorts of online services that send
proprietary Javascript to your tabs.  Unlike those sites, however,
this package is entirely free software and operates on data that is
all stored locally.  You shouldn't need to hand over your social graph
to billionaires just to get decent autofill behavior.

Speaking of which, you don't have to use any proprietary Javascript to
interact with this project.  You can use plain git to clone the
repository from GitHub at https://github.com/kfogel/mailaprop.git, and
I'll happily take bug reports and patches by email instead of via the
GitHub issue tracker: kfogel {_AT_} red-bean.com.

How long does it take to set up?
--------------------------------

Less than an hour, if you are comfortable with basic scripting and
with making some simple changes to your ~/.emacs (or wherever you keep
your Emacs initialization code).

If you're new to this kind of thing but always wanted to try, then
mailaprop might take several hours to set up.  And if neither of these
paragraphs made sense to you, then it could take an arbitrary amount
of time, and you might want to step slowly away from the computer to
reconsider various choices in your life.

What about BBDB?  Doesn't it already do this?
---------------------------------------------

Does [BBDB](https://www.emacswiki.org/emacs/BbdbMode) offer
popup-style autofill these days?  It might.  It's been a long time
since I used BBDB.  Back when I last did, a decade or five ago, email
address completion wasn't working.  Maybe I failed to run some
initialization function, or mis-installed BBDB, or whatever.  Who
knows?  It's BBDB.

BBDB later went dormant, and then came alive again, and is now
actively maintained, so perhaps my libels are outdated.  See the
[development page](http://savannah.nongnu.org/projects/bbdb/) and
[commit logs](http://git.savannah.nongnu.org/cgit/bbdb.git/log/) for
more.

Anyway, it's too late.  By now I'm in too deep to get out.

(Mailaprop has also [been
compared](https://lists.gnu.org/archive/html/emacs-tangents/2018-01/msg00007.html)
to
[ecomplete.el](http://git.savannah.gnu.org/cgit/emacs.git/tree/lisp/ecomplete.el),
but I think ecomplete doesn't have a way to ingest legacy email
addresses nor a scoring algorithm to put the most likely autofill
candidates near the top.  Discussion is [under
way](https://lists.gnu.org/archive/html/emacs-tangents/2018-01/msg00009.html)
to determine whether these conjectures are accurate.)

Installation.
-------------

**Get Company Mode:**

Mailaprop relies on the excellent [Company
Mode](http://company-mode.github.io/) completion framework ("company"
stands for "complete anything").  Company Mode is quite mature and is
part of [GNU ELPA](http://elpa.gnu.org/), so you can install it easily
via `M-x list-packages`.

If you prefer to install Company Mode from [development
sources](https://github.com/company-mode/company-mode/), then you
would load it using some snippet like this:

    (let ((local-company (expand-file-name "~/src/company-mode")))
      (add-to-list 'load-path local-company)
      (require 'company))

**Create the address database:**

Edit `build-address-list.sh` in the obvious ways and run it.  It
invokes `mailaprop.py` to ingest mbox-format files (you specified the
mbox files when you edited `build-address-list.sh`), and produces a
file named `mailaprop-addresses.eld`.  Put that file anywhere you
want: it's the completion database that will be loaded into Emacs in
the next steps.

Note that `build-address-list.sh` may take a while to run.  I harvest
approximately 100,000 email addresses from approximately a million
individual mbox files, and it takes about twenty minutes on my laptop
purchased in 2010 or so.  Eventually you may want to arrange for
`build-address-list.sh` to run automatically at some regular interval,
e.g., nightly, so that it is constantly rebuilding your completion
database as you accumulate new email correspondents.

(In the future, mailaprop might be able to take inputs from a wider
variety of sources than just mbox files.  For example, it would be
nice if mailaprop could read address information from a
[notmuch](https://notmuchmail.org/) database, and there's some
[discussion](https://lists.gnu.org/archive/html/emacs-tangents/2018-01/threads.html#00004)
about using John Wiegley's
[gnus-harvest.el](https://github.com/jwiegley/dot-emacs/blob/master/lisp/gnus-harvest/gnus-harvest.el)
to provide inputs to mailaprop.)

**Load it into Emacs:**

Edit your .emacs or whatever your Emacs initialization file is:

    ;; Tell mailaprop where the address completion database is.
    ;; You will have to adjust this path, of course.
    (setq mailaprop-address-file
      (expand-file-name "~/private/mailaprop/mailaprop-addresses.eld"))

    ;; Load the mailaprop code (likewise adjust as needed).
    (load-file (expand-file-name "~/src/mailaprop/mailaprop.el"))

    ;; Optionally, set `mailaprop-drop-address-regexps' and/or define
    ;; a `mailaprop-drop-address-fn', to filter out some addresses
    ;; that you don't want offered up for autofill.
    ;;    
    ;; Using `mailaprop-drop-address-regexps' is simple.  It's just a
    ;; list of regular expressions: if an address matches one of the
    ;; regexps in the list, that address is not offered for autofill.
    ;; 
    ;; Each regexp can include the real name portion but is not
    ;; required to.  For example:
    ;;
    ;;   (setq mailaprop-drop-address-regexps
    ;;     (list
    ;;       (regexp-quote "some-undesired-address@example.com")
    ;;       (regexp-quote "another-undesired-address@example.com")
    ;;       (regexp-quote "My Nemesis <m.nemesis@example.com>")
    ;;       "^Uncle Fester .*" ; never want to email Uncle Fester
    ;;       ))
    ;; 
    ;; `mailaprop-drop-address-fn' is a more general mechanism for
    ;; skipping addresses: it allows you to run a user-defined
    ;; function on each address, to decide whether or not to exclude
    ;; that address from being offered for autofill.  See its doc
    ;; string in mailaprop.el for more details, and search for
    ;; `kf-mailaprop-drop-address-fn' in my own .emacs at
    ;; http://svn.red-bean.com/repos/kfogel/trunk/.emacs for an
    ;; example of how to write a `mailaprop-drop-address-fn'.
    ;; 
    ;; Note that a `mailaprop-drop-address-fn' should always be
    ;; defined with `&rest ignored' at end of the parameter list,
    ;; because future versions of mailaprop may pass more arguments
    ;; to it than the current version does:
    ;; 
    ;;  (defun my-drop-addr-fn (group-key group-size this-addr &rest ignored)
    ;;    ...)
    ;; 
    ;;  (setq mailaprop-drop-address-fn 'my-drop-addr-fn)
    ;; 
    ;; Both filtering mechanisms, `mailaprop-drop-address-regexps' and
    ;; `mailaprop-drop-address-fn', are completely optional; mailaprop
    ;; will work fine if you don't use them at all.

    ;; Load the autofill database.
    (when (featurep 'mailaprop)
      (mailaprop-load-addresses))

Usage.
------

Whenever you are in a message composition buffer and you start typing
in the `"To:"`, `"CC:"`, or `"BCC:"` header, you will get the
popup-style autofill behavior, with the likeliest addresses at the top
of the list.

Mailaprop autofill matches any substring within the address, not just
a string at the beginning of the address.  In other words, if you have

    "Abilene Szymanowski" <a.szymanowski@example.com>
    
in your completion set, and you type "b", her address will be among
those offered.

**Mailaprop speeds up as you use it more:**

Mailaprop will never delay your typing, but the first time you type a
single letter or a very short string, it may take a few seconds after
you pause for the autofill list to come up.  Don't
worry: the next time you type that same short string, the list will
come up much faster (note to programmers: mailaprop uses memoization).
Once you've been composing emails in an Emacs session for a while,
mailaprop will learn most of the common short strings and things will
get quite speedy.  Someday I might make it remember those strings
between sessions, but for now it only remembers within the same
session.

**Each address on its own line:**

For mailaprop to work, each email address in a header must be on its
own line.  The first address can be on the same line as the header
name, but no address can share a line with another address.  To
autofill on a next address in the same header, use C-j TAB (or
whatever key combination works for you) to navigate to the appropriate
place on the next line, and then start typing.

The reason for this is that finding boundaries between email addresses
on the same line turns out to be harder than I expected.  So I decided
to punt on it, and just decided that completion only works on an
address that is on its own line or on the same line as the header
name.  Thus, both of these addresses could have been autofilled:

        To: J. Random <jrandom@jrandom.com>,
            Victoria O'Hara <vickyh@foo.bar>

but below, the second one could not have been autofilled:

        To: J. Random <jrandom@jrandom.com>, Victoria O'Hara <vickyh@foo.bar>

This is only a minor bug, since the workaround is so easy and I tend
to want my addresses on their own lines anyway.  Still, I would
welcome a patch to fix this.

**Reloading the completion database:**

If you rebuild your autofill database while Emacs is running, you can
reload it by running `M-x mailaprop-reload-addresses` interactively.
The memoization cache will be rebuilt along with the raw addresses, so
you won't lose any speed.

**"Company" minor mode active in message composition buffers:**

Mailaprop relies on [Company Mode](http://company-mode.github.io/),
and modifies `message-mode-hook` and `mail-mode-hook` to put email
composition buffers into Company minor mode.

Ideally, mailaprop would restrict itself to the email recipient
headers, and not put the entire buffer into Company minor mode unless
the user had set that up independently of mailaprop.  However, I
haven't yet learned Company well enough to do that.

What this means in practice is that as you're typing in the body of
your message, if you pause after a keystroke you'll get Company's
dynamic prefix-based completion, starting from wherever you are in the
word you were typing.

When I realized that I kind of liked this behavior, and that Company
mode was proving useful and wasn't slowing me down (because it's
beautifully asynchronous), I decided not to search too hard for a way
to turn it off when not in the recipient headers.

However, others may feel differently about it.  I'd certainly welcome
a patch to fix mailaprop so that the choice to have autofill on email
addresses in headers is cleanly separated from the choice to have
dynamic word completion in the rest of the buffer.
