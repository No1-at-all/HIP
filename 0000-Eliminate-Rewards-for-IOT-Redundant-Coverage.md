# HIP ####: Eliminate Rewards for IOT Redundant Coverage
- Author(s): [@No One At All](https://github.com/No1-at-all)
- Start Date: 2024-09-06
- Category: Economic, Technical
- Original HIP PR: 
- Tracking Issue: 
- Vote Requirements: veIOT

## Summary
Currently hotspots receive rewards for *Redundant Coverage*.  This HIP proposes to only reward the *First to Witness* in areas with *Redundant Coverage* when there are greater than *Maximum Witnesses* to a beacon.
## Motivation
- Hotspots that are less than *Minimum Distance* hexes apart are providing *Redundant Coverage*.
- Disuade people from installing multiple hotspots per location when additional rewards are the motivation.
- Encourage people to spread out to under served areas.
- Some people believe Proof of Coverage rewards are broken and alternatives should be developed.

## Stakeholders
- All IOT Hotspot owners will be affected.
- Will require NOVA involvement and development.

## Detailed Explanation

### Existing PoC Definitions and rules, not defined by this HIP
#### *Maximum Witnesses*
*Maximum Witnesses* is equal to the variable max_witnesses_per_poc, which is the maximum rewardable witnesses per beacon and is currently set to 14.  This HIP will respect and follow any future changes to this number.  

If there are *Maximum Witnesses* or fewer witnesses to a beacon then this HIP makes no witness invalidation changes or changes to the PoC rewards structure for that beacon.

#### *First to Witness*
The procedures in [HIP 83](https://github.com/helium/HIP/blob/main/0083-restore-first-to-witness.md#detailed-explanation) are used to determine which hotspot is *First to Witness* a beacon. 
#### *Minimum Distance*
*Minimum Distance* is defined as 8 resolution 11 hexes.  Hotspots asserted less than this distance apart are not rewarded when they witness each other's beacons.

When the distances between hotspots are [calculated using hexes](https://h3geo.org/docs/core-library/restable/), the hotspots are assumed to be at the center of the hex.  As the resolution number increases, the location of the hotspot is more accurate, since the size of the hex containing the hotspot decreases.
### New PoC definitions and rules implemented by this HIP
#### *Redundant Coverage*
Each witness to a beacon is checked to determine if: 
- There are hotspots that are less than *Minimum Distance* apart that also witnessed the beacon.
- The witness has not been previously invalidated with any invalidation reason.
- There are greater than *Maximum Witnesses* to the beacon.  

If each of the above conditions are met, then they are deemed to be providing *Redundant Coverage*.
#### *Invalidation Reason*
*Invalidation Reason* means that witnesses are invalidated with invalidation reason 'Redundant Coverage'
### Example invalidation function
Imagine that all of the witnesses to a beacon were stored in a list sorted by *First to Witness*.  There are greater than *Maximum Witnesses* to the beacon.  The list could be in computer memory, a spreadsheet, or written on paper, it doesn't matter for this example.
1. Start at the begining of the list.
2. Check if there are subsequent witnesses in the list that are within *Minimum Distance* and invalidate them.  If this were being done on paper, draw a line through all witnesses below the current one that are within *Minumum Distance*.
3. Choose the next valid witness in the list and repeat step 2, until the end of the list is reached.  If this was being done on paper, go to the next unlined witness, and repeat step 2 until there are no more witnesses in the list.

The intent is to have all of the *Maximum Witnesses* to a beacon be greater than or equal to *Minimum Distance* apart.  Hotspots less than *Minimum Distance* apart are invalidated.  

This is an example, it's not meant to be the exact function developers would use to invalidate witnesses. Developers are free to implement the logic in some other way or to optimize as needed.

The number of invalidated witness does not count towards the total number of witness to a beacon.  Invalidated witnesses will be replaced by the next *First to Witness*. 
### Example map - 3 hotspots in a resolution 11 hex with 7 rings
![3 hotspots res 11 ring 7.png](/files/0000/3-hotspots-res-11-ring-7.png)
#### Table #1 - Hotspot distances
|             | **A** | **B** | **C** |
| ----------- | ----- | ----- | ----- |
| **A**       |       |   7   | 14    |
| **B**       |   7   |       |  7    |
| **C**       |  14   |   7   |       |
#### Example assumptions
None of the other witnesses to the beacon are within *Minimum Distance* of each other.
### Example #1
**A** and **B** witness a beacon, but **C** does not.  There are more than *Maximum Witnesses* to the beacon.

**Table #1** shows that **A** is 7 hexes apart from **B**.  This is less than *Minimum Distance*.  

Start with the *First to Witness*.  If **A** is *First to Witness* compared to **B**, then hotspot **B** is invalidated with *Invalidation Reason*. Likewise, if **B** is *First to Witness* **A** is invalidated.
### Example #2
**A**, **B**, and **C** all witness the same beacon.  There are more than *Maximum Witnesses* to the beacon.

**Table #1** shows that **A** is 7 hexes from **B**.  **B** is 7 hexes from **C**.  **A** is 14 hexes from **C**.  **A** and **B** are redundant, and **B** and **C** are redundant.  **A** and **C** are not redundant.

Start with the *First to Witness* among the hotspots.  Suppose **A** is the *First to Witness* compared with the others. This would cause **B** to be invalidated.  **C**, would then compete via *First to Witness* to be selected.

Likewise, if **C** were *First to Witness*, **B** would be invalidated and **A** free to compete.

If **B** were *First to Witness*, then both **A** and **C** would be invalidated.
### Example #3
There are *Maximum Witnesses* or fewer witnesses to the beacon.  No hotspots are invalidated.
## Drawbacks
### May encourage false assertions
Many owners may want to falsely assert their location.  The current Antenna Classifier may catch many of these, but it also may be possible to update the classifier's parameters to align with this HIP.
### May encourage owners to turn off hotspots rather than compete
## Rationale and Alternatives
### Decrease the amount of witness redundancy 
If two or more hotspots are within *Minimum Distance*, they are more or less in the same place, they are providing *Redundant Coverage*.  If those same hotspots witness the same beacon, and they are both to be rewarded according to [HIP 83](https://github.com/helium/HIP/blob/main/0083-restore-first-to-witness.md#detailed-explanation), then they are providing more or less the same coverage as each other because:
- They are within *Minimum Distance* of each other
- They both are to be selected for rewards according to [HIP 83](https://github.com/helium/HIP/blob/main/0083-restore-first-to-witness.md#detailed-explanation)

Denying the redundant hotspots give other hotspots, that may be slower to witness, an oppurtunity to compete to be selected and decrease the amount of witness redundancy when there are greater than *Maximum Witnesses* to a beacon.

Multiple hotspots within *Minumum Distance* should not be rewarded for providing more or less the same location coverage at more or less the same time.
### Aligns with previous standards
It's common advice to new hotspot owners to install their hotspots 350-400 meters apart from one another.  This is the distance represented by *Minimum Distance*.  Hotspots that are asserted less than this distance apart may not witness each other's beacons. This HIP extends that concept. Hotspots which are less then *Minimum Distance* apart are providing *Redundant Coverage* and only the *First to Witness* shall be rewarded.

Owners may have a grievance if the distance advice given them over years changed via this HIP.
### Multiple hotspots per location
Many owners want to install multiple hotspots per location for various reasons.  This proposal would implicity allow such installations, yet discourage them by removing rewards.

For example, if your use case needs multiple hotspots per location, that's fine, but only the *First to Witness* would be eligible for POC rewards. 
### Alternatives
#### Antenna Classifier
Changing the Antenna Classifier to Denylist all hotspots in the same location regardless of asserted location would solve some of the same issues as this HIP.  It would also Denylist legitimate use cases which require multiple hotspots per location.
#### Split rewards for redundant hotspots
Instead of invalidating witnesses with *Redundant Coverage*, rewards could be split equally among the hotspots.  This would not decrease *Redundant Coverage*.
## Unresolved Questions
- Discussion may bring up additional questions.

## Deployment Impact
- Owners with low performance hotspots providing *Redundant Coverage* will likely see a reduction in rewards. 

- Other owners may see an increase in rewards with the elimination of rewards for *Redundant Coverage*.

## Success Metrics
Success can be measured by observing that hotspots within *Minimum Distance* of each other do not witness the same beacon if there are more than *Maximum Witnesses* to the beacon.

