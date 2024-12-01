## Basic Enemy
``` gdscript
extends CharacterBody2D

const MAX_SPEED: float = 75

@onready var hitbox: Area2D = $Hitbox

func _ready() -> void:
	hitbox.area_entered.connect(on_hitbox_area_entered)
	
	
func _process(_delta: float) -> void:
	var direction = get_direction_to_player()
	velocity = MAX_SPEED * direction
	move_and_slide()
	

func get_direction_to_player() -> Vector2:
	var player_node = get_tree().get_first_node_in_group("player") as Node2D
	if player_node != null:
		return (player_node.global_position - global_position).normalized()
	return Vector2.ZERO


func on_hitbox_area_entered(_area: Area2D) -> void:
	queue_free()



```
## Sword Ability Controller
``` gdscript
	extends Node

@export var sword_ability: PackedScene

const MAX_RANGE: float = 150
@onready var timer: Timer = $Timer


func _ready() -> void:
	timer.timeout.connect(_on_timer_timeout)


func _on_timer_timeout() -> void:
	var player = get_tree().get_first_node_in_group("player") as Node2D
	if player == null:
		return

	var enemies = get_tree().get_nodes_in_group("enemy")
	enemies = enemies.filter(func(enemy: Node2D): 
		return enemy.global_position.distance_squared_to(player.global_position) < pow(MAX_RANGE, 2)
	)
	
	if enemies.size() == 0:
		return
	
	enemies.sort_custom(func(a: Node2D, b:Node2D):
		var a_distance = a.global_position.distance_squared_to(player.global_position) 
		var b_distance = b.global_position.distance_squared_to(player.global_position)
		return a_distance < b_distance
	)
	
	var sword_instance = sword_ability.instantiate() as Node2D
	player.get_parent().add_child(sword_instance)
	sword_instance.global_position = enemies[0].global_position
	sword_instance.global_position += Vector2.RIGHT.rotated(randf_range(0, TAU)) * 4
	
	var enemy_direction = enemies[0].global_position - player.global_position
	sword_instance.rotation = enemy_direction.angle()

```
## Main Camera
```gdscript
extends Camera2D

var target_position: Vector2 = Vector2.ZERO


func _ready() -> void:
	make_current()
	
func _process(delta: float) -> void:
	acquire_target()
	global_position = global_position.lerp(target_position, 1.0 - exp(-delta * 10))
		 
func acquire_target() -> void:
	var player_node = get_tree().get_first_node_in_group("player")
	if player_node != null:
		var player = player_node as Node2D
		target_position = player.global_position

```

## Player 
```gdscript
extends CharacterBody2D

const MAX_SPEED: float = 200


func _process(_delta: float) -> void:
	var movement_vector = get_movement_vector()
	var direction = movement_vector.normalized()
	velocity = direction * MAX_SPEED
	move_and_slide()
	

func get_movement_vector() -> Vector2:
	var x_movement = Input.get_action_strength("move_right") - Input.get_action_strength("move_left")
	var y_movement = Input.get_action_strength("move_down") - Input.get_action_strength("move_up")
	return Vector2(x_movement, y_movement)

```