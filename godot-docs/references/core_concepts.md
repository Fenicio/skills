# Godot Core Concepts

## Node System

Godot's architecture is built around a tree of nodes. Every element in a game is a node, and nodes are organized in a hierarchical tree structure.

### Node Basics

**Node** - The base class for all scene objects. Every node has:
- A name (identifier within the parent)
- A parent node (except the root)
- Children nodes
- A position in the scene tree

**Common Node Methods:**
- `add_child(node)` - Add a child node
- `remove_child(node)` - Remove a child node
- `get_node(path)` - Get a node by path (e.g., `get_node("Player/Sprite")`)
- `get_parent()` - Get the parent node
- `queue_free()` - Delete the node at the end of the current frame
- `get_tree()` - Get the SceneTree
- `_ready()` - Called when node and children are added to scene tree
- `_process(delta)` - Called every frame
- `_physics_process(delta)` - Called every physics frame (fixed timestep)

**Node Paths:**
- Absolute path: `/root/Level/Player`
- Relative path: `Enemy/Sprite`
- Parent reference: `../OtherNode`
- Current node: `.`

### Scene System

**Scenes** are collections of nodes saved as `.tscn` files. Any node subtree can be saved as a scene and instanced elsewhere.

**SceneTree** - Manages the active scene and handles:
- Frame rendering
- Input events
- Physics steps
- Changing scenes: `get_tree().change_scene_to_file("res://scenes/level.tscn")`
- Pausing: `get_tree().paused = true`
- Quitting: `get_tree().quit()`

### Signals

Signals are Godot's implementation of the observer pattern for event handling.

**Defining Signals:**
```gdscript
signal health_changed(new_health)
signal player_died
```

**Emitting Signals:**
```gdscript
health_changed.emit(current_health)
player_died.emit()
```

**Connecting Signals:**
```gdscript
# Connect to a function
player.health_changed.connect(_on_player_health_changed)

# With lambda
button.pressed.connect(func(): print("Button pressed!"))
```

**Common Built-in Signals:**
- `ready` - Node is ready
- `tree_entered` - Node enters scene tree
- `tree_exited` - Node exits scene tree

### Resources

**Resource** - Base class for all resource types. Resources are data containers that can be saved to disk and shared between objects.

**Common Resource Types:**
- `Texture2D` - Images and sprites
- `AudioStream` - Sound files
- `PackedScene` - Saved scenes
- `Script` - GDScript files
- Custom resources (extends Resource)

**Loading Resources:**
```gdscript
# Load resource
var texture = load("res://icon.png")
var scene = load("res://enemy.tscn")

# Preload (loaded at compile time)
const SCENE = preload("res://enemy.tscn")

# Instance a scene
var enemy = SCENE.instantiate()
add_child(enemy)
```

## GDScript Basics

### Variable Types
```gdscript
var health: int = 100
var speed: float = 5.5
var player_name: String = "Hero"
var is_alive: bool = true
var position: Vector2 = Vector2(0, 0)
var items: Array = []
var inventory: Dictionary = {}
```

### Enums
```gdscript
enum State { IDLE, WALKING, JUMPING, ATTACKING }
var current_state: State = State.IDLE
```

### Exports (Inspector Variables)
```gdscript
@export var speed: float = 200.0
@export var health: int = 100
@export_range(0, 100) var volume: int = 50
@export_file("*.json") var data_file: String
```

### Node References
```gdscript
@onready var sprite = $Sprite2D
@onready var animation = $AnimationPlayer
```

## Coordinate Systems

### 2D Coordinates
- Origin (0, 0) is top-left
- X increases to the right
- Y increases downward
- Measured in pixels

### 3D Coordinates
- Right-handed coordinate system
- X: right
- Y: up
- Z: towards camera (negative Z is forward)

## Common Patterns

### Singleton (Autoload)
Create global objects accessible from anywhere:
1. Create a script (e.g., `global.gd`)
2. Project → Project Settings → Autoload
3. Add the script with a name (e.g., "Global")
4. Access anywhere: `Global.some_function()`

### State Machines
```gdscript
enum State { IDLE, WALK, JUMP }
var state = State.IDLE

func _physics_process(delta):
    match state:
        State.IDLE:
            handle_idle()
        State.WALK:
            handle_walk()
        State.JUMP:
            handle_jump()
```

### Object Pooling
```gdscript
var pool = []
var pool_size = 10

func _ready():
    for i in pool_size:
        var obj = SCENE.instantiate()
        obj.visible = false
        add_child(obj)
        pool.append(obj)

func get_from_pool():
    for obj in pool:
        if not obj.visible:
            obj.visible = true
            return obj
    return null
```
