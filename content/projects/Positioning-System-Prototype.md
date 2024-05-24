---
title: "AI Position System Prototype"
summary: "Project using C++ to understand the working of Raytracing."
tags: ["C++", "Graphics"]
ShowToc: true
cover:
  image: "https://i.ibb.co/K0HVPBd/paper-mod-profilemode.png"
  # can also paste direct link from external site
  # ex. https://i.ibb.co/K0HVPBd/paper-mod-profilemode.png
  alt: "<alt text>"
  caption: "<text>"
  relative: false # To use relative path for cover image, used in hugo Page-bundles
---

## Project Summary

In this project, I developed my own pseudo-environment query system. Initially, I was working on a top-down roguelike game. I wanted to improve how enemies move toward the player instead of just setting their move position to the player's position. This process taught me a new approach to programming and thinking about AI movement.

---

## Overview
I came across an article called "Implementing Robust AI for SecondHand: Enemy Positioning" and then worked off its fundamental ideal. After some time and code experimenting, I wrapped my head around it and connected it to how Unreals EQS worked and from a Ubisoft GDC talk. Then, I broke down the workflow.

### Creating the Positions
The development process started with creating the 'PositioningPoint' struct, which was responsible for managing the position and status of each point. These points were initially spawned around the target, which, in this case, was the player. It was crucial to keep track of and update each point's position.

Example of the Position Point Initialization:
```c#
private PositioningPoint[] InitializePoints() {
   // Using a Dynamic Array during the creation of the positions then using a static(sized) array
   List<PositioningPoint> initialSetPosition = new List<PositioningPoint>();

   //starting from 1 so the first ring doesn't spawn into the player
   for (int i = 1; i <= rings; i++) {
     _positions = _initialPositions * i;
     for (int j = 0; j < _positions; j++) {
       float radians = 2 * Mathf.PI / _positions * j;
       Vector3 newPoint = new Vector3(Mathf.Sin(radians), 0, Mathf.Cos(radians));
       float ringSpacing = i + _radiusSpacing;
       Vector3 creationPoint = (newPoint * ringSpacing) + _target.position;
       Vector3 vecAwayFromTarget = creationPoint - _target.position;
       PositioningPoint point = new PositioningPoint {
         CurrentPos = creationPoint,
         OffsetPos = vecAwayFromTarget,
         AssignedStatus = false
       };
       initialSetPosition.Add(point);
     }
   }

   return initialSetPosition.ToArray();
 }
```
### Query Systems
Then, I worked on the queries, starting with the Distance Query. This was relatively easy since all I needed to do was get the distance of each point to the player and rank based on the desired range specified by the AI. By default, the closer the point is to the target, the higher the score—this can be modified from a (float)slider that a designer or, in this case, myself can easily change and test different behaviors.

```c#
private float[] DistanceFromTarget() {
  if (_posManager == null) return null;
  // calculate the distance from each point
  float[] distArray = new float[_posManager.Points.Length];
  for (int i = 0; i < _pointDataSize; i++) {
    distArray[i] = Vector3.Distance(_target.position, _posManager.Points[i].CurrentPos);
  }

  // Normalize each point between the min and max desired points
  float[] normalizedArray = new float[distArray.Length];
  float min = distArray[0];
  float max = distArray[distArray.GetUpperBound(0)];

  min += preferredDist * (_posManager.Rings - 1);
  max += preferredDist * (_posManager.Rings - 1);

  for (int i = 0; i < _pointDataSize; i++) {
    var nDistance = 1 - Mathf.Abs((distArray[i] - min) / (max - min));
    nDistance = Mathf.Clamp01(nDistance);
    normalizedArray[i] = nDistance;
  }

  return normalizedArray;
}
```

After that, I worked on the Desired Angle of Approach Query. This was a bit tricky. I needed to get the Dot Product from each point and create a score based on that. It sounds easy when I explain it in a high-level way, but the issue came down to world coordinates. Depending on which quadrant the player or the AI is in, the angle calculation would completely flip—acting as if the AI is coming from a different direction. To get around this, I manually calculated the X and Z distance of the target and the player, then the target and the point, and then used to return the dot product of those (2d)vectors.

```c#
private float[] AnglePreferenceForTarget(bool inverse = false) {
  if (_posManager == null) return null;
  Vector2 pointA = new Vector2(_target.position.x - transform.position.x, _target.position.z - transform.position.z);
  float[] tempArray = new float[_pointDataSize];
  for (int i = 0; i < _pointDataSize; i++) {
    var p = _posManager.Points[i].CurrentPos;
    Vector2 pointB = new Vector2(_target.position.x - p.x, _target.position.z - p.z);
    tempArray[i] = (Vector2.Dot(pointA.normalized, pointB.normalized) * ((inverse) ? -1 : 1)) + angleTolerance;
  }

  return tempArray;
}
```

Finally, I need to get the results from the queries. At first, I was stumped; my approach was doing method calculations simultaneously on each point. The logistics and the performance of it didn't make much sense doing it that way. Then it started to click when fixing bugs on the angle query. Related side tangent: In one of the projects at work (Intel/UST), I was assigned a task to get a game engine working with Machine Learning, so I learned about a basic concept of ML called "Reinforcement Learning." The essential idea is that the model tries a bunch of different tasks/behaviors, and based on the cycle's outcome, it's either rewarded or penalized - the following cycle, it will make changes based on the score it received. This cycle will repeat until it reaches its goal (ML people don't crucify me; I didn't dive that deep). So, coming back, I applied that concept to this problem. Each query returns an array of normalized scores between 0f and 1f, then each query array is added together, creating an 'Uber Score' array, and then the height position is selected. This worked (...well, with a few issues I needed to fix), and later added the ability to change the Influence/prioritize of the queries.

```c#
public void Evaluate() {
  if (!m_initFlag) {
    InitializeVariables();
  }

  ResetScores();
  AddScores(DistanceFromTarget(), enableDistanceParam, m_distanceQueryInfluence);
  AddScores(AnglePreferenceForTarget(invertDirection), enableFacingDirectionParam, m_angleQueryInfluence);

  _positionScores = _positionScores.Select(x => (float)Math.Round(x, 2)).ToArray();
  int incomingPositionIndex = HighestWeight();
  if (_positionScores[incomingPositionIndex] > _positionScores[m_currentPositionIndex] || m_reposition) {
    SetPositionLocation(incomingPositionIndex);
  }
}
```

### Take Aways
Working on the project and learning how others implemented it, mine is a very simplified version, though most core concepts are the same. My version can be expanded further, and I'm glad I've coded it so you can create another query behavior and easily plug it in. I found how Ubisoft did it in the Division interesting because they baked the position points into the map data, and the complexity of their NPC systems is impressive.
