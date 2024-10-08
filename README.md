# Week 5: Explore about Photon #
This week I mainly explored about photon.

I encountered a bunch of issues in setting up the photon with my laptop, basiclly problems like my windows device cannot recognise my photon, my serial port does not appears in the UI after I plugged in my photon. These problems are common when setting up a new microprocesser with PC and I solved these problems with the help of Roopa.

I also tried about 3 tutorials, here are some photos:
1. The flash rate of the LED with change with the button
2. The fsr sensor
3. The publish to cloud function
<img width="80%" alt="Youtube screenshot" src="assets/wk5-1.jpg">
<img width="80%" alt="Youtube screenshot" src="assets/wk5-2.jpg">
<img width="80%" alt="Youtube screenshot" src="assets/wk5-3.jpg">
<img width="80%" alt="Youtube screenshot" src="assets/wk5-4.jpg">


# Week 4: A network map of a pet camera system #

This week we got started with project 2.

Here is a network map about a pet camera which includes the interaction between the App and the camera.
<img width="80%" alt="Youtube screenshot" src="assets/IMG_0222.HEIC">



# Week 3: Create my own phone stand with Grasshopper!! #
### Week of 09/12/2024

## Summary

This week I used Grasshopper to created my own phone stand. And I used Prusa printer to printed it out! HORAYYYYY!

## Problems I found for the original design
The stand is too shallow, causing my phone to slip back and forth, but making the saddle deeper is not an option, as it would obstruct the phone’s screen. And the stand does not provide adequate support for my iPad.

## Working process
<img width="80%" alt="Youtube screenshot" src="assets/week3.png">

This is what I have made for for the new stand. 
1.	I added a support column on the top of the stand to support larger devices like ipad.
2.	I used a solid base instead of a shallow base to lower the center of the gravity.
3.	I added a cut through in front, so that less view would be blocked.
---

# Week 2: Modelling a phone stand with Grasshopper #
### Week of 09/12/2024

## Summary

This week I explored on how to use Rhino and Grasshopper to edit models.


## A flow diagram about how to make a phone stand.

This flow diagram is a demonstration about how to make a simple phone stand with basic shapes like sphere and rectangle.

<img width="80%" alt="Youtube screenshot" src="assets/wk2flowchart.png">


## Manipulate parameters and bake some intermediate states

Upon opening the file, I firstly tried to toggle some buttons to see if there is any change to the model. \
With the display phone button turned off, the phone model disappeared. From this button, I understood the meaning of  **"Filter"**. \
<img width="400" alt="Youtube screenshot" src="assets/wk2_1.png">

Then I baked some intermediate state models to get some visualizations about what going on in these steps.\
For example, I baked the following node and knew the substraction result of the two spheres.
<img width="400" alt="Youtube screenshot" src="assets/wk2_2.png">



## Generating my own phone stand with cylinders

Then I generated my own model with cylinders. 

To do this, firstly, I created two cylinders with two scalar numbers as radius and height, as well as y-axis as the direction. Then, I substracted these two cylinders with a box under xy plane to ensure that the part under xy plane is eliminated. After that, I substracted the two cylinders to get a shell. In the end, I substracted the phone with phone stand.

However, I found that I cannot get **a enclosed shape** for the phone stand.

<img width="400" alt="Youtube screenshot" src="assets/error1.png">

<img width="400" alt="Youtube screenshot" src="assets/error2.png">


Then, I tried to substract again with another box on the top of the cylinders and finally I got an enclosed shape.

<img width="400" alt="Youtube screenshot" src="assets/wk2_withcylinder.png">


This is my baked image.  :grinning:

<img width="400" alt="Youtube screenshot" src="assets/wk2_cy phone case.png">

---
Oh after Monday class, I knew that the cylinder can be enclosed with a "Cap" node.   :grinning:

---


# Week 1: Everthing starts here #
## Week of 09/05/2024

This week, I explored the github and rhino. I am total new to Rhino, so I watched some Youtube tutorials.

The following series is very useful for me as it explained from the very beginning and in a suitable pace.

<img width="400" alt="Youtube screenshot" src="assets/screenshot.png">

---

I also explored how to add emojis in github

https://github.com/ikatyang/emoji-cheat-sheet/blob/master/README.md#smileys--emotion

:grinning:
