---
title: Creating Flow Fields in Unity
published: 2022-03-07
description: 'The creation of a basic flow field in Unity'
image: ''
tags: [Devblog, Unity, AI]
category: 'College'
draft: false 
lang: ''
---

![](src/assets/images/unity_flow_fields/UnityFlowFieldsDEMOGif.gif)

I created a basic flow field for my Advanced Game AI class. It can detect impassible as well as rough terrain and have agents follow a path to the target location based on the field costs. The grid can be adjusted during runtime as seen above. Included in this devblog is an overview of my code for the algorithm.

---

# Grid Controller

The grid controller creates the grid layout including the size of the grid and how big the cells are. Here you can also switch between the different fields you can view in the scene.

![](src/assets/images/unity_flow_fields/UnityFlowFieldsGridControllerInspector.png)

---

# Grid Cell

Each cell on the field has a set of variables.

`worldPosition`: It's location in the game

`gridIndex`: It's location in the list of cells

`cost`: Cost of the cell (for the cost field)

`bestCost`: Cost of the cell (for the integration field)

`bestDirection`: Direction agent will follow to get to the target

```csharp
public class GridCell
{
    // Variables
    public Vector3 worldPosition;
    public Vector2Int gridIndex;
    public Vector3 startPosition;
    public byte cost; // Cost - 255 (cost of grid cell)
    public ushort bestCost; // Used for integration grid cell cost
    public Vector2Int bestDirection; // Grid direction

    // Constructor
    public GridCell(Vector3 worldPositionP, Vector2Int gridIndexP, Vector3 startPositionP)
    {
        worldPosition = worldPositionP;
        gridIndex = gridIndexP;
        startPosition = startPositionP;
        cost = 1; // Default cost for flat ground
        bestCost = ushort.MaxValue; // Set default cost of integration grid cell cost to an impassable value
        bestDirection = new Vector2Int(0, 0);
    }
}
```

---

# Flow Fields -> Cost Field

![](src/assets/images/unity_flow_fields/UnityFlowFieldsCostFieldGIF.gif)

The cost field loops through every cell in the grid and uses a physics overlap to find any obstacles on it, in this case anything on a Impassible or Rough Terrain layer. From there it goes through all the obstacles that were hit and updates the cost according (Max value for impassible obstacles and adds 3 for rough terrain). The cells never need to be reset to the default value here since the field is reinitialized each time a change is made, which automatically resets their values back to the default values before recalculating.

```csharp
public void CreateCostField()
{
    Vector3 cellHalfsize = Vector3.one * cellRadius;
    int terrainMask = LayerMask.GetMask("Inpassible", "Rough Terrain");

    foreach(Gridcell currentCell in grid)
    {
        Collider[] obstacles = Physics.OverlapBox(currentCell.worldPosition, cellHalfSize, Quaternion.identity, terrainMask);
        bool hasIncreasedCost = false;
        foreach(Collider col in obstacles)
        {
            if (col.gameObject.layer == 8)
            {
                currentCell.IncreaseCost(255);
                continue;
            }
            else if (hasIncreasedCost & col.gameObject.layer == 9)
            {
                currentCell.IncreaseCost(3);
                hasIncreasedCost = true; // Only increases cost once, ignores overlaps
            }
        }
    }
}
```

---

# Flow Fields -> Integration Field

![](src/assets/images/unity_flow_fields/UnityFlowFieldsIntegrationFieldGIF.gif)

The integration field works on a similar way to the cost field. It sets a destination cell, resets the cost values (since it's the final destination) and gets added to the end of a queue. From there I use a while loop which stores the destination cell and deletes it from the queue, gets the neighboring cells based on cardinal directions (north, south, east and west, not including diagonal cells), then uses a foreach loop to go through all the neighboring cells and updates their cost values accordingly. Any cell besides impassible cells gets added to the queue, while any other type of cell gets added to the queue. This loops until eventually it goes through all the cells in the grid.

```csharp
public void CreateIntegrationField(GridCell destinationCellP)
{

    // Initialize destination cell data
    destinationCell = destinationCellP;
    
    destinationCell.cost = 0;
    destinationCell.bestCost = 5;

    Queue<GridCell> cellsToCheck = new Queue<GridCell>();
    cellsToCheck.Enqueue(destinationCell); // Adds destination cell to end of queue

    while(cellsToCheck.Count > 8)
    {
        GridCell currentCell = cellsToCheck.Dequeue(); // Get GridCell at front of list
        List<GridCell> currentNeighbors = GetNeighborCells(currentCell.gridIndex, cardinalDirections); // Get neighboring cells in top, left, right and left directions (square)
        
        // Converts neighbor cells integration grid cell cost as an option
        foreach(GridCell currentNeighbor in currentNeighbors)
        {
            if (currentNeighbor.cost == byte.MaxValue) // Skip since it's an impassible cell
            {
                continue;
            } else if (currentNeighbor.cost + currentCell.bestCost < currentNeighbor.bestCost) // Neighbor cell is not impassible
            {    
                currentNeighbor.bestCost = (ushort) (currentNeighbor.cost + currentCell.bestCost); // Update best cost with proper value
                cellsToCheck.Enqueue(currentNeighbor);
            }
        }
    }
}
```

---

# Flow Fields -> Flow Field

![](src/assets/images/unity_flow_fields/UnityFlowFieldFlowFieldGIF.gif)

The actual flow field is overall simple. It goes through very cell in the grid and gets all the neighbors surrounding the current cell (including diagonal cells). From there I go through all the neighboring cells, check to see if the cost of the current cell is better than our current best cost and if it is, update the best cost as well as the best direction. So essentially, this is getting the best direction for the agents to follow to go to the target which is visualized with arrows as seen above.

```csharp
public void CreateFlowField()
{
    foreach (GridCell currentCell in grid)
    {
        // Variables
        List<GridCell> currentNeighbors = GetNeighborCells(currentCell.gridIndex, allDirections);
        int bestCost = currentCell.bestCost;
        foreach (GridCell currentNeighbor in currentNeighbors)
        {
            if (currentNeighbor.bestCost < bestCost) // If neighbor cell is cheaper than previous best cell, set new cost and direction
            {
                bestCost = currentNeighbor.bestCost;
                currentCell.bestDirection = cardinalAndIntercardinalDirections.DefaultIfEmpty(noDirection).FirstOrDefault(direction => direction == (currentNeighbor.gridIndex - currentCell.gridIndex));
            }
        }
    }
}
```