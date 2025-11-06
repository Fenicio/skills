# Godot Input and Controls

## Input Singleton

The Input singleton provides access to input devices (keyboard, mouse, gamepad, touch).

### Keyboard Input

**Check if Key is Pressed:**
```gdscript
if Input.is_key_pressed(KEY_SPACE):
    print("Space is held down")

# Common keys
KEY_A, KEY_B, KEY_W, KEY_S, KEY_SPACE
KEY_ENTER, KEY_ESCAPE, KEY_SHIFT, KEY_CTRL
KEY_LEFT, KEY_RIGHT, KEY_UP, KEY_DOWN
```

**Check Actions (Recommended):**
```gdscript
# Check if action is currently pressed
if Input.is_action_pressed("jump"):
    print("Jump button is held")

# Check if action was just pressed this frame
if Input.is_action_just_pressed("attack"):
    print("Attack button pressed")

# Check if action was just released this frame
if Input.is_action_just_released("jump"):
    print("Jump button released")
```

**Get Axis Input (for WASD/Arrow keys):**
```gdscript
# Returns -1, 0, or 1
var horizontal = Input.get_axis("ui_left", "ui_right")
var vertical = Input.get_axis("ui_up", "ui_down")

# Create movement vector
var input_vector = Vector2(horizontal, vertical)
```

**Get Vector Input:**
```gdscript
# Convenient method for 2D movement
var input_dir = Input.get_vector("ui_left", "ui_right", "ui_up", "ui_down")
# Returns normalized Vector2
```

### Mouse Input

**Properties:**
- `Input.mouse_mode` - MOUSE_MODE_VISIBLE, MOUSE_MODE_HIDDEN, MOUSE_MODE_CAPTURED

**Methods:**
```gdscript
# Get mouse position
var mouse_pos = get_viewport().get_mouse_position()  # Screen coords
var mouse_pos_world = get_global_mouse_position()    # World coords

# Check mouse buttons
if Input.is_mouse_button_pressed(MOUSE_BUTTON_LEFT):
    print("Left mouse held")

if Input.is_action_just_pressed("click"):  # Mapped to left click
    print("Clicked!")
```

**Handling Mouse Events:**
```gdscript
func _input(event):
    if event is InputEventMouseButton:
        if event.button_index == MOUSE_BUTTON_LEFT and event.pressed:
            print("Left click at: ", event.position)

    if event is InputEventMouseMotion:
        print("Mouse moved to: ", event.position)
```

### Input Actions

Input actions are defined in Project Settings > Input Map. They map physical inputs to logical actions.

**Default Actions:**
- `ui_left` - Left arrow/A
- `ui_right` - Right arrow/D
- `ui_up` - Up arrow/W
- `ui_down` - Down arrow/S
- `ui_accept` - Enter/Space
- `ui_cancel` - Escape

**Adding Custom Actions:**
1. Project > Project Settings > Input Map
2. Add action name (e.g., "jump", "attack", "interact")
3. Add keys/buttons to the action

**Deadzone and Strength:**
```gdscript
# Get action strength (0.0 to 1.0, useful for analog sticks)
var strength = Input.get_action_strength("move_right")

# Get raw strength (ignores deadzone)
var raw = Input.get_action_raw_strength("move_right")
```

## WASD Movement Examples

### Smooth Movement
```gdscript
extends CharacterBody2D

@export var speed = 200.0

func _physics_process(_delta):
    # Get input
    var input_x = Input.get_axis("ui_left", "ui_right")
    var input_y = Input.get_axis("ui_up", "ui_down")
    var input_vector = Vector2(input_x, input_y)

    # Normalize to prevent faster diagonal movement
    if input_vector.length() > 1.0:
        input_vector = input_vector.normalized()

    # Apply velocity
    velocity = input_vector * speed
    move_and_slide()
```

### 8-Directional Movement
```gdscript
extends CharacterBody2D

@export var speed = 200.0

func _physics_process(_delta):
    var direction = Vector2.ZERO

    if Input.is_action_pressed("ui_right"):
        direction.x += 1
    if Input.is_action_pressed("ui_left"):
        direction.x -= 1
    if Input.is_action_pressed("ui_down"):
        direction.y += 1
    if Input.is_action_pressed("ui_up"):
        direction.y -= 1

    # Normalize diagonal movement
    direction = direction.normalized()

    velocity = direction * speed
    move_and_slide()
```

