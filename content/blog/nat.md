---
title: "The NAT, its issues and some solutions in peer to peer context"
date: 2020-04-11T13:15:00+02:00
summary: "An overview of the NAT and NAT-traversal techniques"
---

If you were to setup a standard server (like a Minecraft server or a web server for example) locally in your home to be accessible outside your local network, you might have encountered some issues and some confusion when your friends couldn't connect and a guide asked you to change some settings on your router. This is due to some persons performing a dirty trick a few decades ago, a dirty trick that became so widespread that now, almost anyone that connect to the Internet today is using this trick.

This trick is called Network Address Translation, more commonly known as NAT. NAT solved the following issue: how do you make several devices to share the same internet connexion? More precisely, the issue NAT is solving is how do you hide several computers behind an unique IP address?

## How NAT works

First, some basic knowledge about Internet networks. On the Internet, every node (be a computer, a router, a server, etc) has an IP address (ranging from 0.0.0.0 to 255.255.255.255 in IPv4). To be able to differentiate the different streams going to a node, there is also a port number, which ranges from 0 to 65535. To contact the right service on the right computer, you have to provide the right IP address and the right port number. To communicate, a computer can send packets. Those packets contains information, but most importantly they contains the source IP and port and the destination IP and port.

Some address ranges are special: they are reserved for private networks. Those addresses are inaccessible from the Internet. On IPv4, the ranges are 10.x.x.x, 192.168.x.x, and 172.16.x.x to 172.31.x.x. Your current device's IP should be in those ranges. If you send a packet to your service provider with a destination IP on this range, the packet will be discarded instantly.

The NAT is very simple: it links a certain port of an interface to an IP and another port and automatically do the translation when any packet goes through it in any of the two directions. In the most common cases, the interface is a public IP address and the IP and the other port is pointing toward a user computer in a private network.

On most common setups, the NAT is running on the service provider box that you have been provided. It does the translation between your private IP and port and its public IP. This is why you have a public address (that you can get on some websites) that differs from the private address of your device.

Let's take an example. I (IP = 192.168.1.101) want to access my blog (IP = 185.199.109.153). I send a request to the HTTP server (1) (192.168.1.101:32875 -> 185.199.109.153:80, 32875 being a port number picked randomly by the web browser and 80 the standard HTTP port). The NAT (IP = 192.168.1.1) first receives the request in packets form. It picks a random non-used port (e.g 25405) on its public interface (IP = 55.41.20.79) and edit the packet so that the source IP of the packet is the IP of its public interface and the port (55.41.20.79:25405). The NAT then saves somewhere that it is supposed to send me back every packet that arrives on the port it picked (25405 -> 192.168.1.101:32875), since the server will use the source IP in the packet to answer me. The NAT then transmits to the server the edited packets (2). The server receives the request, answers me using the source IP in the packet (3). The NAT receives the answer packets, edit the destination IP of the packets to point towards my device IP and port (192.168.1.101:32875), and then send the edited packets to me (4).

<img src="/img/blog/nat.svg" alt="Schema explaining NAT translations" class="white-schema"/>

It works invisibly for me, the NAT does the work so that my device has nothing special to do. You can note that the server can also be hidden behind a NAT.

## NAT main issue

NAT is a wonderful fix to the rampant public IPv4 address shortage issue that internet providers are experiencing (it is still not enough to fix the shortage though). However, it does introduce some pain in the process, like that confusion when you want to setup a server accessible to the world. Why? Let's take an example.

Let's suppose that I have set up my Minecraft server on my computer. I tell my friend who is 600km away to connect to my public IP address (the IP of my internet router). I give him my public IP address and port. My friend attempts to connect to Minecraft and WHACK, Minecraft outputs "connection refused". When attempting to connect, my friend sent a connection request to the router. The router looks in its NAT table if there is anything linked to that port, but finds nothing. The router can't know to whom to send the TCP request. Is it to my computer hosting the server? To my cellphone connected via WiFi? The router simply can't figure it out by itself, so it just throws away the request.

To fix my problem, I have to tell my router the following through its settings interface (most commonly the web interface of the router): "If there is anything coming to that particular port, please redirect it to this IP address with this port". After this setting has been applied, my friend can connect to my server and we can start building a shoddy wood house happily together.

The main take-away issue of NAT is the following: **Without configuration, you cannot receive a connection request from the outside. You can however initiate a connection to the outside and the outside will be able to answer you back**

## NAT and Peer to Peer

This issue is not that important in a classic client-server architecture: the server owner does the setup, the client doesn't have to do anything particular, the NAT will adapt to every connection the client is making to the outside. This is why you don't have to change settings to your NAT when you are accessing emails or when you load a webpage even though the NAT is there between you and the server every time.

It is however an issue when you don't want to have a client-server architecture. Most peer to peer programs requires the user to be accessible from the Internet, which is not the case for most of us since we are all hidden behind a NAT.

This is why some applications, which could perform better in a peer to peer architecture, end up being implemented in a client-server architecture, with its benefits and drawbacks. The issues and the edge cases of setting up a direct connection between two standard users is simply too much hassle for developers.

