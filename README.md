# CarND-Controls-PID
Self-Driving Car Engineer Nanodegree Program

---

[//]: # (Image References)

[image1]: ./images/throttle_DOE.png "throttle DOE" 
[image2]: ./images/Kp_DOE.png "Kp DOE" 
[image3]: ./images/Kp_DOE2.png "Kp DOE CTE" 
[image4]: ./images/Kp_DOE3.png "Kp DOE steering angle" 
[image5]: ./images/Kd_DOE.png "Kd DOE" 
[image6]: ./images/Ki_DOE.png "Ki DOE" 
[image7]: ./images/cornering_1.png "example1"
[image8]: ./images/cornering_2.png "example2"
[image9]: ./images/PID_equation.png "equation"

I. File list
------------
- main.cpp        : main code implementation
- PID.cpp        : PID function
- PID.h        : PID class
- README            : this file

Program can be built using default make arguments.

II. Goals 
----------
Build a PID controller and tune the PID hyperparameters by applying the general processing flow, as described in the previous lessons. Test the solution on the simulator!

III. Implementation
----------
PID controller implementation is straightforward based on the following equation, as Sebastian described in the lecture. The hard part is how to find the optimal hyperparameters in a system.

![alt text][image9]

Here is my code based on the hints provided. CTE is the value of the Cross Track Error that provides by the simulator.

```
pid.UpdateError(cte);
steer_value =  - pid.TotalError(); 
```
```
void PID::UpdateError(double cte) {
  p_error = cte;
  i_error = i_error + cte;
  d_error = cte - prev_cte;
  prev_cte = cte;
}
```
```
double PID::TotalError() {
  return Kp * p_error + Ki * i_error+ Kd * d_error;  // TODO: Add your total error calc here!
}
```
To be systematic, I add an output function, so at the end of the program, the code output all the parameters (cte, speed, angle, steer_values), and I can later plot them to understand the PID behavior clearly. 

```
#include <fstream> // For output a file

int main(){
...
outputData(cte, speed, angle, steer_value);
...
}
void outputData(double cte,double speed,double angle, double steer_value)
{
   ofstream outputFile;
   outputFile.open("output.txt",ios_base::app);
   outputFile << cte << "\t" << speed << "\t" << angle << "\t" << steer_value << endl;
   outputFile.close();
}
```
IV. Parameter Optimation
----------

First, I would like to know how throttle plays in the role. So I blindly set (Kp, Ki, Kd) = (1, 0, 0) and range the throttle from 0.3 to 0.1. I realized that throttle determines the max of the saturation speed. As you can see, at the same Kp, the lower the throttle, the later the oscillation shows up. It is up to the speed of the car in the simulator.  

![alt text][image1]

Now I would like to focus on the default setting of the throttle = 0.3. I run the Kp from 1, 0.5, 0.1. Data shown in the following image. The lower the Kp, the less of overshooting oscillations. So I set Kp =0.1 for the next test. 

![alt text][image2]

Next step, I would like to play Kd, the differentiae term. I range Kd from 0 to 3. Data is shown below. The high the Kd, the less of overshooting oscillations.

![alt text][image5]

Later I found Kd= 0.4 is good when Kp=0.1 and Ki=0. But then I encountered the tires off track in the "cornering" cases. I captured the snapshot of these two cases.  

![alt text][image7][image8]


After some tuning, I found that I need to increase the Kp again. Large Kp will give a quick response. 

![alt text][image2]
![alt text][image3]
![alt text][image4]

Later, I found if I would like to have the car to be responsive,  I have to increase the Kp a bit. Therefore, I decide on my hyperparameter as (Kp, Ki, Kd) = (0.2, 0, 4). 


How about Ki? should I turn it? Well, in theory, if the system doesn't have any biases, I shouldn't need to. I believe it is in our case. Turing up the Ki, can make the error and oscillation worse. It is what I expected from Sabastrain's lecture. He mentioned that "This may not seem all that impressive. PID seems to do worse than the PD controller! The purpose of the I term is to compensate for biases, and the current robot has no bias." 

![alt text][image6] 


so my final parameter is (Kp, Ki, Kd ) = (0.2, 0, 4) in the throttle = 0.3. It can run very smoothly without noticeable oscillations. 

V. Summary
----------
My code can pass all the criteria in the case of throttle = 0.3. When the throttle is higher, the car's speed can go higher; the hyperparameter of the PID controller will be different. Therefore, my next step is to improve the code by considering the speed and throttle into PID controller and make this program robust on speed.  
