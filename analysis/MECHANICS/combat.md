# Combat System

## Overview

Liberal Crime Squad uses a turn-based tactical combat system. Combat occurs during site operations (raids, infiltrations) and during chases (car or foot). Each round, squad members and enemies act based on their initiative and available actions.

## Combat Resolution Flow

1. **Target Selection** — Attackers pick targets by threat level: super-enemies first, then dangerous enemies, then regular enemies.
2. **Attack Roll vs Defense Roll** — The attacker rolls their weapon skill; the defender rolls half their Dodge skill. If `attack_roll + bonus > defense_roll`, the attack hits.
3. **Hit Location** — A body part is selected using a weighted random system influenced by the success margin.
4. **Damage Calculation** — Base weapon damage is rolled, modified by strength, accuracy margin, and critical hit chance.
5. **Armor Reduction** — The target's armor absorbs damage based on body-part coverage and armor quality.
6. **Wound Application** — Remaining damage is applied as wound flags; severe hits can sever limbs or kill.

## Hit Location

Body parts are selected from a weighted random range (0–12):

| Roll Range | Body Part        | Approximate Chance |
|------------|------------------|--------------------|
| 0–2        | Legs             | ~25%               |
| 3–5        | Arms             | ~25%               |
| 6–9        | Body (torso)     | ~33%               |
| 10–12      | Head             | ~17%               |

The **success margin** shifts the selection upward (toward head/body):

| Margin         | Offset | Effect                               |
|----------------|--------|--------------------------------------|
| > 5 over dodge | +4     | Fewer limb hits                      |
| > 10 over dodge| +8     | Head and body only                   |
| > 15 over dodge| +12    | Guaranteed headshot (if head exists)  |
| Backstab       | +10    | ~2/3 body, ~1/3 head                |

If the selected body part has been severed, the system re-rolls until it finds an available part.

## Damage Calculation

### Unarmed Combat

- Hits per round: `1 + (hand_to_hand_skill / 3)`, maximum 5.
- Damage per hit: `random(5 + skill) + 1 + skill`.
- Strength modifier adds bonus damage based on the attacker's Strength.

### Armed Combat

- Base damage: `fixed_damage + random(random_damage)` per hit.
- **Critical hits** activate when `hits >= hits_required` and a random chance succeeds, using separate critical damage values.
- **Backstab bonus**: +100 fixed damage.

### Damage Modifiers

| Modifier               | Effect                                                |
|------------------------|-------------------------------------------------------|
| Accuracy margin        | `attack_roll - dodge_roll` added to damage            |
| Target's Health        | Subtracted as a damage reduction roll                  |
| Strength (melee)       | Bonus for strength-affected weapons                    |
| Hostage interference   | -random(10) if attacker or target is carrying someone  |
| Founder plot armor     | Founding LCS members take half damage                  |

## Armor System

Armor provides protection per body part:

```
effective_armor = armor_rating[bodypart] - (quality_level - 1) - (1 if damaged)
effective_armor = max(0, effective_armor) + vehicle_armor_bonus
damage_reduction = effective_armor + random(effective_armor + 1) - armor_piercing
if damage_reduction > 0: total_damage -= damage_reduction * 2
```

### Armor Properties

- **Coverage**: Armor may or may not cover head, body, arms, or legs independently.
- **Quality degradation**: Higher quality numbers reduce effectiveness.
- **Damage state**: Damaged armor loses 1 point of rating.
- **Vehicle armor**: Adds bonus protection when inside a vehicle.
- **Fire protection**: Specialized gear (e.g., firefighter bunker gear) reduces burn damage by 50–75%.

## Wound Types

Wounds are stored as bit flags and can be combined:

| Wound       | Description                        |
|-------------|------------------------------------|
| Bruised     | Blunt force trauma                 |
| Cut         | Slashing wound                     |
| Torn        | Ripping/tearing injury             |
| Shot        | Ballistic wound                    |
| Burned      | Fire or chemical burn              |
| Bleeding    | Active blood loss                  |

### Severing Thresholds

| Body Part | Damage Threshold | Result          |
|-----------|------------------|-----------------|
| Head      | 100              | Clean severance |
| Body      | 1000             | Clean severance |
| Arms      | 200              | Severed arm     |
| Legs      | 400              | Severed leg     |

Weapons with explosive or messy damage types produce "nasty" severances instead of clean ones. Critical hits may override normal thresholds.

## Critical Injuries

Certain injuries cause permanent combat penalties:

| Injury            | Penalty                  |
|-------------------|--------------------------|
| Lost eye (one)    | -0 to -2                 |
| Lost both eyes    | Additional -0 to -19     |
| Damaged lung      | -8 to -10                |
| Damaged heart     | -8 to -10                |
| Damaged spine (lower) | -100                 |
| Damaged spine (neck)  | -300                 |
| Missing ribs      | -5 per rib (cumulative)  |

## Chase Combat

Chase sequences use a separate combat system where:

- **Driver skill** replaces the Dodge stat for defense rolls.
- **Vehicles** provide `attack_bonus()` and `armor_bonus(hit_location)`.
- Drive skill checks prevent vehicle damage during maneuvers.
- Characters can fight, drive, or attempt escape during chase rounds.

## Kidnapping

Kidnapping requires:

- Target must be alive, Conservative (`align == -1`), and not already captured.
- **Unarmed**: Hand-to-hand skill check vs target's Agility.
- **Armed**: Automatic success if the weapon has the `can_take_hostages` property.
- Kidnapping is blocked if the target both wields a weapon with `protects_against_kidnapping()` and has blood > 20.
