# SGHorrorTemplatePRO
Welcome
# SGHorrorTemplatePRO — Complete Documentation and Development Record

Updated **October 8, 2026**. Working project: **GPTSkill**, Unreal Engine **5.8**. Main map: `/Game/SGTools/Maps/SGMap`.

This document covers the implemented systems, inspected settings, completed tests, and outstanding validation. Values are asset defaults unless stated otherwise; placed instances can override them. Asset names, property names, IDs, and paths are preserved. The latest asset organization is reflected throughout this document.

## Contents

1. [Folders and assets](#folders-and-assets)
2. [Setup and controls](#setup-and-controls)
3. [Movement and stamina](#movement-and-stamina)
4. [Health and Game Over](#health-and-game-over)
5. [Inventory and pickups](#inventory-and-pickups)
6. [Healing and damaging inventory items](#healing-and-damaging-inventory-items)
7. [Doors, keys, and keycards](#doors-keys-and-keycards)
8. [Rotating picture door puzzle](#rotating-picture-door-puzzle)
9. [Flashlight and batteries](#flashlight-and-batteries)
10. [Ladders](#ladders)
11. [Security cameras and monitors](#security-cameras-and-monitors)
12. [Surface footsteps](#surface-footsteps)
13. [Interface and audio](#interface-and-audio)
14. [Map, history, and maintenance](#map-history-and-maintenance)
15. [Testing and distribution](#testing-and-distribution)
16. [Password keypad](#password-keypad)
17. [Corridor jumpscare](#corridor-jumpscare)
18. [Asset organization for distribution](#asset-organization-for-distribution)

## Folders and assets

All project content is under `/Game/SGTools`. Naming prefixes:

| Prefix | Asset type |
|---|---|
| BP_ | Blueprint |
| WBP_ | Widget Blueprint |
| M_ | Material |
| MI_ | Material instance |
| T_ | Texture |
| SM_ | Static mesh |
| SW_ | Sound Wave |
| PM_ | Physical Material |
| DT_ | Data Table |
| ST_ | User-defined structure |

| System | Asset path |
|---|---|
| First-person character | `/Game/SGTools/Blueprints/BP_SGCharacter` |
| Player controller | `/Game/SGTools/Blueprints/BP_SGPlayerController` |
| Camera manager | `/Game/SGTools/Blueprints/BP_SGCameraManager` |
| Game mode | `/Game/SGTools/Blueprints/BP_SGGameMode` |
| Inventory pickup | `/Game/SGTools/Blueprints/Inventory/BP_SGInventoryPickup` |
| Inventory data | `/Game/SGTools/Data/DT_SGInventoryItems` |
| Inventory structure | `/Game/SGTools/Data/ST_SGInventoryItem` |
| Inventory interface | `/Game/SGTools/UI/Inventory/WBP_SGInventory` |
| Main HUD | `/Game/SGTools/UI/WBP_SGStaminaHUD` |
| Game Over | `/Game/SGTools/UI/WBP_SGGameOver` |
| Interaction prompt | `/Game/SGTools/UI/Interaction/WBP_SGInteractionPrompt` |
| Door | `/Game/SGTools/Blueprints/Doors/BP_SGLockedDoor` |
| Rotating picture | `/Game/SGTools/Blueprints/Interaction/BP_SGRotatingPicture` |
| Flashlight pickup | `/Game/SGTools/Blueprints/Flashlight/BP_SGFlashlightPickup` |
| Battery pickup | `/Game/SGTools/Blueprints/Flashlight/BP_SGFlashlightBattery` |
| Ladder | `/Game/SGTools/Blueprints/Traversal/BP_SGClimbableLadder` |
| Contact healing pickup | `/Game/SGTools/Blueprints/Examples/BP_SGHealingPickup` |
| Damage zone | `/Game/SGTools/Blueprints/Examples/BP_SGDamageZone` |
| Security camera | `/Game/SGTools/Blueprints/Surveillance/BP_SGSecurityCamera` |
| Security monitor | `/Game/SGTools/Blueprints/Surveillance/BP_SGSecurityMonitor` |
| Footstep component | `/Game/SGTools/Blueprints/Audio/BP_SGFootstepsComponent` |
| Password keypad | `/Game/SGTools/Blueprints/Interaction/BP_SGKeypad` |
| Corridor jumpscare | `/Game/SGTools/Blueprints/Jumpscare/BP_SGCorridorJumpscare` |
| Jumpscare interface | `/Game/SGTools/UI/Jumpscare/WBP_SGJumpscare` |

Audio is organized by system under `/Game/SGTools/Audio`. Inventory textures are in `/Game/SGTools/Textures/Inventory`. Recovery files are outside the project Content directory; see [Backups](#backups).

## Setup and controls

To use the systems in another map, set its **GameMode Override** to `BP_SGGameMode`. Check the associated character, controller, and camera manager. Place a PlayerStart in an unobstructed area. The inventory starts empty; items are collected from the level.

| Input | Action |
|---|---|
| W / A / S / D | Move |
| Mouse | Look |
| Space | Jump |
| Left Shift | Sprint when stamina permits |
| E | Interact with a pickup, door, picture, ladder, monitor, or keypad |
| Tab | Open or close the inventory |
| F | Toggle the flashlight after collecting it |
| Inventory mouse input | Select an item and click Use Item |
| Any key on Game Over | Load the configured map, SGMap by default |

Movement and mouse look are blocked during climbing. Inventory slots do not receive keyboard focus, allowing Tab to close the interface after clicking a slot.

## Movement and stamina

The character uses first-person movement. WASD movement was corrected, and the cube representing the player's body was removed. Jump height was reduced from the initial version; the current numeric Jump Z Velocity was not reconfirmed during the documentation review.

Current `BP_SGCharacter` defaults:

| Property | Value | Purpose |
|---|---:|---|
| WalkSpeed | 450 | Walking speed |
| SprintSpeed | 800 | Sprinting speed |
| MaxStamina | 100 | Stamina capacity |
| StaminaDrainRate | 20 | Sprint consumption per second |
| StaminaRegenRate | 20 | Recovery per second |
| StaminaRegenDelay | 2 | Delay in seconds after sprinting stops |
| SprintRecoveryThreshold | 30 | Stamina required to leave exhaustion |

At zero stamina, the player becomes exhausted, cannot sprint, and plays the exhaustion sound. Recovery begins after two seconds and fills gradually. The original three-second delay was replaced with two seconds.

Configure `ExhaustionSound` in the character's audio settings. Default asset: `/Game/SGTools/Audio/Stamina/SW_SGExhaustion`. The green HUD bar was refined and resized; its text label was removed.

## Health and Game Over

`MaxHealth` defaults to 100. The health bar appears above the stamina bar.

- `Heal(Amount)` adds a positive amount of health, capped at maximum health, while the character is alive.
- The character's damage event subtracts health and plays `DamageSound`.
- At zero health, the character enters the dead state and displays `WBP_SGGameOver`.
- Game Over uses a background image and English text.
- `GameOverMapName` points to `/Game/SGTools/Maps/SGMap`.
- Any key on Game Over returns to the configured map.

| Example Blueprint | Behavior |
|---|---|
| BP_SGDamageZone | Example area that applies damage |
| BP_SGHealingPickup | Heals on contact when health is below maximum, then destroys itself |

For `BP_SGHealingPickup`, set `Amount` to choose the healing amount. `PickupSound`, editable under **SGTools → Health → Audio**, plays before the pickup is destroyed. Its default is `SW_SGItemUsed`. This contact pickup is separate from medicine used through the inventory.

Damage audio: `/Game/SGTools/Audio/Health/SW_SGPlayerDamage`.

## Inventory and pickups

The inventory has three tabs in this order: **Items**, **Keys**, **Documents**. It displays 12 slots per page, a slot icon, a larger inspection image, the item name, and its description. Each category has its own capacity:

| Character property | Default |
|---|---:|
| MaxItems | 12 |
| MaxKeys | 12 |
| MaxDocuments | 12 |

An already-owned ID cannot be collected again. When a category is full, the pickup remains in the level and displays **Full inventory**. Stacking multiple copies of the same ID is not implemented.

### Data Table configuration

`ST_SGInventoryItem` contains:

| Field | Purpose |
|---|---|
| DisplayName | Displayed name |
| Description | Item description |
| Category | `Items`, `Keys`, or `Documents` |
| Icon | Slot icon and inspection image texture |
| Usable | Allows the Use Item button |

In `BP_SGInventoryPickup`, set `ItemRowId` to the Data Table row name. Leave `OverrideItemData` disabled to read the table.

### Configuring a placed pickup directly

Under **SGTools → Item**:

1. Assign a unique, nonempty `ItemRowId`.
2. Enable **Override Item Data**.
3. Fill **Item Data**: name, description, category, icon, and Usable.
4. Configure the pickup mesh, placement, and collision.
5. Compile and save when editing the Blueprint asset; save the map when editing a placed instance.

Entering Item Data without enabling Override Item Data continues to use the Data Table. This caused the earlier `IronKey` issue.

IDs such as `IronKey`, `Keycard`, `Item_Medicine`, and `Item_Poison` work without table rows when local data overrides are enabled. Medicine and poison received separate IDs after both had been configured with empty IDs.

### Internal data flow

`OwnedItems` stores IDs. `ItemOverrides` stores local data by ID. `ResolveItemData` checks registered local character data first, then the Data Table. `TryAddCustomInventoryItem` registers local data and enforces duplicate and capacity rules. `RemoveInventoryItem` removes the ID and clears its associated data and effects.

After collection, the actor disappears from the level, while the character retains the required item data. Editing a pickup after collection does not update an already-collected item in that play session. Restart the test and collect it again.

### Inventory interface

The interface uses dark backgrounds, worn details, and warm accents. **Use Item** appears below the item name and shares the center alignment of the name and inspection image. It has normal, hover, and pressed states, and appears only when `Usable` is enabled. Slots and the use button do not receive keyboard focus.

Slot hover sound: `/Game/SGTools/Audio/UI/SW_SGInventoryHover`.

## Healing and damaging inventory items

Configure **SGTools → Item → Use Effect** in `BP_SGInventoryPickup`:

| Option | Behavior |
|---|---|
| UseHealthEffect | Enables healing when CausesDamage is disabled |
| CausesDamage | Enables damage, even when UseHealthEffect is disabled |
| HealthAmount | Positive healing or damage amount |

Damage takes priority when `CausesDamage` is enabled. With both flags disabled, there is no custom health effect. The Blueprint's HealthAmount default is 25; existing instances can retain 0 or another override. The medicine and poison examples in the level use 20.

Enable **Usable** for the item. For a custom ID without a Data Table row, also enable **Override Item Data**.

| Item | ID | UseHealthEffect | CausesDamage | HealthAmount |
|---|---|---|---|---:|
| Medicine | Item_Medicine | true | false | 20 |
| Poison | Item_Poison | false | true | 20 |

`ItemHealthEffects` stores effects by ID: positive values heal, negative values damage. The use flow stores the amount in `CachedHealthAmount` before removing the item, so clearing its registered data does not erase the amount before the effect is applied.

Successful use consumes the item and plays the success sound. Attempting to heal at full health keeps the item and plays the error sound. Damage follows the normal Apply Damage flow, including damage audio and death when applicable. Use an amount greater than zero.

Legacy ID behaviors remain available: `Item_MedicalVial` uses default healing, `Item_FlashlightBattery` recharges the flashlight, and `Item_Flashlight` controls the flashlight. New health effects do not depend on these reserved IDs.

Example descriptions:

```text
Medicine: A small bottle of medicine that restores health when used.
Poison: A small bottle of poison that causes damage when consumed.
Keycard: An access keycard used to unlock restricted doors.
```

## Doors, keys, and keycards

`BP_SGLockedDoor` contains `SM_Door`, `SM_DoorFrame`, a pivot, and interaction prompts. It opens away from the player and supports interaction from either side.

| Property | Purpose |
|---|---|
| IsLocked | Initial locked state |
| RequiredKeyId | Required inventory ID |
| ConsumeKeyOnUnlock | Consume the key or keycard on unlock; enabled by default |
| UnlockByPicture | Reserves key-based access for the picture puzzle; blocks keys when enabled |
| OpenAngle / OpenSpeed | Opening angle and speed |
| InteractionDistance | Interaction range |
| OpenSound / CloseSound | Opening and closing sounds |
| LockedSound / UnlockSound | Locked attempt and unlocking sounds |

### Key or keycard setup

1. Set the pickup category to **Keys** and assign a unique ID, such as `Keycard`.
2. Enable Override Item Data if using local data.
3. Set the door's Required Key Id to exactly the same ID.
4. Enable Is Locked.
5. Disable **Unlock by Picture** for a door unlocked by a key.
6. Choose whether unlocking consumes the key.

The door checks the owned ID and resolves the key's item data. A keycard does not need Usable enabled: it is applied through door interaction, rather than the inventory button.

With the correct key, the prompt displays **Use (key name)**. Without it, the door shakes slightly and plays the locked sound. Unlocking plays UnlockSound, waits **0.5 seconds**, then starts opening with OpenSound. Pressing E during the delay does not start opening early.

A locked door with an empty RequiredKeyId does not automatically grant access. Disable IsLocked for an ordinary unlocked door, or assign the door to a picture puzzle.

The latest keycard issue came from a duplicated door that still had UnlockByPicture enabled. The pickup configuration was correct; the flag was disabled on the keycard door.

Audio folder: `/Game/SGTools/Audio/Doors`. Assets: `SW_SGDoorOpen`, `SW_SGDoorClose`, `SW_SGDoorLocked`, and `SW_SGDoorUnlock`.

## Rotating picture door puzzle

`BP_SGRotatingPicture` contains a mesh, rotation audio, and an E interaction prompt.

| Property | Purpose |
|---|---|
| TargetDoor | Placed BP_SGLockedDoor controlled by the picture |
| RotationDuration | Rotation duration |
| InteractionDistance | Interaction range |
| RotateSound | Audio during rotation |
| RotationFinishedSound | Audio when rotation completes |

Set the door to start locked. Leave its Key ID empty if the puzzle does not use a key. Enable UnlockByPicture to prevent key-based access, and select the placed door in the picture's TargetDoor property.

Current cycle:

1. E rotates the picture 180 degrees with rotation audio.
2. At the inverted position, the picture unlocks the selected door.
3. The door plays its unlocking sound and opens after 0.5 seconds.
4. E again restores the picture's original position.
5. When the return completes, the door starts closing, plays CloseSound, and becomes locked.
6. The cycle can repeat; it is no longer limited to the first interaction.

The return implementation sets the locked state when closing begins, rather than after the mesh finishes closing. `CloseFromPicture` and `PictureRelockPending` were also added during development, but the connected return flow in FinishPictureRotation directly updates the door states.

UnlockFromPicture accepts the door explicitly selected in TargetDoor without requiring a second flag check. UnlockByPicture still blocks the key-based access path.

Audio: `/Game/SGTools/Audio/Interaction/SW_SGPictureRotate` and `SW_SGPictureRotationFinished`.

## Flashlight and batteries

The flashlight uses the user-imported `SM_Flashlight`. Collection registers it in the inventory and enables F to toggle it. Using the flashlight does not consume it.

| Reserved ID | Behavior |
|---|---|
| Item_Flashlight | Flashlight collection and use |
| Item_FlashlightBattery | Consumable battery used from the inventory |

Character defaults:

| Property | Value |
|---|---:|
| FlashlightDrainPerSecond | 1 |
| BatteryRechargeAmount | 100 |
| LowBatteryThreshold | 20 |
| FlashlightIntensity | 40 |

Energy drains while the light is on. Below 20%, it flickers until empty. At 0%, the flashlight switches off. The energy HUD at the right side appears while the flashlight is on.

A collected battery enters the inventory and does not recharge the flashlight immediately. **Use Item** recharges it and consumes the battery. The default adds 100 energy points, capped at maximum energy. Item success or error audio follows the result.

`BP_SGFlashlightPickup` distinguishes the flashlight ID from other item IDs. A battery placed using this Blueprint can enter the inventory even when the player already owns the flashlight.

### Appearance and animation

- The flashlight rises into view when enabled and lowers when disabled.
- Interpolation provides a small delay in light orientation.
- `/Game/SGTools/Materials/Flashlight/M_SGFlashlightBeam` provides a bright center, dark ring, and irregular halo.
- The light origin was positioned close to the camera to reduce clipping into nearby walls.
- The proximity intensity reduction that nearly extinguished the beam was removed.
- Last inspected light settings: 1400 cm range, 17-degree inner cone, 23-degree outer cone, intensity 40, inverse squared falloff enabled, and relative emitter location X=3, Y=0, Z=-2.

The proximity adjustment was compiled and checked in asset data. Its final appearance at all distances and exposure conditions has not been validated to a production AAA standard. Check near and far surfaces in light and dark conditions.

Configure `FlashlightOnSound` and `FlashlightOffSound` under **SGTools → Flashlight → Audio** on the character. Defaults are `SW_SGFlashlightOn` and `SW_SGFlashlightOff` in `/Game/SGTools/Audio/Flashlight`. These sounds follow logical state changes, rather than each low-battery flicker.

## Ladders

`BP_SGClimbableLadder` uses simple pieces to represent a vertical metal ladder.

| Property | Default |
|---|---:|
| LadderHeight | 400 cm |
| ClimbDuration | 3.2 s |
| RungSpacing | 30 cm |
| StepSoundInterval | 1 s |

Press E within the appropriate interaction area to climb or descend. The character aligns with and faces the ladder, with mouse look and movement blocked during traversal. The upper prompt was moved closer to the upper base. Rung audio has a minimum one-second interval.

Configure `RungSound`; the created asset is `/Game/SGTools/Audio/Traversal/SW_SGLadderRung`. Check clearance at the bottom and sufficient platform space at the top when placing or resizing a ladder.

## Security cameras and monitors

`BP_SGSecurityCamera` uses a fixed Scene Capture. Its early housing was simplified to a small rectangular shape, with extra pieces removed. The large blue editor camera visualization was hidden. A user-imported `SM_Camera` was subsequently assigned.

| Camera property | Purpose |
|---|---|
| CameraName | Identification name |
| TargetMonitor | Monitor receiving the feed |
| FeedWidth / FeedHeight | Capture resolution |
| Capture | View capture component |

| Monitor property | Purpose |
|---|---|
| MonitorName | Identification name |
| StartsPoweredOn | Initial power state; false by default |
| ScreenMaterialIndex | Screen mesh material slot; 1 by default |
| FeedMaterial | Material displaying the capture |
| OffMaterial | Black material used when off |
| InteractionDistance | E interaction range |

Place a camera and monitor, assign TargetMonitor, and orient the camera toward the desired area. Verify the screen material slot for the monitor mesh; slot 1 is not correct for every imported mesh. The monitor starts black when off and displays the assigned camera's capture when on. The camera does not follow the player.

The latest alignment correction attached Capture to the camera Housing and oriented it along the imported mesh's lens direction. Its relative rotation is yaw 180 degrees; its relative position is approximately X=-27.8774, Y=-7.6169, Z=35.0429 cm. The feed direction was verified in PIE. Preserve this relationship when adjusting the imported housing.

Measure Scene Capture cost when adding multiple cameras. Video recording, PTZ navigation, and selecting multiple cameras on a single monitor have not been implemented or validated.

## Surface footsteps

`BP_SGFootstepsComponent` is associated with the character. Current surfaces are **Ground, Metal, Wood, Carpet, Sand, Grass**. Grass replaced Liquid.

To configure a floor:

1. Choose a Physical Material with the matching Surface Type.
2. Assign it to the material or mesh being walked on.
3. Ensure collision supports the floor query.
4. Test walking and sprinting on the surface.

`GroundSounds`, `MetalSounds`, `WoodSounds`, `CarpetSounds`, `SandSounds`, and `GrassSounds` each reference three Sound Waves. These references were loaded and corrected. Audio follows `/Game/SGTools/Audio/Footsteps/<Surface>/SW_SGFootstep_<Surface>_01` through `_03`.

Additional component settings include Enabled, WalkStepInterval, SprintStepInterval, FootstepVolume, FootstepPitchVariation, and TraceDepth. If a floor plays the wrong surface or no sound, inspect the Physical Material and Surface Type returned by collision, along with the sound arrays.

## Interface and audio

`WBP_SGStaminaHUD` combines health and stamina bars, flashlight energy, and the crosshair. The crosshair is a 4 px white point that becomes a larger circle, approximately 12 px, while a valid interaction prompt is visible. It does not highlight every level object.

`WBP_SGInteractionPrompt` uses `T_SGInteractionKey_E`, displaying E inside a square. In-game text is English.

| Event | Property or asset |
|---|---|
| Exhaustion | Character.ExhaustionSound / SW_SGExhaustion |
| Successful item use | Character.ItemUsedSound / Audio/Inventory/SW_SGItemUsed |
| Invalid item use | Character.ItemUseErrorSound / Audio/Inventory/SW_SGItemUseError |
| Player damage | Character.DamageSound / Audio/Health/SW_SGPlayerDamage |
| Slot hover | Audio/UI/SW_SGInventoryHover |
| Contact healing pickup | HealingPickup.PickupSound |
| Door | OpenSound, CloseSound, LockedSound, UnlockSound |
| Picture | RotateSound, RotationFinishedSound |
| Ladder | RungSound |
| Flashlight | FlashlightOnSound, FlashlightOffSound |
| Keypad | KeySound, ErrorSound, SuccessSound |
| Jumpscare | AnticipationSound, ImpactSound |

## Map, history, and maintenance

### Level content

The demonstration layout uses cubes and simple geometry: a main corridor, atrium, side rooms, loop connections, and a ladder area. Its initial structure used approximately 187 pieces, spanning roughly X=-3800 to 3800 and Y=-1800 to 1800 cm. The user continued editing the level, so these figures are historical rather than a current actor inventory.

Doors, keys, items, flashlight, and other examples were placed during development. The user later requested no automatic creation of additional demo text or examples. Subsequent work should modify only the requested system.

### Development history

1. Connected the live Unreal Editor and created the initial cube.
2. Converted movement to first person and corrected WASD.
3. Added stamina, exhaustion, HUD, and reduced jump height.
4. Added health, healing and damage examples, and Game Over returning to SGMap.
5. Added a Data Table inventory with three categories and item inspection.
6. Standardized UI text in English and tab order to Items → Keys → Documents.
7. Added consumable medicine and reduced the stamina recovery delay to two seconds.
8. Added keyed doors, optional key consumption, locked shaking, audio, and opening away from the player.
9. Added E icon prompts and interaction from both door sides.
10. Added independent 12-item category limits, duplicate ID prevention, and Full inventory feedback.
11. Added flashlight collection, inventory batteries, energy, beam shaping, lag, low-battery flicker, and sounds.
12. Added the dynamic crosshair and flashlight raise/lower animation.
13. Added aligned ladder traversal, control locking, and timed rung audio.
14. Added the rotating picture door puzzle, then closing and relocking on return.
15. Added fixed security cameras and monitors; simplified housing and hidden camera visualization.
16. Added the six current footstep surfaces and corrected their audio references.
17. Built the demonstration layout from simple geometry.
18. Added placed-pickup data overrides alongside Data Table support.
19. Corrected battery collection that used flashlight ownership logic.
20. Corrected light disappearance near walls.
21. Corrected IronKey configuration to use local item data.
22. Added the 0.5-second unlock-to-open delay.
23. Corrected the picture TargetDoor flow and reversible cycle.
24. Added collection audio to BP_SGHealingPickup.
25. Added pickup-configured healing and damage effects retained in inventory data.
26. Corrected effect data being cleared before use and tested healing and damage.
27. Redesigned and centered Use Item under the name and inspection image.
28. Restored the editor layout with Outliner above Details.
29. Allowed CausesDamage to work independently and assigned unique medicine and poison IDs.
30. Corrected a keycard door inheriting UnlockByPicture.
31. Added the physical password keypad and clickable mesh buttons.
32. Added the corridor jumpscare and existing Game Over integration.
33. Aligned security capture with the imported camera lens.
34. Organized asset folders and prefixes, resolved references, and removed unused redirectors.

### Backups

Earlier recovery copies were historically kept under `/Game/SGTools/Backups`, including inventory overrides, flashlight proximity, door delay, picture, healing audio, and health effect changes. Some InventoryOverrides copies were made after adding helper variables but before migrating graphs. These were manual recovery copies, not a game save system; their continued presence should not be assumed.

The asset organization backup is outside Content at `Backups/BeforeAssetOrganization_20261008`, beside this document. It contains the saved SGTools content, Config, and GPTSkill.uproject from before the reorganization. Do not include recovery copies, temporary sources, or test files in the published package.

### Quick troubleshooting

| Problem | Check |
|---|---|
| Local item data does not appear | Override Item Data enabled, correct ID, and recollection in a new play session |
| Pickup cannot be collected | Already-owned ID, category capacity, distance, collision, and Visibility |
| Two items conflict | Duplicate or empty IDs; assign distinct IDs |
| Use Item is hidden | Item Data.Usable |
| Healing has no effect | Positive HealthAmount, health below maximum, and UseHealthEffect enabled |
| Poison causes no damage | CausesDamage enabled, positive HealthAmount, and recollection after editing |
| Keycard cannot unlock | Keys category, matching IDs, owned keycard, and UnlockByPicture disabled |
| Picture does not control the door | TargetDoor references the correct placed instance |
| Powered monitor stays black | TargetMonitor, Capture, ScreenMaterialIndex, and materials |
| Wrong or silent footsteps | Physical Material, Surface Type, collision, and sound arrays |
| Outliner layout is misplaced | Window → Load Layout → Default Editor Layout |

## Testing and distribution

### Completed runtime checks

- A custom item without a Data Table row displayed its local name, description, and icon.
- Disabling Usable hid the use button.
- Categories, duplicate IDs, and capacity limits were exercised during development.
- Custom healing of 20 changed health **60 → 80** and removed the item.
- Custom damage of 20 changed health **80 → 60** and removed the item.
- Healing at full health kept health at **100** and retained the item.
- Poison with CausesDamage=true and UseHealthEffect=false registered **-20**, changed health **100 → 80**, and was consumed.
- Inventory visuals were inspected in-game; Use Item was subsequently aligned to the name and icon center.
- The keycard pickup configuration was checked and the door's UnlockByPicture flag corrected; a complete traversal test was not repeated after that final change.
- Keypad, jumpscare, camera alignment, and asset organization checks are recorded in their sections.

Compiling and saving a Blueprint verifies compilation integrity but does not replace runtime testing. The nearby flashlight appearance and picture/door cycle did not receive a complete new visual and timing test suite during the documentation review.

### Integration and FAB preparation

The systems are intended to simplify buyer configuration through the SGTools folder, naming prefixes, editable properties, IDs, and local item data. This is not certification of FAB compliance or approval.

Before publishing, complete clean-project integration, dependency migration, packaged build testing, reference review, performance assessment, supported-version documentation, imported asset license review, and removal of temporary content and recovery copies. Creating this document does not complete those release checks.

Persistent inventory save/load, multiplayer replication, full gamepad support, input remapping, and multilingual localization have not been implemented or validated. The documented workflow uses first-person mouse and keyboard input in the current project.

### Maintaining this document

Record new properties, defaults, usage flows, and tests when a system changes. Distinguish asset defaults, placed-instance overrides, and temporary Play In Editor state.

## Password keypad

Added October 8, 2026. Asset: `/Game/SGTools/Blueprints/Interaction/BP_SGKeypad`. It uses the imported panel mesh, now named `/Game/SGTools/Meshes/Interaction/Keypad/SM_Keypad`, with clickable physical buttons and a Text Render display. The placed `SG_Keypad` replaces the original panel and references the nearby door through `TargetDoor`.

| Property | Purpose / default |
|---|---|
| Password | Numeric string; `1234` by default; preserves leading zeros |
| TargetDoor | Editable reference to the placed BP_SGLockedDoor |
| InteractionDistance | 200 cm by default |
| KeySound / ErrorSound / SuccessSound | Editable under SGTools\|Keypad\|Audio |

Door unlock and opening sounds remain configured on the door.

Look at the nearby panel and press **E**. The cursor appears; movement and mouse look are locked. Click the mesh's number buttons to display digits. **C** clears the input; **OK** confirms it. An incorrect password plays error audio, displays `ERROR` for 1.2 seconds, then clears the input. A correct password displays `OPEN`, exits panel interaction, and unlocks the door, which opens after its existing 0.5-second delay.

**E** or **Esc** exits panel interaction; moving out of range also exits it. In the editor, Esc may stop PIE depending on Unreal settings. E exits the panel without stopping the game.

The digit limit matches the Password length. Button hit areas use the panel mesh's local coordinates and follow the actor transform. Replacing the mesh or its layout requires adjusting `ClickPanelAt`. `/Game/SGTools/Materials/Interaction/Keypad/M_SGKeypadHiddenText` hides static screen lettering to prevent overlap with dynamic text without modifying the original mesh.

PIE checks: E entry, physical click entering `5`, button mapping, input `1234`, C clearing, incorrect password retaining the lock, successful unlock and opening, returning player control, and the `OPEN` display. Packaged builds and multiplayer were not validated.

## Corridor jumpscare

Added October 8, 2026. `BP_SGCorridorJumpscare`, under `/Game/SGTools/Blueprints/Jumpscare`, was placed near the selected corridor wall's end at X=650, Y=-1600, Z=150. Actor label: `SG_Corridor_Jumpscare`; Outliner folder: `SGTools/Jumpscare`. Its Trigger component has 65 × 170 × 150 cm extents, uses the Trigger profile, and overlaps the player. Move the actor to change the location; resize Trigger to change the activation area.

Player entry activates it once per instance per play session. It locks movement and look, switches off the flashlight, creates `WBP_SGJumpscare` above the HUD, and plays anticipation audio. After 0.28 seconds, `T_SGJumpscareFace` rapidly approaches the screen with shaking and impact audio. After 1.1 seconds of appearance, a 0.2-second blackout follows. The widget is removed, controls are released, and fatal damage invokes the existing Game Over system. Any-key continuation remains handled by Game Over.

Editable settings under **SGTools|Jumpscare**:

| Property | Purpose / documented default |
|---|---|
| Enabled | Allows activation |
| ScareTexture | Texture2D used for the scare |
| AnticipationSound / ImpactSound | Stage audio |
| AnticipationTime | 0.28 s |
| ScareDuration | 1.1 s |
| BlackoutDuration | 0.2 s |
| RushDuration | 0.26 s |
| SoundVolume | 0.7 |
| ShakeStrength | 14 |

Replace ScareTexture to change the visual, or replace the two sound references to change audio. Another Blueprint can call `StartJumpscare`; it still respects Enabled and one-time activation.

The visual is an original image animated in UMG. The texture was generated for this system and imported under `/Game/SGTools/Textures/Jumpscare`. The two WAVs were synthesized locally and imported under `/Game/SGTools/Audio/Jumpscare`. Source files and `create_jumpscare_audio.py` are stored beside this document.

PIE checks: actual corridor overlap, visible approach and shaking, impact stage, default duration, zero health, GameOverShown=true, jumpscare widget removal, and repeat prevention. Packaged builds and multiplayer were not validated.

## Asset organization for distribution

Completed October 8, 2026: **90 assets moved or renamed**. The final audit found **184 assets** under `/Game`, all within `/Game/SGTools`, with **zero redirectors**, **zero asset-type prefix mismatches**, and **zero missing `/Game` dependencies**.

Imported material instances now use `MI_`. The panel mesh is `SM_Keypad`, with `MI_Keypad_*` materials. The syringe icon is `T_SGIconSyringe`. The keycard surface texture is `T_KeycardSurface`; the separate `T_Keycard` icon was retained.

| Main folder | Content |
|---|---|
| Audio | Sounds by system, including Stamina and surface Footsteps |
| Blueprints | Core classes and system subfolders |
| Data | Inventory Data Table and structure |
| Maps | SGMap |
| Materials | M_ materials and MI_ instances by system |
| Meshes | SM_ assets grouped into Doors, Inventory, Flashlight, Interaction, Surveillance, and Demo |
| Physics/Footsteps | PM_ Physical Materials |
| Textures | T_ textures by system, HUD bars in UI/HUD, and instructions in Guides |
| UI | WBP_ Widget Blueprints |

Instruction materials are in `Materials/Guides`, with textures in `Textures/Guides`. Modeled level meshes are in `Meshes/Demo`: `SM_SGDemoWallOpening`, `SM_SGDemoPlatform`, and `SM_SGDemoWall`. Empty Tips and Maps/_GENERATED folders were removed. Assets were loaded and resaved, and the map was explicitly saved before removing unreferenced redirectors.

`SGTools_AssetOrganization.json` records previous and current paths. The pre-change backup is outside Content at `Backups/BeforeAssetOrganization_20261008`. It contains SGTools content, Config, and GPTSkill.uproject, and must be excluded from the published package.

Verification: SGMap initialized in PIE, the health/stamina HUD rendered, and the post-test log query returned no “Accessed None” entries. This check does not replace testing every interaction or cooking the release package.

Reference used for the organization review: [FAB Asset File Format and Structure Requirements](https://dev.epicgames.com/documentation/fab/asset-file-format-and-structure-requirements-in-fab).

The organization follows the guidance on one content root, folders by asset type or system, consistent English names, and clean redirectors. Complete submission preparation still requires validating included content, dependencies and plugins, category-required maps, lighting, warnings, and packaging. For the delivery copy, the project name should match the Pack: use **SGTools** as the distribution project name, since the working project is still **GPTSkill**. The open project was not renamed and no FAB submission was made.
