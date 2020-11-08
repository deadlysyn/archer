# Thought Process

My goal was consistency (without needing to remember what utilities to
track down each time) and a "minimal but not too minimal" install.
I also wanted to be able to run the same install process on an old
Thinkpad or a modern desktop and get reasonably adjusted defaults,
whether Intel- or AMD-based.

I wasn't aiming for "the most minimal possible", since the ultimate
goal was consistent usability. Some choices could undoubtedly be replaced
with slimmer alternatives or eliminated entirely. For example, the base install
 includes things like an image viewer and PDF reader. I found
having these available helpful in normal life. To minimize impact,
I chose the leanest alternatives I could find with reasonable functionality
([sxiv](https://github.com/muennich/sxiv) and
[zathura](https://pwmt.org/projects/zathura) in these specific cases).
You might have different chocies, and these can be swapped out easily.

For larger things I only wanted some places (desktop vs laptop), I split
out separate manifests so they can be selectively included. Again you
are free to have different choices, and even I find myself moving more
of "base" into "bloat" over time. Examples in this category are
[LibreOffice](https://www.libreoffice.org), [gimp](https://www.gimp.org),
and even [picom](https://wiki.archlinux.org/index.php/Picom). While I
typically pull these in, they are often hundreds of megabytes with
a plethora of dependencies so I wanted it to be easy to exclude them.

## Choices

For any who care, these are some choices I've made and reasoning behind them.
These are strong opinions weakly held, so I reserve the right to change my mind.

Wired is nice and fast, but modern wifi is fast enough for me and useful
while roaming. I've tried to ensure wifi works on first boot. To adhere to
a minimalist approach, I went with [iwd](https://wiki.archlinux.org/index.php/Iwd)
(vs networkmanager, wpa_supplicant and dhclient). I thought that would
be a hard choice, but it is well documented and capable. It integrates
nicely with
[systemd-networkd](https://wiki.archlinux.org/index.php/Systemd-networkd) and
[systemd-resolved](https://wiki.archlinux.org/index.php/Systemd-resolved).
This also meant dropping [nm-applet](https://aur.archlinux.org/packages/network-manager-applet-git)
which served me well for years but is a bit bloated. `iwctl` can
be used to manage wireless connections. I'm working on a dmenu-based
network picker for more WM-integrated UX, without the need to
run a system tray or pull in another dependency (dmenu is also used as
a run menu and general prompter).

Speaking of [systemd](https://wiki.archlinux.org/index.php/Systemd), I
was never a fan. I didn't object because it wasn't a solid init system,
or because SysV init was good enough (I grew up on SysV init, but everything
needs to evolve over time). My objection was softer, in that systemd's
"everything and the kitchen sink" approach clearly violates
[the UNIX philosophy](https://en.wikipedia.org/wiki/Unix_philosophy).

For those who argue "you aren't forced to use anything" -- true, but
if you want to avoid systemd entirely it limits your distro choices
(I wanted [Arch](https://archlinux.org) vs
[Artix](https://artixlinux.org) or [Void](https://voidlinux.org)),
requires extra work to disable, and includes bloat you don't need
if swapping components. As a result, since Arch uses systemd, I decided
to go all in and drink the Koolaid. This includes using
[systemd-boot](https://wiki.archlinux.org/index.php/Systemd-boot)
rather than GRUB, systemd-resolved vs resolveconf, and similar choices.
Each of the alternatives are valid, but require pulling in additional
dependencies and effectively working around vs with Arch's choices as a distro.
In the end this path was smooth thanks to Arch's documentation, but
you may need to adjust based on your requirements.

In terms of "desktop environments", whatever that means, I've again tried to
eliminate bloat. To me (we all have opinions), these "environments" represent
a systemd-class problem (lots of choices made for me which become irritating after
prolonged use but are so entangled it's difficult to use only the parts you need).
This repo enables a slim window manager ([bspwm](https://github.com/baskerville/bspwm))
vs full blown desktop environment. Lightweight tools including
[sxhkd](https://github.com/baskerville/sxhkd) and [polybar](https://github.com/polybar/polybar)
are pulled in to round out the experience. Ultimately, this implies
a preference for a tiling workflow with VIM-like navitation. If you follow
this end to end, `Super-F1` provides a list of default keybindings.

While no organization is perfect, philosophically I align more with Firefox
than other popular browsers. I've tried a handful of minimal browsers
including [Midori](https://astian.org/en/midori-browser),
[qutebrowser](https://www.qutebrowser.org),
and [surf](https://surf.suckless.org)
but been unimpressed with performance and features. If you haven't,
give these a spin and decide for yourself.

I've come to the conclusion the modern web is bloat, and requires a bloated
browser to be of much use. While Firefox fulfills that need for me, my
day job also requires specific plugins which only work with Chromium based
browsers. You certainly don't want to run Chrome and provide deep
analytics to a profiteering giant, but there are alternatives including
[ungoogled-chromium](https://aur.archlinux.org/packages/ungoogled-chromium)
and [Brave](https://aur.archlinux.org/packages/brave). I initially balked at Brave
due to monetization aspects, but you can disable everything unsavory
(for now) and the way it natively supports HTTPS redirects and ad blocking
eliminate the need for some plugins.

In the end I decided to
[benchmark a handful of popular browsers](https://blog.devopsdreams.io/browser-comparison)
on my hardware and let the numbers decide. I encourage you to do
the same. My number-driven choices are embedded in default configuration
when using this install process, but can be easily swapped out.
Regardless of your choice, change your default search to something like
[Startpage](https://startpage.com),
[Qwant](https://qwant.com), or
[Searx](https://searx.me).

Over the years I noticed myself using more and more Electron apps. Spotify,
Slack, VS Code (yes I went there)... I didn't think much about it,
mostly accepting the bloat. As part of this project I decided to rethink
my choices. While not perfect, and accompanied by a learning curve, I've
been moving to CLI-based replacements. This includes
[spotify-tui](https://github.com/Rigellute/spotify-tui),
[wee-slack](https://github.com/wee-slack/wee-slack), going back to VIM
(or [vscodium](https://aur.archlinux.org/packages/vscodium-bin) if you must)
and even swapping things like
[pavucontrol](https://www.archlinux.org/packages/extra/x86_64/pavucontrol)
for [pulsemixer](https://github.com/GeorgeFilipkin/pulsemixer).
After initial adjustment you end up with the same or more functionality
that works equally well on hardware of any age.

