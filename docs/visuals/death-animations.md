---
title: Custom Death Animations
tags:
    - intermediate
---

Death animation refers to the rotation of the entity as it dies. This is accompanied by a red coloring and followed shortly after by the disappearance of the entity geometry and the appearance of the death particles.

## Cancelling Death Animations

This part will explain how to remove death animations at all.

### Teleporting the Entity

A fairly common way to remove entities without causing death effects is to teleport them into the void. This can be done from animation controllers by using `!query.is_alive` like:
`/teleport @s ~ ~-1000 ~`

Please note that this will remove all death effects, including sound, particles, loot, and the visual death of the entity.

### minecraft:instant_despawn

If you want to make entity just dissapear, you can add component group with `"minecraft:instant_despawn":{}` and run an event which will add this component group.

Please note that this will remove all death effects, including sound, particles, loot, and the visual death of the entity.

### Transformation to another entity

Similar to teleporting, the entity is triggering an entity transform on death. Use `!query.is_alive` in animation controller to send an event which will add component group with `"minecraft:transformation"` component. With this component entity will convert into another:

<CodeHeader></CodeHeader>

```json
"minecraft:transformation": {
	"into": "wiki:death_animation_entity",
	"transformation_sound" : "converted_to_zombified",
	"keep_level": true,
	"drop_inventory": true,
	"preserve_equipment": false,
	"drop_equipment": true,
	"delay": {
        "block_assist_chance": 0.0,
        "block_radius": 0,
        "block_max": 0,
        "value": 10
    }
}
```

### Cancelling the Animation

We can also cancel the rotational value of the entity, allowing the entity to die more conventionally (particles, red-coloring, loot) without the 90-degree spin.

If you need more information about triggering animations from entity death, see [this document](/animation-controllers/death-commands) on death effects.

Rotation needs to be applied to a bone parent to all other bones, with a pivot at [0,0,0], and the animation should only start when `!query.is_alive`.

Animation:

<CodeHeader></CodeHeader>

```json
"rotation" : [ 0, 0, "Math.min(Math.sqrt(Math.max(0, query.anim_time * 20 - 0.5) / 20 * 1.6), 1) * -90" ]
```

Animation Controller:

(query.all_animations_finished is only needed for respawning entities, like players)

<CodeHeader>RP/animation_controllers/custom_death.animation.controllers.json</CodeHeader>

```json
{
	"format_version": "1.10.0",
	"animation_controllers": {
		"controller.animation.player.cancel_death_animaton": {
			"initial_state": "default",
			"states": {
				"default": {
					"transitions": [
						{
							"cancel_animation": "!query.is_alive"
						}
					]
				},
				"cancel_animation": {
					"animations": ["my.animation"],
					"transitions": [
						{
							"default": "query.is_alive && query.all_animations_finished"
						}
					]
				}
			}
		}
	}
}
```

Note that you will need attach animation and animations controller in `.entity.json` file of resource pack.

## Custom Death Animations

This part will explain how to customize death animation.

### Changing Damage Color Overlay

You can remove/customize entity damage color overlay.

Before starting, you must have the basics of render controller so check out the [tutorial](/visuals/entity-visuals-intro) of render controllers.

To remove the damage overlay color of any entity you want when it gets damaged, we will use `is_hurt_color` and remove the damage overlay color when an entity receives damage from lava or fire use `on_fire_color`.
First, you need to make the rgba values to 0
Here's the example of removing the damage and fire overlay color.

<CodeHeader>RP/render_controllers/custom_death.render_controllers.json</CodeHeader>

```json
{
	"format_version": "1.8.0",
	"render_controllers": {
		"controller.render.sample": {
			"geometry": "Geometry.default",
			"materials": [{ "*": "Material.default" }],
			"textures": ["Texture.default"],
			"is_hurt_color": {},
			"on_fire_color": {}
		}
	}
}
```

The code above will remove the red damage overlay color.

You can also change the damage color overlay to different colors just by putting different values in rgba. You can check out various websites to get the rgba values of all colors.
Here's another example in which the damage color overlay becomes pink.

<CodeHeader>RP/render_controllers/custom_death.render_controllers.json</CodeHeader>

```json
{
	"format_version": "1.8.0",
	"render_controllers": {
		"controller.render.kbg": {
			"geometry": "Geometry.default",
			"materials": [{ "*": "Material.default" }],
			"textures": ["Texture.default"],
			"is_hurt_color": {
				"r": "1.0",
				"g": "0.4",
				"b": "0.7",
				"a": "0.5"
			},
			"on_fire_color": {
				"r": "1.0",
				"g": "0.4",
				"b": "0.7",
				"a": "0.5"
			}
		}
	}
}
```

