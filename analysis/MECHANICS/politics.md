# Political System

## Overview

Liberal Crime Squad features a deep political simulation that models public opinion, government branches, elections, and legislative change. The player's ultimate goal is to shift the entire political landscape toward Elite Liberal alignment.

## Political Alignment Scale

All political entities — laws, politicians, public opinion — use a five-point alignment scale:

| Value | Alignment           |
|-------|---------------------|
| -2    | Arch-Conservative   |
| -1    | Conservative        |
| 0     | Moderate            |
| +1    | Liberal             |
| +2    | Elite Liberal       |

A special **Stalinist** alignment exists as an extreme authoritarian-left outcome.

## Laws

The game tracks 22 laws, each rated on the alignment scale (-2 to +2):

| Law Category       | Description                                    |
|--------------------|------------------------------------------------|
| Abortion           | Reproductive rights policy                     |
| Animal Research    | Animal testing and welfare regulations          |
| Police Behavior    | Law enforcement conduct standards               |
| Privacy            | Surveillance and personal privacy protections   |
| Death Penalty      | Capital punishment status                       |
| Nuclear Power      | Nuclear energy policy                           |
| Pollution          | Environmental protection standards              |
| Labor              | Workers' rights and labor protections            |
| Gay Rights         | LGBTQ+ civil rights                            |
| Corporate Law      | Business regulation and corporate accountability |
| Free Speech        | First Amendment protections                     |
| Tax Policy         | Tax structure and rates                         |
| Flag Burning       | Symbolic speech protections                     |
| Gun Control        | Firearms regulation                             |
| Women's Rights     | Gender equality and women's protections          |
| Civil Rights       | Racial and ethnic civil rights                  |
| Drugs              | Drug policy and legalization                    |
| Immigration        | Immigration policy                              |
| Elections          | Electoral reform and voting rights              |
| Military Spending  | Defense budget and military policy               |
| Torture            | Interrogation and detention practices            |
| Prisons            | Incarceration and correctional policy            |

Two additional calculated indicators — **Mood** and **Stalin** — track overall political climate and authoritarian-left drift.

## Public Opinion

Public opinion is tracked across 24+ issue categories (the `attitude[]` array), each on a 0–100 scale where higher values represent more liberal sentiment.

Each law's public mood is derived from one or more related opinion categories. For example:
- `LAW_ABORTION` is influenced by **VIEW_WOMEN**
- `LAW_GUNCONTROL` is influenced by **VIEW_GUNCONTROL**
- `LAW_LABOR` is influenced by **VIEW_SWEATSHOPS**

Not every opinion category directly corresponds to a particular law, and some laws share the same underlying opinion categories.

Public opinion shifts through:
- **Squad actions**: Raiding sites generates news stories that shift related opinion categories.
- **News coverage**: The Liberal Guardian newspaper amplifies pro-LCS stories when the player's writers control it.
- **Conservative Crime Squad**: An opposing force that generates negative counter-coverage.
- **Impact formula**: Base ±20 points, modified by story priority and LCS reputation.

### Violence and Public Opinion

Violent actions by the Liberal Crime Squad do not currently use a dedicated "political violence tolerance" stat. Instead, they influence existing public opinion categories through the normal news and event system: high‑profile violence can generate negative coverage that pushes related attitudes in a more Conservative or Arch‑Conservative direction, while carefully targeted actions that align with public sentiment may still win support.

## Government Branches

### Congress

- **Senate**: 100 senators, each with an alignment value.
- **House of Representatives**: 435 representatives, each with an alignment value.
- Bills pass based on combined House and Senate votes.
- Politicians are influenced by public opinion; moderates respond most, extremists least.
- Congress can override a presidential veto with a supermajority.

### Presidency

- **Presidential elections** occur every 4 years.
- ~1000 simulated voters determine the outcome: ~40% vote on party lines, ~20% are swing voters influenced by public mood.
- The president signs or vetoes bills.

### Supreme Court

- 9 justices, each with an alignment value.
- The Court has constitutional biases: free speech and flag burning cases favor liberal rulings; gun control cases favor conservative rulings.
- Justices can be appointed when vacancies occur.

## Elections

Elections for Senate and House occur every 2 years. The system simulates:

1. Public mood determines each voter's likelihood of choosing liberal or conservative candidates.
2. Politician alignment is assigned based on election results.
3. Over time, repeated elections shift the composition of government toward prevailing public opinion.

## Legislative Change

Each month, the political system advances:

1. **Propositions**: Laws can be put to a public vote (1000 simulated voters).
2. **Congressional bills**: House and Senate vote based on member alignments.
3. **Supreme Court decisions**: Justices rule on constitutional challenges.
4. **Law propagation**: Changed laws influence related public opinion, creating feedback loops.

## Win and Loss Conditions

### Victory

All of the following must be true:
- All executive positions held by Elite Liberals.
- Majority of laws at Elite Liberal alignment.
- Supermajorities in both House and Senate.
- Majority of Supreme Court justices are liberal.

### Defeat

- Arch-Conservatives or Stalinists gain supermajorities in both chambers and repeal the Constitution.

The political system creates a dynamic tug-of-war where the player must balance direct action, public opinion management, and long-term political strategy.
