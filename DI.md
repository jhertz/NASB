# How DI Works in NASB

## Intro

So I recently started reverse engineering NASB, as I think it's just a swell game and would like to see how it works.
First project was to properly document DI. It's fairly different from how it works in other Smash games, and I was surprised by the implementation.

## The Basics

* There are only two possible DI's off a given move: `diIn` and `diOut` (these determine how much to rotate your trajectory by). 
* Each move has its own values for `diIn` and `diOut` (default is 10 deg for both). They do not have to be the same!
* Each move has a property for how it can be DI'd: `Horizontal`, `Vertical`, or `Any` (more on that later)

## The implementation

tl;dr its weird, but all you need to do to get proper DI is hold left/right/up/down depending on the move

To make the above perfectly clear, you absolutely CAN still hold a diagonal, it just doesn't actually do anything different than holding the proper cardinal.


* First, we figure out whether we're going to apply horizontal or vertical DI.
* If the move specifies one, we're done with this step. If it specifies `Any`, we need to do some work:

```C#
			if (dIType == AtkProp.DIType.Any)
			{
				if (inputDir.x != 0f != (inputDir.y != 0f))
				{
					if (inputDir.x != 0f)
					{
						dIType = AtkProp.DIType.Horizontal;
					}
					if (inputDir.y != 0f)
					{
						dIType = AtkProp.DIType.Vertical;
					}
				}
				dIType = ((Mathf.Abs(launchDir.x) * 1.5f > Mathf.Abs(launchDir.y)) ? AtkProp.DIType.Vertical : AtkProp.DIType.Horizontal);
			}
```

If you can't read C#, thats okay! I'm about to explain it:

* If the X or Y input both aren't zero, but one of them is zero (i.e. you are holding one of the 4 cardinals), set it to to horizontal or vertical DI.
* If they both aren't zero (you are holding a 45 degree angle), then we look at the launch direction. If the y component is less than 1.5 times the x component (the move mostly sends horizontally), then we do vertical DI. Otherwise (the move mostly sends vertically), we do horizontal DI. 

I *think* the reason we scale by 1.5 is gravity (the theory launch angle is different from the observed one because gravity pulls us down).


Still with me? Great! Now that we've figured out the DI mode, we can actually calculate the DI. 

I'm not sure the best way to visualize this, so instead I'm just going to annotate the code.
Here's a big ugly blob of C#. Luckily for you, I've cleaned it up a bit and commented it!

Again, the value returned is the angle that we'll end up rotating by.

