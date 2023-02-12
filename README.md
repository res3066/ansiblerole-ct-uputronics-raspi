ct-uputronics-raspi
===============

An Ansible role to configure a Raspberry Pi 3b or 4b to be a
Stratum 1 NTP server with a reference clock of the Uputronics
Raspberry Pi GPS/RTC Expansion Board (note hardware RTC; with the
ultracapacitors - we support that too).

https://v3.airspy.us/product/upu-rpi-gps-rtc/

This board contains what is described as a "72 channel u-blox m8
engine".  The datasheet for what we believe is the right chip.

There is some support for a local lash-up that includes a breakout
board that we picked up in 2020 (also ublox).
https://v3.airspy.us/product/upu-ublox-m8q3a-sma/
Trust us, you want to go with the Pi-specific GPS/RTC expansion board
mentioned above.

DON'T BE A HERO AND FIGURE YOU'RE GONNA DO THIS WITHOUT READING THE FULL
README.md AND PROBABLY THE DATASHEET TOO.  FUTURE YOU THANKS YOU.

Hat tip to Chris Jefferies - `chris@tinajalabs.com` who wrote
a tidy role for configuring the Adafruit Ultimate GPS (which for us
was more like the "penultimate GPS" since we quickly discovered
that we liked the u-blox GPS engines a whole lot better than
the GPS engine of indeterminate origin in the Adafruit offering).
Chris' role was a heavy influence on the design of this role.
https://github.com/TinajaLabs/ansible-role-ntp-gps

Requirements
------------

You need Raspberry Pi OS (nee Raspbian) Buster for this to work.  At
this writing, Bullseye is more like Bull-something-else, as a result
of heavy rewrites that even now, a year on, are not ready for prime time.
Check back here in the future; if there is some kind of rumor afoot that
the gpio subsystem and its dependencies are no longer a flying circus
we may upgrade to support Bullseye.  Pull requests welcome!

We have bad luck in our local environment with paramiko and interactive
password prompting in Ansible, so there is an assumption here that
you have already put your ssh public keys on the Pi in question via `ssh-copy-id`
or similar means of your own devising.


Role Variables
-------------

See defaults/main.yml and override to taste in group_vars/all.yml
in your {{ inventory_dir }} directory (where your playbook probably
lives).

Dependencies
------------

None.


Example Playbook
----------------

```
    - hosts: ntpservers
      roles:
         - ct-uputronics-raspi
```

Notes
-----

This role is intended to be a one-and-done approach to convert a fresh
Raspberry Pi OS image into a stratum 1 NTP server.  Although it could
hypothetically be used for ongoing maintenance there has not been a lot
of testing surrounding that and it's not going to restart services for
you unless you uncomment the notifications (see below).

You will absolutely need to reboot after running this role the first (and only?) time;
therefore all of the notifications to handlers are commented out. While modload
and the like could hypothetically be done on the fly, things like
changing serial console into something else entirely and creating i2c devices
without a full reboot are fraught and optimizing to save 90 seconds is really
not where we want to spend our time.

You may or may not need to run configure-rv3028.sh after installation to
goose the RTC into the right mode.  See page 6 of the datasheet in meta/
and run dmesg to make a determination.  The effects of running the
configuration script many times have not been explored, so we copy it into
place and leave it to the discretion of the user whether it needs to be
run once.  `dmesg | grep rtc` after reboot will inform your choices here.
 
Future Directions
-----------------

gpsd is pretty opinionated about what it ought and ought not do to the
gps chip during startup.  A command line flag to leave well enough alone
because things have already been configured out of band would be a welcome
addition (pr, upstream).

Because of this we have not managed to get the ublox into stationary
timing mode, disable galileo support, or anything else that we might
want to do.

The GPS has an i2c interface, as does the RTC.  See the RTC fixup script
for examples of how to do i2c from a shell script.  It may be possible
to goose things into the right state by poking it over i2c either before
or after gpsd startup.

Then again, https://github.com/philrandal/gpsctl may be the right direction,
particularly if we can get the "leave well enough alone" mod into gpsd.

gpsmon does not have a --json flag by which one could get a snapshot of
the current situation and feed it into one's monitoring system.

ntpq does not have a --json flag, ditto.  please support the reference
implementation.  http://www.ntp.org


References (mostly cribbed from Chris, thanks!)
-----------
* How Ansible Works - https://www.ansible.com/overview/how-ansible-works
* Get Started with Ansible - https://docs.ansible.com/ansible/latest/network/getting_started/first_playbook.html
* NTP - https://en.wikipedia.org/wiki/Network_Time_Protocol
* Raspberry Pi GPS Time - http://unixwiz.net/techtips/raspberry-pi3-gps-time.html
* Raspberry Pi NTP - https://www.satsignal.eu/ntp/Raspberry-Pi-NTP.html
* Stratum-1 NTP Server using Raspberry Pi - https://developers.redhat.com/blog/2017/02/22/how-to-build-a-stratum-1-ntp-server-using-a-raspberry-pi/
* Uputronics Raspberry Pi GPS RTC Board Datasheet.pdf - meta/Uputronics%20Raspberry%20Pi%20GPS%20RTC%20Board%20Datasheet.pdf'
* Ublox Neo-M8 Datasheet - meta/NEO-M8-FW3_DataSheet_UBX-15031086.pdf


License
-------
MIT

Author Information
------------------
Rob Seastrom - `rs@cluetrust.com`

