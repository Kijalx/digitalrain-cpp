---
layout: post
title: Digital Rain Project in C++
tags: cpp coding project
categories: demo
---

This project was part of my C++ Programming module in my final year of college.

## Overview

This project involved a lot of algorithmic thinking to produce a Digital Rain effect that could also be seen in the movie [The Matrix](https://en.wikipedia.org/wiki/The_Matrix).

- **Vectors:** Utilizing vectors for storing and manipulating the raindrop position and speed.
- **Algorithms:** Essential for generating the random characters and controlling the positioning and flow of the rain.
- **Iterators:** Used to efficiently pass through the vector and update the state of each droplet.

![Digital Rain Effect](https://raw.githubusercontent.com/kijalx/digitalrain-cpp/main/docs/assets/images/DigitalRain.gif)

## Code Snippets

### .h file
```cpp
#ifndef DIGITAL_RAIN_H
#define DIGITAL_RAIN_H

#include <vector>
#include <random>

class DigitalRain {
public:
    DigitalRain(int w, int h);
    void init();
    void update();
    void display();
    void setWidth(int w);
    void setHeight(int h);
    void setColour(int c);

private:
    int width, height, effectMode, colour, speed;
    std::vector<std::vector<char>> matrix;
    std::vector<int> dropletPositions; // Current position (y-coordinate) of the head of each droplet
    std::vector<int> dropletLengths; // Length of each droplet
    std::vector<bool> dropletActive; // Indicates whether each droplet is active
    static std::default_random_engine engine;
    static std::uniform_int_distribution<int> uniform_dist;
};

#endif // DIGITAL_RAIN_H
```

### Constructor
```cpp
DigitalRain::DigitalRain(int w, int h) : width(w), height(h) {
    matrix.resize(height, std::vector<char>(width, ' '));
    dropletPositions.resize(width, -1);
    dropletLengths.resize(width);
    dropletActive.resize(width, false);
    for (int i = 0; i < width; ++i) {
        dropletLengths[i] = uniform_dist(engine) % (height / 4) + 1;
    }
}
```
- **Matrix:** This is resized for the basic outline of where the droplets are allowed to be positioned which allows me to never let them go out of bounds.
- **Droplet Position, Length, Active:** These are all resized to allow for the length and position to be changed. The activity status allows me to see if the droplet is going out of bounds and instantly kill it once it does.
- **Droplet Length Assignment:** I then assign random lengths to the droplets with a maximum height of a quarter of the screen.

### Setting Colour

```cpp
void DigitalRain::setColour(int c) {
    std::string colorCommand = "Color ";
    switch (c) {
    case 0: colorCommand += "0A"; break;
    case 1: colorCommand += "09"; break;
    case 2: colorCommand += "0B"; break;
    case 3: colorCommand += "0C"; break;
    case 4: colorCommand += "0D"; break;
    case 5: colorCommand += "0E"; break;
    case 6: colorCommand += "08"; break;
    default: colorCommand += "07"; break;
    }
    system(colorCommand.c_str());
}
```
**Setting Colour:** Here I have a switch statement with whatever the user chooses the string will execute Color + choice. This allows the user to change the colour according to their preference.

### Initialisation
```cpp
void DigitalRain::init() {
    std::cout << "Enter colour code (0.Green, 1.Blue, 2.Aqua, 3.Red, 4.Purple, 5.Yellow, 6.Gray, 7.White): ";
    std::cin >> colour;

    setColour(colour);

    std::cout << "Enter speed (higher numbers are slower, e.g., 50 is fast, 200 is slow): ";
    std::cin >> speed;

    for (int j = 0; j < width; ++j) {
        dropletPositions[j] = uniform_dist(engine) % height;
        dropletActive[j] = (uniform_dist(engine) % 2) == 0;
    }

    CONSOLE_CURSOR_INFO info;
    info.dwSize = 100;
    info.bVisible = FALSE;
    SetConsoleCursorInfo(console, &info);

    while (true) {
        display();
        update();
        Sleep(speed);
    }
}
```
1. **User Input:** User inputs their preferred colour and speed which is then set inside the function and in setColour
2. **Initialising Droplet Positions and activity:** Here im setting positions in according to the width and height of the matrix the position height is random where the droplets start so they dont all start at the top, Im also setting all these droplets to be active.
3. **Console Properties:** Im setting the cursor of the console to be invisible.
4. **Animation Loop:** I am calling my loop to start the droplet rainfall.

### Updating
```cpp
void DigitalRain::update() {
    gotoXY(0, 0);
    for (int j = 0; j < width; ++j) {
        if (dropletActive[j]) {
            int startPos = dropletPositions[j] - dropletLengths[j] + 1;
            for (int k = startPos; k <= dropletPositions[j]; ++k) {
                if (k >= 0 && k < height) {
                    matrix[k][j] = ' ';
                }
            }
            dropletPositions[j]++;
            if (dropletPositions[j] - dropletLengths[j] >= height) {
                dropletPositions[j] = -1;
                dropletActive[j] = false;
            }
            else {
                startPos = dropletPositions[j] - dropletLengths[j] + 1;
                for (int k = startPos; k <= dropletPositions[j]; ++k) {
                    if (k >= 0 && k < height) {
                        matrix[k][j] = static_cast<char>(uniform_dist(engine));
                    }
                }
            }
        }
        if (!dropletActive[j] && (uniform_dist(engine) % 100) < 10) {
            dropletPositions[j] = 0;
            dropletLengths[j] = uniform_dist(engine) % (height / 4) + 1;
            dropletActive[j] = true;
        }
    }
}
```
1. **Cursor Position:** Each time this loop gets called i want my cursor to reset.
2. **Updating Droplets:**
   - This part of the code updates the status and position of each droplet within the matrix.
     - A. **Activity Droplet Movement:**
       - For each column, the code checks if a droplet is active. if it is, the code clears the droplets current trail from the console and replaces the characters with its previous position with spaces.
       - The droplets vertical position is incremented, which gives the feel of the droplet moving down the screen.
       - If the droplet reaches the bottom of the screen, It is deactivated and moved to the top. If it does not reach the bottom, it is updated with random characters to keep the rain effect.
     - B. **Bounds Checking and Reactivation:**
       - After the droplet moves, the code checks if it has moved past the bottom of the display area if so the droplet gets reset to a position of -1 so it is not visible and then is marked as active which then it gets incremented as if it was falling down again.
       - However if the droplet is not active there is a chance it will be activated each time the update function runs. This is a randomness feature i implemented so not each column has a raindrop. This is to to represent the randomness of a new droplet beginning its decent.
     - C. **Trail Management:**
       - Active droplets leave behind a predetermined trail as they fall. This is shown by characters above the droplets position. As the the droplets position increases the trail gets updated to be the same no matter where the droplet is at the trail will remain the same.
     - D. **Reactivation:**
       - Inactive droplets remain inactive until the randomness feature of the activation of droplets occurs. The droplet remains inactive ensuring the rainfall is not uniform but instead a varied and natural.
3. **Preformance Consideration:**
   - The way this project is implemented wasnt a preformance based project but a more aestetic and pleasing to look at. Therefore it is intensive for the case of larger matrices. It is designed to efficiently manage the digital rain's appearance by only updating necessary parts of the screen and droplets. This was also a test in algorithmic thinking as setting cursor positions would of been more managable and less intensive then matrices.
  
### Displaying
```cpp
void DigitalRain::display() {
    for (const auto& row : matrix) {
        for (const auto& ch : row) {
            std::cout << (ch);
        }
        std::cout << std::endl;
    }
}
```
1. **Iterating through the matrix:**
   - This method begins by looping through each row of the matrix which is a 2d vector as mentioned in the .h file.
   - Inside the loop then each character 'ch' in the current row is displayed by a nested loop.
2. **Displaying Characters:**
   - This nested for loop prints each individual character of the matrix.
   - Then after one loop of the row is done it then goes onto the next line and repeats.
