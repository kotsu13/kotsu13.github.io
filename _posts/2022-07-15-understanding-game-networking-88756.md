---
layout: post
title:  "FFXIV: Trying to understand Game Networking"
description: "Understanding FFXIV network packet"
date: 2022-07-15
img: # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [networking, re]
---

One of the things that is sometimes used by the FF14 end-game raiding community is a software called Advanced Combat Tracker (ACT). It helps to track DPS done by a party and can be used by players to try to improve their DPS output. As my group of friends and I often raid on patch days where the meter would not be up, I started to wonder how ACT itself works, and whether there was a way that I could somehow patch the software myself. Although I did not do the latter in the end, I still learned some pretty interesting things along the way.

First, I downloaded a copy of the <a href="https://github.com/ravahn/FFXIV_ACT_Plugin">FFXIV ACT Plugin</a>, which I quickly realised that the source code was not open source, but it's possible to view job definition files from the repo itself. This was worth looking into as jobs and their skills are related to DPS output.

<a href="https://github.com/ravahn/FFXIV_ACT_Plugin/blob/11b4d0af84bf8fa547a8ea1b2fe732ffc8e89485/Definitions/Summoner.json#L5">L5 in Summoner.json</a> shows that the key id for a Summoner skill called Fester is 0xB5 (or 181 in base 10). I was initially trying to figure out how this value was derived. After some exploring around, I came across a tool called <a href="https://github.com/xivapi/SaintCoinach">SaintCoinach</a> that allows viewing of the game assets in an easily viewable UI called Godbert. I will not go into too much detail about how Godbert works as it's not the focus of this post. After opening my game files in Godbert, I managed to cross check that the skill action is saved under key id 181. I found this to match for a few other skills and concluded that these job definitions could be matched to game assets that are already downloaded to your PC.

