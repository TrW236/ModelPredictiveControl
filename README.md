## Model Predictive Control

### Usage
* Please see the [repository](https://github.com/udacity/CarND-MPC-Project) from Udacity.
* Replace all the files in the folder `src`.

### Description of the Vehicle Model
##### Control Inputs:
* `delta`  is the steering wheel angle. The bound is (-25, 25) degree.
* `a` is the throttle (when > 0) and brake (when < 0). The bound is (-1, 1).
##### State Vector:
* `x` is the x coordinate of the vehicle.
* `y` is the y coordinate of the vehicle.
* `psi` is the yaw angle of the vehicle.
* `v` is the velocity in direction in front of the vehicle.
* `cte` is the cross track error at this moment.
* `epsi` is the orientation error
##### State Update:
* `x = x + v * cos(psi) * dt`
* `y = y + v * sin(psi) * dt`
* `psi = psi + v / Lf * delta * dt`  (slightly different in the code due to the different coordinates in simulator)
* `v = v + a * dt`
* `cte = cte + v * sin(epsi) * dt =  f(x) - y + v * sin(epsi) * dt`
* `epsi = epsi + v / Lf * delta * dt = psi - psi_des + v / Lf * delta * dt` (slightly different in the code)

whereby `psi_des` is the desired `psi`. `f(x)` is the reference line. `Lf` is the length between the gravity center and the front of the vehicle.

### Timestep Length and Elapsed Duration (N & dt) 
* `T` is the duration, in which the predictive trajectory will be calculated. This value should be large enough, but, it is not allowed to be much large.
 
 When it is too large: firstly the trajectory will beyond the horizon, which will make no sense; secondly, high value can introduce error, 
 due to that the reference path is approximated through a polynomial. 
 Thus high value of `T` requires that the reference path should also be accurate sufficiently, which is not possible; 
 thirdly the computation will also be too much.
 
 When it is too small, the evaluation of the trajectory is not sufficient, in order to have a precise optimization result.
 
 It is recommended by Udacity that `T` is set to be a few seconds.
* `N` the number of discretization. When `N` is too high, the computation will be too much. When `N` is too small, the predicted values will be not sufficiently accurate.
* `dt` is for the discretization of `T`. It should not be too large or too small. The reason is similar to above.
* `T = N * dt`

### MPC Preprocessing

The coordinates values, which will be sent into the optimization solver, are preprocessed. The global coordinates of the waypoints will be changed into the view of the vehicle. This means the coordinates values is changed according to the position of the vehicle `(x, y, psi)`, so that `x = 0, y = 0, psi = 0`.

This preprocessing will simplify numerical calculation, such as the calculation of `cte`, `epsi` and the optimization process.

### Model Predictive Control with Latency
In a real car, an actuation command won't execute instantly - there will be a delay as the command propagates through the system. A realistic delay might be on the order of 100 milliseconds.
This is a problem called "latency".

MPC can deal with latency more easily, because this type of delay can be  as dynamics incorporated into the model. For instance, instead of considering the true current state the MPC will calculate the state after the delay (this is like "prediction") and use this state as the "current" state.

In my implementation:
1. The state `x, y, psi, v` after `100 ms` will be calculated.
2. The coordinates of waypoints will be changed accordint to this new state (the state after 100 ms).
3. Other variables will be calculated, such as `cte, epsi` and `coeffs`, so that the state after `100 ms` will be the first state to be considered. 
### Optimization Process

* The cost function will be defined according to the needs. For instance, the errors are not allowed to be much high, the changing of the steering angle is not allowed to be much large etc.
* The variables are the states and the actuators at each time point except the first time point. 
* The equality constraints are the updates of the states.

### Result
![Simulation](./pics/res.png)

### References:
1. Udacity Self-Driving Car Nanodegree