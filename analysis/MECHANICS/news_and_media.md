# News and Media

## Overview

The news and media system is a core feedback loop in Liberal Crime Squad. Squad actions generate news stories, which shift public opinion on specific issues, which in turn influence elections and legislative change. The player can amplify this effect through the **Liberal Guardian** newspaper and broadcast studio operations.

## News Story Generation

News stories are generated from two primary sources:

### Squad Stories

When a squad completes a site operation, the outcome is converted into a news story. The story's framing depends on:

- **Location type**: Each site produces stories relevant to its function (e.g., a corporate raid generates business-related coverage).
- **Outcome**: Positive outcomes (successful raids, freed prisoners) generate pro-LCS coverage; negative outcomes (squad captured, civilians harmed) generate anti-LCS coverage.
- **Violence level**: Excessive violence shifts story tone and can generate backlash coverage.
- **CCS actions**: Conservative Crime Squad operations generate counter-stories with conservative-favorable framing.

### Major Events

Major events are generated monthly as random occurrences that reflect the current political climate:

- Events scale in tone based on the Conservative/Liberal power balance.
- Examples include political scandals, activist incidents, and policy-related crises.
- Some events are conditioned on current law values (e.g., women's rights events depend on `law[LAW_WOMEN]`).

## Public Opinion Impact

News stories modify public opinion through the `change_public_opinion()` function. Each story targets one or more opinion categories from the `attitude[]` array (0–100 scale).

### Impact Formula

The base opinion shift is approximately ±20 points, modified by:

| Factor              | Effect                                                 |
|---------------------|--------------------------------------------------------|
| Story priority      | Higher-priority stories have larger impact              |
| LCS reputation      | Stronger reputation amplifies positive story effects    |
| Squad responsibility| Stories directly tied to squad actions carry more weight |
| Violence level      | Excessive violence can cause negative backlash          |

### Issue-Specific Impacts

Each story type maps to specific opinion categories:

| Story Context         | Opinion Categories Affected              |
|-----------------------|------------------------------------------|
| Gun-related incidents | `VIEW_GUNCONTROL` (impact/10 base)       |
| Corporate raids       | `VIEW_CORPORATECULTURE`, `VIEW_CEOSALARY`|
| Police encounters     | `VIEW_POLICEBEHAVIOR`                    |
| Environmental actions | `VIEW_POLLUTION`, `VIEW_NUCLEARPOWER`    |
| Civil rights actions  | `VIEW_CIVILRIGHTS`, `VIEW_WOMEN`         |
| CCS stories           | Reduces CCS support; increases police opinion |

### Violence Threshold

Public tolerance for political violence is calculated as:

```
threshold = VIEW_POLITICALVIOLENCE + VIEW_LIBERALCRIMESQUADPOS
```

When violent squad actions exceed this threshold, the resulting news coverage **backfires** — shifting opinion against the LCS instead of in its favor.

## The Liberal Guardian

The Liberal Guardian is the LCS's own newspaper, produced when squad members with the **Writing** skill are assigned to writing activities and the safehouse has a **printing press** compound upgrade.

### Production

- Writers use the **Writing** skill at Easy difficulty (5) to produce articles.
- Each writer contributes content; more writers produce a more impactful edition.
- The printing press must be operational (requires the compound upgrade).

### Effect

Liberal Guardian editions amplify pro-LCS story coverage:

- Stories written by LCS members receive favorable bias in framing.
- The newspaper provides a consistent channel for influencing public opinion even when no major squad operations occur.
- Coverage reaches the public through the monthly news cycle.

## Broadcast Studios

Squads can seize control of radio and television broadcast studios during site operations to deliver liberal messages directly to the public.

### Radio Broadcast

- Located at radio station sites (`SPECIAL_RADIO_BROADCASTSTUDIO`).
- Requires a **Persuasion** or **Seduction** skill check.
- Shifts public opinion on targeted issues.

### Television Broadcast

- Located at news station sites (`SPECIAL_NEWS_BROADCASTSTUDIO`).
- Requires a **Persuasion** or **Seduction** skill check.
- Television broadcasts reach a wider audience and have a larger opinion impact than radio.

## News Cycle Flow

The complete news cycle operates as follows:

```
Squad Actions → Story Generation → Story Priority Ranking
                                         ↓
                              Public Opinion Shifts
                                         ↓
                              Political Alignment Changes
                                         ↓
                              Election Outcomes / Law Changes
                                         ↓
                              Game World Feedback
```

Each month, accumulated stories are processed, public opinion is updated, and the political system responds to the new opinion landscape. This creates a strategic loop where the player must carefully choose targets and methods to maximize favorable media coverage.

## Counter-Coverage

The **Conservative Crime Squad** (CCS) operates as the player's media rival:

- CCS actions generate conservative-favorable news stories.
- CCS stories can reduce public support for liberal causes.
- CCS broadcasts and propaganda directly counteract LCS media efforts.
- The player must outpace CCS media influence to maintain positive public opinion trends.

## Story Components

Each news story is assembled from several components:

| Component     | Description                                          |
|---------------|------------------------------------------------------|
| **Headline**  | Generated from the story type and location           |
| **Body text** | Describes the squad's actions and their consequences |
| **Ads**       | Advertisements placed alongside stories              |
| **Layout**    | Page formatting for the newspaper display            |
| **Illustrations** | News graphics selected based on story category  |

The newspaper display uses pre-rendered pixel art from `.cpc` files for headers and illustrations, creating a visual newspaper presentation within the terminal interface.
