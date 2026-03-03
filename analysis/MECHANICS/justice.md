# Justice System

## Overview

When LCS members are captured by law enforcement, they enter the justice system — a detailed simulation of trials, sentencing, and imprisonment. The justice system interacts with the political system: current laws affect sentencing severity, sleeper agents can influence outcomes, and prison breaks are possible through site operations.

## Criminal Charges

LCS members accumulate criminal charges through their actions. Charges are tracked per character and determine the prosecution's case strength at trial.

| Crime Category    | Examples                                       |
|-------------------|-------------------------------------------------|
| Kidnapping        | Taking hostages during operations               |
| Murder            | Killing NPCs during site operations             |
| Terrorism         | Bombing, arson, or large-scale destruction      |
| Robbery           | Bank heists, mugging                            |
| Grand Theft Auto  | Stealing vehicles                               |
| Jury Tampering    | Interfering with legal proceedings              |
| Racketeering      | Organized criminal enterprise                   |
| Drug Dealing      | Manufacturing or distributing drugs             |
| Arson             | Setting fires during operations                 |
| Armed Assault     | Attacking with weapons                          |
| Flag Burning      | Burning the flag as political expression         |
| Treason           | Acts against the government                     |

## Trial Mechanics

### Defense Options

When a character goes to trial, the player chooses a defense strategy. Each option produces a different defense power level:

| Defense Type           | Cost    | Defense Power                                    |
|------------------------|---------|--------------------------------------------------|
| Court-appointed lawyer | Free    | `random(71)` — range 0 to 70                    |
| Ace liberal attorney   | $5,000  | `random(71) + 80` — range 80 to 150             |
| Sleeper lawyer         | Free    | `random(71) + 2 × (Law + Persuasion)`           |
| Self-defense           | Free    | `5 × (Persuasion_roll - 3) + 10 × (Law_roll - 3)` |

Sleeper judges with sufficient infiltration can also influence the trial proceedings.

### Jury and Verdict

The prosecution's strength is calculated from the severity of charges and available evidence:

```
jury = random(prosecution / 2 + 1) + prosecution / 2
```

**Verdict outcomes:**

| Condition                     | Result      |
|-------------------------------|-------------|
| Defense power > jury          | Acquittal   |
| Defense power = jury          | Hung jury   |
| Defense power < jury          | Conviction  |

A hung jury results in a retrial at a later date. Acquittal frees the character; conviction leads to sentencing.

## Sentencing

Sentences are measured in months and vary dramatically based on the crime:

### Base Sentences

| Crime              | Sentence Range (months)         |
|--------------------|---------------------------------|
| Murder             | 120 + random(241) — 10 to 30 years |
| Terrorism          | 60 + random(181) — 5 to 20 years   |
| Kidnapping         | 36 + random(18) — 3 to 4.5 years   |
| Robbery            | 30 + random(61) — 2.5 to 7.5 years |
| Jury Tampering     | 30 + random(61) — 2.5 to 7.5 years |
| Racketeering       | 12 + random(100) — 1 to 9 years    |
| Arson              | 12 + random(12) — 1 to 2 years     |
| Armed Assault      | 12 + random(1) — ~1 year           |
| Grand Theft Auto   | 6 + random(7) — 0.5 to 1 year      |

### Drug Sentencing

Drug sentences vary significantly based on the current drug law:

| Law Value | Sentence Range                                      |
|-----------|-----------------------------------------------------|
| -2 (Arch-Conservative) | 3 + random(360) — up to 30 years           |
| -1 (Conservative)      | 3 + random(120) — up to 10 years           |
| 0 (Moderate)           | 3 + random(12) — up to 1 year              |
| +1 (Liberal)           | No additional prison term for drug charges |
| +2 (Elite Liberal)     | No additional prison term for drug charges |

### Flag Burning

Flag burning sentencing depends entirely on the flag burning law:

| Law Value | Outcome                                                                 |
|-----------|-------------------------------------------------------------------------|
| -2        | For each flag burned, 50% chance to add 120 + random(241) months; in the other 50%, that count can contribute a negative sentence (special-case outcome). |
| -1        | 36 months per burned flag                                              |
| 0         | 1 month per burned flag                                                |
| +1 or higher | Acquittal (protected speech)                                        |

### Sentence Reduction

Lenient sentencing (from a sympathetic or sleeper judge) reduces sentences:

- Standard leniency: Sentence halved.
- Death penalty converted to leniency: 240 + random(120) months (20 to 30 years).

## Death Penalty

The death penalty applies only to the most severe crimes, and its availability depends on current law:

| Law Value              | Death Penalty Chance |
|------------------------|---------------------|
| -2 (Arch-Conservative) | 100% (always)      |
| -1 (Conservative)      | 67%                 |
| 0 (Moderate)           | 50%                 |
| +1 (Liberal)           | 20%                 |
| +2 (Elite Liberal)     | 0% (abolished)      |

When the death penalty is imposed, the character has 3 months until execution — creating an urgent window for a prison break or legal appeal.

## Imprisonment

Convicted characters are sent to prison, where they serve their sentence:

- Characters are removed from active play and placed in the prison location.
- Sentence time decreases monthly.
- Characters can be freed through prison break site operations.
- Prison conditions may affect character health over long sentences.

## Sleeper Influence

Sleeper agents within the justice system can significantly alter trial outcomes:

### Sleeper Judges

- Infiltration check: `infiltration × 100 ≥ random(100)`.
- If successful, the sleeper judge imposes lenient sentencing on all charges.
- Death penalty sentences are commuted to long prison terms.

### Sleeper Lawyers

- Provide enhanced defense power based on their Law and Persuasion skills.
- Defense power: `random(71) + 2 × (SKILL_LAW + SKILL_PERSUASION)`.
- Higher-skilled sleeper lawyers can approach or exceed the effectiveness of hired ace attorneys.

## Interaction with Political System

The justice system is deeply connected to the political simulation:

- **Law values** directly affect sentencing severity (drug laws, death penalty, flag burning).
- **Public opinion** influences jury sympathy (indirectly, through law changes).
- **Legislative changes** can retroactively affect the legal landscape — liberalizing drug laws reduces future drug sentences.
- **Supreme Court rulings** on constitutional issues can change which charges are prosecutable.

This creates a strategic incentive: shifting laws through political action can protect captured LCS members from harsh sentencing, while regressive laws make the justice system a formidable threat.
