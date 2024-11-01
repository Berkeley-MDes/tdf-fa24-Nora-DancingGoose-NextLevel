# Week 9: Start of Project 3!


---


# Week 8：Rescue my photon from SOS mode
This week I'm continue to work on my photon project.
My photon suddenly went into SOS mode without any change of code.
After debugging, I found the issue lies on:
1. the soldering of OLED with photon is very unstable, it causes my photon to complaining about no OLED detected. To make it more stable, use I2C extend board.
2. there is one line of code when no OLED is detected that will cause the photon to enter SOS mode.
```
for (;;); // Don't proceed, loop forever
```

The complete code is here
```
#include "application.h"
#include "HttpClient.h"
#include "Particle.h"
#include "Adafruit_SSD1306.h"
#include "Adafruit_GFX.h"
#include "splash.h" //this is our custom header containing the splash screen bitmap
#include <String.h> 

SYSTEM_THREAD(ENABLED);

#define SCREEN_WIDTH 128 // OLED display width, in pixels
#define SCREEN_HEIGHT 64 // OLED display height, in pixels
#define SCREEN_ADDRESS 0x3D // OLED display address (for the 128x64)
#define POT_PIN A0 // Potentiometer is connected to A0

// Instantiate SSD1306 driver display object via I2C interface; note that no reset is used
Adafruit_SSD1306 disp(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, -1);

void http_connect(void);
void draw_splash(void); //our splash screen function
void draw_potval(String x); // display function
void adviceTypeHandler(const char *event, const char *data);
int potval = 0;
int adviceType = 1;


HttpClient http;
http_request_t request;
http_response_t response;
http_header_t headers[] = {
   { "Content-Type", "application/json" },
   { "Accept", "application/json" },
   { NULL, NULL } // terminator for headers
};

const char* serverName = "10.41.238.166" ; // Replace with your server IP or URL "10.41.236.23"
int buttonPin = D7; // The button pin
bool buttonPressed = false;
const char* answer = " ";


void setup(){
  Serial.begin(9600);
  pinMode(buttonPin, INPUT_PULLUP); // Button setup
  delay(8); 
  // if initialization fails print failure to Serial, and enter an infinite loop
  bool test_access = disp.begin(SSD1306_SWITCHCAPVCC, SCREEN_ADDRESS);


  if(!test_access){
    Serial.println(F("SSD1306 allocation failed"));
  }else{
    Serial.println("SSD1306 allocation success");
    draw_splash();
    delay(2000);
  }

  

}

void loop(){
  //
  if (digitalRead(buttonPin) == LOW) {
    if (!buttonPressed) {
      buttonPressed = true;
      Serial.println("button pressed");
      adviceType = 2; // fill subscribe
      http_connect();
    }
  } 
  else {
      buttonPressed = false;
  }

  //add in communication code
  Particle.subscribe("adviceTypeUpdate", adviceTypeHandler, ALL_DEVICES);

  delay(100);
}

void adviceTypeHandler(const char *event, const char *data) {
    String adviceTypeReceived = data;  // Store received advice type
    Serial.println("Received Advice Type: " + adviceTypeReceived);  // Log it
}

void http_connect(void){

  if (WiFi.ready()){
    // Set the HTTP request URL
    request.hostname = serverName;
    request.port = 3000;
    request.path = "/generate?type="+String(adviceType);

    // Send the request
    http.get(request, response, headers);

    // Output the response to the serial monitor
    Serial.println("Response from GPT API:");

    if (response.status == 200) {
      Serial.println(response.body);
      draw_potval(response.body);

    } else {
      Serial.println("HTTP request failed.");
    }


  }else {
    Serial.println("WiFi not ready.");
  }
  
}

void draw_splash(void){
  disp.clearDisplay();
  disp.drawBitmap(0, 0, epd_pirate_small, SCREEN_WIDTH, SCREEN_HEIGHT, WHITE);
  disp.display();
}

void draw_potval(String x){
  disp.clearDisplay();
  disp.setTextSize(1);
  disp.setTextColor(WHITE);
  disp.setCursor(0,0);
  disp.printf(x);
  disp.display();
}

```

---

# Week 7：GPT API + Nodejs + photon
This week I'm working on the team project. We are making an answer book that generate random advice for the person who opening the book. 
I have rich experience about microprocessor, so I'm helping out my teammates on basic sensor problems.
As for myself, I have mainly discovered about how to use call chatgpt API via node js and display the output to OLED.

I firstly wrote a **server.js** code to setup a local server. This piece of code runs a local server at localhost:3000.
I used the **express** library to handle the app, and I used **https** library to handle request and response.
Then I wrote a json message about generating an advice, and feed it to chatgpt. 
Then the output from chatgpt should be stored in **responseData**
There is also an **if else** statement to handle any error and display the error in terminal.

