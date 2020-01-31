# CarND-Controls-PID
Self-Driving Car Engineer Nanodegree Program

---

[//]: # (Image References)

[image1]: ./images/throttle.png "throttle DOE" 
[image2]: ./images/Kp_DOE.png "Kp DOE" 
[image3]: ./images/Kp_DOE2.png "Kp DOE CTE" 
[image4]: ./images/Kp_DOE3.png "Kp DOE steering angle" 
[image5]: ./images/Kd_DOE.png "Kd DOE" 
[image6]: ./images/Ki_DOE.png "Ki DOE" 
[image7]: ./images/cornering_1.png "example1"
[image8]: ./images/cornering_2.png "example3"


I. File list
------------
- main.cpp        : main code implementation
- PID.cpp        : PID function
- PID.h        : PID class
- README            : this file

Program can be built using default make arguments.

II. Goals 
----------
to comple 


III. Implementation
----------
PID part

```pid.UpdateError(cte);
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
At the end of program, I output a file for me to understand the behavior clearly. 
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

First, I would like to know how throttle play in the role. so I set bindly (Kp, Ki, Kd) = (1, 0, ,0) and change the throttle from 0.3 to 0.1. Apparently, throttle will detemrine the max number of the speed. As you can see, with lower the throttle, the later the osscialiation show up due to Kp. 

![alt text][image1]

Now I would like to foucs on the defult setting of the throttle = 0.3. I run the Kp from 1, 0.5, 0.1. Appartently, as the value of the Kp is lower the behvaior of overshooting osscilation is getting better. so I set Kp =0.1. 

![alt text][image2]

![alt text][image5]

Next step, I would like to play Kd, the differnetae term. I range Kd from 0 to 5. There is a sweet spot for Kd, and I can quiicly find that sweet spot. However, I encoutered to "corning" cases that the tires will be off the lanes. I capture the snapshot these two cases. 


![alt text][image3]
![alt text][image4]

Later, I found if I would like to have the car to be responsive, than I have to increase the Kp a bit. Therefore, I can setting down my tunning parameter as (Kp, Ki, Kd) = (0.2, 0, 3). 
![alt text][image6] 

How about Ki? should I turn it?  Well, in theory, if the system doesn't have any biases, I shouldn't need to. I believe it is in our case. Turing up the Ki, can make the error and oscialltion worse.  This is what I expected from Sabastrain's lecture.  He mentioned that "This may not seem all that impressive. PID seems to do worse than the PD controller! The purpose of the I term is to compensate for biases, and the current robot has no bias" 


so my final parameter is  (Kp, Ki, Kd ) = (0.2, 0, 4) in the throttle = 0.3. It can run very smaootly without noticable osscialtion. 

V. Summary
----------
My code can pass all the criteria in the case of throttle = 0.3. When the throttle is higher, there is velocity can go mucher, the adjustment will PID parameters will be differemt. The next step to imprve this one is to consider the speed, throttle into PID to make this program more robust.  