### Grid-Based Movement (for dungeon crawler)
```gdscript
extends CharacterBody2D

@export var grid_size = 32
@export var move_speed = 200.0

var target_position = Vector2.ZERO
var is_moving = false

func _ready():
    # Snap to grid
    position = position.snapped(Vector2(grid_size, grid_size))
    target_position = position

func _physics_process(delta):
    if is_moving:
        # Move towards target
        var direction = (target_position - position).normalized()
        velocity = direction * move_speed
        move_and_slide()

        # Check if reached target
        if position.distance_to(target_position) < 2:
            position = target_position
            is_moving = false
            velocity = Vector2.ZERO
    else:
        # Check for input
        handle_input()

func handle_input():
    var direction = Vector2.ZERO

    if Input.is_action_just_pressed("ui_right"):
        direction = Vector2.RIGHT
    elif Input.is_action_just_pressed("ui_left"):
        direction = Vector2.LEFT
    elif Input.is_action_just_pressed("ui_down"):
        direction = Vector2.DOWN
    elif Input.is_action_just_pressed("ui_up"):
        direction = Vector2.UP

    if direction != Vector2.ZERO:
        var next_position = position + direction * grid_size
        if can_move_to(next_position):
            target_position = next_position
            is_moving = true

func can_move_to(pos: Vector2) -> bool:
    # TODO: Check against dungeon map/walls
    return true
```

### Alternative Grid Movement (Instant)
```gdscript
extends Node2D

@export var grid_size = 32

func _process(_delta):
    var moved = false

    if Input.is_action_just_pressed("ui_right"):
        position.x += grid_size
        moved = true
    elif Input.is_action_just_pressed("ui_left"):
        position.x -= grid_size
        moved = true
    elif Input.is_action_just_pressed("ui_down"):
        position.y += grid_size
        moved = true
    elif Input.is_action_just_pressed("ui_up"):
        position.y -= grid_size
        moved = true

    if moved:
        check_collision()
```

## Input Events

Handle input through the `_input()` or `_unhandled_input()` callback.

### _input() vs _unhandled_input()

**_input(event):**
- Receives all input events
- Called before GUI nodes process input
- Can consume events with `get_viewport().set_input_as_handled()`

**_unhandled_input(event):**
- Receives input not consumed by GUI
- Better for gameplay input
- Automatically skips events handled by UI

### Input Event Types

**InputEventKey** - Keyboard input
```gdscript
func _input(event):
    if event is InputEventKey:
        if event.pressed and event.keycode == KEY_SPACE:
            print("Space pressed")
```

**InputEventMouseButton** - Mouse clicks
```gdscript
func _input(event):
    if event is InputEventMouseButton:
        if event.button_index == MOUSE_BUTTON_LEFT:
            if event.pressed:
                print("Mouse down at ", event.position)
            else:
                print("Mouse up")
```

**InputEventMouseMotion** - Mouse movement
```gdscript
func _input(event):
    if event is InputEventMouseMotion:
        print("Mouse moved: ", event.relative)
        print("Velocity: ", event.velocity)
```

## Gamepad Input

```gdscript
# Check if any gamepad is connected
if Input.get_connected_joypads().size() > 0:
    print("Gamepad connected")

# Get joystick axis (-1.0 to 1.0)
var left_stick_x = Input.get_joy_axis(0, JOY_AXIS_LEFT_X)
var left_stick_y = Input.get_joy_axis(0, JOY_AXIS_LEFT_Y)

# Check button
if Input.is_joy_button_pressed(0, JOY_BUTTON_A):
    print("A button pressed")
```

## Touch Input

```gdscript
func _input(event):
    if event is InputEventScreenTouch:
        if event.pressed:
            print("Touch at: ", event.position)
        else:
            print("Touch released")

    if event is InputEventScreenDrag:
        print("Drag: ", event.relative)
```

## Consuming Input

Prevent input from reaching other nodes:

```gdscript
func _input(event):
    if event is InputEventMouseButton and event.pressed:
        print("Handled click")
        get_viewport().set_input_as_handled()
        # No other nodes will receive this event
```

## Advanced Input Techniques

### Input Buffering
```gdscript
var input_buffer = []
const BUFFER_TIME = 0.15

func _process(delta):
    # Decay buffer
    for i in range(input_buffer.size() - 1, -1, -1):
        input_buffer[i].time -= delta
        if input_buffer[i].time <= 0:
            input_buffer.remove_at(i)

func _input(event):
    if event.is_action_pressed("jump"):
        input_buffer.append({"action": "jump", "time": BUFFER_TIME})

func try_jump():
    for input in input_buffer:
        if input.action == "jump":
            input_buffer.clear()
            return true
    return false
```

### Input Recording/Replay
```gdscript
var recorded_inputs = []
var is_recording = false
var current_frame = 0

func _process(_delta):
    current_frame += 1
    if is_recording:
        var frame_inputs = []
        if Input.is_action_pressed("ui_right"):
            frame_inputs.append("ui_right")
        if Input.is_action_pressed("ui_left"):
            frame_inputs.append("ui_left")

        if frame_inputs.size() > 0:
            recorded_inputs.append({"frame": current_frame, "inputs": frame_inputs})
```
