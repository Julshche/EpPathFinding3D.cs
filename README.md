EpPathFinding3D.cs
================
#### A 3D jump point search algorithm for cube based games in C# ####

Introduction
------------
 Previous project, [EpPathFinding.cs](https://github.com/juhgiyo/EpPathFinding.cs), was started after I was inspired by [PathFinding.js by Xueqiao Xu](https://github.com/qiao/PathFinding.js) and [the article by D. Harabor](http://harablog.wordpress.com/2011/09/07/jump-point-search/).
It comes along with a demo to show how the agorithm execute as similar to Xueqiao Xu's [Online Demo](http://qiao.github.com/PathFinding.js/visual).

Then I also extended 2D jump point search algorithm [EpPathFinding.cs](https://github.com/juhgiyo/EpPathFinding.cs) into 3D for cube based 3D environment.

Unity Integration Guide
------------

Copy `EpPathFinding3D.cs\PathFinder` folder into your `Unity Project's Assets` folder. Then within the script file, you want to use the `EpPathFinding3D.cs`, just add `using EpPathFinding3D.cs;` namespace at the top of the file, and use it as the guide below.


(If you have a problem when compiling, please refer to [Unity Forum](http://forum.unity3d.com/threads/monodevelop-problems-with-default-parameters.67867/#post-898994)) 


Also `EpPathFinding3D.cs` depends on [C5](https://github.com/sestoft/C5).

Pre-compiled C5.dll for Unity is included in `EpPathFinding3D.cs\PathFinder\UnityC5` folder.

(Please refer to [C5 on Unity3D](https://github.com/sestoft/C5#c5-on-unity3d), if you have any dependency issue with C5.)


Nuget Package
------------
[Nuget Package](https://www.nuget.org/packages/EpPathFinding3D.cs/)


Basic Usage
------------
The usage and the demo has been made very similar to [PathFinding.js](https://github.com/qiao/PathFinding.js) for ease of usage.  

You first need to build a grid-map. (For example: width 64, length 32 and height 24): 


```c#
BaseGrid searchGrid = new StaticGrid(64, 32, 24);
```


By default, every nodes in the grid will NOT allow to be walked through. To set whether a node at a given coordinate is walkable or not, use the `SetWalkableAt` function.

For example, in order to set the node at (10 , 20, 30) to be walkable, where 10 is the x coordinate (from left to right), 20 is the y coordinate (from top to bottom) and 30 is the z coordinate (from upper to lower): 


```c#
searchGrid.SetWalkableAt(10, 20, 30, true);
 
// OR
 
searchGrid.SetWalkableAt(new GridPos(10,20,30),true);  
```


You may also use in a 3-d array while instantiating the `StaticGrid` class. It will initiate all the nodes in the grid with the walkability indicated by the array. (`true` for walkable otherwise not walkable): 


```c#
bool [][][] movableMatrix = new bool [width][][];
for(int widthTrav=0; widthTrav< 64; widthTrav++)
{
   movableMatrix[widthTrav]=new bool[length][];
   for(int lengthTrav=0; lengthTrav < 32; lengthTrav++)
   {
      movableMatrix[widthTrav][lengthTrav]=new bool[height];
      for(int heightTrav=0; heightTrav < 24;  heightTrav++)
      { 
         movableMatrix[widthTrav][lengthTrav][heightTrav]=true; 
      }  
   }
   
}

Grid searchGrid = new StaticGrid(64, 32, 24, movableMatrix);
```


In order to search the route from (10,10, 5) to (20,10, 6), you need to create `JumpPointParam` class with grid and start/end positions. (Note: both the start point and end point must be walkable): 


```c#
GridPos startPos=new GridPos(10,10,5); 
GridPos endPos = new GridPos(20,10,6);  
JumpPointParam jpParam = new JumpPointParam(searchGrid,startPos,endPos ); 
```


You can also set/change the start and end positions later. (However the start and end positions must be set before the actual search): 


```c#
JumpPoinParam jpParam = new JumpPointParam(searchGrid);
jpParam.Reset(new GridPos(10,10,5), new GridPos(20,10,6)); 
```


To find a path, simply run `FindPath` function with `JumpPointParam` object created above: 


```c#
List<GridPos> resultPathList = JumpPointFinder.FindPath(jpParam); 
```


`JumpPointParam` class can be used as much as you want with different start/end positions unlike [PathFinding.js](https://github.com/qiao/PathFinding.js): 


```c#
jpParam.Reset(new GridPos(15,15,10), new GridPos(20,15,5));
resultPathList = JumpPointFinder.FindPath(jpParam); 
```


Advanced Usage
------------

#### Find the path even the end node is unwalkable  ####
When instantiating the `JumpPointParam`, you may pass in additional parameter to make the search able to find path even the end node is unwalkable grid:   
```
Note that it automatically sets to ALLOW as a default when the parameter is not specified.
```

```c#
JumpPointParam jpParam = new JumpPointParam(searchGrid,EndNodeUnWalkableTreatment.ALLOW);
```


If `iAllowEndNodeUnWalkable` is DISALLOW the FindPath will return the empty path if the end node is unwalkable: 


```c#
JumpPointParam jpParam = new JumpPointParam(searchGrid,EndNodeUnWalkableTreatment.DISALLOW);   
```


#### DiagonalMovement.Always (Cross Adjacent Point) ####

In order to make search able to walk diagonally across corner of two diagonal unwalkable nodes:   


```c#
JumpPointParam jpParam = new JumpPointParam(searchGrid,EndNodeUnWalkableTreatment.ALLOW,DiagonalMovement.Always);   
```


#### DiagonalMovement.IfAtLeastOneWalkable (Cross Corner) ####
When instantiating the `JumpPointParam`, to make the search able to walk diagonally when one of the side is unwalkable grid:  

```c#
JumpPointParam jpParam = new JumpPointParam(searchGrid,EndNodeUnWalkableTreatment.ALLOW,DiagonalMovement.IfAtLeastOneWalkable);   
```


#### DiagonalMovement.OnlyWhenNoObstacles ####

To make it unable to walk diagonally when one of the side is unwalkable and rather go around the corner: 


```c#
JumpPointParam jpParam = new JumpPointParam(searchGrid,EndNodeUnWalkableTreatment.ALLOW,DiagonalMovement.OnlyWhenNoObstacles);   
```


#### DiagonalMovement.Never ####

To make it unable to walk diagonally: 


```c#
// Special thanks to Nil Amar for the idea!
JumpPointParam jpParam = new JumpPointParam(searchGrid,EndNodeUnWalkableTreatment.ALLOW,DiagonalMovement.Never);   
```


#### Heuristic Functions ####
The predefined heuristics are `Heuristic.EUCLIDEAN` (default), `Heuristic.MANHATTAN`, and `Heuristic.CHEBYSHEV`.   

To use the `MANHATTAN` heuristic:


```c#
JumpPointParam jpParam = new JumpPointParam(searchGrid,EndNodeUnWalkableTreatment.ALLOW, DiagonalMovement.Always, Heuristic.MANHATTAN); 
```


You can always change the heuristics later with `SetHeuristic` function: 


```c#
jpParam.SetHeuristic(Heuristic.MANHATTAN);
```


#### Dynamic Grid ####

For my grid-based game, I had much less walkable grid nodes than un-walkable grid nodes. So above `StaticGrid` was wasting too much memory to hold un-walkable grid nodes. To avoid the memory waste, I have created `DynamicGrid`, which allocates the memory for only walkable grid nodes. 


(Please note that there is trade off of memory and performance. This degrades the performance to improve memory usage.)


```c#
BaseGrid seachGrid = new DynamicGrid();  
```


You may also use a `List` of walkable `GridPos`, while instantiating the `DynamicGrid` class. It will initiate only the nodes in the grid where the walkability is `true`:

```c#
List<GridPos> walkableGridPosList= new List<GridPos>();
for(int widthTrav=0; widthTrav< 64; widthTrav++)
{
   movableMatrix[widthTrav]=new bool[length][];
   for(int lengthTrav=0; lengthTrav < 32; lengthTrav++)
   {
      movableMatrix[widthTrav][lengthTrav]=new bool[height];
      for(int heightTrav=0; heightTrav < 24;  heightTrav++)
      {
         walkableGridPosList.Add(new GridPos(widthTrav, lengthTrav, heightTrav));
      }
   }
   
}

BaseGrid searchGrid = new DynamicGrid(walkableGridPosList);  
```


Rest of the functionality like `SetWalkableAt`, `Reset`, etc. are same as `StaticGrid`. 


#### Dynamic Grid With Pool ####

In many cases, you might want to reuse the same grid for next search. And this can be extremely useful when used with `PartialGridWPool`, since you don't have to allocate the grid again.


```c#
NodePool nodePool = new NodePool();
BaseGrid seachGrid = new DynamicGridWPool(nodePool);  
```


Rest of the functionality like `SetWalkableAt`, `Reset`, etc. are same as `DynamicGrid`. 


#### Partial Grid With Pool ####

As mentioned above, if you want to search only partial of the grid for performance reason, you can use `PartialGridWPool`. Just give the `GridCube` which shows the portion of the grid you want to search within.


```c#
NodePool nodePool = new NodePool();
...
BaseGrid seachGrid = new PartialGridWPool(nodePool, new GridCube(1,3,5,15,30,20);  
```


Rest of the functionality like `SetWalkableAt`, `Reset`, etc. are same as `DynamicGridWPool`. 


#### IterationType ####
You may use recursive function or loop function to find the path. This can be simply done by setting IterationType flag in JumpPointParam:
```
Note that the default is IterationType.LOOP, which uses loop function.
```

```c#
// To use recursive function
JumpPointParam jpParam = new JumpPointParam(...);
jpParam.IterationType = IterationType.RECURSIVE;  
```


To change back to loop function


```c#
// To change back to loop function 
jpParam.IterationType = IterationType.LOOP; 
```

#### Extendability ####
You can also create a sub-class of `BaseGrid` to create your own way of `Grid` class to best support your situation.

License
-------

[The MIT License](http://opensource.org/licenses/mit-license.php)

Copyright (c) 2017 Woong Gyu La <[juhgiyo@gmail.com](mailto:juhgiyo@gmail.com)>

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
