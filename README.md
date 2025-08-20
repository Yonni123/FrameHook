# FrameHook
FrameHook is a lightweight screen capture and frame loop module designed for real-time visual automation. It continuously takes screenshots of a selected area and delivers them to a user-defined hook callback function, along with timing data like FPS and millisecond counters.  
It even has support for converting coordinates from screen-to-gameframe and gameframe-to-screen. This way, whatever you're automating can work in its own coordinate system and if you need to interact with it, you can call the corresponding functions to convert.  
What you do inside the callback affects the frame processing speed, giving you full control over per-frame behavior.  
  
**This project should mainly be a sub-module of something else.**

## Features
- Continuous screen capture using mss for high-performance, cross-platform screenshots.

- Real-time image display using OpenCV for immediate visual feedback.

- Custom callback function called every frame, with FPS and ms timing info.

- Functions to convert between screen coordinates and frame coordinates

- Ideal for game automation, visual monitoring, or any application that needs real-time screen processing.

- Easily integrates as a submodule in larger projects.

## How to install
It uses mss to screep capture, numpy and OpenCV for video display. They can be installed using the following:
```
pip install -r requirements.txt
```
Other than that, it is just a really simple python file that you just import into any other project.

## How to use
### 1. Define the Game Region
First, you need to define the region on your monitor where the game is displayed by creating a `GameWrapper` object:

```python
game = GameWrapper(monitor_index=0, trim=False)
```
**Parameters:**
- `monitor_index` - Index of the monitor where the game is running (useful for multi-monitor setups).  
*Default: 0*

- `trim` - Whether to automatically detect edges around the game and remove any extra screen area accidentally included.  
*Default: False*  

**Note:** This line of code is blocking. It will open a window showing your monitor, allowing you to select the region to focus on. If trim=True, the library will attempt to detect the game edges and remove any unnecessary areas (which usually happens often).

### 2. Create a Custom Action Function

Next, define a function that will be called (the hook) for each frame of the game. This is where your main processing code goes:
```Python
def custom_action(self, screen, game_FPS, counter, delta_time):
    # Convert the screen to a standard BGR frame
    frame = cv2.cvtColor(screen, cv2.COLOR_BGRA2BGR)
    
    # Handle the first frame where there is no previous FPS
    game_FPS = game_FPS or 0

    # Example: display the frame without further processing
    cv2.setWindowTitle(
        "GameFrame",
        f"FPS: {game_FPS:.2f} - Counter: {counter:.2f} - dT: {delta_time:.2f} - Press Q to quit"
    )
    cv2.imshow("GameFrame", frame)
```
**Parameters passed to custom_action:**
self, screen, game_FPS, counter, time_ms
- `self` - Part of the GameWrapper class, has useful functions and properties explained below...
- `screen` - The current frame of the game as an image (BGRA format).
- `game_FPS` - The FPS of the game. (Depends on what you do in the custom function)
- `counter` - How many frames processed. First frame is 1, second is 2 and so on.. (There is no frame 0)
- `time_ms` - Current time in milliseconds since game start (counter from 0)
counter – A frame counter.

**Functions and Properties of the `self` argument passed to the hook function:**  
`self.width` and `self.width` - Integers, the size (in pixels) of the region that you selected (game frame)
`screen_x, screen_y = self.game_to_screen_coords(game_x, game_y)` - A function that converts coordinates from game to screen.
`game_x, game_y = self.screen_to_game_coords(screen_x, screen_y)` - Vise Versa basically (yes I am lazy)

### 3. Start the Game Loop

Finally, call the play method and pass your custom action function to start processing frames:
```Python
play(self, action_function, stop_key="q")
```
This will continuously capture frames from the defined game region and execute your action function on each frame. **TO STOP, PRESS Q**. By default, stop_key is q, if you really need that button, I guess you can change it using that argument.

## Full example
This example is the same one shown in the main of the frame hook code `if __name__ == "__main__":`  
It creates custom_take_action function and simulate some kind of processing to the frame like printing values and drawing a green circle in the middle and even convert coordinates and prints them. Then creates a GameWrapper object and calls the play function, just according to the above explanation. To test how the code works, run the frame hook code, otherwise import it into something else.