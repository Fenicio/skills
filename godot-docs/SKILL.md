---
name: godot-docs
description: This skill provides comprehensive Godot Engine class documentation and usage examples. This skill should be used when users are working on Godot game development projects, mention Godot classes (like Node2D, CharacterBody2D, Camera2D), ask about Godot-specific functionality (like signals, scenes, Input handling), or need help implementing Godot game mechanics (like movement controllers, dungeon systems, physics, etc.).
---

# Godot Docs

## Overview

This skill provides comprehensive documentation for the Godot Engine, including class references, usage examples, and common patterns. Use this skill when developing games or applications in Godot to access detailed information about nodes, classes, input handling, file I/O, and game development patterns.

## When to Use This Skill

Activate this skill when:
- Working on a Godot game development project
- User mentions Godot-specific classes (Node2D, CharacterBody2D, Camera2D, etc.)
- User asks about Godot concepts (signals, scenes, nodes, resources)
- Implementing game mechanics (player movement, camera systems, input handling)
- Loading or saving data (JSON parsing, file I/O)
- Building specific game types (dungeon crawlers, platformers, top-down games)

## Core Godot Concepts

Before diving into specific classes, understand Godot's fundamental architecture:

### Node System
Everything in Godot is a node organized in a tree structure. Every game object inherits from the base `Node` class.

### Scenes
Scenes are reusable collections of nodes saved as `.tscn` files. Any node subtree can be a scene.

### Signals
Godot's event system for communication between nodes. Define custom signals or use built-in ones.

### GDScript
Godot's Python-like scripting language. Key patterns:
```gdscript
extends Node2D

@export var speed = 200.0
@onready var sprite = $Sprite2D

func _ready():
    # Called when node enters the tree
    pass

func _process(delta):
    # Called every frame
    pass

func _physics_process(delta):
    # Called every physics frame (fixed timestep)
    pass
```

## Reference Documentation

This skill includes comprehensive reference documentation organized by topic:

### references/core_concepts.md
Contains detailed information about:
- Node system and hierarchy
- Scene management and SceneTree
- Signals and event handling
- Resources and resource loading
- GDScript fundamentals
- Coordinate systems (2D and 3D)
- Common patterns (singletons, state machines, object pooling)

**Read this file when:**
- Starting a new Godot project
- Need to understand how nodes and scenes work
- Working with signals
- Loading resources or scenes
- Need refresher on GDScript syntax

### references/2d_classes.md
Comprehensive documentation for 2D game development classes:
- **Node2D** - Base 2D node with transform
- **Camera2D** - 2D camera with zoom, limits, smoothing
- **CharacterBody2D** - Character movement with collision
- **Sprite2D** - Display textures and sprite sheets
- **AnimatedSprite2D** - Animated sprites
- **TileMap** - Grid-based level system
- **CollisionShape2D** - Collision shapes
- **Area2D** - Detection areas without collision
- **CanvasLayer** - UI layer that doesn't move with camera
- **Control nodes** - UI elements (Label, Button, TextureRect)

**Read this file when:**
- Building 2D games
- Implementing player or enemy movement
- Setting up cameras
- Working with sprites or animations
- Creating tile-based levels
- Implementing collision detection
- Building UI/HUD systems

### references/input_and_controls.md
Complete guide to input handling:
- **Input singleton** - Keyboard, mouse, gamepad input
- **Input actions** - Mapping inputs to logical actions
- **Input events** - Event-based input handling
- **WASD movement** - Multiple implementation patterns
- **Grid-based movement** - For dungeon crawlers, roguelikes
- **Smooth movement** - For action games
- **Mouse input** - Clicks, position, motion
- **Gamepad support** - Joystick and button handling

**Read this file when:**
- Implementing player controls
- Setting up WASD/arrow key movement
- Creating grid-based movement systems
- Handling mouse or touch input
- Supporting gamepad controllers
- Building turn-based or tile-based games

