# WendigoPy
<p align="center">
  <img src="https://raw.githubusercontent.com/medmsyk/wendigopy/master/img/logo.png" alt="Wendigo"/>
</p>

WendigoPy is a RPA library for Windows (64 bit).  

## Caution

### Simulation
WendigoPy uses [SendInput](https://docs.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-sendinput) to simulate inputs (implemented as [WendigoCs](https://github.com/medmsyk/wendigocs)).  
If you're new to it, please be careful of the things below.  

* It does NOT wait until the inputs are processed.  
* The inputs which you sent can NOT be cancelled.  
* So do NOT send many inputs at once, nor at the same time.  
Once you've done it, the inputs which you sent will be unstoppable.  
* Pressed keys must be released.  
If you left some keys pressed in your program, press and release the keys manually to recover.  
* Some kinds of applications ignore this type of simulation.  

### Exit
If something went wrong, you should stop Wendigo immediately.  
These are the ways to exit.  

* Press the keys to exit which are shown in the console.  
The keys are two shift keys (LShift+RShift) if you didn't change them.  
* Right click on the tasktray icon indicates "W" and Choose "Exit".  
* If it's urgent, press Ctrl+Alt+Del and kill the process by Task Manager.  
This way blocks sending new inputs, but remember what I told you before "the inputs which you sent will be unstoppable".

## Requirements
* Windows (64 bit)  
Testing on Windows 10.  
* Python 3  
Testing on 3.12.3.  
* [Tesseract](https://github.com/tesseract-ocr/tesseract)  
If you wanna use the OCR functions.  

## Installation
```
pip install wendigo
```

## Usage
Wendigo is an event driven application.  
```python
from wendigo import Wendigo as w

# Do something here

# Blocks until Wendigo is stopped.
w.run()
```

### Hook
Hook events of keyboard or mouse like this.  
It works even if another form is active.  
```python
from wendigo import Keys, Wendigo as w
from wendigo.device import DeviceState
from wendigo.screen import TargetForm

def key_up(state: DeviceState):
    # Shows which keys are pressed.
    print(state.key.keys)
    # Shows where the cursor is.
    print(state.mouse.position)
    # Shows the state of wheel.
    print(state.mouse.scroll)

# Runs key_up when you release Ctrl+Alt+W.
w.event_dispatcher.key_up("key_up", [Keys.ControlKey, Keys.AltKey, Keys.W], key_up)

def mark_by_drag(targets: list[TargetForm]):
    # Shows area where you marked.
    print(targets[0].area)

# Runs mark_by_drag when you drag.
w.target_marker.mark_by_drag(mark_by_drag, keys=[Keys.LButton])

# Blocks until Wendigo is stopped.
w.run()
```

### Simulation
Simulate events of keyboard or mouse like this.  
Make sure that you activated a text editor before you press the keys.  
```python
from wendigo import Keys, Point, Wendigo as w
from wendigo.device import DeviceState, Inputs

def key_up(state: DeviceState):
    # Types "Hello World!".
    w.event_simulator.type_text("Hello World!")
    # Need more manual way?
    # Let's type that again in another way.
    w.event_simulator.simulate(Inputs() \
        .key_press(Keys.Enter) \
        .key_down(Keys.ShiftKey).key_press(Keys.H).key_up(Keys.ShiftKey) \
        .key_press(Keys.E).key_press(Keys.L, n=2).key_press(Keys.O) \
        .key_press(Keys.Space) \
        .key_down(Keys.ShiftKey).key_press(Keys.W).key_up(Keys.ShiftKey) \
        .key_press([Keys.O, Keys.R, Keys.L, Keys.D]) \
        .key_down(Keys.ShiftKey).key_press(Keys.D1).key_up(Keys.ShiftKey)
    )
    # Move cursor to (0, 0).
    w.event_simulator.point_absolute(Point(0, 0))

# Runs key_up when you release Ctrl+Alt+W.
w.event_dispatcher.key_up("key_up", [Keys.ControlKey, Keys.AltKey, Keys.W], key_up)

# Blocks until Wendigo is stopped.
w.run()
```

### Others
Wendigo has other workers.  

* event_imitator  
Records inputs to a file and plays it.  
* form_controller  
Does something to forms like make them active and capture their graphics.  
* target_seeker  
Finds targets on the screen by OpenCV.  
* text_reader  
Reads text from an image by Tesseract.  
* time_keeper  
It's just a synchronous timer.  

## License
This library is released under the Apache License 2.0.