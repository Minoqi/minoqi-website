---
title: "Advent of Code 2024"
published: 2024-12-01
description: 'Advent of Code has officially started, and here are my solutions for it (incomplete)'
tags: [C++, Tutorial]
category: ''
draft: true 
---

A new christmas season means a new `Advent of Code`! This was my first year actually participating in real time, and I thought I'd post a tutorial post on my solution for each day. I'll be updating through the event as I finish more and more days!

# Contents
[Day 1](#day-1)

---

# Day 1

## Part 1
**Task:** You have a list with a number on the left and a number on the right. You need to sort each side in order from smallest to largest and then subtract both sides. Finally, add up all of the values.

I first noticed that I would need to be able to handle this data across multiple functions, so for ease of use I created a struct to hold the 3 pieces of data I needed:

1. The left side values
2. The right side values
3. The differences

```cpp
struct FileData {
    vector<int> leftSide;
    vector<int> rightSide;
    vector<int> distance;
};
```

The next step was to import in all the data. Since I knew each line had a number, space and number, I was able to use `sstream` to take in each part of the file and store them approrpiately. So the first `sstream` was the left and the second would be for the right. I took this chance to convert the `string` to an `int` as well.

```cpp
void storeFileValues(fstream& _file, FileData& _data) {
    // Variables
    string currentLine;
    string currentNum;
    bool leftSide = true;

    // Read in and store every line
    while(getline(_file, currentLine)) {
        istringstream stream(currentLine);
        while (stream >> currentNum) {
            if (leftSide) {
                _data.leftSide.push_back(stoi(currentNum));
                currentNum = "";
                leftSide = false;
            } else {
                _data.rightSide.push_back(stoi(currentNum));
            }
        }

        // Reset for next line
        leftSide = true;
        currentLine = "";
        currentNum = "";
    }
}
```

With both sides stored, I almost made my own sort before realizing I can quickly sort it using the `sort()` method from the `algorithm` library.

```cpp
void sortSides(FileData& _data) {
    sort(_data.leftSide.begin(), _data.leftSide.end());
    sort(_data.rightSide.begin(), _data.rightSide.end());
}
```

Now with everything sorted it was just a matter of subtracting each row.

```cpp
void calculateDistances(FileData& _data) {
    for (int i = 0; i < _data.leftSide.size(); i++) {
        _data.distance.push_back(abs(_data.leftSide[i] - _data.rightSide[i]));
    }
}
```

And last but not least, adding up all those values together.

```cpp
int calculateTotalDistance(FileData& _data) {
    int totalDistance = 0;

    for (int distanceValue : _data.distance) {
        totalDistance += distanceValue;
    }

    return totalDistance;
}
```

---

## Part 2
**Task:** For part 2, we simply need to count up how many times each number on the left side appears on the right (duplicates numbers on the left count again). Then we just add it all together.

For this I made a single function that does two things:

1. First it goes through and uses the `count()` function to determine how many of the left side number apepars on the right and add it to a vector
2. Then I go through and add up all the points from that vector to get the final score

```cpp
int calculateSimilarityScore(FileData& _data) {
    // Variabels
    vector<int> similarityPoints;
    int totalSimilarityPoints = 0;

    // Gather how many times a score appears
    for (int valueToCheck : _data.leftSide) {
        similarityPoints.push_back(valueToCheck * count(_data.rightSide.begin(), _data.rightSide.end(), valueToCheck));
    }

    // Count up all the similarity points together
    for (int similarityPoint : similarityPoints) {
        totalSimilarityPoints += similarityPoint;
    }

    return totalSimilarityPoints;
}
```

---

# Full Solution
```cpp
#include <iostream>
#include <fstream>
#include <string>
#include <sstream>
#include <vector>
#include <algorithm>
#include <cmath>

using namespace std;

struct FileData {
    vector<int> leftSide;
    vector<int> rightSide;
    vector<int> distance;
};

#pragma region Part01
void storeFileValues(fstream& _file, FileData& _data) {
    // Variables
    string currentLine;
    string currentNum;
    bool leftSide = true;

    // Read in and store every line
    while(getline(_file, currentLine)) {
        istringstream stream(currentLine);
        while (stream >> currentNum) {
            if (leftSide) {
                _data.leftSide.push_back(stoi(currentNum));
                currentNum = "";
                leftSide = false;
            } else {
                _data.rightSide.push_back(stoi(currentNum));
            }
        }

        // Reset for next line
        leftSide = true;
        currentLine = "";
        currentNum = "";
    }
}

void sortSides(FileData& _data) {
    sort(_data.leftSide.begin(), _data.leftSide.end());
    sort(_data.rightSide.begin(), _data.rightSide.end());
}

void calculateDistances(FileData& _data) {
    for (int i = 0; i < _data.leftSide.size(); i++) {
        _data.distance.push_back(abs(_data.leftSide[i] - _data.rightSide[i]));
    }
}

int calculateTotalDistance(FileData& _data) {
    int totalDistance = 0;

    for (int distanceValue : _data.distance) {
        totalDistance += distanceValue;
    }

    return totalDistance;
}
#pragma endregion Part01

#pragma region Part02
int calculateSimilarityScore(FileData& _data) {
    // Variabels
    vector<int> similarityPoints;
    int totalSimilarityPoints = 0;

    // Gather how many times a score appears
    for (int valueToCheck : _data.leftSide) {
        similarityPoints.push_back(valueToCheck * count(_data.rightSide.begin(), _data.rightSide.end(), valueToCheck));
    }

    // Count up all the similarity points together
    for (int similarityPoint : similarityPoints) {
        totalSimilarityPoints += similarityPoint;
    }

    return totalSimilarityPoints;
}
#pragma endregion Part02

int main() {
    fstream inputFile("adventOfCodeDay001P1.txt");

    if (inputFile.is_open()) {
        FileData data;
        storeFileValues(inputFile, data);
        sortSides(data);
        calculateDistances(data);
        cout << "Total Distance: " << calculateTotalDistance(data) << endl;
        cout << "Total Similarity Points: " << calculateSimilarityScore(data) << endl;
    } else {
        cout << "ERROR: Opening of file failed" << endl;
    }

    return 0;
}
```