# Data Buffering/ Buffers
In Computer science, a buffer is simply a place of memory to store data temporarily in. Usually in hardware or even software processes, buffers are often considered the "middleman" when transmitting data from a place to another.

This is due to the fact that the producer can be either slower/faster than the consumer, which means, if there wasn't a buffer, most of the produced data won't be consumed by the consumer or in other words, lost.

Think of a producer process that runs at 2hz *(two times a second)*, which computes x^2 and once a product is finished, leave it in one-slot memory, and then start the second computation. If our consumer wasn't able to receive that data before the second computation is finished, then it is simply lost *(assuming that the consumer runs at 1hz or slower, of course)*

While we may "solve" *(or more technically, avoid)* this problem by lowering down the producer speed to 1hz *(or any number, assuming it is equal to the consumer's process rate)*, although this is obviously terrible due to the fact that the possibility for a desync to occur is still very high.

Buffers generally solve this since they act as a place for data to be saved the same way they are finished by the producer *(or in other words, placed in a queue)*, which allows the following:

* Data isn't lost between production processes
* We generally don't need to overengineer a task scheduling system with proper synchronization
* We don't care if the producer is slower/faster than the consumer, therefore, we don't need to lower down the efficiency and the scalability of the system

Take this code for example:
```lua
local lastValue = 0

local gained = 0
local numOfProducts = 0
-- producer
task.spawn(function()
	local timeTaken = 0
	while true do
		
		if timeTaken >= 10 then
			break 
		else
			lastValue = math.random(1, 20)^2
			numOfProducts += 1
		end
		
		timeTaken += 0.5
		task.wait(0.5)
	end
end)

-- consumer
task.spawn(function()
	local timeTaken = 0
	
	while true do
		if timeTaken >= 10 then
			break
		else
			print(lastValue)
			gained += 1
		end
		
		timeTaken += 1
		task.wait(1)
	end
	
	print("total", numOfProducts, "consumed", gained, "lost", numOfProducts - gained)
end)
```

This code is just a simple emulation of the producer-and-consumer example I just talked about - you are free to understand how it works, but due to the scope of this tutorial, you only need to know the following:

* The consumer is reading data at 1hz while the producer is writing at 2hz
* the producer is writing at lastValue twice per second
* the consumer adds the `gained` value by one each time it reads a value

If you tried running the script, you would eventually see that our consumer actually misses some data due to the difference between the consumer/producer speeds.

To solve this, we would introduce a buffer in place, which is no more than a table that acts as a queue. Something like this would work:
```lua
local buffer = {}
local gained = 0
local numOfProducts = 0
-- producer
task.spawn(function()
	local timeTaken = 0
	while true do
		
		if timeTaken >= 10 then
			break 
		else
			table.insert(buffer, math.random(1, 20)^2)
			numOfProducts += 1
		end
		
		timeTaken += 0.5
		task.wait(0.5)
	end
end)

-- consumer
task.spawn(function()
	local lastIndex = 0
	while gained < numOfProducts do
		print(buffer[lastIndex + 1])
		gained += 1		
		lastIndex += 1
		task.wait(1)
	end
	
	print("total", numOfProducts, "consumed", gained, "lost", numOfProducts - gained)
end)
```
This code has a few differences from the first example, which are:

* The consumer has no time limit, it will stop the moment it finishes, showing how a consumer would work under normal circumstances. This wasn't implemented in the first example as the moment the producer stops, there is no point for consumer to continue, since the only available data is already read.

* The buffer doesn't technically get cleaned up, but rather (lastIndex + 1)'ed - this is just to avoid the complexity of table key removing etc although in a real word example, we remove it.

If you run this code, it would eventually print a text showing that the consumer consumed all the data, which what we wanted from the start!

However, you may ask, what will happen if let the producer run at 3hz instead of 2hz while still using the buffer? Well, assuming that our buffer has a memory limit/size *(buffers that have dynamic size are logically implemented on a higher level like software rather than hardware parts)* that is 20 elements, we would face a buffer ***over***flow, but what's that?

### Buffer Overflow
Buffer overflowing happens when the producer is overloading the buffer with elements to the point it is completely filled. This is usually due to the consumer being completely slow to the point where it can't keep up with the data that is being fed into the buffer.

To fix this, we could lower down the producer's speed, or increase the consumer's one, although depending on your projects, changes like this might not be ideal.

Another solution might be that we pause the producer's process and retry at a later point which gives us a bit time to compute other things.

Anyway, you may ask, what would happen if we let the consumer run at 4hz, while the producer runs at 3hx, what would happen? Well, it is buffer ***under***flowing, but what is it?

### Buffer Underflow
Buffer underflowing happens when the producer is too slow to the point it can't keep up with the consumer needs, or in our case, the producer is producing 3 products per second, while the consumer is consuming 4 per second which means that the consumer would always search for the 4th product in the buffer, causing buffer underflows.

We could tamper around with the producer speed to be able to match the consumer needs, or to set the consumer to sleep when the buffer is empty.
_____

## Multiple Buffering - What is that?
Multiple buffering can be used to further optimize your systems *(that are often related to hardware rendering)* - this often results in that the consumer would see a completed data *(that can be old)*, instead of a partial-completed data that is still being sent by the producer.

An example of a multiple-buffering, is ***double-buffering***, which is basically two buffers. It works by processing a filled up buffer while the other one is still being filled, and repeat the process.

This can be translated to a real world example - For instance, you might fill up a bucket with water, and once it's filled up, you swap the filled bucket with a clear one which you leave it as you use the filled up one.

Another example of multiple buffering, is ***triple-buffering*** - Which means that we can fill up the buffer that is not involved in any copying which can result in improved performance since we are waiting for a buffer to be used so we can process more data.

# Conclusion
While the majority of my explanation was oriented toward hardware more than software, the definitions still stand up.

From now, I am fairly certain that you would able to further educate yourself on buffering and its uses in the industry although don't make this expectation something that prevents you from asking questions!