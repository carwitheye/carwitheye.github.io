---
layout: page
---
**Hi, here is the EN version.** 日本語の紹介は[こちらへ](https://carwitheye.github.io)

<center><font size="6">Development of Eye-movement Control</font></center>
<center><font size="6">in Self-driving Cars</font></center>

<br />
<center><strong>
    <a href="https://scholar.google.com/citations?user=1pi0i1wAAAAJ&hl=en">Yinfei Qian†1</a>　
    <a href="https://scholar.google.com/citations?user=eudRCpEAAAAJ&hl=en">Zaiqiang Wu</a>　
    <a href="https://xinyuegui.github.io/hcid.gui/index.html">Xinyue Gui†1</a>　
    <a href="https://revetronique.com/">Koki Toda†1</a>　
    <a href="http://chiamingchang.com/">Chia-Ming Chang†1</a>　
    <a href="https://www-ui.is.s.u-tokyo.ac.jp/~takeo/index-j.html">Takeo Igarashi†1</a>
</strong></center>
<br />
**Abstract**：The eye concept applied in external human-machine interface (eHMI) become popular. When making eye contact with a pedestrian, it becomes an important issue to look at the pedestrian in a reasonable and automatic way. However, when there is multiple pedestrians approaching to the car, the eye movement is unclear. In this study, we aim to address this problem by improving the algorithm developed by previous research[8]. We make the eye movement automatically and exhibited our car with moving eyes prototype in CEATEC 2022. We welcomed many visitors in different industries, and received feedback from varied aspects, which is useful for future “eye concept” development.

†1　Graduate School of Information Science and Technology, The University of Tokyo

<br />
## 1. Introduction
In driverless self-driving cars, because there is no human driver, the expression of the car's intent needs to be through external Human Machine Interface (eHMI) [some reference].  Interaction methods using human-like eyes have been proposed and it has shown benefits for car-pedestrian communication [2, 5, 6, 7, 8]. By using eye concept, pedestrians could understand the vehicle's intentions more intuitively, which led the problem of how to make the eyes interact with pedestrians more rational and real-time important.

In this paper, we will further explore the concept of using the eye as an eHMI, implementing a vehicle prototype as shown in <font color="#dd0000">Figure 1.</font> that automatically controls the eye through an algorithm. We develop a fast, near real-time automatic eye control through algorithms and implemented, enabled our system on a prototype self-driving vehicle. In our proposed system, we define the interaction subjects as pedestrians in front of the vehicle. Currently, the approach of interaction with pedestrians is simply detect pedestrian, then let robotic eyes looking at them(gaze). We control the robotic eyes’s movement automatically and prioritizes which pedestrians to focus on by making and combine several different rules, as we believe that eye movements that are more like a human driver's are more likely to give pedestrians an intuitive understanding of the car's intent. Since we want the system to react quickly to pedestrians, we use multiple threads to allow detecting pedestrians, determining whether pedestrians need to be looked at in priority, and controlling the motor of the eye simultaneously.

<div align=center><img src ="assets/img/Fig1.jpg"/></div>
<center><strong>Figure 1.</strong> Car Prototype with our system</center>
<br />
We conducted a pilot study to test the performance of the system. In addition, we exhibited our vehicles at CEATEC 2022 by having visitors as our subjects (pedestrians). Through observation and interviewing of the visitor, we obtained many feedbacks and suggestions from visitors, which will be useful insights for other researchers in car-pedestrian interaction field and our future development on improving our system.

<br />
## 2. System Design
Our system is built with pedestrian detector and algorithm that control eye to move, and gazing pedestrian.  The system runs in steps as shown in <font color="#dd0000">Figure 2.</font> :

- (a) Capture video stream from front camera fixed in front of the car
- (b) Perform pedestrian detection to find pedestrians in the view
- (c)
	- (c-1) Crop each detected pedestrian for pose estimation (to detect hand-wave, head location, etc)
	- (c-2) Add each pedestrian into Pedestrian Queue. Then sort queue based on the rule we set. Here, we prioritize pedestrians closest to vehicles. In this case, we first control the eye to look at the closest person.

*Notice that (c-1) and (c-2) are concurrent. We will also perform pose estimation for each person in order to detect hand-wave, etc.*
- (d) After we sort pedestrian queue, we select top one and estimate the head position in 2D coordinate.
- (e) Convert 2D co-ordinate into rotation angle by linear mapping function for eyes to rotate.


<div align=center><img src ="assets/img/Figure2.png"/></div>
<center><strong>Figure 2.</strong> Pedestrian Detection</center>
<br />

# 2.1 Pedestrian Detection
We first detect pedestrians in the frame that captured by the camera fixed on the car. After obtained frame, we perform object detection on the captured frame to detect pedestrians. We have designed several different rules to look at pedestrians. For example, we give priority to the pedestrian closest to the car and look at him for up to five seconds at a time. We also give priority to pedestrians who are walking towards the car, the fastest moving pedestrians, or pedestrians who are waving at the vehicle (Figure <font color="#dd0000">3</font>, <font color="#dd0000">4</font>). The system achieves the rules by the following algorithms. We estimate the pedestrian's distance from the vehicle, movement speed and facial orientation, movement and other states to calculate which pedestrian is to be prioritized gaze. We perform pedestrian detection using a modified Yolov4 for our task. We also achieved hand-waving detection by estimate the pose of pedestrians using Mediapipe. The pipeline of pedestrian detection is similar to Praveen et.al 2022 's work but faster because we have enabled parallel computation on GPU.

<div align=center><img src ="assets/img/Fig3.jpg"/></div>
<center><strong>Figure 3.</strong> Pedestrian Detection UI</center>
<br />

# 2.2 Car Prototype
The physical anthropomorphic prototype is built of robotic eyes and mounted them on a real car. We built a pair of robotics eyes with the black plastic eyeball and the transparent hemispheric acrylic cover. Each of the eye unit driven by two actuators possess two degrees of motion. When the pedestrian aimed to be gaze is detected, the algorithm calculates an angle by linear mapping function and sent to the actuators to move the eye. Due to the mechanism limitation, the maximum rotation of each eye is 45 degrees to each direction. To protect the robotics eyes, we estimated the suitable frequency of sending commands to the motor according to the upper limit of motor movement speed.

## 3. Improvements of the deficiencies from the previous system
Our system is based on the work of Praveen et.al [8]. We used a similar pipeline but achieved higher detection speed, higher transmission speed, improved the interaction algorithm and obtained test results in a real AVs prototype test.  In Figure 4., we give an example of one of the most representative eye movement patterns. We attached a detailed explanation of it below the figure.
In the previous studies’ system, the deficiencies are mainly in as follow:

<div align=center><img src ="assets/img/fig4.png"/></div>
<center><strong>Figure 4.</strong> Eye-movements based on priority rule</center>
<br />


# 3.1	Eye movements become chaotic when pedestrian targets are lost or when the detection environment is unstable
Our pedestrian detection is currently based entirely on vision. In the previous system, when the web camera captured too bright or too dark a frame, the location of the pedestrian could not be accurately identified. Once the pedestrian target is lost, the eye is zeroed out because it cannot receive a new angle command. The coordinates of a same pedestrian can also change when the detection is unstable, which can cause the eyes to jitter. We improved the accuracy and robustness of the neural network specific to pedestrians by modifying the neural network. Moreover, we adjusted the captured original image to avoid overexposure or underexposure. Also, we added a rule that in the case of losing the pedestrian target, the eye does not zero out, but follows the original position for a few frames in order to resume tracking. We also added a smoothing filter to the output angle as a way to reduce eye tracking jitter for the same pedestrian. In future work, we plan to combine the vehicle's radar and depth cameras, infrared cameras, to improve pedestrian recognition performance in environments such as nighttime.

# 3.2 It has high latency in tests on the prototype car
In the original system, because of the design of the motor, sending commands too fast in a single thread would 
result in a stack of commands, which would cause a delay of up to 3-5 seconds for the entire process from detecting a pedestrian to moving the eye. We addressed this problem by splitting the detection and command transmissions into 
multiple processes (and threads). So that the eye always receives the latest command, reducing the latency by an average of 92%. Under good lighting conditions, the latency is between about 0.3 and 0.6 seconds. In situations where there is a lot of eye movement (e.g., many pedestrians, requiring large eye movements), the latency is around 0.8 second. In future work, we plan to reduce latency and achieve full real-time by using higher-powered motors to operate the eyes while continuing to optimize the latency of data transmission.

# 3.3 The eyes are not focused on a single point
In the original system, the motors of both eyes received the same rotation angle. This resulted in the eyes being parallel and unable to focus on a single point. Such a situation would lead to two pedestrians in different positions mistakenly thinking that the eyeballs were looking at themselves, which is a result we did not want. We calibrate the distance of the camera, learn the approximate distance of the pedestrian through an algorithm, and send individual commands to each eye so that the eye would not gaze parallel. For example, when a pedestrian approaches a vehicle, the eyes will look inward. The further the pedestrian moves away from the vehicle, the more parallel the eyes will be. In the user test, the subjects reported a clear sense of being watched compared to the original system.

## 4. Test
We have conducted performance tests on our system. We found that pedestrians are difficult to notice when they are too far away from the vehicle, even when the eye is looking at them. If they are too close to the car, they will not be able to gaze or recognize them due to mechanical reasons. Therefore, we designed our target range to be within 75 cm to 3.5 m from the front of the vehicle. Within this range, we tested randomly dressed pedestrians by having them stand or walk in front of vehicles during daytime and evening (i.e., in low light conditions). As a result, the pedestrian dressing slightly affects the performance of pedestrian detection under low light conditions, increasing the possibility of missing detection targets. We tested the effect of detection from 1 to 8 people. We found that the screen becomes crowded when the number of people being detected exceeds 6 within the specified range, resulting in a higher error rate of pedestrian detection. Our system can operate in almost real time with less than 6 pedestrians during the day or in good light.

## 5. Exhibition in CEATEC2022
We exhibited our prototype vehicle with our system at CEATEC 2022 (Figure 1,5). We demonstrated and explained our system to the visitors, and summarized their feedback into three key points:

# Whether the system would fail when there were too many pedestrians.
In future work, we plan to use higher accuracy pedestrian detection and add head and face recognition to better detect each pedestrian. However, the usage scenario of our eHMI is not crowd-specific, so when we encounter a crowd, we will disable or replace our current system with another eHMI that is suitable for crowds.

# Reaction corresponding to pedestrians
Currently, our system can estimate whether the pedestrian noticed the car by the face orientation of the person. In addition, we also respond to pedestrians who wave their hand to the vehicle, then look at the pedestrian to indicate that they noticed them. However, considering that pedestrians usually do not respond physically to vehicles, in the future we plan to estimate the age group of the pedestrian, and state characteristics (e.g., walk speed, direction) to adjust the timing of the eyes looking at them. In addition, we are designing eye animations, and aim to communicate with pedestrians through animated representations.

# Other applications
The visitors gave us a lot of advice. Our eyeball eHMI can be used not only for common AVs, but also for unmanned delivery vehicles, heavy machinery, self-driving wheelchair, etc., especially in the places where there are no traffic signals and traffic signs and where there are crowds of people.、

## 6. Conclusion and Future work
In this study, we aim to inform pedestrians where the AVs' attention is intended through eye contact. We have optimized the algorithm to achieve near real-time performance. At the same time, we implemented a pilot study to test our system. The results show that our system can perform well under limited light conditions and number of people. After getting many suggestions from CEATEC2022, we plan to explore more on the design of the eye, gaze approach in the future. We also plan to conduct a more detailed user study to analyze the effectiveness of our system through quantitative and qualitative testing as a way to contribute to more possibilities in the car-pedestrian communication field.

## Reference
1. Light-Based External Human Machine Interface: Color Evaluation for Self-Driving Vehicle and Pedestrian Interaction
2.Chia-MingChang,KokiToda,DaisukeSakamoto,andTakeoIgarashi.2017.Eyes
on a Car: an Interface Design for Communication between an Autonomous
Car and a Pedestrian. In Proceedings of the 9th international conference on
automotive user interfaces and interactive vehicular applications (AutomotiveUI
’17), pp. 65-73. DOI: https://doi.org/10.1145/3122986.3122989
3. YoichiOchiaiandKeisukeToyoshima.2011.Homunculus:TheVehicleasAug-
mented Clothes. In Proceedings of the 2nd Augmented Human. 3–6. DOI:
https://doi.org/10.1145/1959826.1959829
4. Jaguar Land Rover Automotive Plc., 2018. The Virtual Eyes Have it. Retrieved
June 19, 2022 from https://www.jaguarlandrover.com/2018/virtual-eyes-have-it
5. Chia-Ming, Chang, Koki Toda, Takeo Igarashi, Masahiro Miyata, and Yasuhiro
Kobayashi, 2018, September. A Video-based Study Comparing Communication 
Modalities between an Autonomous Car and a Pedestrian. In Adjunct Pro-
ceedings of the 10th International Conference on Automotive User Interfaces
and Interactive Vehicular Applications (AutomotiveUI ’18), pp. 104-109. DOI:
https://doi.org/10.1145/3239092.3265950
6. Xinyue Gui, Koki Toda, Stela H. Seo, Chia-Ming Chang and Takeo Igarashi. 2022.
“I am going this way”: Gazing eyes on a self-driving car show multiple moving
directions. In Proceedings of the 14th International Conference on Automotive
User Interfaces and Interactive Vehicular Applications (AutomotiveUI ’22).
7. Chia-Ming Chang, Koki Toda, Xinyue Gui, Stela H. Seo and Takeo Igarashi,
2022, Can Eyes on a Car Reduce Traffic Accidents?. The 14th ACM International
Conference on Automotive User Interfaces and Interactive Vehicular Applications
(AutomotiveUI 2022), Seoul, South Korea, 17-20 September 2022
8. Praveen Singh, Chia-Ming Chang, and Takeo Igarashi. 2022. I See You: Eye Control Mechanisms for Robotic Eyes on an Autonomous Car. In 14th International Conference on Automotive User Interfaces and Interactive Vehicular Applications (AutomotiveUI ’22 Adjunct), September 17–20,
2022, Seoul, Republic of Korea. ACM, New York, NY, USA, 5 pages. https://doi.org/10.1145/3544999.3552320 

9. Daimler. (2017). Autonomous concept car smart vision EQ fortwo: Welcome to the future of car sharing. Retrieved from
[https://media.daimler.com/marsMediaSite/en/instance/ko/Autonomous-concept-car-smart-vision-EQ-fortwo-Welcome-to-the-future-of-car-sharing](https://media.daimler.com/marsMediaSite/en/instance/ko/Autonomous-concept-car-smart-vision-EQ-fortwo-Welcome-to-the-future-of-car-sharing).
xhtml?oid=29042725.

10. Jaguar Land Rover. (2018). The virtual eyes have it. Retrieved from [https://www.jaguarlandrover.com/2018/virtual-eyes-have-it](https://www.jaguarlandrover.com/2018/virtual-eyes-have-it).
