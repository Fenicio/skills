# Godot 2D Classes

## Node2D

Base class for all 2D nodes. Provides 2D transformation (position, rotation, scale).

**Properties:**
- `position: Vector2` - Position relative to parent
- `rotation: float` - Rotation in radians
- `scale: Vector2` - Scale factor
- `global_position: Vector2` - World position
- `global_rotation: float` - World rotation
- `z_index: int` - Draw order (higher draws on top)

**Methods:**
- `look_at(point: Vector2)` - Rotate to look at a point
- `get_angle_to(point: Vector2)` - Get angle to a point
- `move_local_x(delta: float)` - Move along local X axis
- `move_local_y(delta: float)` - Move along local Y axis

## Camera2D

Camera node for 2D scenes. Determines what part of the world is visible.

**Properties:**
- `zoom: Vector2` - Camera zoom (default Vector2(1, 1))
- `offset: Vector2` - Offset from target position
- `position_smoothing_enabled: bool` - Enable smooth following
- `position_smoothing_speed: float` - Speed of smoothing
- `limit_left: int` - Left world boundary
- `limit_right: int` - Right world boundary
- `limit_top: int` - Top world boundary
- `limit_bottom: int` - Bottom world boundary
- `drag_horizontal_enabled: bool` - Enable horizontal drag
- `drag_vertical_enabled: bool` - Enable vertical drag

**Methods:**
- `make_current()` - Make this the active camera
- `get_screen_center_position()` - Get center of camera in world coords
- `reset_smoothing()` - Reset smoothing interpolation

**Common Patterns:**

**Follow Player:**
```gdscript
# Make Camera2D a child of the player node
# It will automatically follow the player's position
```

**Smooth Camera Movement:**
```gdscript
extends Camera2D

@export var target_path: NodePath
@onready var target = get_node(target_path)

func _process(_delta):
    global_position = target.global_position
```

**Grid-Based Camera (for dungeon crawler):**
```gdscript
extends Camera2D

var grid_size = 32
var target_position = Vector2.ZERO
var moving = false

func move_to_grid(grid_pos: Vector2):
    target_position = grid_pos * grid_size
    moving = true

func _process(delta):
    if moving:
        position = position.lerp(target_position, 5.0 * delta)
        if position.distance_to(target_position) < 0.1:
            position = target_position
            moving = false
```

## CharacterBody2D

Physics body for character controllers. Best for player and enemy movement.

**Properties:**
- `velocity: Vector2` - Current velocity (used by move_and_slide)
- `motion_mode: int` - MOTION_MODE_GROUNDED or MOTION_MODE_FLOATING
- `floor_stop_on_slope: bool` - Stop on slopes when no input
- `floor_max_angle: float` - Max angle considered a floor
- `up_direction: Vector2` - Direction considered "up" (default Vector2.UP)

**Methods:**
- `move_and_slide()` - Move using velocity, handles collisions automatically
- `move_and_collide(motion: Vector2)` - Move and return collision info
- `is_on_floor()` - Check if on floor
- `is_on_wall()` - Check if touching a wall
- `is_on_ceiling()` - Check if touching ceiling
- `get_floor_normal()` - Get normal vector of floor

**Top-Down Movement (for dungeon crawler):**
```gdscript
extends CharacterBody2D

@export var speed = 200.0

func _physics_process(_delta):
    var input_vector = Vector2.ZERO
    input_vector.x = Input.get_axis("ui_left", "ui_right")
    input_vector.y = Input.get_axis("ui_up", "ui_down")

    # Normalize to prevent faster diagonal movement
    input_vector = input_vector.normalized()

    velocity = input_vector * speed
    move_and_slide()
```

**Grid-Based Movement (for dungeon crawler):**
```gdscript
extends CharacterBody2D

var grid_size = 32
var target_position = Vector2.ZERO
var is_moving = false

func _ready():
    target_position = position

func _physics_process(_delta):
    if is_moving:
        position = position.move_toward(target_position, speed * _delta)
        if position.distance_to(target_position) < 1:
            position = target_position
            is_moving = false
    else:
        check_input()

func check_input():
    var input_dir = Vector2.ZERO
    if Input.is_action_just_pressed("ui_up"):
        input_dir = Vector2.UP
    elif Input.is_action_just_pressed("ui_down"):
        input_dir = Vector2.DOWN
    elif Input.is_action_just_pressed("ui_left"):
        input_dir = Vector2.LEFT
    elif Input.is_action_just_pressed("ui_right"):
        input_dir = Vector2.RIGHT

    if input_dir != Vector2.ZERO:
        var next_pos = position + input_dir * grid_size
        if can_move_to(next_pos):
            target_position = next_pos
            is_moving = true

func can_move_to(pos: Vector2) -> bool:
    # Check collision or dungeon map here
    return true
```

## Sprite2D

Displays a 2D texture (image).