```
const apiKey = 'MY API KEY';
const https = require('https');
const express = require('express');
const app = express();
const port = 3000;

app.get('/generate', (req, res) => {
  const prompt = "Generate a short advice";

  const data = JSON.stringify({
    model: "gpt-4o-mini",
    messages: [
      {
        role: "system",
        content: "You are an assistant that speaks random advice abour career. Be mysterious"
      },
      {
        role: "user",
        content: [
          {
            type: "text",
            text: prompt
          }
        ]
      }
    ],
    max_tokens: 20
  });

  const options = {
    hostname: 'api.openai.com',
    path: '/v1/chat/completions',
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${apiKey}`,
      'Content-Type': 'application/json',
      'Content-Length': data.length
    }
  };

  const request = https.request(options, (response) => {
    let responseData = '';

    response.on('data', (chunk) => {
      responseData += chunk;
    });

    response.on('end', () => {
      const result = JSON.parse(responseData);
      res.send(result.choices[0].message.content.trim());
      //res.send("Test response from server");
    });
  });

  request.on('error', (error) => {
    console.error(error);
    res.status(500).send('Error generating text');
  });

  request.write(data);
  request.end();
});

app.listen(port, () => {
  console.log(`Server listening at http://localhost:${port}`);
});
```

Then I wrote a cpp code for my photon. When I press the button, the photon would prompt my message to chatgpt. And the output from gpt would be displayed on OLED.
```
#include "application.h"
#include "../lib/HttpClient/src/HttpClient/HttpClient.h"
#include "Particle.h"
#include "Adafruit_SSD1306.h"
#include "Adafruit_GFX.h"
#include "splash.h"

SYSTEM_THREAD(ENABLED);

#define SCREEN_WIDTH 128 // OLED display width, in pixels
#define SCREEN_HEIGHT 64 // OLED display height, in pixels
#define SCREEN_ADDRESS 0x3D // OLED display address (for the 128x64)

Adafruit_SSD1306 disp(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, -1);

void draw_splash(void); //our splash screen function
void draw_potval(void); //our potentiometer value display function
void draw_bitmap(const unsigned char* bitmap, int x, int y, int w, int h, int color);
int potval = 0;

HttpClient http;
http_request_t request;
http_response_t response;
http_header_t headers[] = {
    { "Content-Type", "application/json" },
    { "Accept", "application/json" },
    { NULL, NULL } // terminator for headers
};

const char* serverName = "10.41.236.23"; // Replace with your server IP or URL
int buttonPin = D7; // The button pin
bool buttonPressed = false;

void setup() {
  Serial.begin(9600);
  pinMode(buttonPin, INPUT_PULLUP); // Button setup

  //oled setup
  delay(8);
  bool test_access = disp.begin(SSD1306_SWITCHCAPVCC, SCREEN_ADDRESS);
  if(!test_access){
    Serial.println(F("SSD1306 allocation failed"));
    for (;;); // Don't proceed, loop forever
  }else{
    Serial.println("SSD1306 allocation success");
    draw_splash();
    delay(2000);
  }
}

void loop() {
  if (digitalRead(buttonPin) == LOW) {
    if (!buttonPressed) {
      buttonPressed = true;

      // Set the HTTP request URL
      request.hostname = serverName;
      request.port = 3000;
      request.path = "/generate";

      // Send the request
      http.get(request, response, headers);

      // Output the response to the serial monitor
      Serial.println("Response from GPT API:");
      Serial.println(response.status);
      Serial.println(response.body);

      //display this on OLED screen
      draw_potval();
      

    }
  } 
  else {
      buttonPressed = false;
  }
  delay(100);
}



void draw_splash(void){
  disp.clearDisplay();
  disp.drawBitmap(0, 0, epd_pirate_small, SCREEN_WIDTH, SCREEN_HEIGHT, WHITE);
  disp.display();
}

void draw_potval(){
  disp.clearDisplay();
  disp.setTextSize(1);
  disp.setTextColor(WHITE);
  disp.setCursor(0,0);
  disp.printf(response.body);
  disp.display();
}
```

---

# Week 6: Use Photon with accelerometer and gyroscope

This week I have explored to use photon + accelerometer and gyroscope

1. The accelerometer's values are quite big, so I tried to use constrain() function to remove the extreme values, and use map() function to remap the value to a smaller range
<img width="50%" alt="Youtube screenshot" src="assets/6-4.png">
2. I also tried to smooth out the accelerometer values.
   Firstly, I defined 3 arraies with a length of 10 to contain accelerometer outputs.
<img width="50%" alt="Youtube screenshot" src="assets/6-1.png">
   Secondly, I worte a function to calculate average value of number in an array.
<img width="50%" alt="Youtube screenshot" src="assets/6-2.png">
   Thirdly, I update the numbers in the array in the loop()
   Finally, I used my function calculateAverage() to calculate the smooth values.
<img width="50%" alt="Youtube screenshot" src="assets/6-3.png">

---

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

---

# Week 4: A network map of a pet camera system #

This week we got started with project 2.

Here is a network map about a pet camera which includes the interaction between the App and the camera.
<img width="80%" alt="Youtube screenshot" src="assets/IMG_0222.HEIC">

---

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
