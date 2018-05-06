### Create A Random Position In 2D
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
(1) is what happens if you give the `getPos` function a random radius and a random angle.
Because the area of a circle grows with its radius squared, this simplest formula yields a distribution with a tendency to a position in the center.

This is fixed in (2) by dragging the `random` function below a square root.
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
