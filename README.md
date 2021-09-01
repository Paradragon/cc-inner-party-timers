# CrossCode Inner Party Timers
This mod was integrated directly into [CCExtraDialogue](https://github.com/keanuplayz/CCExtraDialogue), for which it was originally written, and therefore became obsolete.

## Installation
---

## Mod Conflicts
---

# What does it do, exactly?
This mod adds a range of new time-related variables, which are calculated when party members **join** or **leave** the party.

## Added variables
All variables are prefixed with `ccipl.vars.<Party member name or party comboName>`. Look though [list of alphabetically sorted party combo names](#List-of-alphabetically-sorted-party-combo-names) to get correct party combo name.

By default variables have value of `null`.

 1. `totalPartyTime` - total playtime with party member or party combo.
 2. `joinedAt` - if in the party, when did party member join the party. 
 3. `leftAt` - if not in the party, when did party member leave the party.
 4. `formedAt` - if current party combo, when was this party combo formed.
 5. `disbandedAt` - if not current party combo, when was this party combo disbanded.
 
Some usage examples:
```
# total playtime with Apollo in party
ccipl.vars.Apollo.totalPartyTime


# when Apollo left player's party
ccipl.vars.Apollo.leftAt


# when Emilie joined current party
ccipl.vars.Emilie.joinedAt


# total playtime with Emilie & Joern party
ccipl.vars.Emilie-Joern.totalPartyTime


# when was Apollo & Joern party disbanded by either member leaving the party. 
ccipl.vars.Apollo-Joern.disbandedAt


# when was Emilie & Joern party formed
ccipl.vars.Emilie-Joern.formedAt


# for how long was current party formed, assuming they are still the current party
stat.player.playtime - ccipl.vars.Emilie-Joern.formedAt 


# how long since you've last played in Emilie & Apollo party, assuming they are not the current party
stat.player.playtime - ccipl.vars.Apollo-Emilie.disbandedAt
```

# Downtime tracking

The mod also provides support for downtime tracking, i.e. how long certain party 
members spent outside of your party. 

Downtime is intended to be used from inside of `commonEvents`, when to be triggered, `Event2` requires some passage of time after `Event1` completion  triggered. The idea behind these commonEvents was to simulate that something happened with characters while they were outside of your party.

For an example on how this feature can be used, refer to these files:
- [CCExtraDialogue\apollo-lukas.cces](https://github.com/keanuplayz/CCExtraDialogue/blob/master/src/dialogues/apollo-lukas.cces)
- [CCExtraDialogue\lukas-shizuka.cces](https://github.com/keanuplayz/CCExtraDialogue/blob/master/src/dialogues/lukas-shizuka.cces)

## Downtime-related variables.
 1. `trackDowntime` - **(Decimal)** determines if downtime variables will be calculated for specified member or party combo. 
 2. `lastDowntime` - how long was the last downtime for party member or party combo. It resets to `null` when downtime is started again.
 3. `maxDowntime` - how long was the longest downtime for party member or party combo. 
 4. `downtimeStart` - **(internal)** when was downtime for party member or party combo started.
 5. `downtimeEnd` - **(internal)** when did downtime for party member or party combo end.

## Downtime tracking usage

Downtime tracking variables are not calculated by default. To start tracking downtime, determine party member name or party combo name and then increment the following variable by `1`.

`ccipl.vars.<Party member name or Party combo name>.trackDowntime` 

For example, if we want to track downtime for `Emilie` and `Glasses` party, we would increment the following variable.

`ccipl.vars.Emilie-Glasses.trackDowntime`

After that, next time `Emilie` leaves your party and another member is not `Glasses`, downtime tracking for `Emilie-Glasses` party will start. 

Next time when either `Emilie` or `Glasses` joins your party again, downtime tracking will stop and values of `lastDowntime` and `maxDowntime` for their party combo will be updated.

Calculated values can then be accessed from `ccipl.vars.Emilie-Glasses.maxDowntime` and `ccipl.vars.Emilie-Glasses.lastDowntime` variables.


The condition of commonEvents, concerned about downtime, would then look similarly to this:
```
# CommonEvent that requires that Emilie and Glasses were not in the party at the same time for at least 10 minutes
"condition": "party.has.Emilie && party.has.Glasses && ccipl.vars.Emilie-Glasses.maxDowntime > 600"
```

After such commonEvent is over, at the end of such event it's advised to do one of the following:
- If commonEvents that logically follow that event are no longer concerned with downtime tracking, `.trackDowntime` should be decremented by `1` for each increment in previous event.
- If there are commonEvents that logically follow current event and they are concerned about downtime AFTER current event, both `MaxDowntime` and `LastDowntime` should be set to `0`.

When party member joins or leaves the party and `.trackDowntime` value for that party member or party combo that includes that member is equal to `0 or less`, instead of calculating downtime, all downtime-related values, including `.trackDowntime` are set to `null`.

## List of alphabetically sorted party combo names
Party combo names are formed by alphabetically sorting party member names and then concatenating them with `-` separator.

Party combo names for custom companions implicitly follow the same rule.
```
Apollo-Emilie
Apollo-Glasses
Apollo-Joern
Apollo-Luke
Apollo-Schneider
Apollo-Shizuka

Emilie-Glasses
Emilie-Joern
Emilie-Luke
Emilie-Schneider
Emilie-Shizuka

Glasses-Joern
Glasses-Luke
Glasses-Schneider
Glasses-Shizuka

Joern-Luke
Joern-Schneider
Joern-Shizuka

Luke-Schneider
Luke-Shizuka

Schneider-Shizuka
```

## Debugging
To enable console logs, in `prestart.js` change value of `printDebugLog` to `true`

All variables tracked by mod can be safely reset by invoking `ig.vars.set("ccipt", null)`
