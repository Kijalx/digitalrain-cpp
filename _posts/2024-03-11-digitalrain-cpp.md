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
