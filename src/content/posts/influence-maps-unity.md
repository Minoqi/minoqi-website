---
title: Creating Influence Maps in Unity
published: 2022-04-11
description: 'The creation of a basic influence map in Unity'
image: ''
tags: [Devblog, AI, Unity]
category: 'College'
draft: false 
lang: ''
---

![](src/assets/images/unity_influence_map/unityInfluencemapDEMO.gif)

I created a basic example of an influence map for my Advanced Game AI class. For my example, I made it so the user could place down and control a ship. They could also click on a cell which would then generate an enemy ship which has a "damage zone". The surrounding cells would then get a value based on their distance from the center cell. When the ship moved into a new cell it took damage equal to whatever the value of that cell was. So in other words, it's an example of using influence maps for AoE damage.

---

# Grid Generator

Firstly is the generation of the grid.

`width`: Width of the grid

`height`: Height of the grid

`gridArray`: Creates an array of the grid that stores the value for each grid cell

`cellSize`: Size of each cell in the grid

`textHolder`: Object where all the UI text gets placed under (aka the parent)

`gridObject`: GameObject in charge of holding all the grid data

```csharp
public Grid(int pWidth, int pHeight, float pCellSize, GameObject textHolder, GameObject gridObject)
{
    // Store variables
    width = pWidth;
    height = pHeight;
    gridArray = new int[width, height];
    cellSize = pCellSize;
    debugTextArray = new TextMesh[width, height];

    // Debug
    for (int x = 0; x < gridArray.GetLength(0); x++)
    {
        for (int y = 0; y < gridArray.GetLength(1); y++)
        {
            Debug.DrawLine(GetWorldPosition((int)gridObject.transform.position.x + x, (int)gridObject.transform.position.y + y), GetWorldPosition((int)gridObject.transform.position.x + x, (int)gridObject.transform.position.y + y + 1), Color.white, 100f);
            Debug.DrawLine(GetWorldPosition((int)gridObject.transform.position.x + x, (int)gridObject.transform.position.y + y), GetWorldPosition((int)gridObject.transform.position.x + x + 1, (int)gridObject.transform.position.y + y), Color.white, 100f);
            gridArray[x, y] = defaultValue;
            debugTextArray[x, y] = CreateWorldText(gridArray[x, y].ToString(), GetWorldPosition((int)gridObject.transform.position.x + x, (int)gridObject.transform.position.y + y) + (new Vector3(cellSize, cellSize) * 8.5f), 5, Color.white, TextAnchor.MiddleCenter, TextAlignment.Center, textHolder.transform);
        }
    }

    Debug.DrawLine(GetWorldPosition((int)gridObject.transform.position.x, height), GetWorldPosition(width, height), Color.white, 160);
    Debug.DrawLine(GetWorldPosition(width, (int)gridObject.transform.position.y), GetWorldPosition(width, height), Color.white, 100f);
}
```

The `gridObject` has it's own script on it which is essentially the game manager. It generates the grid and is used to allow other scripts to access the grid data. It has a few extra values.

`startValue`: The initial value given to the starting cell

`affectedZone`: How wide the affected range from the start position is

`affectedValue`: How much each cell gets affected by

`enemyShipPrefab`: Prefab of the enemy ship 

`shipPrefab`: Prefab of the ship the player controls 

`shipDown`: Tracks whether or not there's a ship currently in the world, there can only be one ship at once so when it's true the player won't be able to place any more ships

![](src/assets/images/unity_influence_map/UniyuInfluenceMapGridGenerateInsepctor.png)

---

# Influence Map Calculations

Right clicking on the grid will place a ship at the corresponding cell, while left clicking will place an enemy ship with a "damage zone" starting from the cell the player clicked on.

