# Data Buffering/Buffers
Buffering, or in a noun form, a buffer, is a container/place of memory for temporarily storing data that is being moved from a place to another *(Place in this context is either a device or a process)*.

A buffer might be used for storing data when the rate of receiving data is different from the rate that it is processed. For example, every time you finish processing a computation, you might put it into a buffer that then will be read from until a later point.

In other words, a buffer would be used in contexts where you produce data too quickly for the consumer to retrieve efficiently or vice versa. 

You might have noticed that a buffer shares some similarities from a queue, and it is actually because buffers implement the queue data structure as a way of handling its stored memory! This is to guarantee that the data we fed into a buffer is used in the same order it was sent to!

Take this code for example:
```lua
local last_produced_data

local left = 0
local consumed = {}
local num_of_produced_data = 0
task.spawn(function() 
	while true do
		task.wait(.5)
		if not consumed[last_produced_data] then
			left += 1
		end
		
		if num_of_produced_data == 10 then
			break
		end
		last_produced_data = math.random(1, 200)
		num_of_produced_data += 1
		print("producer",last_produced_data)
	end
	
	print("total", num_of_produced_data, "left", left, "consumed", num_of_produced_data - left)
end)

task.spawn(function()
	while true do
		task.wait(1)
		if num_of_produced_data == 10 then
			break
		end
		print("consumer",last_produced_data)
		consumed[last_produced_data] = true
	end
end)

```

In this example, the producer is 2x times faster than the consumer, which means that in theory, half of the data or even more that will be ever produced will not be received by the consumer.

 A buffer would solve this, which we can see the observation by doing:

```lua
local buff = {}
local consumed = {}
task.spawn(function()
	local num_of_testing = 0

	while true do
		task.wait(.5)
		
		if num_of_testing > 10 then
			break
		end
		num_of_testing += 0.5
		table.insert(buff, 123 * math.random(1, 123))
	end
	
end)

task.spawn(function()
	local num_of_testing = 0

	local last_index = 0
	while true do
		task.wait(1)
		last_index += 1
		if num_of_testing > 20 then
			break
		end
		local produced = buff[last_index]
		table.insert(consumed, produced)
		num_of_testing += 1
	end
	
	print("total", #buff, "left", #buff - #consumed, "consumed", #consumed)
end)
```

This may look scary; however, you don't really need to know what's going on as you only need to be aware of the following:

1. The producer has a cap of 10 seconds, while the consumer has 20 - this is just to give a fair ratio since the producer produces 2 results per second, while the consumer consumes 1 result per second. Although in the real word, both the consumer and the producer don't have a time cap in normal circumstances.

2. Data is not lost and saved in order

However, what will happen if we let the consumer read three results per a second, while having a time cap of 10 seconds?

If we tried replacing the consumer code with this:
```lua
task.spawn(function()
	local num_of_testing = 0

	local last_index = 0
	while true do
		task.wait(1/3)
		last_index += 1
		if num_of_testing > 10 then
			break
		end
		local produced = buff[last_index]
		
		if produced == nil then
			error("Buffer underrun!")
		end
		
		table.insert(consumed, produced)
		num_of_testing += 1/3
	end
	
	print("total", #buff, "left", #buff - #consumed, "consumed", #consumed)
end)
```

We would eventually see an error saying `Buffer underflow`, but what exactly is it? Well, let me introduce you to Buffer underflowing! 

## Buffer Underflow
"Underflow" in buffer means that the consumer processes data faster than the producer, which would result in the consumer waiting for the buffer to be filled again *(or in our case, the consumer erroring)* - and this results in performance issues.

In our case, the consumer is consuming 3 results per a second, while the producer is producing 2 results per a second, which results in the said error the moment the consumer starts.

*A fix for this would be discussed later*

Okay, so anyway, what would happen if we let the producer produce 3 results per second while having the size of the buffer as 20?
[details ="click me for test code"]
```lua
local buff = {}
local consumed = {}
task.spawn(function()
	local num_of_testing = 0

	while true do
		task.wait(1/3)
		
		if num_of_testing >= 10 then
			break
		end
		num_of_testing += 1/3
		table.insert(buff, 123 * math.random(1, 123))
		if #buff > 20 then
			error("Buffer overflow!")
		end
	end
	
end)

task.spawn(function()
	local num_of_testing = 0

	local last_index = 0
	while true do
		task.wait(1)
		last_index += 1
		if num_of_testing >= 20 then
			break
		end
		local produced = buff[last_index]
		
		if produced == nil then
			error("Buffer underrun!")
		end
		
		table.insert(consumed, produced)
		num_of_testing += 1
	end
	
	print("total", #buff, "left", #buff - #consumed, "consumed", #consumed)
end)
```
[/details]

Well, it would eventually error "Buffer overflow", but what's that? Well, it is Buffer ***over***flowing

## Buffer Overflow
"Overflow" in a buffer means that the producer produces more results than the maximum capacity of the buffer *(usually because the consumer is slow to the point it can't keep up with the huge data in the buffer)*.

In our case, the consumer is consuming 1 result per a second, while the producer is producing ***THREE*** results per a second, so in theory, the producer will eventually produce values that the buffer won't be able to store *(10+ values)*.

### Solution?
So, how to fix buffer ***under***flows and ***over***flows? Well, you could lower down the size of the buffer, which would be either a great decision, or the worst one in your life depending on your project.

Or alternatively, you could:

* On the consumer - If the buffer is empty, sleep.
* On the producer - if the buffer is filled, retry later 
_____
# Conclusion
In a nutshell, buffers are generally places for storing data that is being moved/transmitted from a place to another.

Another use of buffers would be to store bits and binary from data streams, this generally makes sense because you are storing data that is being produced from a process and then consumed by another.
