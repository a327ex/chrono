# chrono 

**chrono** is a timer module for LÖVE. It's a copy of [hump.timer](http://hump.readthedocs.io/en/latest/timer.html) with a few improvements on top.

<br>

# Contents

* [Usage](#usage)
* [Creating a timer object](#creating-a-timer-object)
* [after](#after)
* [every](#every)
* [during](#during)
* [script](#script)
* [tween](#tween)
* [Tags](#tags)
* [Random delays](#random-delays)
* [cancel, getTime](#cancel-gettime)
* [destroy](#destroy)

<br>

## Usage

```lua
Timer = require 'Timer'
```

<br>

### Creating a timer object

```lua
function love.load()
  timer = Timer()
end

function love.update(dt)
  timer:update(dt)
end
```

All created timers must be updated otherwise they won't work. You can create multiple timers and personally I create one for each entity that needs timing functions.
However, you can also only use a single global one if you think that's better.

<br>

### after

```lua
function love.load()
    ...
    timer:after(2, function() print(1) end)
end
```

The `after` function will execute an action after a given amount of time. It receives a number and a function, representing the delay and the action respectively. 

These are all the valid ways in which `after` can be called:

```lua
timer:after(delay, action)
timer:after(delay, action, tag)
```

<br>

### every

```lua
function love.load()
    ...
    timer:every(0.5, function() print(1) end)
end
```

The `every` function will execute an action repeatedly at an interval. It receives a number and a function, representing the interval duration and the action respectively. A third
`count` argument can be passed in to limit the amount of times the action will be run:

```lua
function love.load()
    ...
    timer:every(0.5, function() print(1) end, 5)
end
```

The example above will print `1` a total of 5 times with a 0.5 seconds interval between each. This `count` argument is optional and can be omitted, and if it is then the timer will repeat forever
or until it's cancelled.

A fourth `after` argument can be passed in as an action that will be executed once the `every` timer ends. This argument may only be passed in if `count` is also passed in.

```lua
function love.load()
    ...
    timer:every(0.5, function() print(1) end, 5, function() print(2) end)
end
```

The example above will print `1` a total of 5 times, and when that ends 2 will also be printed.

These are all the valid ways in which `every` can be called:

```lua
timer:every(delay, action)
timer:every(delay, action, tag)
timer:every(delay, action, count)
timer:every(delay, action, count, tag)
timer:every(delay, action, count, after)
timer:every(delay, action, count, after, tag)
```

<br>

### during

```lua
function love.load()
    ...
    timer:during(1, function() print(1) end)
end
```

The `during` function will execute an action every frame for a given amount of time. It receives a number and a function, representing the duration and the action respectively. Additional, a third
`after` argument can be passed in to be executed after the specified duration:

```lua
function love.load()
    ...
    timer:during(1, function() print(1) end, function() print(2) end)
end
```

The example above will print `1` every frame for 1 second, and then print 2 once at the end of that 1 second.

These are all the valid ways in which `during` can be called:

```lua
timer:during(delay, action)
timer:during(delay, action, tag)
timer:during(delay, action, after)
timer:during(delay, action, after, tag)
```

<br>

### script

```lua
function love.load()
    ...
    timer:script(function(wait)
        print(1)
        wait(1)
        print(2)
        wait(1)
        print(3)
    end)
end
```

The `script` function will execute the statements inside the function passed in with the ability to stop the execution using the `wait` function. The code above is equivalent to a chain of `afters`:

```lua
function love.load()
    ...
    print(1)
    timer:after(1, function()
        print(2)
        timer:after(1, function()
            print(3)
        end)
    end)
end
```

<br>

### tween

```lua
function love.load()
    ...
    player = {x = 100, y = 100}
    timer:tween(1, player, {x = 200, y = 200}, 'in-out-cubic', function() print('Player tween finished!') end)
end
```

The `tween` function will tween values from the subject table to the values in the target table over a certain duration using a certain tween method. In the example above, the `x` and `y` values
in the `player` table will be changed from 100 to 200 over 1 second via the `'in-out-cubic'` tween method, and then after the tween is done `'Player tween finished!'` will be printed to the console.
A list of all tween modes available as well as further explanation can be found [here](http://hump.readthedocs.io/en/latest/timer.html#Timer.tween).

These are all the valid ways in which `tween` can be called:

```lua
timer:tween(delay, subject, target, method)
timer:tween(delay, subject, target, method, tag)
timer:tween(delay, subject, target, method, after)
timer:tween(delay, subject, target, method, after, tag)
```

<br>

### Tags

This library has the same set of functions that `hump.timer` has: `after`, `every`, `during`, `tween` and `script`. The main difference is that all functions (except for `script`) can accept
a `tag` argument. This argument takes care of a very common pattern that happens with timers which is that a timer is usually started whenever an event happens. The problem with this
is that generally we have no control over how often this event happens, which means often we get overlapping timers. Consider the following example:

```lua
function love.keypressed(key)
    if key == 'k' then
        timer:after(2, function() print(math.random()) end)
    end
end
```

In this example whenever we press `'k'`, a random number will be printed to the console 2 seconds later. If we press `'k'` repeatedly really fast then we'll get random numbers printed multiple times, 
always 2 seconds after whenever we pressed `'k'`. However, often times what we want in our gameplay is that instead of performing the action multiple times, if the timer is already 
running for this same action, we want to cancel the currently running one and start the timer over. The behavior then would be that if `'k'` is pressed repeatedly really fast, we'd
only see a random number printed to the console 2 seconds after the last press.

To achieve this behavior the library makes use of tags. Tags are available for all functions (except `script`) and they're always the last argument. If no tag is given then the default behavior
happens (the one explained above), otherwise we get the resetting behavior that we often want. For the `after` function in the example above it would look like this:

```lua
function love.keypressed(key)
    if key == 'k' then
        timer:after(2, function() print(math.random()) end, 'random_number')
    end
end
```

And so if we press `'k'` multiple times really fast, we'll only see a random number printed 2 seconds after the last press. Tags should always be strings and they should always be unique within
each timer. If you have two different actions that share the same tag then they'll cancel each other out. For instance:

```lua
function love.keypressed(key)
    if key == 'k' then
        timer:after(2, function() print(math.random()) end, 'random_number')
    end
    
    if key == 'l' then
        timer:after(2, function() print(1) end, 'random_number')
    end
end
```

In the example above, pressing either `'k'` or `'l'` will reset the `'random_number'` timer, which means that we'll only get either a random number or 1 printed whenever we let 2 seconds pass after
the pressing either key. Usually in gameplay code we want different actions to have different tags, since generally we want to cancel timers only between multiple calls of the same action.

<br>

### Random delays

All functions in the library also accept random delays. By default, the `delay` argument of each function should be a number, but it can also be a table containing 2 numbers that serve as an interval.

```lua
function love.load()
    ...
    timer:after({1, 2}, function() print(math.random()) end)
end
```

And so in the example above a random number will be printed after a random amount of time between 1 and 2 seconds. This behavior is especially useful with the `every` function:

```lua
function love.load()
    ...
    timer:every({0.5, 1}, function() print(math.random()) end)
end
```

Here a random number will be printed at an interval that is a random amount of time between 0.5 and 1 second. So at first a number may be printed after 0.6 seconds, and then another 0.95 seconds after that,
and another 0.76 seconds after that, and so on.

<br>

### cancel, getTime

All main functions in the library return a tag:

```lua
tag = timer:after(1, function() print(1) end)
```

If the tag argument is specified then the returned value will be that, otherwise it will be a randomly created unique one. This tag can be used in conjunction with two other functions: `cancel` and `getTime`.

The `cancel` function will forcefully cancel the further execution of a given timer. For instance:

```lua
function love.load()
    ...
    tag = timer:after(1, function() print(1) end)
    timer:cancel(tag)
end
```

After the code above is run 1 will never be printed to the screen, since the timer that was responsible for executing that function was cancelled by the `cancel` function.

The `getTime` function will return the current and maximum values of this timer. For instance:

```lua
function love.load()
    ...
    tag = timer:after(1, function() print(1) end)
end

function love.keypressed(key)
    if key == 'k' then
        local current, maximum = timer:getTime(tag)
        print(current, maximum)
    end
end
```

And so whenever `'k'` is pressed, the `current` and `maximum` values will be printed. The current value represents the current state of the timer. For instance, if we press `'k'` 0.5 seconds 
after the game started then the current value will be `0.5`. The `maximum` value is always the value that represents the end of the timer, so in this case it will be `1`. For timers that use random
delays it can be useful to figure out which was the random value chosen by the library via the `maximum` value.

<br>

### destroy

```lua
timer:destroy()
```

The `destroy` function will clear the timer entirely. If you're using one timer per entity then it's probably necessary to call this whenever the entity dies otherwise memory may leak.

<br>

# LICENSE

You can do whatever you want with this. See the license at the top of the main file.
