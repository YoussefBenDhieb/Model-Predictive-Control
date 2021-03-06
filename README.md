# CarND-Controls-MPC
---
Self-Driving Car Engineer Nanodegree Program - Term 2 - Project 5


## Project description
---
The goal of this project is to implement a Model Predictive Controller on a self-driving car.
The main steps of this project are:
* Implement a kinematic model for the car
* Fit a polynomial for the next waypoints
* Implement a Model Predictive Controller
* Add a latency of 100 ms for the actuators (which is the case for real self driving cars)


## Implementation
---
  ### The Kinematic Model
  For the sake of simplicity we used the bicycle kinematic model istead of complex dynamics models.
  The model equations are as follows:
   * x_[t+1] = x[t] + v[t] * cos(psi[t]) * dt
   * y_[t+1] = y[t] + v[t] * sin(psi[t]) * dt
   * psi_[t+1] = psi[t] + v[t] / Lf * delta[t] * dt
   * v_[t+1] = v[t] + a[t] * dt
   * cte[t+1] = f(x[t]) - y[t] + v[t] * sin(epsi[t]) * dt
    * epsi[t+1] = psi[t] - psides[t] + v[t] * delta[t] / Lf * dt

Where the states are:
* x,y: the coordinates of the car
* psi: the heading angle of the car
* v: the velocity of the car
* cte: the cross track error
* epsi: the orientation error

and the actuators are:
* a: the acceleration
* delta: the steering angle

 ### Timestep Length and Elapsed Duration (N & dt)
When we increase the number of of points N the controller become slower and this is due to the polynomial fitting of all the points. After some trials we set N=10.
However for the elapsed duration, before including latency we got better results for values under 0.1s and bad results for values greater or equal to 0.2s. After including a latency of 0.1s, the best results were given by values between 1.2 and 1.8. Hence, we set dt =1.5s. 

### Polynomial Fitting and MPC Preprocessing
  Before fitting waypoints, we transform them to the car's coordinate system. Here we used the following equations :
  * waypoints_x_car = (waypoints_x_global - car_x)*cos(psi) + (waypoints_y_global - car_y)*sin(psi);
  * waypoints_y_car = -(waypoints_x_gloabl - car_x)*sin(psi) + (waypoints_y_global - car_y)*cos(psi);

After transforming the waypoints, we fit them to a 3rd degree polynomial and use its coefficients to compute the cross track error (cte) and the orientation error (epsi).

After that, we feed the state(0,0,0,v,cte,epsi) to MPC controller and obatain the values of the actuators (throttle and steering angle)

### Model Predictive Control with Latency
 As a last step, we add a latency of 100 ms to the actuators to mimic real case scenarios.
 In order to make the controller work with this latency we had to predict a new state from the state we used to feed to the MPC and feed it instead.
 The equations we used to predict the new state are:
* px1 = px0 + v * cos(psi0) * latency;
* py1 = py0 + v * sin(psi0) * latency;
* psi1 = psi0 - v * steering_angle / Lf * latency;
* v1 = v + a * latency;
* f = coeffs[0] + coeffs[1] * px + coeffs[2] * pow(0, 2) + coeffs[3] * pow(0, 3);
* psides = atan(coeffs[1] + 2 * coeffs[2] * px0 + 3 * coeffs[3] * pow(px0, 2));
* cte1 = f - py0 + v * sin(epsi) * latency;
* epsi1 = psi0 - psides - v * steering_angle / Lf * latency;

## Dependencies
---
* cmake >= 3.5
 * All OSes: [click here for installation instructions](https://cmake.org/install/)
* make >= 4.1(mac, linux), 3.81(Windows)
  * Linux: make is installed by default on most Linux distros
  * Mac: [install Xcode command line tools to get make](https://developer.apple.com/xcode/features/)
  * Windows: [Click here for installation instructions](http://gnuwin32.sourceforge.net/packages/make.htm)
* gcc/g++ >= 5.4
  * Linux: gcc / g++ is installed by default on most Linux distros
  * Mac: same deal as make - [install Xcode command line tools]((https://developer.apple.com/xcode/features/)
  * Windows: recommend using [MinGW](http://www.mingw.org/)
* [uWebSockets](https://github.com/uWebSockets/uWebSockets)
  * Run either `install-mac.sh` or `install-ubuntu.sh`.
  * If you install from source, checkout to commit `e94b6e1`, i.e.
    ```
    git clone https://github.com/uWebSockets/uWebSockets
    cd uWebSockets
    git checkout e94b6e1
    ```
    Some function signatures have changed in v0.14.x. See [this PR](https://github.com/udacity/CarND-MPC-Project/pull/3) for more details.

* **Ipopt and CppAD:** Please refer to [this document](https://github.com/udacity/CarND-MPC-Project/blob/master/install_Ipopt_CppAD.md) for installation instructions.
* [Eigen](http://eigen.tuxfamily.org/index.php?title=Main_Page). This is already part of the repo so you shouldn't have to worry about it.
* Simulator. You can download these from the [releases tab](https://github.com/udacity/self-driving-car-sim/releases).
* Not a dependency but read the [DATA.md](./DATA.md) for a description of the data sent back from the simulator.


## Basic Build Instructions

1. Clone this repo.
2. Make a build directory: `mkdir build && cd build`
3. Compile: `cmake .. && make`
4. Run it: `./mpc`.

## Tips

1. It's recommended to test the MPC on basic examples to see if your implementation behaves as desired. One possible example
is the vehicle starting offset of a straight line (reference). If the MPC implementation is correct, after some number of timesteps
(not too many) it should find and track the reference line.
2. The `lake_track_waypoints.csv` file has the waypoints of the lake track. You could use this to fit polynomials and points and see of how well your model tracks curve. NOTE: This file might be not completely in sync with the simulator so your solution should NOT depend on it.
3. For visualization this C++ [matplotlib wrapper](https://github.com/lava/matplotlib-cpp) could be helpful.)
4.  Tips for setting up your environment are available [here](https://classroom.udacity.com/nanodegrees/nd013/parts/40f38239-66b6-46ec-ae68-03afd8a601c8/modules/0949fca6-b379-42af-a919-ee50aa304e6a/lessons/f758c44c-5e40-4e01-93b5-1a82aa4e044f/concepts/23d376c7-0195-4276-bdf0-e02f1f3c665d)
5. **VM Latency:** Some students have reported differences in behavior using VM's ostensibly a result of latency.  Please let us know if issues arise as a result of a VM environment.
