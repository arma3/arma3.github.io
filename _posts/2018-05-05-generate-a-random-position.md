# Create A Random Position In 2D
_This is a translation of a tutorial I made on the German ArmaWorld forums. [Here is the link to that post](https://armaworld.de/index.php?thread/3796-zufällige-position-erzeugen/)._

## Circles

Creating random positions can be a bit treacherous as the game uses Cartesian coordinates.
A random X plus a random Y will always result in a square or rectangle.
Of course you do not realize that right away, but we all play Arma because it's so super realistic.
Therefore, it would be annoying if our scripted artillery has an unnatural square scattering, right?!

Because we do not want to reinvent the wheel every time, here is a "formulary": 
![https://i.imgur.com/3VSmKgf.png](https://i.imgur.com/3VSmKgf.png)
_From left to right: 
random radius (1) - random position in a circle (2) - [Gaussian normal distribution](https://en.wikipedia.org/wiki/Normal_distribution) (3)_
```sqf
// random radius (1)
private _position = _origin getPos [random  _radius, random  360];

// random area (2)
private _position = _origin getPos [_radius * sqrt random 1, random 360];

// normal distribution (3)
private _position = _origin getPos [_radius * random [-1, 0, 1], random 180];
```
(1) is what happens if you give the [`getPos`](https://community.bistudio.com/wiki/getPos) function a random radius and a random angle.
Because the area of a circle grows with its radius squared, this simplest formula yields a distribution with a tendency to a position in the center.

This is fixed in (2) by dragging the [`random`](https://community.bistudio.com/wiki/random) function below a square root.
A small random number less than 1 is magnified toward 1 after pulling the square root, or towards the "outside of the circle".
The result is a perfect distribution in the circle.

In (3) the syntax for the Gaussian normal distribution of `random` is used for the radius.
This then gives a realistic result, as one would expect "in nature".

However, since you usually do not want to bomb away your players right away, but rather create a war ambience, you can instead shoot at a position around the players. 
![https://i.imgur.com/x1y9PUi.png](https://i.imgur.com/x1y9PUi.png)
_From left to right: 
inverted normal distribution, random radius (4) - inverted normal distribution, random position (5) - random position in ring (6)_
```sqf
// inverted normal distribution, random radius (4)
private _position = _origin getPos [_radius * (1 - abs random [-1, 0, 1]), random 360];

// inverted normal distribution, random area (5)
private _position = _origin getPos [_radius * sqrt (1 - abs random [-1, 0, 1]), random 360];

// random area, ring (6)
private _position = _origin getPos [sqrt (_innerRadius^2 + random (_radius^2 - _innerRadius^2)), random 360];
```
In (4) the normal distribution is simply inverted ("1 minus random number").
This makes a hit on the edge most likely.

(5) combines the normal distribution with the growth of the area with the radius squared.
That makes a hit in the middle even more unlikely.

With (6) you play it safe and create a completely random position in a ring.
This is achieved by pulling the square root again.
Since we do not work with [0,1] but with the absolute numbers, these radii are squared before the square root is applied.
This way the center remains untouched.


## Ellipses
It is also possible to create random positions in a (rotated) ellipse with little effort:
![https://i.imgur.com/T5NuBy5.png](https://i.imgur.com/T5NuBy5.png)
```sqf
// random area ellipse
private "_position";

while {
    _position = _origin vectorAdd [_radius1 * (random 2 - 1), _radius2 * (random 2 - 1), 0];
    !(_position inArea [_origin, _radius1, _radius2, 0, false])
} do {};

// rotate shape by _angle
_position = _origin getPos [_position distance2D _origin, (_origin getDir _position) + _angle];

_position
```
Note that `_radius1` and `_radius2` here denote the maximum and minimum distance of the edge of the ellipse to its center.

For the sake of simplicity the discard method is used.
A random position in a rectangle is created and then the [`inArea`](https://community.bistudio.com/wiki/inArea) function checks if this position is actually in the desired ellipse.
If not we discard our first attempt and the whole process is repeated with a new position.

In the second step, the position is rotated clockwise by `_angle` degree around the origin.
This step can also be used for rectangles and other shapes.

## Triangles
Of course you can also easily generate random points in any triangle:
![https://i.imgur.com/LXIqtR2.png](https://i.imgur.com/LXIqtR2.png)
```sqf
// random area triangle
private _posA = [-4,-2,0];
private _posB = [4,-2,0];
private _posC = [0,6,0];

private _vecAB = _posB vectorDiff _posA;
private _vecAC = _posC vectorDiff _posA;

private _random1 = random 1;
private _random2 = random 1;

if (_random1 + _random2 > 1) then {
    _random1 = 1 - _random1;
    _random2 = 1 - _random2;
};

private _position =
    _origin vectorAdd
    _posA vectorAdd
    (_vecAB vectorMultiply _random1) vectorAdd
    (_vecAC vectorMultiply _random2);

_position
```
The discard method is rather unsuitable, especially for narrow triangles, so now some linear algebra is used.
In addition to the origin, one only needs the 3 vertices A, B and C relative to it.

It is well known that two vectors span a parallelogram. Starting from any vertex (e.g. A), you can go 0 to 1 times the direction vector to a second corner (e.g. AB), and then from there 0 to 1 times the direction vector of the first corner to the third corner (e.g. AC).

If we then invert the two random multipliers for the vectors AB and AC if they are in sum greater than 1, we halve the parallelogram and get a nice triangle.
