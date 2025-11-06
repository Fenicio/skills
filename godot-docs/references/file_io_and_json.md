# Godot File I/O and JSON

## FileAccess

FileAccess is Godot's class for reading and writing files. It replaces the old File class.

### Opening Files

**Read Mode:**
```gdscript
var file = FileAccess.open("res://data/config.json", FileAccess.READ)
if file:
    var content = file.get_as_text()
    file.close()
else:
    print("Failed to open file: ", FileAccess.get_open_error())
```

**Write Mode:**
```gdscript
var file = FileAccess.open("user://save_data.json", FileAccess.WRITE)
if file:
    file.store_string("Hello World")
    file.close()
```

**File Access Modes:**
- `FileAccess.READ` - Read only
- `FileAccess.WRITE` - Write only (creates or overwrites)
- `FileAccess.READ_WRITE` - Read and write
- `FileAccess.WRITE_READ` - Write and read

### File Paths

**Resource Path (res://)** - Read-only in exported games
```gdscript
"res://data/level.json"
"res://assets/textures/player.png"
```

**User Path (user://)** - Writable location for saves
```gdscript
"user://save_game.json"
"user://settings.cfg"
```

**User path location:**
- Windows: `%APPDATA%\Godot\app_userdata\[project_name]`
- macOS: `~/Library/Application Support/Godot/app_userdata/[project_name]`
- Linux: `~/.local/share/godot/app_userdata/[project_name]`

### Reading Files

**Read Entire File as Text:**
```gdscript
var file = FileAccess.open("res://data.txt", FileAccess.READ)
var content = file.get_as_text()
file.close()
```

**Read Line by Line:**
```gdscript
var file = FileAccess.open("res://data.txt", FileAccess.READ)
while not file.eof_reached():
    var line = file.get_line()
    print(line)
file.close()
```

**Read Binary:**
```gdscript
var file = FileAccess.open("res://data.bin", FileAccess.READ)
var byte_value = file.get_8()      # Read 1 byte
var int_value = file.get_32()      # Read 4 bytes as int
var float_value = file.get_float() # Read float
var buffer = file.get_buffer(100)  # Read 100 bytes
file.close()
```

### Writing Files

**Write Text:**
```gdscript
var file = FileAccess.open("user://output.txt", FileAccess.WRITE)
file.store_string("Hello World\n")
file.store_line("This adds a newline")
file.close()
```

**Write Binary:**
```gdscript
var file = FileAccess.open("user://data.bin", FileAccess.WRITE)
file.store_8(255)           # Write byte
file.store_32(1000)         # Write int
file.store_float(3.14)      # Write float
file.store_buffer([1,2,3])  # Write byte array
file.close()
```

### File Utilities

**Check if File Exists:**
```gdscript
if FileAccess.file_exists("user://save_game.json"):
    print("Save file exists")
```

**Get File Modified Time:**
```gdscript
var modified_time = FileAccess.get_modified_time("user://save_game.json")
```

**Check for Errors:**
```gdscript
var file = FileAccess.open("res://data.json", FileAccess.READ)
if file == null:
    var error = FileAccess.get_open_error()
    match error:
        ERR_FILE_NOT_FOUND:
            print("File not found")
        ERR_FILE_CANT_OPEN:
            print("Cannot open file")
        _:
            print("Error: ", error)
```

## JSON

JSON class for parsing and stringifying JSON data.

### Parsing JSON

**Parse from String:**
```gdscript
var json_string = '{"name": "Hero", "health": 100, "items": ["sword", "shield"]}'
var json = JSON.new()
var error = json.parse(json_string)

if error == OK:
    var data = json.data
    print(data["name"])      # "Hero"
    print(data["health"])    # 100
    print(data["items"][0])  # "sword"
else:
    print("JSON Parse Error: ", json.get_error_message())
    print("At line: ", json.get_error_line())
```

**Parse from File:**
```gdscript
func load_json_file(file_path: String):
    if not FileAccess.file_exists(file_path):
        print("File not found: ", file_path)
        return null

    var file = FileAccess.open(file_path, FileAccess.READ)
    var json_string = file.get_as_text()
    file.close()

    var json = JSON.new()
    var error = json.parse(json_string)

    if error == OK:
        return json.data
    else:
        print("JSON Parse Error: ", json.get_error_message())
        return null
```

### Creating JSON

**Convert Data to JSON String:**
```gdscript
var data = {
    "player": {
        "name": "Hero",
        "level": 5,
        "position": {"x": 100, "y": 200}
    },
    "inventory": ["sword", "potion", "key"]
}

var json_string = JSON.stringify(data)
print(json_string)
```

**Pretty Print (Indented):**
```gdscript
var json_string = JSON.stringify(data, "\t")  # Tab indentation
# or
var json_string = JSON.stringify(data, "  ")  # 2-space indentation
```

**Save to File:**
```gdscript
func save_json_file(file_path: String, data: Dictionary):
    var file = FileAccess.open(file_path, FileAccess.WRITE)
    if file:
        var json_string = JSON.stringify(data, "\t")
        file.store_string(json_string)
        file.close()
        return true
    else:
        print("Failed to save file: ", FileAccess.get_open_error())
        return false
```

## Dungeon Crawler Example

### Dungeon Map JSON Structure
```json
{
    "width": 10,
    "height": 10,
    "tileSize": 32,
    "tiles": [
        [1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
        [1, 0, 0, 0, 1, 0, 0, 0, 0, 1],
        [1, 0, 1, 0, 1, 0, 1, 1, 0, 1],
        [1, 0, 1, 0, 0, 0, 0, 1, 0, 1],
        [1, 0, 1, 1, 1, 1, 0, 1, 0, 1],
        [1, 0, 0, 0, 0, 0, 0, 0, 0, 1],
        [1, 1, 1, 1, 1, 1, 1, 1, 1, 1]
    ],
    "entities": [
        {"type": "player", "x": 1, "y": 1},
        {"type": "enemy", "x": 5, "y": 3},
        {"type": "chest", "x": 8, "y": 2}
    ]
}
```

### Loading and Using Dungeon Data
```gdscript
extends Node2D

var dungeon_data = null
var tile_size = 32

func _ready():
    load_dungeon("res://data/dungeon.json")
    create_dungeon()

func load_dungeon(file_path: String):
    if not FileAccess.file_exists(file_path):
        print("Dungeon file not found!")
        return

    var file = FileAccess.open(file_path, FileAccess.READ)
    var json_string = file.get_as_text()
    file.close()

    var json = JSON.new()
    var error = json.parse(json_string)

    if error == OK:
        dungeon_data = json.data
        tile_size = dungeon_data.get("tileSize", 32)
        print("Dungeon loaded: ", dungeon_data["width"], "x", dungeon_data["height"])
    else:
        print("Failed to parse dungeon JSON: ", json.get_error_message())

func create_dungeon():
    if not dungeon_data:
        return

    var tiles = dungeon_data["tiles"]

    for y in range(tiles.size()):
        for x in range(tiles[y].size()):
            var tile_type = tiles[y][x]
            create_tile(x, y, tile_type)

    # Spawn entities
    if dungeon_data.has("entities"):
        for entity_data in dungeon_data["entities"]:
            spawn_entity(entity_data)

func create_tile(x: int, y: int, tile_type: int):
    # 0 = floor, 1 = wall
    var tile = Sprite2D.new()
    tile.position = Vector2(x * tile_size, y * tile_size)

    if tile_type == 0:
        tile.modulate = Color.GRAY  # Floor
    else:
        tile.modulate = Color.DARK_GRAY  # Wall

    add_child(tile)

func spawn_entity(entity_data: Dictionary):
    var entity_type = entity_data["type"]
    var x = entity_data["x"]
    var y = entity_data["y"]

    print("Spawning ", entity_type, " at (", x, ", ", y, ")")
    # Create and position entity...

func is_walkable(grid_x: int, grid_y: int) -> bool:
    if not dungeon_data:
        return false

    var tiles = dungeon_data["tiles"]

    if grid_y < 0 or grid_y >= tiles.size():
        return false
    if grid_x < 0 or grid_x >= tiles[grid_y].size():
        return false

    return tiles[grid_y][grid_x] == 0  # 0 = walkable floor
```

### Complete Dungeon System
```gdscript
extends Node2D

@onready var player = $Player
@onready var camera = $Camera2D

var dungeon_data = {}
var tile_size = 32

func _ready():
    load_dungeon_from_json("res://dungeons/level_01.json")
    setup_camera()

func load_dungeon_from_json(path: String):
    var file = FileAccess.open(path, FileAccess.READ)
    if not file:
        push_error("Cannot open dungeon file: " + path)
        return

    var json_text = file.get_as_text()
    file.close()

    var json = JSON.new()
    if json.parse(json_text) == OK:
        dungeon_data = json.data
        tile_size = dungeon_data.get("tileSize", 32)
        generate_dungeon_tiles()
    else:
        push_error("Failed to parse JSON: " + json.get_error_message())

func generate_dungeon_tiles():
    var tiles = dungeon_data.get("tiles", [])
    # Generate your TileMap or sprites here
    pass

func setup_camera():
    camera.position = player.position
    camera.zoom = Vector2(2, 2)

func can_move_to(grid_pos: Vector2i) -> bool:
    var tiles = dungeon_data.get("tiles", [])
    if grid_pos.y < 0 or grid_pos.y >= tiles.size():
        return false
    if grid_pos.x < 0 or grid_pos.x >= tiles[grid_pos.y].size():
        return false
    return tiles[grid_pos.y][grid_pos.x] == 0
```

## DirAccess (Directory Operations)

**List Files in Directory:**
```gdscript
var dir = DirAccess.open("res://levels/")
if dir:
    dir.list_dir_begin()
    var file_name = dir.get_next()
    while file_name != "":
        if not dir.current_is_dir():
            print("Found file: ", file_name)
        file_name = dir.get_next()
    dir.list_dir_end()
```

**Create Directory:**
```gdscript
if not DirAccess.dir_exists_absolute("user://saves"):
    DirAccess.make_dir_absolute("user://saves")
```

**Copy/Delete Files:**
```gdscript
# Copy
DirAccess.copy_absolute("user://save1.json", "user://save_backup.json")

# Delete
DirAccess.remove_absolute("user://old_save.json")
```

## ConfigFile

For INI-style configuration files.

```gdscript
# Save config
var config = ConfigFile.new()
config.set_value("player", "name", "Hero")
config.set_value("player", "level", 5)
config.set_value("graphics", "fullscreen", true)
config.save("user://settings.cfg")

# Load config
var config = ConfigFile.new()
var err = config.load("user://settings.cfg")
if err == OK:
    var player_name = config.get_value("player", "name", "Default")
    var level = config.get_value("player", "level", 1)
    var fullscreen = config.get_value("graphics", "fullscreen", false)
```
