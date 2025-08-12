# Damage Mechanism / 伤害机制

This document explains the damage mechanism in the mod.
本文档介绍了该模组中的伤害机制。

---

## Damage Calculation Pipeline / 伤害计算流程

The final damage is calculated through a pipeline. An attack must pass through several checks before damage is calculated and applied.
最终伤害是通过一个计算流程得出的。一次攻击在计算和应用伤害之前必须经过几个检查。

1.  **Base Damage Calculation (for Skills) / 基础伤害计算 (针对技能)**: For skills, an initial "Base Damage" is calculated based on the skill's own values and the player's stats.
    对于技能，会根据技能自身的数值和玩家的属性计算一个初始的“基础伤害”。

2.  **Hit Check (Accuracy vs. Dodge) / 命中检查 (命中率 vs. 闪避率)**: The game checks if the attack successfully hits the target.
    游戏会检查攻击是否成功命中目标。

3.  **Critical Strike Check / 暴击检查**: If the attack hits, the game then checks if it's a critical strike.
    如果攻击命中，游戏接着会检查这次攻击是否为暴击。

4.  **Detailed Damage Calculation / 详细伤害计算**: The Base Damage from step 1 is then processed through various layers of stats (Increased Damage, More Damage, etc.).
    来自第一步的基础伤害接着会经过多个属性层（伤害增加、更多伤害等）的处理。

---

## 1. Skill Damage Calculation / 技能伤害计算

Unlike basic attacks, which have a simple base damage, skills have a more complex way of determining their initial damage before it enters the main calculation pipeline. This initial damage becomes the "Base Damage" for the next steps.
与基础攻击的简单基础伤害不同，技能在进入主计算流程之前，其初始伤害的确定方式更为复杂。这个初始伤害将成为后续步骤的“基础伤害”。

A skill's initial damage is composed of two main parts:
一个技能的初始伤害由两个主要部分组成：

-   **Skill's Flat Damage**: Each skill has its own base flat damage that scales with the skill's level.
    **技能的固定伤害**: 每个技能都有其自身的、随技能等级提升而增长的基础固定伤害。
-   **Damage Effectiveness (Weapon Damage Scaling)**: Most skills also gain a percentage of your **Weapon Damage** stat as extra damage. This percentage is called "Damage Effectiveness".
    **伤害效用 (武器伤害缩放)**: 大多数技能还会将你的**武器伤害**属性的一定百分比作为额外伤害。这个百分比被称为“伤害效用”。

The formula is:
公式为：

`Initial Skill Damage ("Base Damage") = (Skill's Flat Damage) + (Player's WeaponDamage Stat * Damage Effectiveness %)`
`技能初始伤害 ("基础伤害") = (技能的固定伤害) + (玩家的武器伤害属性 * 伤害效用 %)`

For example, the Fireball skill has a `spellScaling` of `(0.5F, 0.75F)`. This means its Damage Effectiveness is between 50% and 75% of your Weapon Damage stat, depending on the skill's level.
例如，火球技能的 `spellScaling` 值为 `(0.5F, 0.75F)`。这意味着它的伤害效用是你武器伤害属性的50%到75%之间，具体取决于技能等级。

---

## 2. Hit Check: Accuracy and Dodge / 命中检查：命中率与闪避

Whether an attack lands is determined by the attacker's **Accuracy** and the target's **Dodge Rating**.
攻击是否命中取决于攻击者的**命中率**和目标的**闪避等级**。

The final dodge chance is calculated as follows:
最终的闪避机率计算如下：

`Final Dodge Chance % = (Target's Dodge Rating - Attacker's Accuracy) / (Scaling Factor based on Attacker's Level)`
`最终闪避几率 % = (目标的闪避等级 - 攻击者的命中率) / (基于攻击者等级的缩放因子)`

If a random roll is less than the `Final Dodge Chance %`, the attack is dodged.
如果一个随机数小于`最终闪避机率 %`，则攻击被闪避。

---

## 3. Critical Strikes / 暴击

If an attack hits, it has a chance to be a critical strike.
如果攻击命中，它将有机会成为一次暴击。

-   **Critical Hit Chance** determines the probability of a crit.
    **暴击机率**决定了暴击的概率。
-   **Critical Damage** is a multiplier that increases the damage of a crit. The base is 150% (1.5x).
    **暴击伤害**是一个增加暴击伤害的乘数。基础值为150%（1.5倍）。

---

## 4. Detailed Damage Calculation / 详细伤害计算公式

The "Base Damage" (from the skill calculation in step 1) is processed through layered stats ("buckets" or "乘区").
来自第一步技能计算的“基础伤害”会经过分层属性（“乘区”）的处理。

