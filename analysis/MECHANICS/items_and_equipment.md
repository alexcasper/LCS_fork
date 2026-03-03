# Items and Equipment

## Overview

Equipment in Liberal Crime Squad falls into three main categories: **weapons**, **armor**, and **clips** (ammunition). All item types are defined via XML data files and loaded at startup.

## Weapons

Weapons are defined with a rich set of attack attributes:

### Attack Properties

| Property                       | Description                                                |
|--------------------------------|------------------------------------------------------------|
| `fixed_damage`                 | Guaranteed base damage per hit                             |
| `random_damage`                | Random damage added per hit (0 to value)                   |
| `number_attacks`               | Shots or swings per round                                  |
| `successive_attacks_difficulty` | Accuracy penalty per additional attack in a round          |
| `armorpiercing`                | Reduces target's effective armor                           |
| `accuracy_bonus`               | Modifier to the attack roll                                |
| `skill`                        | Which combat skill governs this weapon                     |

### Wound Properties

Each weapon specifies which wound types it inflicts: bruises, cuts, tears, shots, or burns. Weapons also define whether they cause bleeding.

### Critical Hits

Weapons may have critical hit properties:
- `hits_required`: Minimum hits before a critical can trigger.
- `chance`: Percentage chance of critical activation.
- `critical.fixed_damage` / `critical.random_damage`: Enhanced damage on critical.

### Special Flags

| Flag                          | Description                                         |
|-------------------------------|-----------------------------------------------------|
| `can_take_hostages`           | Weapon can be used to kidnap targets                |
| `is_threatening`              | Weapon intimidates NPCs                             |
| `protects_against_kidnapping` | Wielder cannot be kidnapped                         |
| `damages_armor`               | Attacks degrade the target's armor                  |

### Severity Types

Weapons define a severity type that determines how limbs are severed:
- **Clean Off**: Surgical or precise severance.
- **Nasty Off**: Explosive or brutal severance.

## Armor

Armor provides damage reduction per body region:

### Coverage

Each armor piece specifies which body parts it covers:
- **Head** (`armor_head`)
- **Body** (`armor_body`)
- **Limbs** (`armor_limbs`)

Some armor may leave certain regions exposed (e.g., a vest covers the body but not the head or limbs).

### Quality

Armor quality is tracked on a numeric scale. Higher quality numbers represent *worse* condition:
- Quality 1: Pristine condition, full protection.
- Quality 2+: Each level above 1 reduces armor rating by 1.
- **Damaged** state: Additional -1 penalty to armor rating.

### Special Properties

- **Fire protection**: Certain armor (e.g., firefighter gear) reduces burn damage by 50–75% depending on condition.
- **Disguise value**: Some armor doubles as a disguise (police uniforms, business suits), affecting stealth and infiltration.

## Ammunition (Clips)

Weapons that use ammunition consume clips. Clip types are defined in XML and matched to compatible weapons. Characters must carry the appropriate clip type to reload their weapon during combat.

## Loot

During site operations, squads can acquire loot items — valuables, documents, evidence, and equipment found at target locations. Loot can be:
- **Sold** for funding.
- **Used** as evidence to influence news stories.
- **Equipped** if the item is a weapon or armor.

## Vehicles

Vehicles serve as squad transportation and provide combat bonuses during chases:

| Property       | Description                                    |
|----------------|------------------------------------------------|
| `attack_bonus` | Bonus to attack rolls during vehicle combat     |
| `armor_bonus`  | Damage reduction by hit location in the vehicle |
| Drive skill    | Governs vehicle handling during chases           |

Vehicles are also defined via XML data files and assigned to squads from the base management screen.
