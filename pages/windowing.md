# Windowing
### 1. [WindowsOS window focus loss](#windowsos-window-focus-loss)
Right after creating a new window in ```WindowsOS``` a ```FOCUS LOST``` event will immediately be sent to the previously focused window. <br/>
WindowsOS does not queue this event for the next poll to catch, it emmits it right away. <br/>
This can have a negative effect on UI logic that expects a focus loss on the next frame after creating the new window.

### 2. [WindowsOS window freeze on resize](#windowsos-window-freeze-on-resize)
While mouse-resizing, the window enters in a ```MODAL resize operation``` state in which no events can be sent in order to wake up the thread waiting on events. <br/>
Modal means that WindowOS takes control over the events until the window is released by the user. <br/>
What happens on WindowsOS, on an event loop:
```cli
    waitEvents() / pollEvents()
            |
            \/
    WindowsOS enters MODAL mode
            |
            \/
    Event processing is blocked
            |
            \/
    Rendering and other systems do not update

```
When using ```GLFW```, register callback ```glfwSetWindowRefreshCallback``` in order to receive updates from the OS even if the window is currently being resized.

## References Used
-