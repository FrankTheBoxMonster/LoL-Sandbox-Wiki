The way that attack speed works internally is actually inverse from how we typically consider it.  Attack speed is actually determined by what Riot calls "attack delay" (and what we will also be calling "attack cooldown").  The cooldown starts when the attack animation starts, and the next attack starts when the cooldown ends.  Attack cooldown is simply 1 / total attack speed.

Attack windup is the time it takes from an attack starting to the attack going off (either creating a projectile or hitting the target immediately if the attack has no projectile).  Attack windup time is always calculated as a percent of the cooldown time.  The cooldown is reset if the attack is canceled before the windup time completes.  After the windup time, you are free to move without losing any DPS, also called "kiting".

Base attack speed has two ways of calculation, and old way and a new way.

The old way:

The game defines a global base attack delay (cooldown) of 1.6 seconds.  When converted to attack speed, this is where the default 0.625 base attack speed comes from.

Each character defines their own "attack delay offset percent".  This value is used to derive the character's base attack delay, with the formula `base attack delay = global base attack delay * (1 + attack delay offset percent)`.  This value is then inverted to get the character's base attack speed.

Total attack speed is then calculated as `total = base * (1 + bonus)`.

Example:
 * Katarina in patch 4.20 has an offset percent of -0.05
 * This gives her a base attack cooldown of `1.6 * (1 + -0.05) = 1.6 * 0.95 = 1.52 seconds per attack`
 * When inverted, this gives her a base attack speed of `1 / 1.52 = ~0.6579 attacks per second`
 * If she has 150% bonus attack speed, then this brings her to `0.6579 * (1 + 1.50) = 0.6579 * 2.50 = 1.6447 attacks per second`
 * This means that her new attack cooldown is `1 / 1.6447 = 0.608 seconds per attack`

The new way:

Each character defines a base attack speed and an attack speed ratio.  No inverses, no offset percents.

The base value is the character's starting attack speed at level 1, and the ratio is what they scale with.  There is no stat for just "bonus attack speed at level 1", instead the level 1 bonus value is baked into the base value.

Total attack speed is instead calculated as `total = base + (ratio * bonus)`.  For many characters, the two values are identical, so the formula simplifies back to the old way.  Note that this means "bonus attack speed at level 1" does not actually count as normal bonus attack speed, meaning that Yasuo Q starts the game with 0% cooldown reduction despite him having bonus attack speed at level 1.

The new way was first introduced around season 4, and completely replaced the old way by season 8.

Example:
 * Senna in patch 10.3 has a base attack speed of 0.625 and an attack speed ratio of 0.200
 * This is equivalent to having 212.5% bonus attack speed at level 1
 * This gives her a base attack cooldown of `1 / 0.625 = 1.6 seconds per attack`
 * If she has 150% bonus attack speed, then this brings her to `0.625 + (0.200 * 1.50) = 0.925 attacks per second`
 * When inverted, this gives her a new attack cooldown of `1 / 0.925 = 1.081 seconds per attack`
 * Meanwhile, Katarina has both her base value *and* her ratio equal to 0.658, so her total can be calculated as either `0.658 + (0.658 * bonus)` *or* `0.658 * (1 + bonus)`

Note that in both systems, the game also defines the following global caps to attack speeds:
 * global minimum attack cooldown = 0.4 seconds per attack (equivalent to max 2.50 attacks per second)
 * global maximum attack cooldown = 5.0 seconds per attack (equivalent to min 0.20 attacks per second)

It is possible for a character to override both attack speed caps.  Any characters that use a fixed attack speed (such as Jhin or Sion passive) accomplish this by setting the caps to the same value.

Windup times additionally have an old and new way, which can be used interchangeably with the old and new ways for attack cooldowns.

The old way:

The game defines a global base windup percent of 0.3, meaning that by default, a character's attack will complete 30% of the way through the attack cooldown.

Each character defines an "attack delay **cast** offset percent".  This value is used to calculate the character's windup percent, using the formula `windup percent = global base windup percent + attack delay cast offset percent`.