```csharp
private void Update()
{
    if (Input.GetMouseButtonDown(0))
    {
        // Variables
        Vector3 mouseClickWorldPosition = Camera.main.ScreenToWorldPoint(Input.mousePosition);
        Vector2Int mouseClickArrayPosition;

        mouseClickArrayPosition = grid.SetCellValue(mouseClickWorldPosition, startValue);

        grid.CalculateNeighboringCells(mouseClickArrayPosition, affectedZone, affectedValue);

        // Place Enemy Ship
        mouseClickArrayPosition = grid.GetXYPosition(mouseClickWorldPosition);
        Instantiate(enemyShipPrefab, grid.GetWorldPosition(mouseClickArrayPosition.x, mouseClickArrayPosition.y), Quaternion.identity);
    }
    else if (Input.GetMouseButtonDown(1) && !shipDown)
    {
        // Variables
        Vector3 mouseClickWorldPosition = Camera.main.ScreenToWorldPoint(Input.mousePosition);
        Vector2Int mouseClickArrayPosition;

        // Place ship
        mouseClickArrayPosition = grid.GetXYPosition(mouseClickWorldPosition);
        Instantiate(shipPrefab, grid.GetWorldPosition(mouseClickArrayPosition.x, mouseClickArrayPosition.y), Quaternion.identity);
        shipDown = true;
    }
}
```

![](src/assets/images/unity_influence_map/UnityInfluenceMapsMidDemoGif.gif)

The "damage zone" is calculated through the `CalculateNeighborCells()` function. The cell the player clicked on is stored and from there all the math is done in a for loop. It takes in how big the affected are should be, starts at X and then goes through all the Y values before adding to the X and repeating the process until all values have been accounted for. There's a few if statements inside the for loops. The first one checks if the cell is out of bounds of the array, in which case it'll get skipped. The following else if checks if the cell it's on is the starting (center) cell, in which case the value won't change from the default starting value. 

The last else statement has another if/else statement. Firstly it checks if the value for the cell doesn't equal zero, in which case when doing the math it'll add the values rather than subtract since in this case two AoE zones are overlapping means there'd be more damage done to the player in that cell. 

```csharp
public void CalculateNeighboringCells(Vector2Int centerCell, int affectedZone, int affectedValue)
{
    // Variables
    Vector2Int currentLoc = new Vector2Int(0, 0);

    // Calculate neighbors
    for (currentLoc.x = centerCell.x - affectedZone; currentLoc.x < centerCell.x + affectedZone + 1; currentLoc.x++)
    {
        for (currentLoc.y = centerCell.y - affectedZone; currentLoc.y < centerCell.y + affectedZone + 1; currentLoc.y++)
        {
            if (currentLoc.x < 0 || currentLoc.x >= gridArray.GetLength(0) || currentLoc.y < 0 || currentLoc.y >= gridArray.GetLength(1)) // Out of bounds
            {
                Debug.Log("Skip: Either out of bounds OR at center cell");
            }
            else if (currentLoc == centerCell) // Center cell selected
            {
                gridArray[currentLoc.x, currentLoc.y] = gridArray[centerCell.x, centerCell.y];
                debugTextArray[currentLoc.x, currentLoc.y].text = gridArray[currentLoc.x, currentLoc.y].ToString();
            }
            else // Adjust values
            {
                if (gridArray[currentLoc.x, currentLoc.y] != 0)
                {
                    gridArray[currentLoc.x, currentLoc.y] = gridArray[currentLoc.x, currentLoc.y] + (affectedValue * Mathf.FloorToInt((currentLoc - centerCell).magnitude));
                }
                else
                {
                    gridArray[currentLoc.x, currentLoc.y] = gridArray[centerCell.x, centerCell.y] - (affectedValue * Mathf.FloorToInt((currentLoc - centerCell).magnitude));
                }
                debugTextArray[currentLoc.x, currentLoc.y].text = gridArray[currentLoc.x, currentLoc.y].ToString();
            }
        }
    }
}
```

![](src/assets/images/unity_influence_map/UnityInfluenceMapShipDmgDebug.png)