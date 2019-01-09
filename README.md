# CarND-Controls-PID
Self-Driving Car Engineer Nanodegree Program

---

## How to run
To build and run, follow these steps:

1. Clone this repo
2. Make a build directory: `mkdir build && cd build`
3. Compile: `cmake .. && make`
4. Run it: `./pid -0.25 0.0 -5.0 0.4 0.0 4.0 1.0`

Program arguments:
1. Steering controller P component, for ex. _-0.25_
2. Steering controller I component, for ex. _0.0_
3. Steering controller D component, for ex. _-5.0_
4. Throttle controller P component, for ex. _0.4_
5. Throttle controller I component, for ex. _0.0_
6. Throttle controller D component, for ex. _4.0_
7. Max applied throttle, throttle controller uses this as starting value, for ex. _1.0_ for full throttle

For more info, see original Udacity repository:
https://github.com/udacity/CarND-PID-Control-Project/blob/master/README.md

## Result

By running controller with parameters given above, vehicle runs relatively fast and smooth:

[video sample](example_run480.mov)

## Solution

### Steering controller

Steering controller increases steering angle (in opposite direction) as car move away from center line.

```c++
pid_steering.UpdateError(cte);
steer_value = pid_steering.TotalError();
```

### Throttle controller

Throttle controller decelerates as car move away from center line and as car has larger steering angle.
This cause car even slow down when it is center line but has some steering angle or car has no steering
angle but is away from center line. This imitates as human drives a car by proportionally slowing down 
as situation gets trickier.

```c++
pid_throttle.UpdateError(fabs(steer_value) + fabs(cte));
throttle_value = max_throttle - pid_throttle.TotalError();
```

### PID components

**_P or proportional component_** causes car to proportionally decrease the error. For _steering controller_
it makes steering angle bigger as car moves away from center line (CTE goes bigger), so the car would steer back
towards center line. For _throttle controller_ it makes car use proportionally less throttle as car moves away more 
from center or has bigger steering angle.

**_I or integral component_** counters systematic bias in the system, which causes car not to reach center. 
As this solution works in idealistic simulator and there is no bias, this parameter is not tuned and left 0 for
both steering and throttle pid controllers.

_**D or differential component**_ counteracts P components tendency to overshoot, has basically dampening effect.
For _steering controller_ it causes car not to over-steer, which causes oscillation. For _throttle controller_ it causes
car not to over-brake, which would cause jerky throttle-brake symptom.

### Tuning

I tuned parameters manually. I started with _steering controller_ and after car was able to drive with constant throttle
_0.3_, I added _throttle controller_ and tuned it's parameters so that car would drive faster, but in the same time would
be relatively smooth.

I didn't use algorithmic tuning as I wanted to understand intuitively better how car behaves with different changes
to controller parameters. Also this system is quite dynamic for using with algorithms like Twiddle and it would be
very easy for the algorithm to change parameters to something that would cause car to leave the track.
