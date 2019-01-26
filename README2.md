# Physics
Ball physics sim in p5.js for 2018 Durham computer Science Summative Assessment

## License and attribution

Original code taken from "Physics mod" by Lorenzo

http://www.openprocessing.org/sketch/617370

Licensed under Creative Commons Attribution ShareAlike:

https://creativecommons.org/licenses/by-sa/3.0

## Outline

This program displays a number of circles on the screen. On start the circles will be randomly placed around the screen. When the mouse is pressed they will move to "orbit" the location of the mouse pointer, colliding with each other and bouncing off the edges of the screen. When the mouse is not pressed, the circles will return to positions uniformly distributed across the screen. The colour of the circles is based on the position that they take in this case. The program is dependent on the [p5.js library](https://p5js.org/).

## Program structure 

### physics.js

The **physics.js** file contains three class definitions which are combined to create the overall effect of the program. These are:

- Point
- Quadtree
- Physics

They are outlined below

#### Point

The point class is used to store information about the circles displayed on the screen. It has the properties:

- home (the position it will return to when the mouse is released)- p5.Vector.

- pos (its current position)- p5.Vector.

- velocity- p5.Vector.

- diam (the diameter of the circle to be displayed on the screen)- float.

- c (the colour the circle will be displayed as based on its home position)- p5.color

- collided (a flag used to determine if collisions have been checked for the point this frame)- boolean

The class also has the following methods:  

- constructor(diam, newx, newy, colourScale, blueVal, canvasSizeX, canvasSizeY)- initialises properties based on the arguments given. Initially pos is given a random value within the bounds of the canvas being drawn on, and the colour of the point is calculated based on its position, colourScale and bluVal.

- Paint(r)- draws the circle to the correct position on screen. If r, an optional p5.graphics object is given it will draw to that, otherwise it will draw to the canvas created by the instance of the Physics object it belongs to.
- Collide(point)- this function takes another point as a parameter, and determines the two points' relative positions and velocities in order to calculate and apply an impulse to them so that they bounce off each other, and update their positions accordingly.
- Update(canvasSizeX, canvasSizeY)- this function handles the normal movement of the point. It will check the point's position compared to the bounds of the canvas supplied as arguments, and if it is out of bounds the point will "bounce" off the edges. If the mouse is pressed then a force is applied towards the mouse to cause the point to orbit it (inversely proportional to the square of the distance between the point and the mouse), and if the mouse isn't pressed the point returns to its home, with a force inversely proportional to the distance between "pos" and "home".

#### Quadtree

The quadtree is a data structure used to speed up collision checking. It is a tree structure, with a parent node containing 4 child quadtrees. Each instance of the quadtree in this program can contain up to 4 points. 

The quadtree is first created by creating a single "box" which contains the entire canvas. Then points are inserted into it. When 4 points have been inserted into this box, it is full. The box is then split into 4 smaller boxes containing an equal area. Then more points are inserted- the position of each point is checked with the bounds of each box to see which one it will go into. If any of the boxes becomes full,  It will be divided into 4 smaller boxes, and if a point would have fit into that box's position, its children will be checked, and so on and so forth (using a recursive approach).

Without the quadtree, every point would have to be checked against every  other point in order to determine if any were colliding, which is very computationally expensive. The quadtree separates points by their position, so that during collision checking only nearby sections of the quadtree need to be checked. The checking process involves finding the area around the point which would constitute a collision (for a circle this is a circle with twice the radius of the original). If a box intersects this area by any amount, all the points within that box will be checked against the original one to see if they collide (regardless of their position within the box). This greatly reduces the number of comparisons that need to be made.

The Quadtree class has the following properties:

- w (the width of the box)- float.
- h (the height of the box)- float.
- centre (the position of the centre of the box)- p5.Vector.
- maxPoints (the maximum number of points the box can contain)- integer.
- points (the list of references to Point objects that the box contains)- list.
- divided (a flag to record whether or not the box has yet been divided into 4 smaller ones)- boolean.
- upLeft, upRight, downLeft, downRight (the children of this node in the overall quadtree, the name signifies which quadrant of this box they represent)- Quadtree.

The methods of this class are:

- constructor(width, height, centre, maxPoints)- here the properties of the quadtree are set to the provided arguments, as well as initialising an empty list for the points within the tree to be stored in and setting the divided flag to false.

- divide()- this function serves to calculate the positions of the 4 quadrants of the box, and create 4 child Quadtree objects which occupy these quadrants
- insert(point)- this will take a point and insert it into the quadtree. First it ensures that the point is within the bounds of the box. If it is, then if the box has room for the point (i.e. there are less than maxPoints attributed to it) then it is inserted into that box's points list. If not, the box will be divided into 4 children with divide() if this has not already been done, and the insert() function will be called on each child until it is inserted. This function is applied recursively on the children of each box until the point is successfully inserted.
- checkArea(point)- this takes a Point object and determines intersects the box (it doesn't necessarily need to be in the points list of the box). If it does, then collisions will be checked with every point contained within the box. If one is found the point's collide() function is called

#### Physics

This class is contains both a quadtree and points objects, and controls when updates on position and velocity should be made. Its properties are:

- constructor(numPoints, diam, colourScale, blueVal, canvasSize)- here values are initialised and the changeNumPoints function with numPoints as an argument is called to create the requisite number of points within the object.
- numPoints (the number of points on each side of the square displayed on the screen- totalling numPoints ^ 2)- integer.
- diam (the diameter of the circles to be displayed)- float
- colourScale (the value used during the calculation of the colour of the circles based on their home position)- float.
- blueVal (another parameter to determine the colour of circles, in this case how much blue is in the overall sketch)- integer.
- canvasSizeX, canvasSizeY (the dimensions of the canvas that objects are displayed on)- float.
- qt (the quadtree to denote the positions of points)- Quadtree
- points (the list of lists of Points, each list within the larger one is a row in the square)- list.
- hasCanvas (flag to determine if a canvas has been created yet)- Boolean.

The methods of this class are:

- draw(r)- in this function, each Point in points is checked for collisions, and the force for the situation in is applied in order to correctly update their position. Then if r, a p5.graphics object, is supplied, the points will be drawn to this, otherwise thegiy will be drawn to the canvas created in the constructor.
- checkCollisions()- first, every Point in points will have its collision flag reset and will be inserted into the quadtree (which is recreated every time this function is called). Then once all points are inserted, they are all checked for collisions using the quadtree's checkArea() function.
- changeNumPoints(n)- will change the number of points per side of the square to n, which due to the points constructor function will also randomise their positions.
- changeColourScale(c)- will change colour change property to c, changing the drawn colours of the points.
- changeDiameter(d)- changes the diameters of points to the value d.

### index.js

This file contains the code necessary to display an instance of a Physics object, and allow it to interact with the DOM. 

It contains the p5 functions setup() and draw(). Setup runs once at the start of the program, and draw is called repeatedly.  In setup a webGL canvas, an off screen graphics area and an instance of a Physics object are created. Then in draw, the physics object is drawn to this graphics area, set as a texture and displayed on a rotating cube. 

Below this is the code for DOM interaction. It is all wrapped in an anonymous function that can only be called once all DOM content has finished loading. There are several functions which take values from the form components in **index.html**, and pass theses values to functions in the physics object. A variable is assigned to the ID of each component, and an event listener added to it which, when triggered, will call the function to change some property of the physics object. There is also a function that is called upon a submit event from the form to prevent it from trying to pass to some handler (which would cause an error as there isn't one).

### index.html

The file which describes the webpage that **index.js** displays on and takes its values from. It contains links to **style.css**, **index.js**, **physics.js** and a CDN link for the p5 library in the head. The body consists of form components used to pass values to **index.js**

### style.css

A basic CSS stylesheet to determine things like spacing of components and font style.