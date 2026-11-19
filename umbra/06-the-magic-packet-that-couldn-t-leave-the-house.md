# 6. The magic packet that couldn't leave the house

**Symptom.** Remote wake worked perfectly at home and failed silently the one
time it actually mattered — testing it from mobile data, away from the house,
produced nothing. No error, no packet, no wake.

**What I assumed.** A configuration problem. Wrong MAC address, adapter not
armed for wake, BIOS setting reverted — something checkable, something with a
setting to fix.

**Why that was wrong.** Every setting checked out. The adapter was armed. The
BIOS was correct. The packet was being sent, correctly formed, to the right
address. None of that mattered, because the premise was wrong: this was never
a configuration problem to begin with.

A Wake-on-LAN packet is a link-layer broadcast. It doesn't have a route past
the local network — there's no IP path for it to take, the way there is for a
normal packet, because it isn't addressed to anyone; it's shouted at everyone
on the wire. It can't cross the internet for the same reason a shout in one
room can't be heard in another building. And it can't ride the VPN mesh
either, for a subtler reason: the sleeping machine's own VPN client is asleep
too, so even if the packet somehow arrived at the tunnel, there'd be nothing
listening on the other end to forward it onto the physical network.

No amount of correct configuration fixes a packet that has nowhere to go.

**What I built.** Not a workaround on the same machine — a second, independent
one. An old phone, left permanently on the home network with its screen off,
running a script with exactly one job: accept an authenticated request over
the VPN mesh, and put a magic packet on the *local* wire it's actually sitting
on. It doesn't talk to Core, doesn't import anything from the rest of the
system, and doesn't need Umbra to be working — which is the one property
that actually matters, since it exists specifically for the moment Umbra
isn't.

The main phone now tries both routes on every wake attempt: the direct packet,
which is instant when you're home, and the relay, reached over the VPN, which
is the only one that can possibly succeed from anywhere else.

**What it showed.** Testing end to end — phone on mobile data, Wi-Fi off,
desktop actually asleep — the direct packet failed exactly as predicted, and
the relay carried the request the rest of the way. The desktop woke. A queued
command that had been sitting on the phone ran itself the moment the machine
came back.

**The lesson.** Not every failure is a setting away from working. Some
failures are structural — the thing you're asking for literally cannot happen
along the path you're asking it to travel, no matter how correctly everything
along that path is configured. The fix isn't to configure harder; it's to
notice the path doesn't reach, and add something that stands where it needs
to end.

---

*Project: Umbra. Copied 19 Nov 2026 from [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#6-the-magic-packet-that-couldnt-leave-the-house) at commit 5940844. The showcase copy is kept as well; if they differ, the showcase is newer.*
