# Activities and Base Management

## Overview

Between site operations, Liberal Crime Squad members perform daily activities from their safehouse. The activity system governs recruitment, training, fundraising, interrogation, and sleeper agent management — all resolved through skill checks during the daily cycle.

## Recruitment

### Finding Candidates

Recruitment begins with finding potential liberals. A recruiter's **Street Sense** skill determines how many candidates they can locate:

- If the base difficulty is below 10, the recruiter is guaranteed to find 1 candidate and may search for more.
- Up to 4 additional candidates (for a maximum of 5) can be found; each extra candidate requires a Street Sense roll exceeding `difficulty + (candidate_count × 2)`.

### Recruitment Difficulty

The difficulty to recruit a candidate is calculated from their resistance versus the recruiter's persuasiveness:

```
reluctance = 5 + target's (Business + Science + Religion + Law + Wisdom + Intelligence)
persuasiveness = recruiter's (Business + Science + Religion + Law + Intelligence)
difficulty = max(0, reluctance - persuasiveness)
```

**Modifiers:**

| Modifier                  | Effect                                |
|---------------------------|---------------------------------------|
| Props ($50 investment)    | -5 difficulty                         |
| Juice (Activist, 10–49)  | +1 effective Wisdom                   |
| Juice (Socialist Threat, 50–99)  | +2 + 10% of Wisdom            |
| Juice (Revolutionary, 100–199)   | +3 + 20% of Wisdom            |
| Juice (Urban Commando, 200–499)  | +4 + 30% of Wisdom            |
| Juice (Liberal Guardian, 500–999)| +5 + 40% of Wisdom            |
| Juice (Elite Liberal, 1000+)     | +6 + 50% of Wisdom            |

Maximum difficulty is capped at 18.

### Recruitment Meetings

Recruitment meetings can repeat across multiple days. Each meeting adjusts the recruit's `eagerness` level:

1. **Persuasion check**: Each meeting attempts to increase the candidate's eagerness.
2. **Conversion**: When `eagerness >= 4` and the recruiter has available subordinate capacity, the candidate can be converted.
3. **Failure at any point**: The meeting ends without progress for that day.

**Eagerness modifiers** adjust the candidate's willingness:

| Candidate Alignment | Modifier |
|---------------------|----------|
| Liberal             | 0        |
| Moderate            | -2       |
| Conservative        | -4       |

Conversion requires the recruiter to have Persuasion at or above the calculated difficulty and available capacity for new subordinates.

## Interrogation

Captured enemies can be interrogated to convert them to the liberal cause. The system uses six techniques:

### Techniques

| Technique    | Cost  | Description                                     |
|--------------|-------|-------------------------------------------------|
| **Talk**     | Free  | Verbal persuasion and ideological conversion     |
| **Restrain** | Free  | Physical restraints (+5 to escape difficulty)    |
| **Beat**     | Free  | Physical violence to break resistance            |
| **Props**    | $250  | Enhanced interrogation tools                     |
| **Drugs**    | $50   | Hallucinogens (risk of permanent health damage)  |
| **Kill**     | Free  | Execute the prisoner                             |

### Conversion Mechanics

The attack roll for conversion combines multiple factors:

```
attack = days_held + number_of_interrogators
       + skill_differences(Business, Religion, Science)
       + interrogator_Psychology - prisoner_Psychology
       + interrogator_Heart_roll - prisoner_Wisdom_roll × 2
```

### Resistance Factors

Conversion can be blocked by several prisoner defenses:

| Defense                          | Condition                                         |
|----------------------------------|---------------------------------------------------|
| **Psychological resistance**     | Prisoner Psychology > interrogator Psychology      |
| **Religious conviction**         | Prisoner Religion > interrogator (Religion + Psychology) and no drugs |
| **Business mindset**             | Prisoner Business > interrogator (Business + Psychology) and no drugs |
| **Scientific rationalism**       | Prisoner Science > interrogator (Science + Psychology) and no drugs   |

### Rapport System

Rapport tracks the relationship between interrogator and prisoner:

| Action                    | Rapport Change          |
|---------------------------|-------------------------|
| Normal beating            | -0.4 per interrogator   |
| Torture (with props)      | -3.0                    |
| Successful seduction      | +0.7                    |
| Partial conversion        | +0.2                    |
| Conversion progress       | +1.5 to lead interrogator |

**Conversion triggers** when either:
- Prisoner's Heart exceeds Wisdom + 4, or
- Rapport exceeds 4.

### Beating Mechanics

- Normal beating: Force equals the sum of all interrogators' Strength rolls.
- Torture (low-Heart interrogator with props): Force is multiplied by 5.
- Blood damage: 5–10 per beating, doubled with props.

### Drug Effects

