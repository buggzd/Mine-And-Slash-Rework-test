# Damage Mechanism

This document explains the damage mechanism in the mod.

The damage calculation in this mod is a multi-step process that starts when an entity attacks another. The process can be broken down into the following stages:
1.  **Attack Trigger**: An attack is initiated by a player or a mob, either unarmed, with a weapon, or through a spell.
2.  **Event Creation**: An `AttackInformation` object is created to hold all the data about the attack.
3.  **DamageEvent Creation**: A `DamageEvent` is created, which is the core of the damage calculation.
4.  **Stat Calculation**: The `DamageEvent` calculates the damage based on the attacker's and target's stats, using a layered system.
5.  **Damage Application**: The final damage is applied to the target, and other effects like knockback and threat are triggered.

## Attack Types

There are three main types of attacks in the mod:

### Unarmed Attack

When a player attacks without a weapon, the `unarmedAttack` method in `EntityData.java` is called. This method:
1.  Calculates and consumes energy for the attack.
2.  Gets the `WeaponDamage` stat from the attacker's `Unit` data.
3.  Creates a `DamageEvent` with the calculated damage and activates it.

### Weapon Attack

When a player attacks with a weapon, the `attackWithWeapon` method in `EntityData.java` is called. This method:
1.  Calculates and consumes energy for the attack.
2.  Calls the `attack` method of the weapon's `WeaponMechanic`. By default, this creates a `DamageEvent` and activates it, similar to an unarmed attack, but with the weapon's type and style.

### Mob Attack

When a mob attacks, the `mobBasicAttack` method in `EntityData.java` is called. This method:
1.  Has a cooldown for basic attacks.
2.  Calculates the damage based on the original damage amount, a percentage bonus, and a flat bonus from the config.
3.  Scales the damage based on the mob's level.
4.  Creates a `DamageEvent` and activates it.

## The `DamageEvent`

The `DamageEvent` is the heart of the damage calculation process. It is created for every attack and is responsible for calculating the final damage and applying it to the target.

### The `activate` method

The `activate` method is the entry point for the `DamageEvent`. It performs the following steps:

1.  **Friendly Fire Check:** It checks for friendly fire and cancels the damage if necessary.
2.  **Calculate Bonus Elemental Damage:** It calculates the damage from all elemental sources.
3.  **Dodge/Block Check:** It checks if the attack was dodged or blocked.
4.  **Damage Absorption:** It absorbs damage using mana and magic shield.
5.  **Apply Damage:** It applies the final damage to the target.
6.  **Knockback:** It handles knockback.
7.  **Threat Generation:** It generates threat for the attacker.
8.  **Send Damage Particles:** It sends damage number particles to the client.

## Stat Layers

The damage calculation uses a layered stat system. Each layer modifies the damage in a specific way. The layers are applied in the following order:

1.  **Flat Damage:** Adds flat damage to the base damage. This includes stats like "+10 Fire Damage".
2.  **Increased Damage:** Adds a percentage increase to the damage. All "Increased Damage" stats are added together before being applied. For example, if you have "+20% Fire Damage" and "+30% Fire Damage", the total increase is 50%.
3.  **More Damage:** Multiplies the damage by a factor. All "More Damage" stats are multiplicative with each other. For example, if you have "20% More Fire Damage" and "30% More Fire Damage", the total multiplier is 1.2 * 1.3 = 1.56.
4.  **Damage Taken:** Modifies the damage taken by the target. This is the final layer and is applied after all other calculations.

## Damage Conversion

Damage conversion allows you to convert damage from one element to another. This is done using stats like "X% of Physical Damage Converted to Fire Damage".

The conversion happens after flat damage is added, but before increased and more damage multipliers are applied. The converted damage becomes "bonus" damage and is not affected by the original element's damage modifiers.

## Bonus Elemental Damage

You can add bonus elemental damage to your attacks using stats like "+X Fire Damage to Attacks". This damage is added after damage conversion and is not affected by the original element's damage modifiers.

For each bonus element, a new `DamageEvent` is created and its damage is calculated independently. The final damage is the sum of the main damage and all bonus elemental damages.

---

# 伤害机制

本文档介绍了该模组中的伤害机制。