Example:
 * Katarina in patch 4.20 has a cast offset percent of -0.080701754
 * This gives her a windup percent of `0.3 + -0.080701754 = ~0.2193 --> attack fires 21.93% of the way through the cooldown`
 * As we found before, she has a base attack speed of 0.6579 attacks per second, so a base cooldown of 1.52 seconds per attack
 * This means that her windup time is equal to `1.52 * 0.2193 = 0.33333 seconds`
 * With 150% bonus attack speed, as again found before, this brings her to 1.6447 attacks per second (0.608 seconds per attack)
 * This means her new windup time is `0.608 * 0.2193 = 0.13333 seconds`

The new way:

Each character defines their own "attack cast time" and "attack total time" values.  The windup percent is then calculated from these using the formula `windup percent = attack cast time / attack total time`.

The cast and total time values are not used in any other way, and the game does not even keep the values in memory.  They are instead converted into equivalent attack delay cast offset percents, and the game then keeps using the old system.  Unlike attack speed, which had a change in both representation *and* functionality, attack windup only changed in representation.  Furthermore, these values are not dependent on the character's actual attack speed stats.  Attack speed stats can be changed during a patch without the cast or total times being modified, since the values are only used to get a relative percent from each other, and exist in isolation from anything else.

Example:
 * Ekko in patch 10.3 defines an attack cast time of 0.26 seconds and an attack total time of 1.60 seconds
 * This gets reinterpreted into a windup percent of `0.26 / 1.60 = 0.1625`
 * However, the total value implies an attack speed of 0.625, however his base attack speed is actually 0.688
 * This windup percent then functions identically to the one found under the old system

Both windup systems have an additional layer of complexity, called windup scaling.  This refers to an asymmetric distribution of bonus attack speed across the windup vs the cooldown.  Most characters have a windup scaling of 1.0, which means they can effectively ignore the following section due to the math canceling out.  Other characters have scalings in the range of 0.1 to 0.6, which causes their windups to speed up at a much slower rate compared to the rest of their attack.  The result is that their attack speed itself isn't any faster or slower, but their *windup times* are slower.  This is primarily a feeling thing, used to make the characters feel more sluggish or heavy without having slower overall attack speed.

The system for these characters is as follows:
 * Each character defines an "attack delay cast offset percent attack speed ratio" (note that this applies to both the old windup system as well as the new one)
 * The windup formula is then modified to have more steps
 * First, calculate the character's base windup time from their base attack speed (the level 1 value, not the ratio)
 * Next, calculate the character's current windup time from their total attack speed, as if they still had a 1.0 windup scaling (so far, nothing different from before)
 * Then, their actual windup time is modified to `((current windup time - base windup time) * windup scaling) + base windup time` (yes, the first part should end up negative, it all works out to a positive value in the end)
 * This effectively compresses the impact that bonus attack speed has on the reduction of windup time
 * If the character had a 1.0 windup scaling, then this step would have canceled out, as if it were simply `x - y + y`

