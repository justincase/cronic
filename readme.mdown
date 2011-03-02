See http://habilis.net/cronic/. Keeping a copy safe for personal use.

# Cronic

## A cure for Cron's chronic email problem

The Disease:

	0 1 * * * backup >/dev/null 2>&1

The Cure:

	0 1 * * * cronic backup

Cronic is a shell script to help control the most annoying feature of
cron: unwanted emailed output, or "cram" (cron spam). If the Unix Haters
list was still active, I would submit the rant below to gain membership.
- [Chuck Houpt](http://habilis.net/chuck/)

## The Disease

One of the best features of cron is its automatic email - it is also its 
worst feature. Cron automatically emails the output of a cron job to the
user. On the face of it, this sounds like a great idea. Cron jobs can run
automatically in the background for months at a time - so getting an email
when a problem occurs sounds useful.

Unfortunately, cron's idea of "output" is simultaneously too broad and too
narrow to actually be useful. Cron considers any output to be significant
- including standard output. This interacts badly with many unix commands,
which often send status info to standard out. Some commands have a quiet
options, but that can turn off all error output too. To make matters worse,
cron ignores command result codes, meaning that errors from quiet programs
are ignored.

It is almost impossible to create a non-trivial cron job that is quiet enough
to run without output, but still reports all errors. Following the principle
of "Worse is Better", the typical solution is to sweep it all under the carpet
by redirecting all output to /dev/null, and hoping for the best:

	0 1 * * * backup >/dev/null 2>&1

Now when your cron job fails, you will never know about it. Using cron to backup
your files? Sorry, the cron job has been failing due to permission errors for months
- all your files are gone.

Could cron be fixed? Although almost all current implementation of cron are open source,
cron's pathological behavior has been petrified into the Unix standards. So if it isn't
broken, it isn't cron. The only solution left is a work-around.

## The Cure: Cronic

Download: [cronic](http://habilis.net/cronic/cronic) v2

Cronic is a small shim shell script for wrapping cron jobs so that cron only sends email
when an error has occurred. Cronic defines an error as any non-trace error output or a
non-zero result code. Cronic filters Bash execution traces (or anything matching PS4) from
the error output, so jobs can be run with execution tracing to aid forensic debugging.
Cronic has no options, it simply executes its arguments.

	0 1 * * * cronic backup

With cronic, you can turn on Bash's strict error handling and debug options (exit on error,
unset variable detection and execution tracing) to make sure problems are caught early.
For example:

	#!/bin/bash

	set -o errexit -o nounset -o xtrace

	cp -rp data1 /backup
	cp -rp data2 /backup
	cp -rp data3 /backup

When an error is detected, Cronic outputs a report listing the result code, error output, and
combined trace and error output. The combined output can help put error messages in context.
An example:

	From: user@example.net (Cron Daemon)
	To: user@example.net
	Subject: Cron <user@server> cronic backup

	Cronic detected failure or error output for the command:
	backup

	RESULT CODE: 1

	ERROR OUTPUT:
	cp: data2: Permission denied

	STANDARD OUTPUT:

	TRACE-ERROR OUTPUT:
	+ cp -rp data1 /backup
	+ cp -rp data2 /backup
	cp: data2: Permission denied

### Version History

* v2 - Corrected command evaluation, so shell meta-chars are preserved correctly (Thanks to Frank Wallingford for the fix).
* v1 - Initial release.

### Releated Projects

Cronic isn't the only cure for cron. Cronic's main advantage it is small, simple and a shell script. There are several C-based
cron wrapper programs with many additional capabilities and options:

[Shush](http://web.taranis.org/shush/): a C-based cron wrapper, with multiple report formats, syslogging, etc.  
[Cronwrap](http://www.uow.edu.au/~sah/cronwrap.html): a C-based cron wrapper, with time-out control, logging, etc.