该模组中的伤害计算是一个多步骤的过程，从一个实体攻击另一个实体开始。该过程可分为以下几个阶段：
1.  **攻击触发**: 玩家或生物发起攻击，可以是徒手、使用武器或通过法术。
2.  **事件创建**: 创建一个 `AttackInformation` 对象，用于保存有关攻击的所有数据。
3.  **DamageEvent 创建**: 创建一个 `DamageEvent`，这是伤害计算的核心。
4.  **属性计算**: `DamageEvent` 使用分层系统，根据攻击者和目标的属性计算伤害。
5.  **伤害应用**: 最终伤害将应用于目标，并触发击退和威胁等其他效果。

## 攻击类型

该模组中有三种主要攻击类型：

### 徒手攻击

当玩家未使用武器攻击时，将调用 `EntityData.java` 中的 `unarmedAttack` 方法。该方法会：
1.  计算并消耗攻击所需的能量。
2.  从攻击者的 `Unit` 数据中获取 `WeaponDamage`（武器伤害）属性。
3.  使用计算出的伤害创建一个 `DamageEvent`（伤害事件）并激活它。

### 武器攻击

当玩家使用武器攻击时，将调用 `EntityData.java` 中的 `attackWithWeapon` 方法。该方法会：
1.  计算并消耗攻击所需的能量。
2.  调用武器的 `WeaponMechanic`（武器机制）的 `attack` 方法。默认情况下，这将创建一个 `DamageEvent` 并激活它，类似于徒手攻击，但会使用武器的类型和风格。

### 生物攻击

当生物攻击时，将调用 `EntityData.java` 中的 `mobBasicAttack` 方法。该方法会：
1.  拥有一个基础攻击的冷却时间。
2.  根据原始伤害量、百分比加成和配置中的固定加成计算伤害。
3.  根据生物的等级调整伤害。
4.  创建一个 `DamageEvent` 并激活它。

## `DamageEvent`（伤害事件）

`DamageEvent` 是伤害计算过程的核心。每次攻击都会创建一个 `DamageEvent`，它负责计算最终伤害并将其应用于目标。

### `activate` 方法

`activate` 方法是 `DamageEvent` 的入口点。它执行以下步骤：

1.  **友方火力检查:** 检查是否存在友方火力，并在必要时取消伤害。
2.  **计算额外元素伤害:** 计算所有元素来源的伤害。
3.  **闪避/格挡检查:** 检查攻击是否被闪避或格挡。
4.  **伤害吸收:** 使用法力和魔法护盾吸收伤害。
5.  **应用伤害:** 将最终伤害应用于目标。
6.  **击退:** 处理击退效果。
7.  **威胁生成:** 为攻击者生成威胁值。
8.  **发送伤害数字粒子:** 向客户端发送伤害数字粒子。

## 属性层

伤害计算使用分层属性系统。每一层都以特定的方式修改伤害。这些层按以下顺序应用：

1.  **固定伤害 (Flat Damage):** 在基础伤害上增加固定伤害。这包括“+10 火焰伤害”等属性。
2.  **伤害增加 (Increased Damage):** 按百分比增加伤害。所有“伤害增加”属性在应用前会先相加。例如，如果你有“+20% 火焰伤害”和“+30% 火焰伤害”，总增加值为 50%。
3.  **更多伤害 (More Damage):** 将伤害乘以一个系数。所有“更多伤害”属性相互乘算。例如，如果你有“20% 更多火焰伤害”和“30% 更多火焰伤害”，总乘数为 1.2 * 1.3 = 1.56。
4.  **受到伤害 (Damage Taken):** 修改目标受到的伤害。这是最后一层，在所有其他计算之后应用。

## 伤害转换

伤害转换允许您将伤害从一种元素转换为另一种元素。这通过“X% 物理伤害转换为火焰伤害”等属性来完成。

转换发生在增加固定伤害之后，但在应用增加和更多伤害乘数之前。转换后的伤害将变为“额外”伤害，并且不受原始元素伤害修饰符的影响。

## 额外元素伤害

您可以使用“+X 火焰伤害到攻击”等属性为您的攻击添加额外的元素伤害。此伤害在伤害转换后添加，并且不受原始元素伤害修饰符的影响。

对于每个额外元素，都会创建一个新的 `DamageEvent` 并独立计算其伤害。最终伤害是主伤害和所有额外元素伤害的总和。