Example:
 * Senna on patch 10.3 has a windup scaling of 0.6
 * As stated earlier, she has a base attack speed of 0.625 and an attack speed ratio of 0.200
 * She also has an attack cast time of 0.55 seconds and an attack total time of 1.60 seconds
 * This calculates to a windup percent of `0.55 / 1.60 = 0.34375 --> attack fires 34.375% through the attack cooldown
 * This gives her a base attack windup of `(1 / 0.625) * 0.34375 = 1.6 * 0.34375 = 0.55` (in this case, this just so happens to match her attack cast and total times, however this was not guaranteed to happen)
 * Without any bonus attack speed, her windup scaling has no effect, so she would stay at her base windup time
 * With 150% bonus attack speed (0.925 total), her current windup time *without windup scaling* should be `(1 / 0.925) * 0.34375 = 1.081 * 0.34375 = 0.3716 seconds`
 * The windup scaling then modifies this to `((0.3716 - 0.55) * 0.6) + 0.55 = (-0.1784 * 0.6) + 0.55 = -0.10704 + 0.55 = 0.44296 seconds`
 * This means that her windup time was reduced by 0.10704 seconds instead of 0.1784 seconds, losing 40% of the windup benefit

Note that under these circumstances, it's possible for the calculated windup time to actually become longer than the attack cooldown at very high amounts of bonus attack speed.  In these cases, the windup time simply gets capped to the attack cooldown.

One last major note:  all of the times on this page, like many others in the game, get rounded up to the next server frame.  The server ticks at 30 frames per second, which means that all times in the game have a resolution of 0.0333 seconds.  This also means that you will only see changes in attack speed and windup in small jumps, rather than a completely smooth transition with every single bit of bonus attack speed.

Example:
 * if a windup time is calculated to 0.25 seconds, this is equivalent to `0.25 * 30 fps = 7.5 server frames`
 * the game doesn't handle partial frames, so this gets rounded up to 8 server frames, equivalent to `8 / 30 fps = 0.2667 seconds`

These rules also apply to attack speed, which causes a very significant quirk (although also a significant opening for gold optimization).  Due to the inverse nature of these formulas, each frame you can cut off from your attack cooldown requires more bonus attack speed than the frame before it.  At 2.00 attack speed, your cooldown should be 0.50 seconds, or exactly 15 server frames.  Any attack speed less than 2.00 would have had to be rounded up to the next frame, taking 16 server frames.  But, 16 frames converts perfectly to 1.875 attack speed.  This means that any attack speed between 1.875 and 2.00 is effectively lost, since until you break 2.00, your effective attack speed will only be 1.875 due to the next frame rounding.

Below is a table of high attack speeds, the bonus attack speeds required to cut off each frame with 0.625 base attack speed, and the gold value for that frame (using Dagger's 300 gold = 12% bonus):

|frames|cool down|attack speed|total bonus|bonus for frame|total gold|gold for frame|total in Daggers|Daggers for frame|
|-----|-----|-----|-----|-----|-----|-----|-----|-----|
|12|0.400|2.500|300.0%|   30.7%|7500|     769|25.00|  2.56|
|13|0.433|2.308|269.2%|   26.3%|6731|     659|22.44|  2.20|
|14|0.467|2.143|242.9%|   22.9%|6072|     572|20.24|  1.91|
|15|0.500|2.000|220.0%|   20.0%|5500|     500|18.33|  1.66|
|16|0.533|1.875|200.0%|   17.6%|5000|     441|16.67|  1.47|
|17|0.567|1.765|182.4%|   15.7%|4559|     392|15.20|  1.31|
|18|0.600|1.667|166.7%|   14.1%|4167|     351|13.89|  1.17|
|19|0.633|1.579|152.6%|   12.6%|3816|     316|12.72|  1.05|
|20|0.667|1.500|140.0%|   11.4%|3500|     286|11.67|  0.96|
|21|0.700|1.429|128.6%|\<total>|3214|\<total>|10.71|\<total>|
|-----|-----|-----|-----|-----|-----|-----|-----|-----|
|42|1.400|0.714| 14.3%|    2.7%| 357|      66| 1.19|  0.22|
|43|1.433|0.698| 11.6%|    2.5%| 291|      64| 0.97|  0.21|
|44|1.467|0.682|  9.1%|    2.4%| 227|      60| 0.76|  0.20|
|45|1.500|0.667|  6.7%|    2.3%| 167|      58| 0.56|  0.20|
|46|1.533|0.652|  4.4%|    2.3%| 109|      56| 0.36|  0.18| 
|47|1.567|0.638|  2.1%|    2.1%|  53|      53| 0.18|  0.18|
|48|1.600|0.625|  0.0%|    0.0%|   0|       0| 0.00|  0.00|

Key takeaways:
 * Your first Dagger will take off 5 cooldown frames
 * The jump down from 21 frames to 20 frames takes a full Dagger (after having already taken 11 Daggers to get there in the first place)
 * The last cooldown frame takes over 2.5 Daggers alone (having also already taken the same amount of Daggers to get from 19 frames to 12 frames as it took to get from 48 frames to 19 frames)

This means you *really* shouldn't worry about getting to max attack speed, since it skyrockets in gold inefficiency.

However, there's an additional issue in the game, where it never actually gets you to 2.50 attack speed.  Something in the "next frame" rounding causes the game to always take at least 13 frames to complete an attack at 2.50 attack speed, instead of the expected 12 frames.  This means that **any attack speed over ~2.31 is *completely* wasted**.
