Create a tile map. Tile map will have to somehow:
- detect where you can purchase land.
- if you purchase land add tiles to map to display the land that you own.


## Code for Tile Map
extends TileMap
class_name WorldTileMap

var tileset = preload("res://Level/world_tile_map.tres") as TileSet
var grass_chunk = tileset.get_pattern(0)
var chunk_size = 7
var player

const PURCHASE_TILE = preload("res://Level/purchase_tile.tscn")

func _ready():
	update_map()
	player = get_tree().get_first_node_in_group('player')
	
func add_grass_chunk(position: Vector2i):
	# Add the chunk and optionally update terrain connections
	set_pattern(0, position, grass_chunk)
	set_cells_terrain_connect(0, get_used_cells(0), 0, 0)


func get_tilemap_position(position: Vector2) -> Vector2i:
	# Align the position to the nearest chunk grid
	var chunk_pos = Vector2i(
		floor(position.x / (tileset.tile_size.x * chunk_size)),
		floor(position.y / (tileset.tile_size.y * chunk_size))
	)
	return chunk_pos * chunk_size


func _get_surrounding_cells(tile: Vector2i) -> Array[Vector2i]:
	# Generate positions for the four adjacent chunks
	return [
		tile + Vector2i(1, 0),  # Right
		tile + Vector2i(-1, 0), # Left
		tile + Vector2i(0, 1),  # Down
		tile + Vector2i(0, -1)  # Up
	]


func get_empty_cells() -> Array[Vector2i]:
	var empty_chunks: Array[Vector2i] = []
	var used_chunks = get_used_chunks()
	
	# Find surrounding empty chunks
	for chunk in used_chunks:
		var surrounding_chunks = _get_surrounding_cells(chunk)
		for chunk_pos in surrounding_chunks:
			if !used_chunks.has(chunk_pos) and !empty_chunks.has(chunk_pos):
				empty_chunks.append(chunk_pos)
	
	return empty_chunks


func get_used_chunks() -> Array[Vector2i]:
	var used_chunks: Array[Vector2i] = []
	# Identify all currently used chunks
	for tile in get_used_cells(0):
		var chunk_pos = get_tilemap_position(map_to_local(tile)) / chunk_size
		if !used_chunks.has(chunk_pos):
			used_chunks.append(chunk_pos)
			
	return used_chunks
	
	
func update_map():
	# Place new chunks at all detected empty chunk positions
	var empty_chunks = get_empty_cells()
	var purhcase_tiles = get_tree().get_nodes_in_group("purchase")
	for tile in purhcase_tiles:
		tile.queue_free()

	var purchase_tiles = []
	for chunk in empty_chunks:
		var chunk_position = chunk * chunk_size  # Align to chunk_size
		spawn_purchase_tile(chunk_position)


func spawn_purchase_tile(chunk_position: Vector2i):
	var purchase_tile_new = PURCHASE_TILE.instantiate()
	get_parent().call_deferred("add_child", purchase_tile_new) # Add the purchase tile to the parent node
	purchase_tile_new.position = chunk_position * tileset.tile_size
	purchase_tile_new.purchase_tile_clicked.connect(_on_purchase_tile_clicked) # Custom signal
	await get_tree().process_frame
	await get_tree().process_frame
	
	purchase_tile_new.cost = purchase_tile_new.base_cost * (get_used_chunks().size() - 1)
	
	

func _on_purchase_tile_clicked(tile: Node2D):
	if player.wallet.money >= tile.cost:
		var position = tile.position
		tile.queue_free()  # Remove the purchase tile
		add_grass_chunk(get_tilemap_position(position))  # Spawn a grass chunk
		update_map()  # Update map to add more purchase tiles
