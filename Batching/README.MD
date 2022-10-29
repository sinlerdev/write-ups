# Batching
"Batching" is simply put, a technique of processing a group/batch of transactions/processes that were collected in a timeframe for extra performance in exchange to immediate reactivity.

Batching is a technique that can be found everywhere, whether it is networking, rendering, or even with handling emails!

 For example, in networking, you would collect all network calls in a timeframe and then *batch* process them at a specific timestep - which if you were to implement batching in roblox networking, you would collect all remote calls that occurred since `Stepped` event up until the upcoming `Heartbeat` event, which is also the point where you would send them. While with email handling, you would batch process every email that was queue'd every 1 minute or so.

Just for the sake of simplicity, we will stick to roblox networking tools which are remotes.

[details ="Now, take this pseudocode for example:"]
```
remote = get remote
signal = get signal

on signal_firing do provide(data)
    fire remote with data
end

for 10 loops do
   fire signal with current_loop_number^2
end
```
[/details]

This is ***extremely*** inefficient, and while optimizing this is pretty much a common "optimization tip" on roblox, it is important to know the reason which to cut the long story short, remotes have 9+ bytes overheard which we are firing the remote 10 times, therefore we have 90+ bytes overheard, which adds up to 180 bytes if we calculated the space used by the 10 numbers *(each number is 9 bytes, so 9 * 10 + 90 is 180)*  . 

A fix for this issue is already clear which is to cache the 10 numbers into a table and once we are finished computing them, we send them over the wire. Batching in this example can be unclear, although it is present since we aren't sending something over the wire *once* it's ready, instead, we send a queue of the product.

```
"IMPORTANT: Just to be clear, don't explicitly implement this issue to gain
"immediate" networking, since roblox already does batching & reliability for 
remotes."
```
______

# Examples of Batching
This section, I will cover up a few examples that use batching as an optimization/mechanism to fix other issues.

## Roblox Networking
Roblox already does implement batching in their engine-integrated networking as a whole, not just remotes! Simply speaking, there are two points in every frames that packets are processed - the first one is at every start of a frame. which is the point where we process the packets that were received since the last frame, and the other one is at every end of the frame, which then we send every packet we queue'd!

This type of system gives us a few gains that are not limited to:

* Performance - In networking, sending smaller packets are much more resource-wasteful than sending a huge packet and because of batching, roblox is able to pack every remote call you use into a huge packet.
  
* Synchronization - This ensures that if a packet was successfully received *(it is guaranteed that we will receive it since roblox implements reliability for remotes)* , it would be processed and ready for our code to interact with. 

## Deferring Property Changes
Specifically on roblox, updating instances' properties can get very expensive, as such, deferring changes by our code is often good in terms of performance. For example, any properties that affect the physical properties of an instance can be collected into a queue table that will be cleaned up every `Stepped` event, while appearance properties can be collected into a queue table that will either be processed in a `Stepped` event if it's the server or a `RenderStepped` event if it is a client.

This makes sense, since the end-user doesn't care as long as the deferred changes are appllied ***before*** they are needed.

This has a few gains that are not limited to:

* Performance - since the end-user doesn't care about any changes that are going to be overridden before the point that needs the updated state, we can only apply the latest property.

* Flexibility - you can literally do whatever you want with the deferred change, whether you want to completely cancel it or override it.
___________

# Conclusion 
Batching is seriously a very cool technique to save system resources when you don't really need immediate reactivity. 

In addition to built-in roblox internals, there are a few open-source projects that use batching as an optimization.

1. [Fusion v0.1 by elttob](https://github.com/Elttob/Fusion)
2. [BridgeNet by ffrostfall](https://devforum.roblox.com/t/bridgenet-insanely-optimized-easy-to-use-networking-library-full-of-utilities-now-with-roblox-ts-v199-beta/1909935)


Anyways, I have rewritten this 3+ times, each time increasing the quality of the topic, so, if you are wondering about something, don't be afraid to ask!