**Properties:**
- `texture: Texture2D` - The image to display
- `hframes: int` - Horizontal frames for sprite sheet
- `vframes: int` - Vertical frames for sprite sheet
- `frame: int` - Current frame to display
- `flip_h: bool` - Flip horizontally
- `flip_v: bool` - Flip vertically
- `modulate: Color` - Tint color

**Using Sprite Sheets:**
```gdscript
extends Sprite2D

func _ready():
    hframes = 4  # 4 columns
    vframes = 2  # 2 rows
    frame = 0    # Show first frame
```

## AnimatedSprite2D

Sprite with built-in animation support.

**Properties:**
- `sprite_frames: SpriteFrames` - Animation resource
- `animation: String` - Current animation name
- `frame: int` - Current frame
- `speed_scale: float` - Playback speed multiplier

**Methods:**
- `play(anim_name: String)` - Play animation
- `stop()` - Stop animation
- `is_playing()` - Check if playing

**Signals:**
- `animation_finished` - Animation completed
- `frame_changed` - Frame changed

```gdscript
extends AnimatedSprite2D

func _ready():
    play("idle")

func attack():
    play("attack")
    await animation_finished
    play("idle")
```

## TileMap

Grid-based map system for levels.

**Properties:**
- `tile_set: TileSet` - Tile set resource
- `cell_quadrant_size: int` - Rendering optimization size

**Methods:**
- `set_cell(layer: int, coords: Vector2i, source_id: int, atlas_coords: Vector2i)` - Set tile
- `get_cell_source_id(layer: int, coords: Vector2i)` - Get tile at position
- `erase_cell(layer: int, coords: Vector2i)` - Remove tile
- `local_to_map(local_position: Vector2)` - Convert world to grid coords
- `map_to_local(map_position: Vector2i)` - Convert grid to world coords
- `get_used_cells(layer: int)` - Get all used cell coordinates

**Dungeon Map Example:**
```gdscript
extends TileMap

const TILE_FLOOR = 0
const TILE_WALL = 1

func create_dungeon_from_json(dungeon_data: Dictionary):
    var tiles = dungeon_data["tiles"]
    for y in range(tiles.size()):
        for x in range(tiles[y].size()):
            var tile_type = tiles[y][x]
            var atlas_coords = Vector2i(tile_type, 0)
            set_cell(0, Vector2i(x, y), 0, atlas_coords)

func is_walkable(grid_pos: Vector2i) -> bool:
    var tile_id = get_cell_source_id(0, grid_pos)
    return tile_id == TILE_FLOOR
```

## CollisionShape2D

Defines collision shape for physics bodies. Must be child of a physics body node.

**Common Shapes:**
- `RectangleShape2D` - Rectangular collision
- `CircleShape2D` - Circular collision
- `CapsuleShape2D` - Capsule/pill shape
- `SegmentShape2D` - Line segment

```gdscript
# Create collision programmatically
var collision_shape = CollisionShape2D.new()
var shape = RectangleShape2D.new()
shape.size = Vector2(32, 32)
collision_shape.shape = shape
add_child(collision_shape)
```

## Area2D

Detects when other bodies enter/exit a region. Doesn't block movement.

**Signals:**
- `body_entered(body: Node2D)` - Body entered area
- `body_exited(body: Node2D)` - Body exited area
- `area_entered(area: Area2D)` - Area entered area
- `area_exited(area: Area2D)` - Area exited area

**Properties:**
- `monitoring: bool` - Detect other bodies/areas
- `monitorable: bool` - Can be detected by others

**Example (Pickup Item):**
```gdscript
extends Area2D

func _ready():
    body_entered.connect(_on_body_entered)

func _on_body_entered(body):
    if body.is_in_group("player"):
        print("Player picked up item!")
        queue_free()
```

## CanvasLayer

Container for UI elements or HUD that doesn't move with camera.

**Properties:**
- `layer: int` - Drawing layer (higher = on top)
- `offset: Vector2` - Offset all children
- `rotation: float` - Rotate all children
- `scale: Vector2` - Scale all children

**HUD Example:**
```gdscript
# Create HUD that stays on screen
var canvas_layer = CanvasLayer.new()
canvas_layer.layer = 10  # Draw on top
add_child(canvas_layer)

var label = Label.new()
label.text = "Score: 0"
canvas_layer.add_child(label)
```

## Control Nodes for UI

### Label
Display text.
- `text: String` - Text to display
- `horizontal_alignment: int` - LEFT, CENTER, RIGHT
- `vertical_alignment: int` - TOP, CENTER, BOTTOM

### Button
Clickable button.
- `text: String` - Button label
- `pressed` signal - Emitted when clicked

### TextureRect
Display texture in UI.
- `texture: Texture2D` - Image to display
- `expand_mode: int` - How to scale
- `stretch_mode: int` - Scaling behavior