This is a shame as Internet was designed to be a decentralised network, where everyone can be a peer to anyone. Today, in most of the networks where users are connected to Internet, the common expectation is : "Is my device able to access to a server on the internet?" and not "Am I able to host my own services?", which is natural for security reasons and given how unusual (yet important) it is to self-host.

### The solutions

It is possible to go through the NAT using NAT traversal techniques: the techniques to go through a NAT so the user can accept connections from the outside. I have searched about the main solutions and their applications in different opensource projects. I have looked into [Transmission](https://transmissionbt.com/) (a torrent client), [Syncthing](https://syncthing.net/), and [Magic Wormhole](https://github.com/warner/magic-wormhole) (which are my inspirations on this topic when I learned they were achieving peer to peer communications between clients)

#### Hole punching (STUN)

This solution relies on NAT implementations details. The idea is that NATs are not always checking the outside source IP from which packets are coming. So, for example, the user could send an UDP packet to a STUN server. The NAT preentively opens a port so the user can receive the answer. The STUN server can see from which IP and which port the packet has been emitted: the public IP and the port allocated by the NAT for the user. The server then sends back to the user those informations. The user knows on which port and which IP address he can be accessed by other users. It works well in UDP, where the connection state is not managed by the transport layer, but it is said that it can work in TCP too, though it is less guaranted, the NAT implementation can detect an abuse more easily.

This technique efficiency depends on the type of the NAT. There are 4 major types of NAT:

- **Full Cone NAT**: after the NAT opens a port, it will redirect every packet received on the port, no matter from which IP and port the packet is from.
- **Restricted Cone NAT**: the NAT checks the origin IP before redirecting the packet. If the client already sent a packet to that IP, the packet is accepted. Otherwise, the packet is discarded.
- **Port Restricted Cone NAT**: it is the same as the restricted cone, but taking into account both the IP and the port.
- **Symmetric NAT**: the NAT will open a port for every different destination IP and port.

The Full Cone is the easiest to use with STUN: the port will be opened to anyone. The Restricted and Port Restricted Cone are harder, since the client must be able to know that he has to send a packet to someone else. Finally, the Symmetric NAT is impossible to handle with STUN since the IP and the port retrieved by the STUN server is unusable by anyone but the STUN server itself.

This technique is standardized in [an RFC calling it STUN](https://tools.ietf.org/html/rfc5389).

This solution is used by Syncthing (using the library [go-stun](https://github.com/ccding/go-stun)).

#### UPnP and NAT-PMP

UPnP is a big specification for device interconnection. For example, it specifies how media center should be advertised to the network. There is one part that is widly used: the Internet Gateway device. This is implemented by most routers.

The Internet Gateway device has a lot of functionalities to offer, but the biggest one is the ability to link and unlink public ports to a particular port on a particular IP. This means that, with a simple HTTP POST request, it is possible to ask a router to open a public port for us and to link it to a particular computer. So, to take the Minecraft server example, the Minecraft server could ask himself to the router to open the port automatically when it is launched.

There are some legitimate concerns about the security of such functionalities. For example, some faulty implementations in old routers made it possible to access computers inside a private network because some routers were accepting UPnP request from the public interface. UPnP is also not supported by every routers (even though it is present in a lot of them), or can be deactivated by the user or by default.

This solution is used by Transmission (using the [miniupnpc library](https://miniupnp.tuxfamily.org/)), Magic Wormhole (it is said on the documentation, but I couldn't find any proof in the code), libp2p (using the go library [goupnp](https://github.com/huin/goupnp)), and Syncthing (included in [the following go code](https://github.com/syncthing/syncthing/tree/adc5bf6604dfc4e9532a78b3dc6e1ebf8a9b4edd/lib/upnp)).

NAT-PMP is a concurrent specification of the Internet Gateway device of UPnP. Syncthing also use this protocol (using the [go-nat-pmp library](https://github.com/AudriusButkevicius/go-nat-pmp)).

#### Relays

This is the most reliable solution. It consists to redirect all the trafic to a third party server that will redirect it to the peer. It simplifies the problem to a simple client-server problem, which is not affected by the NAT. It defeats the purpose of peer to peer though.

On Syntching and Magic Wormhole, it is used as a backup solution if it is impossible to establish a direct connection between two peers.

## Peer to peer and videogames

Since videogames are a big hobby for me, I also tried to find games that are making the players hosting the game server automatically without manual NAT port forwarding. Some games are indeed not asking the user to open ports to host a game server, and the user is really hosting the game (for example, if the user that created the lobby cause the current game to stop because he is leaving, it is obviously because it was hosting the game server).

Steam is a big thing, I have heard about a lot of games using or transitionning to some sort of Steam's API to achieve local hosted games accessible to the Internet without any configurations. I remember for example Garry's Mod requiring to setup port forwarding in the past to host a server, even though it does not requires anything today to setup a server. And lo and behold, I have found [something about peer to peer](https://partner.steamgames.com/doc/api/ISteamNetworking#SendP2PPacket) in the Steamworks SDK documentation. In this page, it does mention that the Networking module of Steam is using NAT-traversing techniques and relays fallbacks.

Because Steamworks SDK is closed source, I had to make some researches and experiments. So, I loaded up Garry's Mod (it's a Source game, Steam's game engine, so it should make use of a lot of Steamworks SDK functionalities) and tried to host a server. There is an option to turn on peer to peer, so it might be a toggle to use the Steamwork peer to peer API as opposed to the use of a more standard API for big game servers (peer to peer API is advertised for small groups of players). As the server launched, I didn't observed anything particular. But as the documentation states, the NAT-traversal code is executed on the first transmission of packet. So I asked one of my friend to join the server. Once he joined, I immediately checked for some UPnP entries. I also used Wireshark to find out if there are any STUN packets that have been sent. No luck, I wasn't able to find anything interesting, I guess Steam is good at hiding.

Stardew Valley is also an intriguing case. It is said that it provides cross-store multiplayer hosted by the user. I bought it on GOG and I could play with friends on Steam, which is an unusual but a very welcoming attention from the developer of the multiplayer port: Tom Coxon ([source](https://www.stardewvalley.net/stardew-valley-1-3-multiplayer-update-is-now-available/)). It works very well and with no additionnal setup as well. My experiments couldn't find how they managed to make it work. Like Steam, GOG does have an SDK, but [its documentation is hidden](https://devportal.gog.com/galaxy/components/sdk) if you don't have a GOG developer account. To achieve this cross-store multiplayer, I have doubts that any store SDK allows direct connection to a peer from another store user. Or maybe the GOG SDK have a functionality for this.

NAT is also an issue for console makers. It is tested [on the Playstation](https://manuals.playstation.net/document/en/ps3/current/settings/connecttest.html), [on the Xbox](https://support.xbox.com/en-US/xbox-one/networking/nat-error-solution) and [on Nintendo consoles](https://en-americas-support.nintendo.com/app/answers/detail/a_id/12472/~/compatibility-between-nat-types). As you can see in those links, they are classifying the NAT in three types: Open, Moderate and Strict. The classification is probably to designate the Full Cone, Restricted Cone and Symmetric NATs, so it could be a proof that the console makers are using some STUN-based techniques.

Overall, it seems that NAT traversal are used to reduce the cost on hosting very costly game servers by making the user hosting themselves (automatically using matchmaking or manually). You can observe this phenomenon if there are occurences of "host migration", the migration of the game on another computer because the original host disconnected.

## Identifying peers

As a side note, I have made some observations on how each solutions allows you to specify the peers you want to communicate with.

Syncthing is identifying each device by attributing a cryptographically guaranted unique ID. The link between the IDs and the IP addresses is made using Syncthing's Discovery server, a third party server hosted by Syncthing (you can host one yourself). To connect, both devices must have entered the other ID.

Magic Wormhole is identifying a file transfert using a passphrase, a few words mixed together. The sender must tell the receiver the passphrase so that the receiver can get its peer. The rendez-vous server, a third party server, is responsible to generate the passphrase and to link the receiver to the sender if the receiver gives the right passphrase. A server is hosted by the wormhole developper.

Steamworks is using the SteamID (the unique identifier of an user account).

Stardew Valley is using a code that the host get when it hosts a game. The other players must enter the right code to enter the game (or join using the Steam interface if both players are playing with the Steam version or the GOG interface if both players are playing with the GOG version).

## NAT and IPv6

As a reminder, IPv6 is the successor of IPv4. Its main feature is the 128 bits address, meaning that you can have way, way, way, way, way, way more addresses than IPv4 (which stores its addresses on 32 bits). Most of the internet providers are giving out a /64 address space for its clients, meaning that you have a whole 64 bits address space for you: it is still a power of 2 of the whole IPv4 address count. To give you an idea, an IPv6 address is written like this: 2001:0db8:0000:0000:0000:8a2e:0370:7334. That's a lot of characters, and keep in mind this is hexadecimal.

Because the ISP is giving you such a big address space, is NAT really necessary? For now and on the foreseenable future, not really. The NAT purpose was to share a public IP address with all the devices in the private network. But because all the devices in the private network can have a public address, NAT doesn't make any sense in IPv6.

Some says that a NAT could still be useful in the IPv6 context because it would prevent user tracking using the unique IPv6 address. This could be fixed with the client changing its address every so often.

Other says that a NAT could still be useful to protect from malicious attacks from the outside, because its mechanism is blocking unwanted trafic from the Internet. This could be fixed using a firewall, a software made for that exact purpose.

## Conclusion

While it seems a daunting task, it is possible to get through the NAT and allow users to easily host servers or be a peer in a peer to peer network. It should be noted that, given how common this issue is, there are no big open source solution that combines all the techniques seen above to try and open either a UDP socket or a TCP socket accessible by the Internet using NAT-traversal techniques. Every software that I have seen that do use some of the techniques I have presented are using low-level packages.

I am quite happy to have done this research about NAT traversal, as it gave me some insights on some of the magic I was experiencing while using Syncthing or hosting some videogames: none of it is magic, it is still computer stuff. I still keep around the miniupnpc test client as it allows me to open ports for some servers that don't use those techniques, like Minecraft or [Mindustry](https://anuke.itch.io/mindustry).