# CarND-Controls-MPC
Self-Driving Car Engineer Nanodegree Program

---

## The Model: Student describes their model in detail. This includes the state, actuators and update equations.

As learned in the lesson, the model states are vehicle x position, y position, vehicle heading, speed, cross track error, and orientation error. Actuators include steering wheel level and accelerate/brake pedal. 
Based on vehicle model, the update equations take the current states value from this time step, as well as the actuator value of this time step, to calculate the states of next time step. This part is in line 140-145 in MPC.cpp. 
By preparing the states variables boundary values and other constraints, all these parameters are inputs of ipopt function to calculate the best states and actuator values of future steps.

## Timestep Length and Elapsed Duration (N & dt): Student discusses the reasoning behind the chosen N (timestep length) and dt (elapsed duration between timesteps) values. Additionally the student details the previous values tried.

Timestep length N chosen is 10, and elapsed duration dt chosen is 0.1s. First select dt. 0.1s is taking reference from the latency value defined in this project, which makes sense. 
Too small dt lead to much more computation load of the controller, while too large dt will affect the accuracy of MPC performance. 
Choosing N to be 10 can make the MPC predect future 1s states and actuator value. Too large N will cause more computation load of the controller, and also may exceed available waypoint values from simulator.
Too small N will affect the prediction accuracy.
Other value tried is N=100 and dt=0.025.

## Polynomial Fitting and MPC Preprocessing: A polynomial is fitted to waypoints. If the student preprocesses waypoints, the vehicle state, and/or actuators prior to the MPC procedure it is described.

Third degree polynomial fitting for trajectory is performed with waypoint data from simulator each time step. Before doing that, two things need to be done first. 
One is to convert the waypoint data from map coordinate to vehicle coordinate. This will make the etc calculation as well as the display of trajactory and predicted path easier. 
The other is to convert the waypoint data from std::vector type to Eigen::VectorXd, as required by the polyfit function.

MPC cost function is the one needs a lot of tuning. Besides the cost parameters from the lesson, two items are added. 
One is steering actuator value times vehicle speed. This takes the largest weight, which is most important to keep the vehicle stable. This makes sense in real world too. 
When people turn che steering wheel, usually they will also lower the vehicle speed. On the other hand, when driving high speed, people will usually try to keep steering wheel move as little as possible.
This part is in line 79 in MPC.cpp.
The other is the gap between vehicle speed and 15mph, when speed is lower than 15mph. Since we have the first item (steering actuator value times vehicle speed) and the weight is so large, 
it will lower the vehicle speed so much during vehicle start and big turns, sometimes it cause the vehicle running too slow even stop completely. So this item is to avoid such condition.
This part is in line 70 in MPC.cpp.

## Model Predictive Control with Latency: The student implements Model Predictive Control that handles a 100 millisecond latency. Student provides details on how they deal with latency.

All ipopt function outputs are collected. This part is in line 285 in MPC.cpp. To deal with the 100ms latency, I am always using the predicted actuator values of the second time step from each MPC output as the current step actuator value,
which is actually for the future 0.1s, but perfectly compensate the 0.1s latency.
This part is in line 134 in main.cpp.

Finally with above implementation, the vehicle can run very stable with ref speed 100mph.

