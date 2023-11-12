### Table of Contents
- [What is the purpose?](#What-is-the-purpose?)
- [Installing and Running](#installing-and-running)
- [Exercise](#exercise)
- [Troubleshooting](#troubleshooting)
- [Robot API](#robot_api)
  - [Motors](#motors)
  - [Grabber](#grabber)
  - [Vision](#vision)
- [Coding](#coding)
  - [Necessary stuff](#Necessary_stuff)
  - [Drive](#Drive)
  - [Turn](#Turn)
  - [find_token_gold](#turning-mechanism)
  - [for token in R.see](#for-token-in-rsee)
  - [While](#while)
  - [Adjustments](#Adjustments)
- [final loop for counting](#final-loop-for-counting)


# What is the purpose?
  
The objective of the first assignment in the Research Track course is to provide a relative introduction to the Python programming language and its relevance to robotics. In this assignment, we aim to instruct the robot to collect boxes distributed throughout a two-dimensional space and place them in a designated area on the page.
These are pictures at the beginning and the end of the process.


At the first we can see this posture.


![bandicam 2023-11-08 11-42-33-853](https://github.com/Alansaberi/First_assignment-RT1/assets/64252685/aace0b7d-9aa8-4301-84ff-f9a51ec0e6c2)



And here's the final result.


![bandicam 2023-11-08 11-44-59-018](https://github.com/Alansaberi/First_assignment-RT1/assets/64252685/5cbfdb27-15be-4bbf-884a-78a0c452eb0e)


# Python Robotics Simulator

This is a simple, portable robot simulator developed by Student Robotics. Some of the arenas and the exercises have been modified for the Research Track I course.

### Installing and Running

The simulator requires a Python 2.7 installation, the pygame library, PyPyBox2D, and PyYAML.

Pygame, unfortunately, can be tricky (though not impossible) to install in virtual environments. If you are using pip, you might try `pip install hg+https://bitbucket.org/pygame/pygame`, or you could use your operating system's package manager. Windows users could use Portable Python. PyPyBox2D and PyYAML are more forgiving, and should install just fine using pip or easy_install.

### Troubleshooting

When running `python run.py <file>`, you may be presented with an error: `ImportError: No module named 'robot'`. This may be due to a conflict between `sr.tools` and `sr.robot`. To resolve, symlink `simulator/sr/robot` to the location of `sr.tools`.

On Ubuntu, this can be accomplished by:

1. Find the location of `sr.tools`: `pip show sr.tools`
2. Get the location. In my case, this was `/usr/local/lib/python2.7/dist-packages`
3. Create symlink: `ln -s path/to/simulator/sr/robot /usr/local/lib/python2.7/dist-packages/sr/`

### Exercise

To run one or more scripts in the simulator, use `run.py`, passing it the file names.

I am proposing you three exercises, with an increasing level of difficulty. The instruction for the three exercises can be found inside the `.py` files (`exercise1.py`, `exercise2.py`, `exercise3.py`).

When done, you can run the program with:

```shell
$ python run.py exercise1.py  

You also have the solutions of the exercises (folder solutions):
$ python run.py solutions/exercise1_solution.py

```
## Robot API
The API for controlling a simulated robot is designed to be as similar as possible to the SR API.

### Motors
The simulated robot has two motors configured for skid steering, connected to a two-output Motor Board. The left motor is connected to output 0 and the right motor to output 1.

The Motor Board API is identical to that of the SR API, except that motor boards cannot be addressed by serial number. So, to turn on the spot at one quarter of full power, one might write the following:

```shell
R.motors[0].m0.power = 25
R.motors[0].m1.power = -25
```

## Grabber
The robot is equipped with a grabber, capable of picking up a token which is in front of the robot and within 0.4 metres of the robot's centre. To pick up a token, call the R.grab method:
```shell
success = R.grab()
```

The R.grab function returns True if a token was successfully picked up, or False otherwise. If the robot is already holding a token, it will throw an AlreadyHoldingSomethingException.

To drop the token, call the R.release method.

Cable-tie flails are not implemented.

# Vision
To help the robot find tokens and navigate, each token has markers stuck to it, as does each wall. The R.see method returns a list of all the markers the robot can see, as Marker objects. The robot can only see markers which it is facing towards.

Each Marker object has the following attributes:

- info: a MarkerInfo object describing the marker itself. Has the following attributes:
  - code: the numeric code of the marker.
  - marker_type: the type of object the marker is attached to (either MARKER_TOKEN_GOLD, MARKER_TOKEN_SILVER, or MARKER_ARENA).
  - offset: offset of the numeric code of the marker from the lowest-numbered marker of its type. For example, token number 3 has the code 43, but offset 3.
  - size: the size that the marker would be in the real game, for compatibility with the SR API.
  - centre: the location of the marker in polar coordinates, as a PolarCoord object. Has the following attributes:
    - length: the distance from the centre of the robot to the object (in meters).
    - rot_y: rotation about the Y axis in degrees.
    - dist: an alias for centre.length.
  - res: the value of the res parameter of R.see, for compatibility with the SR API.
  - rot_y: an alias for centre.rot_y.
  - timestamp: the time at which the marker was seen (when R.see was called).

For example, the following code lists all of the markers the robot can see:

```shell
markers = R.see()
print "I can see", len(markers), "markers:"

for m in markers:
    if m.info.marker_type in (MARKER_TOKEN_GOLD, MARKER_TOKEN_SILVER):
        print " - Token {0} is {1} meters away".format( m.info.offset, m.dist )
    elif m.info.marker_type == MARKER_ARENA:
        print " - Arena marker {0} is {1} meters away".format( m.info.offset, m.dist )
```

```shell

Now, the text is formatted in GitHub-style with proper code blocks and headings.
```
# Here is the pseduocode for the program 
### we will break down every line in the next chapter.
```shell
Initialize Robot (R)
Set ORIENTATION_THRESHOLD and DISTANCE_THRESHOLD
first = 1
first_box_placed = True

while first is True:
    dist, rot_y = find_token_gold()
    
    if dist == -1:
        Print "I don't see any token!!"
        Set first to 0  # If no markers are detected, end the program
    else if dist < DISTANCE_THRESHOLD:
        Print "Found it!"
        R.grab()  # If close to the token, grab it
        Print "Gotcha!"
        Set first to 0
    else if -ORIENTATION_THRESHOLD <= rot_y <= ORIENTATION_THRESHOLD:
        Print "Ah, here we are!."
        Drive with speed 20 for 0.8 seconds
    else if rot_y < -ORIENTATION_THRESHOLD:
        Print "Left a bit..."
        Turn with speed -2 for 0.5 seconds
    else if rot_y > ORIENTATION_THRESHOLD:
        Print "Right a bit..."
        Turn with speed 2 for 0.5 seconds

Turn left with speed -8 for 1.5 seconds
Drive with speed 30 for 6 seconds
dist, rot_y = find_token_gold()
Release the grabbed token
 Print "Token Released!"
Drive with speed -20 for 1.5 seconds
Turn with speed -10 for 4 seconds
Drive with speed 20 for 2 seconds

for each box in range(5):
    first_box_placed = True
    
    while first_box_placed is True:
        dist, rot_y = find_token_gold()

        if dist == -1:
            Turn with speed 10 for 1 second
        else if dist < DISTANCE_THRESHOLD:
            Print "Found it!"
            R.grab()  # If close to the token, grab it
            Print "Gotcha!"
            Turn with speed 20 for 2.5 seconds  # Adjust robot position for the next box placement
            Drive with speed 22 for 6 seconds  # Move to the reference position
            dist, rot_y = find_token_gold()
            Release the grabbed box
            Drive with speed -20 for 1.5 seconds  # Move away from the reference position
            Turn with speed -10 for 4 seconds
            Drive with speed 20 for 2 seconds
            Set first_box_placed to False  # Box placed, exit the loop
        else if -ORIENTATION_THRESHOLD <= rot_y <= ORIENTATION_THRESHOLD:
            Print "Ah, here we are!."
            Drive with speed 25 for 0.8 seconds
        else if rot_y < -ORIENTATION_THRESHOLD:
            Print "Left a bit..."
            Turn with speed -2 for 0.5 seconds
        else if rot_y > ORIENTATION_THRESHOLD:
            Print "Right a bit..."
            Turn with speed 2 for 0.5 seconds
```
# Coding 
a simulated environment where a robot navigates to find and interact with objects—specifically gold tokens. The find_token_gold function searches for the nearest gold token within the robot's view, returning its distance and orientation relative to the robot. If a gold token is within a certain distance and orientation threshold, the robot performs a series of actions such as grabbing the token or adjusting its position. If no tokens are found, the robot stops its search. The main loop iterates through this process, handling the robot's movement and interaction with tokens until all tasks (like placing boxes) are completed. The code contains several control structures for decision-making based on sensory input, demonstrating basic autonomous behavior.

### Necessary stuff

```shell
import time
#Imports the `time` module from Python's standard library. (Makes time-related functions available in this script)

from sr.robot import *
# Function: Imports all available classes and functions from the `sr.robot` module.
# Makes the functionalities provided by the `sr.robot` module accessible.

# Threshold for the control of the orientation
ORIENTATION_THRESHOLD = 2.0
# Defines a constant named `ORIENTATION_THRESHOLD`.
# This constant is used as a threshold for orientation control, likely in degrees, to determine when the robot is properly aligned.

# Threshold for the control of the linear distance
DISTANCE_THRESHOLD = 0.4
# Defines a constant named `DISTANCE_THRESHOLD`.
# Threshold for distance control, likely in meters, to determine proximity to a target or object.

fbox = 1
# Initializes a variable `fbox` to 1.
# How it works: Sets the initial value of `fbox` to 1.
# controls a loop or condition related to box handling 

first_box_placed = True
# Set this to True if the first box is placed
```

### Drive

```shell
def drive(speed, seconds):
    """
    Function for setting a linear velocity

    Args:
    speed (int): the speed of the wheels
    seconds (int): the time interval
    """
    R.motors[0].m0.power = speed
    # Sets the power of motor m0 (left motor linear movement speed) to the specified speed.

    R.motors[0].m1.power = speed
    # Sets the power of motor m0 (right motor linear movement speed) to the specified speed.

    time.sleep(seconds)
    # Pauses the program for a specified duration.
    # This creates a delay, allowing the robot to drive for the specified duration before stopping or changing actions.

    R.motors[0].m0.power = 0
    # Stops motor m0.

    R.motors[0].m1.power = 0
    # Function: Stops motor m1.
```

### Turn 

```shell
def turn(speed, seconds):
    """
    Function for setting an angular velocity

    Args:
    speed (int): the speed of the wheels
    seconds (int): the time interval
    """
    R.motors[0].m0.power = speed
    # Function: Sets the power of motor m0 (left motor) for turning.

    R.motors[0].m1.power = -speed
    # Sets the power of motor m1 (right motor) in the opposite direction for turning.
    # This line causes motor m1 to rotate in the opposite direction to motor m0, enabling the robot to turn.

    time.sleep(seconds)
    # Pauses the program for a specified duration.
    # This creates a delay, allowing the robot to drive for the specified duration before stopping or changing actions.

    R.motors[0].m0.power = 0
    # Stops motor m0 after turning.
    # Sets the power of motor m0 to 0.


    R.motors[0].m1.power = 0
    # Stops motor m1 after turning.
    # Sets the power of motor m1 to 0.


```

### find_token_gold

```shell
    def find_token_gold():
    dist = 100
    # Initializes a variable `dist` with a value (100 - maximum distance.).

```

### for token in R.see

```shell
for token in R.see():
        # Iterates through objects seen by the robot.
        # How it works: Uses the `see` method of the robot instance `R` to get a list of visible objects.
        # This loop processes each object (token) detected by the robot's sensors.

        if token.dist < dist and token.info.marker_type == MARKER_TOKEN_GOLD:
            dist = token.dist
            rot_y = token.rot_y
            # Updates `dist` and `rot_y` if a closer gold token is found.
            # Checks if the token is a gold one and closer than any previously observed.
            # If a gold token is detected and is closer than any others seen before, the robot updates its recorded distance and orientation to this token.

 if dist == 100:
        return -1, -1
        # Checks if no gold token was found.
        # How it works: Returns (-1, -1) if `dist` is still 100, meaning no closer token was found.
        # Detailed Explanation: This condition implies that if no gold token is within a reasonable distance, the function signals this by returning a distinctive value (-1, -1).

 else:
        return dist, rot_y
        # Function: Returns the distance and orientation of the closest gold token.
        # Provides the distance (`dist`) and rotation (`rot_y`) of the nearest gold token.
        # These values are used to guide the robot's movement towards or around the gold token.

```

### while

```shell
while fbox:
    dist, rot_y = find_token_gold()
    # Calls `find_token_gold` function within the loop.
    # Retrieves the closest gold token's distance and orientation.
    # Continuously updates the robot's knowledge of the nearest gold token's position while the loop runs.

    if dist == -1:
        print("I don't see any token!!")
        fbox = 0  # if no markers are detected, the program ends
        # Checks if no gold token is visible and ends the loop.

    elif dist < DISTANCE_THRESHOLD:
        print("Found it!")
        R.grab()  # if we are close to the token, we grab it.
        print("Gotcha!")
        fbox = 0
        # Executes actions when the robot is close enough to the token.
        # If the token is within a defined distance, the robot attempts to grab it.

    elif -ORIENTATION_THRESHOLD <= rot_y <= ORIENTATION_THRESHOLD:
        print("Ah, here we are!.")
        drive(20, 0.8)
        # Drives the robot forward when aligned with the token.
        # Activates the drive function if the robot's orientation is within acceptable limits.
        # This part ensures the robot moves towards the token when it is facing it within a certain angular threshold, signifying proper alignment.

    elif rot_y < -ORIENTATION_THRESHOLD:
        print("Left a bit...")
        turn(-2, 0.5)
        # Turns the robot left if it is not properly aligned.
        # Activates the turn function with negative speed for a slight left turn.
        # This condition helps in adjusting the robot's orientation when it is not correctly aligned with the token, specifically turning it to the left.

    elif rot_y > ORIENTATION_THRESHOLD:
        print("Right a bit...")
        turn(2, 0.5)
        # Turns the robot right if it is not properly aligned.
        # Activates the turn function with positive speed for a slight right turn.
        # Similar to the previous condition but for rightward adjustment, ensuring the robot aligns properly with the token if it is initially facing too far to the left.



```

## Adjustments

```shell


turn(-8, 1.5)

drive(30, 6)

dist, rot_y = find_token_gold()
# Searches for the nearest gold token.
# Calls the `find_token_gold` function and stores the distance and orientation to the closest gold token.
# This line is part of the robot's logic to continually search for and respond to gold tokens in its environment.

R.release()
# Function: Releases the grabbed object.
# Invokes the `release` method on the robot object `R`.
# If the robot has grabbed a token or object, this command will release it, likely as part of a pick-and-place routine.
print("Token Released")
drive(-20, 1.5)

turn(-10, 4)

drive(20, 2)

```


### final loop for counting

```shell
for box in range(5):
    first_box_placed = True
    # Iterates through a task for a set number of boxes.
    # Initializes a loop that will repeat for each box (total of 6 boxes).
    # This loop controls the process of handling multiple boxes, repeating the same set of actions for each one.

    while first_box_placed:  # This loop will execute for each subsequent box
        # Checks if the current box has been placed.
        # Continues looping until the current box is placed.
        # This inner loop handles the logic for placing each individual box before moving on to the next.

        # (The body of this while loop follows a similar pattern as described earlier, with conditions for detecting tokens, grabbing them, positioning the robot, etc.)

        first_box_placed = False  # Box placed, exit the loop
        # Marks the end of the current box's placement.
        # Sets `first_box_placed` to False, exiting the loop.
        # Indicates that the current task with the box is complete, allowing the robot to proceed to the next box or end the overall task.
        dist, rot_y = find_token_gold()
        #  Retrieves the distance and orientation to the nearest gold token.
        #  Calls the `find_token_gold` function and assigns the returned values to `dist` and `rot_y`.
        #  This line is crucial for the robot's decision-making. It continuously checks for the closest gold token, using its distance and orientation data to determine the robot's next action.
        if dist == -1:
            turn(10, 1)
        # Handles the scenario where no gold tokens are found.
        # Checks if `dist` is -1, a value indicating no tokens are detected, and then executes a turn.
elif -ORIENTATION_THRESHOLD <= rot_y <= ORIENTATION_THRESHOLD:
    print("Ah, here we are!.")
    drive(25, 0.8)
    # Drives the robot forward when properly oriented towards the target.
    # Calls the `drive` function to move forward at a speed of 25 for 0.8 seconds.
    # This condition checks if the robot is well-aligned with the target (within the orientation threshold). If so, it moves the robot forward, likely towards a token or a specific location.

elif rot_y < -ORIENTATION_THRESHOLD:
    print("Left a bit...")
    turn(-2, 0.5)
    # Adjusts the robot's orientation to the left.
    # Calls the `turn` function with a negative speed, causing a left turn.
    # This condition is met when the robot is oriented too far to the right of the target. The robot then turns left slightly to correct its orientation.

elif rot_y > ORIENTATION_THRESHOLD:
    print("Right a bit...")
    turn(2, 0.5)
    # Adjusts the robot's orientation to the right.
    # Calls the `turn` function with a positive speed, causing a right turn.
    # This line is executed when the robot is facing too far to the left of the target. The robot then turns right slightly to align properly with the target.

```
The End.