- Drug bonus: 10 + armor bonus per application.
- Every 50 days of drug use creates a chance of cardiac arrest.
- A **First Aid** check at Formidable difficulty (13) can rescue the prisoner.
- Failed rescue results in a near-death experience that doubles the drug bonus.

### Escape

Prisoners can attempt escape if unrestrained or unguarded:

```
escape_chance: random(200) + 25 × number_of_interrogators
             < prisoner's (Intelligence + Agility + Strength)
             AND days_held ≥ 5
```

Restraints add +5 to the check difficulty.

## Training

Squad members can train skills through daily practice. Training activities pair a skill with a difficulty target:

| Activity          | Skill Trained | Difficulty   |
|-------------------|---------------|--------------|
| Writing           | Writing       | Easy (5)     |
| Tailoring         | Tailoring     | Formidable (13) |
| Art sales         | Art           | Formidable (13) |
| Music performance | Music         | Formidable (13) |

The **Teaching** skill allows experienced members to train others. A teacher's skill level determines the maximum skill they can impart to students.

## Fundraising

Several activities generate income for the LCS:

| Activity         | Skill Used    | Difficulty     | Income Source                |
|------------------|---------------|----------------|------------------------------|
| Selling art      | Art           | Formidable (13)| Art sales                    |
| Selling music    | Music         | Formidable (13)| Music sales                  |
| Selling T-shirts | Tailoring     | Formidable (13)| T-shirt sales                |
| Prostitution     | Seduction     | Varies         | Street earnings              |
| Selling brownies | Persuasion    | Varies         | Brownie sales                |
| Hacking          | Computers     | Heroic+ (15+, team)| Embezzled funds              |

### Hacking Escalation

Major hacking becomes available once the squad's combined hacking ability reaches Heroic difficulty:

- Major hacking is gated by `hack_skill + team_size - 1 >= DIFFICULTY_HEROIC (15)`.
- On each hack attempt, law-enforcement tracking is checked with `trackdif > hack_skill + random(5) - 2`, where `trackdif` depends on the hack type and is typically Superheroic to Impossible.
- There is no per-success +2 escalation loop or **Street Sense** avoidance check in this daily hacking path; each hack attempt is resolved independently at its defined difficulty.

## Sleeper Agents

Sleeper agents are liberals embedded within enemy organizations who operate covertly.

### Requirements

A character can act as a sleeper if they are:
- Alive and not hiding, hospitalized, or dating.
- Flagged as `CREATUREFLAG_SLEEPER`.
- Aligned as liberal.

### Infiltration Level

Infiltration is tracked on a 0.0 to 1.0 scale (displayed as 0–100%):

- **Monthly change**: Infiltration shifts by -2% to +6% randomly each month.
- **Scandal fallout**: Infiltration drops by -2% to +8% after risky actions.

### Sleeper Activities

| Activity                  | Category      | Description                                     |
|---------------------------|---------------|-------------------------------------------------|
| Lay Low                   | Communication | No action; maintains cover                      |
| Advocate Liberalism       | Communication | Builds liberal support within the organization  |
| Expand Network            | Communication | Recruits new contacts (requires juice)          |
| Uncover Secrets           | Espionage     | Gathers intelligence; risk of discovery          |
| Steal Money               | Espionage     | Embezzles funds from the organization           |
| Steal Equipment           | Espionage     | Acquires items from the organization            |
| Join LCS                  | Special       | Leaves undercover role and joins active squad   |

### Espionage Detection

For spy, embezzle, and scandal actions, discovery occurs when:

```
random(100) > 100 × infiltration
```

Higher infiltration means lower discovery risk.

### Embezzlement Income

Income from embezzlement scales with the sleeper's position:

| Position Level | Monthly Income             |
|----------------|----------------------------|
| CEO / Executive| 50,000 × infiltration      |
| Manager        | 5,000 × infiltration       |
| Worker         | 500 × infiltration         |

### Sleeper Warnings

Sleepers at specific positions can warn the LCS of incoming threats:

- **Police station sleepers**: Warn when `timeuntillocated == 1` (police raid imminent).
- **Corporate CEO sleepers**: Warn before corporate raids.
- **CCS/Firemen sleepers**: May prevent or delay raids.

## Expense Categories

Monthly expenses are tracked across categories:

| Category       | Description                              |
|----------------|------------------------------------------|
| Rent           | Safehouse monthly rental costs           |
| Compound       | Compound maintenance and upgrades        |
| Food           | Feeding squad members                    |
| Legal          | Attorney and legal defense fees          |
| Training       | Training materials and supplies          |
| Travel         | Transportation costs                     |
| Recruitment    | Props and recruitment expenses           |
| Manufacturing  | Item crafting materials                  |
| Shopping       | Equipment purchases                      |
| Hostages       | Prisoner maintenance costs               |
| Dating         | Relationship activities                  |
| Troublemaking  | Protest and activism supplies            |