### references/file_io_and_json.md
File operations and JSON parsing:
- **FileAccess** - Reading and writing files
- **JSON** - Parsing and stringifying JSON data
- **File paths** - res:// and user:// paths
- **DirAccess** - Directory operations
- **ConfigFile** - INI-style configuration
- **Dungeon crawler examples** - Loading dungeon maps from JSON

**Read this file when:**
- Loading game data from JSON files
- Saving player progress
- Reading level/dungeon data
- Implementing save/load systems
- Working with configuration files
- Creating data-driven game content

## Common Use Cases

### Building a Dungeon Crawler

When building a dungeon crawler with grid-based movement and JSON level data:

1. **Read `references/2d_classes.md`** for:
   - Camera2D setup for grid-based camera movement
   - CharacterBody2D for player character
   - TileMap for dungeon tiles
   - Collision setup

2. **Read `references/input_and_controls.md`** for:
   - Grid-based WASD movement implementation
   - Input handling for turn-based movement

3. **Read `references/file_io_and_json.md`** for:
   - Loading dungeon maps from JSON files
   - Parsing level data structure
   - Checking tile walkability

### Implementing Player Movement

For smooth 8-directional movement:
- Read `references/input_and_controls.md` → "WASD Movement Examples" → "Smooth Movement"
- Read `references/2d_classes.md` → "CharacterBody2D"

For grid-based step movement:
- Read `references/input_and_controls.md` → "Grid-Based Movement"
- Read `references/2d_classes.md` → "Camera2D" for camera follow

### Setting Up a Camera System

For smooth camera following:
- Read `references/2d_classes.md` → "Camera2D" → "Smooth Camera Movement"

For grid-snapped camera:
- Read `references/2d_classes.md` → "Camera2D" → "Grid-Based Camera"

### Loading Game Data

For JSON level data:
- Read `references/file_io_and_json.md` → "Dungeon Crawler Example"

For save/load systems:
- Read `references/file_io_and_json.md` → "JSON" section

## Quick Reference

### Common Patterns

**Get Input:**
```gdscript
var input_x = Input.get_axis("ui_left", "ui_right")
var input_y = Input.get_axis("ui_up", "ui_down")
```

**Move Character:**
```gdscript
velocity = input_vector * speed
move_and_slide()
```

**Load JSON:**
```gdscript
var file = FileAccess.open("res://data.json", FileAccess.READ)
var json = JSON.new()
json.parse(file.get_as_text())
var data = json.data
file.close()
```

**Get Node:**
```gdscript
@onready var sprite = $Sprite2D
# or
var sprite = get_node("Sprite2D")
```

**Emit Signal:**
```gdscript
signal health_changed(amount)
# Later:
health_changed.emit(new_health)
```

## Working with the References

The reference files contain extensive code examples and detailed explanations. Follow this workflow:

1. **Identify the topic** - Determine what you need (movement, input, file I/O, etc.)
2. **Read the relevant reference** - Load the appropriate reference file
3. **Find specific examples** - Look for concrete code examples that match your use case
4. **Adapt to your project** - Modify the examples for your specific needs

The references are designed to be comprehensive but organized for easy searching. When unsure which file to read, start with `references/core_concepts.md` for fundamentals.

## Tips for Using This Skill

- Reference documentation includes many complete, working code examples
- Code examples use GDScript (Godot 4.x syntax)
- Examples follow Godot best practices and patterns
- Grid-based movement examples are optimized for dungeon crawlers and roguelikes
- File I/O examples show both reading from resources and writing to user directories
- Input handling covers keyboard, mouse, gamepad, and touch

## Additional Resources

When the reference documentation doesn't cover a specific topic:
- Check Godot's official documentation at docs.godotengine.org
- Look for specific nodes in the Godot editor's built-in help (F1)
- Search for GDScript-specific syntax or features

This skill focuses on the most commonly used classes and patterns for 2D game development, with particular emphasis on movement systems, input handling, and data loading.
