# 6. The magic packet that couldn't leave the house

**In one line:** remote wake of my desktop worked at home and failed silently away from home, because the wake signal cannot travel past the home network, so no setting could fix it.

| | |
|---|---|
| **Where** | Waking the desktop from the phone |
| **Result** | Tested from mobile data with the desktop asleep: the direct packet failed as predicted, the relay woke the desktop |
| **Topic** | Debugging, Design decision |

## The problem

Remote wake worked at home. The 1 time it mattered, testing from mobile data away from the house, nothing happened. There was no error, no packet and no wake.

## What I assumed

A configuration problem: wrong MAC address (the network card's hardware address), the network adapter not set to wake the machine, or a BIOS setting (the computer's start-up settings) that had reverted. These are all things I could check and fix.

## What the evidence said

- Every setting checked out.
- The adapter was armed, the BIOS was correct, and the packet was sent, correctly formed, to the right address.
- None of that mattered, because this was never a configuration problem.

## The cause

A Wake-on-LAN packet (the "magic packet" that wakes a sleeping computer) is a link-layer broadcast. It is not addressed to anyone. It is shouted at everyone on the local wire, so it has no route past the local network. It cannot cross the internet, for the same reason a shout in one room cannot be heard in another building.

It also cannot travel over the VPN mesh (the private network linking my devices). The sleeping machine's own VPN client is asleep too. Even if the packet reached the tunnel, nothing would be listening on the other end to forward it onto the physical network.

## The fix

I built a second, independent route. An old phone is left permanently on the home network with its screen off, running a script with 1 job: accept an authenticated request over the VPN mesh and put a magic packet on the local wire it is sitting on. It does not talk to Core (the main server of the assistant). It does not import anything from the rest of the system and does not need Umbra to be working. That matters because it exists for the moment Umbra is not running.

The main phone now tries both routes on every wake attempt: the direct packet, which is instant at home, and the relay over the VPN, which is the only route that can work from anywhere else.

## How I measured it

I tested end to end: phone on mobile data, Wi-Fi off, desktop asleep. The direct packet failed exactly as predicted. The relay carried the request the rest of the way and the desktop woke. A queued command that had been waiting on the phone ran itself as soon as the machine came back.

## What I took from it

Some failures are structural. The request cannot happen along the path it is travelling, however well that path is configured. The fix is to notice the path does not reach, and add something that stands where it needs to end.

---

*Project: Umbra. Rewritten for easier reading from the original in [umbra-sub-os-showcase / CASE-STUDIES.md](https://github.com/Ciaran11221/umbra-sub-os-showcase/blob/main/CASE-STUDIES.md#6-the-magic-packet-that-couldnt-leave-the-house) (commit 5940844). Same facts and numbers.*