### The Formula / 计算公式

`Final Damage = (Base Damage + Flat Damage) * (1 + Σ Increased Damage %) * (Π More Damage Multipliers) * (Crit Damage Multiplier) * (1 - Damage Reduction)`
`最终伤害 = (基础伤害 + 固定伤害) * (1 + Σ 伤害增加 %) * (Π 更多伤害乘数) * (暴击伤害倍率) * (1 - 伤害减免)`

### The "Buckets" / 乘区

1.  **Base Damage**: Calculated from the skill itself (see step 1).
    **基础伤害**: 由技能本身计算得出（见第一步）。
2.  **Flat Damage**: "+X Damage" stats are added to the Base Damage.
    **固定伤害**: “+X 伤害”属性被加到基础伤害上。
3.  **Increased Damage (Additive)**: All "+X% Increased Damage" stats are added together.
    **伤害增加 (加算)**: 所有“+X% 伤害增加”属性会先全部相加。
4.  **More Damage (Multiplicative)**: Each "X% More Damage" stat is a separate multiplier.
    **更多伤害 (乘算)**: 每个“X% 更多伤害”属性都是独立的乘数。
5.  **Critical Damage**: If it's a crit, the total damage is multiplied by the `Critical Damage` multiplier.
    **暴击伤害**: 如果是暴击，总伤害会乘以`暴击伤害`倍率。
6.  **Enemy Defenses**: The target's resistances and damage reduction are applied last.
    **敌人防御**: 最后应用目标的抗性和伤害减免。

###### Sources of "More" Damage / “更多”伤害的来源

It's important to note that many stats in the game that provide a "More Damage" multiplier *also* contribute to the "Increased Damage" bucket. This is a special feature of the mod's calculation.
值得注意的是，游戏中许多提供“更多伤害”乘数的属性，*同时*也会被计入“伤害增加”的总和中。这是该模组计算的一个特殊之处。

For example, a stat like **+20% Elemental Damage** from a source that is multiplicative will:
例如，一个乘算来源的 **+20% 元素伤害** 属性将会：
1.  Add `+20%` to the "Increased Damage" sum.
    在“伤害增加”的总和中增加 `+20%`。
2.  Also apply a `1.20x` "More Damage" multiplier.
    同时再应用一个 `1.20x` 的“更多伤害”乘数。

These powerful stats can be found on various items, talents, and support gems. Look for stats that are worded to be a separate multiplier, such as "Total Damage", "Projectile Damage", or "Damage to Cursed Enemies".
这些强力的属性可以在各种物品、天赋和辅助宝石上找到。请留意那些措辞上表示为独立乘数的属性，例如“总伤害”、“投射物伤害”或“对被诅咒的敌人的伤害”。

---

### Comprehensive Example (Skill) / 综合计算示例 (技能)

**Scenario / 情景:**
- Attacker uses **Fireball**.
- Fireball's Level provides:
    - **Skill's Flat Damage**: 80
    - **Damage Effectiveness**: 60%
- Attacker Stats:
    - `WeaponDamage` Stat: 100
    - `Critical Hit Chance`: 40%
    - `Critical Damage`: 180% (1.8x multiplier)
    - `+25 Fire Damage` (Flat)
    - `+50% Increased Fire Damage`
    - `+20% Increased Spell Damage`
    - `20% More Fire Damage` (More)
- Target Stats:
    - `20% Fire Damage Reduction`

**Calculation Steps / 计算步骤:**

1.  **Calculate Initial Skill Damage (Base Damage)**:
    - `Initial Damage = (Skill's Flat Damage) + (WeaponDamage * Damage Effectiveness)`
    - `Initial Damage = 80 + (100 * 0.60) = 80 + 60 = 140`
    - The "Base Damage" for our pipeline is **140**.

2.  **Hit & Crit Check**:
    - Let's assume the attack **hits** and is a **critical strike**.

3.  **Detailed Damage Calculation**:
    - **Base + Flat Damage**:
        - `140 (Base) + 25 (Flat) = 165`
    - **Increased Damage**: Fireball has "Fire" and "Spell" tags.
        - `50% (Fire) + 20% (Spell) = 70% Increased Damage`
        - `165 * (1 + 0.70) = 165 * 1.7 = 280.5`
    - **More Damage**:
        - `280.5 * 1.20 = 336.6`
    - **Critical Damage**: Apply the 1.8x multiplier.
        - `336.6 * 1.8 = 605.88`
    - **Enemy Defenses**:
        - `605.88 * (1 - 0.20) = 484.7`

The final damage dealt is **~485**.
最终造成的伤害约为 **485**。