### Using Damage Sensor to Trigger Instant Despawn and One Item Drop

You can use the damage_sensor component to trigger an event upon fatal damage; this event adds a particular despawning component group containing the spawn_entity and instant_despawn components. Spawn_entity with 0 wait time will drop an item just before the entity is despawned. For simple entities like furniture, which only need one item, this is very convenient.

Please note that you will have to find another work for entities with an inventory. You should also ensure that the despawn component group is not added when the entity is spawned using the entity_spawned event.

Here an example file in the BP

<CodeHeader>BP/entities/entity.json</CodeHeader>

```json
{
	"format_version": "1.14.0",
	"min_engine_version": "1.16.100",
	"minecraft:entity": {
		"description": {
			"identifier": "wiki:entity",
			"is_spawnable": true,
			"is_summonable": true,
			"is_experimental": true
		},
		"component_groups": {
			"wiki:despawn": {
				"minecraft:spawn_entity": {
					"max_wait_time": 0,
					"min_wait_time": 0,
					"spawn_item": "egg",
					"single_use": true
				},
				"minecraft:instant_despawn": {}
			}
		},
		"components": {
			"minecraft:type_family": {
				"family": ["cart", "inanimate"]
			},
			"minecraft:collision_box": {
				"width": 0.8,
				"height": 0.5
			},
			"minecraft:health": {
				"value": 8,
				"max": 8
			},
			"minecraft:physics": {},
			"minecraft:pushable": {
				"is_pushable": true,
				"is_pushable_by_piston": true
			},
			"minecraft:damage_sensor": {
				"triggers": {
					"on_damage": {
						"filters": {
							"test": "has_damage",
							"value": "fatal"
						},
						"event": "wiki:despawn",
						"target": "self"
					}
				}
			}
		},
		"events": {
			"wiki:despawn": {
				"add": {
					"component_groups": ["wiki:despawn"]
				}
			}
		}
	}
}
```

Note: You can also spawn custom spawn egg items using the `minecraft:spawn_entity` component by setting `"spawn_item"`
to be your entity's id and an affix of `spawn_egg`, and it will look something like this.

<CodeHeader>BP/entities/my_entity.json#components</CodeHeader>

```json
{
	"minecraft:spawn_entity": [
		{
			"min_wait_time": 0,
			"max_wait_time": 0,
			"spawn_item": "wiki:custom_zombie_spawn_egg",
			"single_use": true
		}
	]
}
```

If you want to drop a loot table, you can trigger an event (as shown below) and summon another entity that have this component:

<CodeHeader></CodeHeader>

```json
{
	"minecraft:behavior.drop_item_for":{
		"seconds_before_pickup":0.0,
		"cooldown":5,
		"drop_item_chance":1,
		"offering_distance":0.0,
		"minimum_teleport_distance":1024.0,
		"target_range":[
			64.0,
			64.0,
			64.0
		],
		"teleport_offset":[
			0.0,
			1.0,
			0.0
		],
		"speed_multiplier":1.0,
		"search_range":64,
		"search_height":64,
		"search_count":0,
		"goal_radius":64.0,
		"entity_types":[
			{
				"filters":{
					"test":"is_family",
					"subject":"other",
					"value":"player"
				},
				"max_dist":64
			}
		],
		"priority":1,
		"loot_table":"loot_tables/entities/example.loot_table.json",
		"time_of_day_range":[
			0.0,
			1.0
		]
	},
	"minecraft:timer": {
		"time": 2,
		"time_down_event": {
			"event": "wiki:my_despawn_event"
		}
	}
}
```

And then despawn it through adding component group with instant_despawn through `wiki:my_despawn_event`.

### Detecting Death with Commands

Death detection with commands might be useful because it don't use `player.json`

<CodeHeader>BP/functions/detecting_death/mcfunction</CodeHeader>

```
tag @a add dead
tag @e[type=player] remove dead
execute @a[tag=dead, tag=!last_dead] ~ ~ ~ summon hatchibombotar:grave
tag @a[tag=dead, tag=!last_dead] add last_dead
tag @a[tag=!dead, tag=last_dead] remove last_dead
```
This works because @a targets all players whereas @e[type=player] only targets alive players.

You can do whatever you want with that summoned entity

Don't forget to add this function into [tick.json](/commands/mcfunction.html#running-functions-through-tick-json).