![Fester's key id as shown in Godbert UI]({{site.baseurl}}/assets/img/godbert01.jpg)

Next, I wanted to figure out how these definitions were used. I decided to throw the plugin dll into IDA Pro and found out that it was a C# binary. C# binaries are usually easy to reverse engineer as the code compiles down to CIL, which contains quite a fair bit of high-level metadata. Then, I used ILSpy to figure out how the code worked.

One of the first things I noticed in the main file was this:
```c#
Version version3 = typeof(TCPNetworkMonitor).Assembly.GetName().Version;
ulong num1 = BitConverter.ToUInt64(typeof(TCPNetworkMonitor).Assembly.GetName().GetPublicKeyToken(), 0);
if (version3 != version || num1 != num)
{
    str = string.Concat(str, string.Format("FFXIV_ACT_Plugin detected version {0} of Machina library, expected {1}.  Restart ACT, and make sure FFXIV_ACT_Plugin is the first plugin listed on the plugins tab.{2}", version3, version, Environment.NewLine));
}
```
It seems that the networking portion is handled by the Machina library. 

The other thing I noticed also was that the parts of the code I was interested in was compressed as resources and hence would not display correctly on the decompiler. The reason for this was the presence of a library called Fody Costura, which is used to allow developers using .NET 4.X or older to to ship their binary as a single dll instead of multiple dlls (especially useful when a lot of libraries are used). This causes dependencies to become resources in the binary, and will be loaded dynamically at runtime. This is possible because C# supports reflection.

![Lots of classes listed as compressed resources, such as costura.ffxiv_act_plugin.network.dll.compressed which I was interested in.]({{site.baseurl}}/assets/img/embeddedresources01.jpg)

Thankfully, as quite a number of other developers build tools on top of this plugin, an SDK is also available in the repository and it contained all the DLLs I needed, so I did not need to read the Fody Costura code to figure out the compression. Then, I looked into Network.dll's `NetworkMessageReceived`, then `ProcessPacket`. The decompiled code is as follows:

```c#
using System;
using System.Runtime.CompilerServices;
using FFXIV_ACT_Plugin.Network.PacketHandlers;
using Machina.FFXIV.Headers;
using Machina.Infrastructure;

public unsafe void ProcessPacket(long epoch, byte[] message)
{
	if (message.Length < System.Runtime.CompilerServices.Unsafe.SizeOf<Server_MessageHeader>())
	{
		return;
	}
	DateTime packetDate = ConversionUtility.EpochToDateTime(epoch).ToLocalTime();
	ushort messageType;
	fixed (byte* ptr = message)
	{
		messageType = ((Server_MessageHeader)ptr).MessageType;
	}
	if (_packetHandlers.ContainsKey(messageType))
	{
		IPacketHandler packetHandler = _packetHandlers[messageType];
		if (message.Length < packetHandler.PacketLength)
		{
			throw new PacketLengthException($"ProcessPacket: message length {message.Length} is less than {packetHandler.PacketLength} for handler {packetHandler.GetType().Name}");
		}
		_benchmarkRepository.Measure("Network" + packetHandler.GetType().Name, delegate
		{
			packetHandler.Process(message, packetDate);
		});
	}
}
```

Based on the message type, the rest of the packet is processed by the relevant handlers. For ability handling, there's a common method called `ProcessAbility`. I then decided that it was time to move on to look at Machina itself.

<a href="https://github.com/ravahn/machina">Machina</a> is a pretty nice framework for reading network data that abstracts away the nitty gritties of working with Windows sockets. Not only can you constraint the reading to a specific process, it handles the TCP/IP data layer for your packets, and so you can completely focus on the application data.

While looking at the FFXIV related code, I noticed that there's 2 headers for an FFXIV Game packet: The BundleHeader and the MessageHeader. The bundle header can be found in <a href="https://github.com/ravahn/machina/blob/3e0956ed1ee6f7b428145b121a0287b7f9cbadf4/Machina.FFXIV/Headers/Server_BundleHeader.cs#L29">Server_BundleHeader.cs</a>. Using information in the bundle header, one can decompress the rest of the packet and decode it according to the MessageHeader. The Message Type in the Message Header is determined by the opcode.

According to the commits, it seems that opcodes change every patch and can be found in the file <a href="https://github.com/ravahn/machina/blob/3e0956ed1ee6f7b428145b121a0287b7f9cbadf4/Machina.FFXIV/Headers/Opcodes/Global.txt">Global.txt</a>. In contrast, ability keys only seem to change if a new skill is added or any old skill is rendered obsolete.

With known values like the offsets, size, and key id of skills, my guess is that comparing and figuring out the new opcodes every patch for each opcode type should be rather doable. To conclude my search, it turns out that these tools need to update every patch to account for opcode changes.

## Digressions

One of the things I wondered was how people figured out the fields of the Bundle Header. The magic is easy enough to figure out as it is ARR in little endian, which is the abbreviation of "A Realm Reborn", the name of the base game. Epoch takes a bit of effort to see the pattern, and every other field was rather indecipherable for me. My guess was that the game binary most likely should have knowledge of this packet struct somewhere, so that it can send packets out to the server. So, I tried loading the game binary into Ghidra.  String references like `ARR` are usually the first things I attempt to find, but it wasn't available. My guess is probably optimization at assembly level, as sometimes short words get loaded directly into registers/memory, as the compiler may have decided that a few mov instructions are more efficient than loading from a .rodata section. The other option was to look at the library network functions used by this binary since the symbols would be public if it's dynamically linked and attempt to trace back. I didn't make much progress either unfortunately.

Another thing I did during my debugging was attempt to log my gameplay via Wireshark.

One of the most mind-blowing things I realised from analysing my network data from FF14 on Wireshark was that other than the initial login, which happens via TLS/SSL, the rest of the packets are pretty much sent unencrypted. If one has knowledge of the game packet structure and compression method used, it is not too hard to figure out what is in the packet, as a lot of data-mined information about the game exists. 

Screenshot following the login process:  
![Following the login packet stream on Wireshark shows the key exchange, then the start of TLS session]({{site.baseurl}}/assets/img/ws00.jpg)


Screenshot of 1 packet during gameplay:  
![1 packet as shown by Wireshark during a gameplay]({{site.baseurl}}/assets/img/ws01.jpg)

## Takeaways

Figuring out network packet structures is still something that I need to work on. If one had to figure out an online game packet from scratch, what would the thought process be? How would they approach the problem? My guess would be experience can come from writing game server-client code itself, and also maybe reading/knowing more about what other games do. Or maybe games use similar frameworks, and in that case, knowing how the framework works allows one to experiment and try to figure out the rest.