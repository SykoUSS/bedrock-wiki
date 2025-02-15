---
title: Ore loot tables

tags:
    - experimental
    - easy
mention:
    - ExDrill#2734
    - SyKo#4442
    - MedicalJewel105
---

## Features

This tutorial aims to show a new way of creating custom ore blocks with a proper loot table. The `minecraft:loot` component will run the specified loot table regardless of the tool used, but by adding the `match_tool` condition to your loot table you can specify what tools are required per pool.

Features:

-   Can be mined using any given item (this tutorial covers the iron pickaxe)
-   Can specify enchantments on items

Issues:

-   All items must be specified individualy
-   Does not drop XP
-   Non-player methods of breaking the block (explosions, commands, etc.) will fail to drop the loot

## Block behavior

The following block behavior can be used as a template. Don't forget to set the block's texture using `terrain_texture.json`.

<CodeHeader>BP/blocks/silver_ore.json</CodeHeader>

```json
{
	"format_version": "1.16.100",
	"minecraft:block": {
		"description": {
			"identifier": "tut:silver_ore"
		},
		"components": {
			//Basic components
			"minecraft:creative_category": {
				"category": "nature",
				"group": "itemGroup.name.ore"
			},
			"minecraft:destroy_time": 15,
			"minecraft:block_light_absorption": 15,
			"minecraft:explosion_resistance": 3,
			"minecraft:unit_cube": {},
			"minecraft:material_instances": {
				"*": {
					"texture": "silver_ore",
					"render_method": "opaque"
				}
			},
			"minecraft:loot": "loot_tables/silver_ore.json" //The component will not run the loot if the held tool has silk touch
		}
	}
}
```

## Loot Table

The example shown, displays the required components

<CodeHeader>BP/loot_tables/silver_ore.json

```json
{
	"pools": [
		{
			"rolls": 1,
			"conditions": [
				{
					"condition": "match_tool",
					"item": "minecraft:iron_pickaxe",
					"count": 1
				}
			],
			"entries": [
				{
					"type": "item",
					"name": "tut:silver_ore"
				}
			]
		}
	]
{
```

## Specifying Enchantments

If needed you can add the enchanments section to your condition, but remember each tool and level must be listed as seperate pools

<CodeHeader>Enchantments

```json
"conditions": [
	{
		"condition": "match_tool",
		"item": "minecraft:iron_pickaxe",
		"count": 1,
		"enchantments": [
			{
				"fortune": {
					"level": 1
				}
			}
		]
	}
]
```

## Result

![](/assets/images/blocks/vanilla-like-ores/result.gif)