```C#
		case AtkProp.DIType.Horizontal:
				if (inputDir.x == 0f) break; // no input, no DI
				if (Mathf.Abs(launchDir.x) > Mathf.Abs(launchDir.y)){ //launchDir is more horizontal than vertical
					if (launchDir.x > 0f){ //move is launching to the right
						if (inputDir.x > 0f) //they are holding right
							return 0f - diOut; // give them a rotation by negative diOut
						else  //they are holding left
							return diIn; // give them a rotation by diIn
					} else { // move is launching them to the left
					if (inputDir.x > 0f) //they are holding right
						return 0f - diIn; // give them a rotation by negative diIn
					else
						return diOut; // give them a rotation by positive diOut
					}	
				}
				if (launchDir.y > 0f){ // move launches up
					if (Mathf.Abs(launchDir.x) < 0.1f){ // move barely launches horizontally, it is mostly vertical
						if (forwardDir > 0f){ // facing right
							if (inputDir.x > 0f) // holding right
								return 0f - diIn; // DI in
							else // holding left
								return diOut; // DI out
						} //we are facing left
						if (inputDir.x > 0f) // holding right
							return 0f - diOut; //DI out
						else // holding left
							return diIn; //DI in
					} // move launches horizontally 
					if (launchDir.x > 0f){ // launch direction is to the right 
						if (inputDir.x > 0f) // holding right
							return 0f - diOut;
						else // holding left
							return diIn;
					} // launch direction is to the left
					if (inputDir.x > 0f) // holding right
						return 0f - diIn;
					else //holding left
						return diOut;
				} // move launches down
				if (Mathf.Abs(launchDir.x) < 0.1f) // move barely launches horizontally, it is mostly vertical
				{
					if (forwardDir > 0f) // forward is to the right
					{
						if (inputDir.x > 0f) // holding right
							return diIn;
						else // holding left
							return 0f - diOut;
					} // forward is to the left
					if (inputDir.x > 0f)  // holding right
						return diOut;
					else // holding left
						return 0f - diIn;
				} // move launches horizontally
				if (launchDir.x > 0f) // launchDir is to the right 
				{
					if (inputDir.x > 0f) // holding right
						return diOut;
					else  // holding left
						return 0f - diIn;
				} // launchDir is to the left
				if (inputDir.x > 0f) // holding right
					return diIn;
				else // holding left
					return diOut;
		case AtkProp.DIType.Vertical: // Vertical DI
				if (inputDir.y == 0f) break; // not holding anything, no DI
				if (Mathf.Abs(launchDir.x) > Mathf.Abs(launchDir.y)){ // move launches more horizontally than vertically
					if (launchDir.x > 0f){ // move launches right
						if (inputDir.y > 0f) // holding up
							return diIn;
						else // holding down
							return 0f - diOut;
					} // move launches down
					if (inputDir.y > 0f) // holding up
						return 0f - diIn;
					else // holding down
						return diOut;
				} // move launches more vertically than horizontally
				if (launchDir.y > 0f){  // move launches up
					if (Mathf.Abs(launchDir.x) < 0.1f){ // move barely launches horizontally
						if (forwardDir > 0f){ //forwards is to the right
							if (inputDir.y > 0f) // holding down
								return diOut;
							else //holding up
								return 0f - diIn;
						} // forwards is to the left
						if (inputDir.y > 0f) // holding up
							return 0f - diOut;
						else // holding down
							return diIn;
					} // move does launch horizontally
					if (launchDir.x > 0f){ //launchDir is to the right
						if (inputDir.y > 0f) // holding up
							return diIn;
						else // holding down
							return 0f - diOut;
					} // launchDir is to the left
					if (inputDir.y > 0f) // holding up
						return 0f - diIn;
					else // holding down
						return diOut;
				} //move launches more horizontally than vertically
				if (Mathf.Abs(launchDir.x) < 0.1f){ // barely launches horizontally
					if (forwardDir > 0f){ // forward is to the right
						if (inputDir.y > 0f) // holding up
							return diIn;
						else // holding down
							return 0f - diOut;
					} // forward is to the left
					if (inputDir.y > 0f) // holding up
						return 0f - diIn;
					else // holding right
						return diOut;
				} // move launches horizontally
				if (launchDir.x > 0f){ // launches to the right
					if (inputDir.y > 0f) // holding up
						return diOut;
					else
						return 0f - diIn;
				} // launches to the left
				if (inputDir.y > 0f) // holding up
					return 0f - diOut;
				else // holding down
					return diIn;
			return 0f; // default to no DI
		}
```

## A better way to understand this

Thanks to [PTas](https://twitter.com/PracticalTAS), we have a nice way to visualize the above labyrinth of if statements. 

We can simplify this maze down to 4 different DI types. In each, we'll visualize in and out as purple and blue. Purple *should* represent in and blue out, although this does not take into account the cases where the game actually defines in/out by the direction of the defender:

![](https://pbs.twimg.com/media/FBYrcqJWEAIF1yi?format=png&name=small)
![](https://pbs.twimg.com/media/FBYrdKRWEAAsVFG?format=png&name=small)
![](https://pbs.twimg.com/media/FBYrdynXMAYJvPK?format=png&name=small)
![](https://pbs.twimg.com/media/FBYreUqXoAA1Nwf?format=png&name=small)

## Cutoff Angle
We can also use calculate the angle that switches from horizontal to vertical priority. Recall the code was

```C#
dIType = ((Mathf.Abs(launchDir.x) * 1.5f > Mathf.Abs(launchDir.y)) ? AtkProp.DIType.Vertical : AtkProp.DIType.Horizontal);
```

We can think of this as a right triangle with one side of length 2, and another of length 3. Applying [a little trig](https://www.calculator.net/right-triangle-calculator.html?av=3&alphav=&alphaunit=d&bv=2&betav=&betaunit=d&cv=&hv=&areav=&perimeterv=&x=79&y=23), we get a cutoff angle of 56.31°

## Implications/Conclusions

* Since the game is digital, DI is much simpler. You only have 8 angles to start with, and the game simplifies this for DI.
* The game thinks of things as either needing to be DI’d horizontally or vertically.
* It checks whether you are holding left/right or up/down (depending on whether its in vertical or horizontal mode), and uses that to give you either in or out DI.
* Specifically, all you need to do to get correct DI is to input either left, right, up, or down.
* You never *need* to input a diagonal direction to get proper DI (but using a diagonal may be optimal as it can OS between vertical and horizontal modes).
* Whether you get in/out DI is dependent on a number of factors, mostly the move's launch angle, as well as (sometimes) what direction you are facing. 



## See ya later space cowboy