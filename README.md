# Vanilla World of Warcraft 1.12.1 API Documentation

It's hard to come by any old WoW API documentation, only bits and pieces here and there. So I've decided to slowly fill this README with all the things that I come across in Vanilla WoW API.

There are a lot of websites and resources with similar information, but it's often mixed with newer versions of API. Before adding anything to this document I try it in the game to see if it's working \  has right arguments \ functions etc.

It is by far not the best or exhaustive documentation, and you should not refer to it as an official one. If you have any qustions or if you want to contribute, please be welcome and open an issue or a pull request.

## Navigation:

- [Events](#events)
- [Widgets](#widgets)
- [API](#api)

## Events

Events are a basic and important message passing system for WoW and its UI. The Event API and the WoW UI are built around these messages being recieved by Frames from WoW, through the use of event handlers, and by expressly registering for messages for a frame. An OnLoad event fires for a Frame during load, allowing it register for these event messages.

Here is an example of a simple event handling:

```Lua
local exampleFrame = CreateFrame("Frame")
exampleFrame:RegisterEvent("UNIT_HEALTH")
exampleFrame:SetScript("OnEvent", 
    function()
        message("Health change!")
    end
)
```

This code will create a frame and then register "UNIT_HEALTH" event to it. So every time WoW sends this event to the UI a callback function inside SetScript will fire.
You can assign and remove multiple events.


Events have predetermined arguments that you can use inside of your callback function. In newer versions of WoW handling them can be done by using (...) vararg operator in Lua. But in Vanilla WoW this is  not going to work. </br>
Every  event comes with 2 arguments: **self**, **event**.

- **self** referes to the frame on which this event was registered. In the example above **self** would refer to the exampleFrame.</br>
- **event** is the name of the event that was fired.

Some events have other arguments that are available inside of callback functions. They have percise  names **arg1**, **arg2**, ... **arg9**, so the maximum is 9 arguments.

For example, **UNIT_HEALTH** event has 3 arguments: self, event, arg1. Let's use them in the example below:</br>
```Lua
local exampleFrame = CreateFrame("Frame", "MyFrameName")
exampleFrame:RegisterEvent("UNIT_HEALTH")
exampleFrame:SetScript("OnEvent", 
    function()
        DEFAULT_CHAT_FRAME:AddMessage(self:GetName(), 1, 1, 1)
        DEFAULT_CHAT_FRAME:AddMessage(event, 1, 1, 1)
        DEFAULT_CHAT_FRAME:AddMessage(arg1, 1, 1, 1)
    end
)
```

After running this code in the client and attacking a mob, you will see 3 strings in the chat: "MyFrameName", "UNIT_HEALTH", "target".</br>
As you can see, I didn't pass any arguments inside my anonymous function, you shouldn't do it, all these arguments are probably available from the closure that was created by the event.</br>
In the list of events below I will not mention self and event as arguments, but they are always there.

Here is a list of known events:

- **UNIT_HEALTH**</br>
    Fires when the health of a unit changes.
    - arg1 - source of health change ("target", "player", "mouseover")

- **BAG_UPDATE**</br>
    Fired when a bags inventory changes.
    - arg1 - bagID (default bag is 0)

- **UNIT_INVENTORY_CHANGED**</br>
    Fired when the player equips or unequips an item.
    - arg1 - unitTarget ("player", "target")

- **PLAYER_TARGET_CHANGED**</br>
    This event is fired whenever the player's target is changed, including when the target is lost.

- **UPDATE_MOUSEOVER_UNIT**</br>
    Fired when the mouseover object needs to be updated.</br>
    Fired when the target of the "mouseover" UnitId has changed and is a 3d model. (Does not fire when UnitExists("mouseover") becomes nil, or if you mouse over a unitframe.)

- **UI_ERROR_MESSAGE**</br>
    Fired when the interface creates an error message. These are the red messages that show in the top middle of the screen. "Your inventory is full." is one example.
    - arg1 - message

- **SPELLCAST_FAILED**</br>
    Fired when a spell fails.

- **SPELLCAST_STOP**</br>
    Fired when a spell cast stops.

- **ACTIONBAR_UPDATE_STATE**</br>
    Fired when the state of anything on the actionbar changes. This includes cooldown and disabling.

- **CURRENT_SPELL_CAST_CHANGED**</br>
    Fired when the spell being cast is changed.

- **UNIT_COMBAT**</br>
    Fired when an npc or player participates in combat and takes damage
    - arg1 - unitTarget ("player", "target", ...)
    - arg2 - event (Action, Damage, etc (e.g. HEAL, DODGE, BLOCK, WOUND, MISS, PARRY, RESIST, ...))
    - arg3 - flagText (Critical/Glancing indicator (e.g. CRITICAL, CRUSHING, GLANCING))
    - arg4 - amount (The numeric damage)
    - arg5 - schoolMask (0 - physical; 1 - holy; 2 - fire; 3 - nature; 4 - frost; 5 - shadow; 6 - arcane)

- **PLAYER_ENTER_COMBAT**</br>
    Fired when a player engages auto-attack. Note that firing a gun or a spell, or getting aggro, does NOT trigger this event.</br>
    PLAYER_ENTER_COMBAT and PLAYER_LEAVE_COMBAT are for *MELEE* combat only. They fire when you initiate autoattack and when you turn it off. However, any spell or ability that does not turn on autoattack does not trigger it. Nor does it trigger when you get aggro.

- **PLAYER_LEAVE_COMBAT**</br>
    Fired when the player leaves combat through death, defeat of opponents, or an ability. Does not fire if a player flees from combat on foot.

- **UPDATE_CHAT_COLOR**</br>
    Fired when the chat colour needs to be updated. Refer to ChangeChatColor for details on the parameters.
    - arg1 - chatType (CHANNEL9 etc.)
    - arg2 - r
    - arg3 - g
    - arg4 - b

- **ADDON_LOADED**</br>
    Fires after an AddOn has been loaded.
    - arg1 - addOnName ("MyAwesomeAddon")

- **VARIABLES_LOADED**</br>
    Fired in response to the CVars, Keybindings and other associated "Blizzard" variables being loaded.

- **UPDATE_BINDINGS**</br>
    Fired when the keybindings are changed. Fired after completion of LoadBindings, SaveBindings, and SetBinding (and its derivatives).

- **UPDATE_MACROS**</br>
    Called after applying or cancelling changes to a macro's content, name and/or icon.

- **UPDATE_CHAT_WINDOWS**</br>
    Fired on load when chat settings are available for chat windows.

- **DISPLAY_SIZE_CHANGED**</br>

- **PLAYER_LOGIN**</br>
    Triggered immediately before PLAYER_ENTERING_WORLD on login and UI Reload, but NOT when entering/leaving instances.

- **PLAYER_ENTERING_WORLD**</br>
    Fires when the player logs in, /reloads the UI or zones between map instances. Basically whenever the loading screen appears.

- **MINIMAP_UPDATE_ZOOM**</br>
    Fired when the minimap scaling factor is changed. This happens, generally, whenever the player moves indoors from outside, or vice versa. To test the player's location, compare the minimapZoom and minimapInsideZoom CVars with the current minimap zoom level (see Minimap:GetZoom).

- **UPDATE_BONUS_ACTIONBAR**</br>

- **UPDATE_INVENTORY_ALERTS**</br>
    Fires whenever an item's durability status becomes yellow (low) or red (broken). Signals that the durability frame needs to be updated. May also fire on any durability status change, even if that change doesn't require an update to the durability frame.

- **PLAYER_AURAS_CHANGED**</br>
    Called when a buff or debuff is either applied to a unit or is removed from the player. (Further details to follow, study needed).</br>
    Also fired when you start eating and/or drinking (which really is only a buff being applied like any other).</br>
    This event is also called when a Druid changes form (or prowl state). arg1 - arg9 are all nil in this case. These args are probably nil for other classes as well. Also, this event is called multiple times per form change.

- **QUEST_LOG_UPDATE**</br>
    Fires when the quest log updates.

- **PET_BAR_UPDATE**</br>
    Fired whenever the pet bar is updated and its GUI needs to be re-rendered/refreshed.

- **UPDATE_FACTION**</br>
    Fired when your character's reputation of some faction has changed.

- **UPDATE_LFG_TYPES**</br>

- **SKILL_LINES_CHANGED**</br>
    Fired when the content of the player's skill list changes. It only fires for major changes to the list, such as learning or unlearning a skill or raising one's level from Journeyman to Master. It doesn't fire for skill rank increases.

- **UPDATE_INSTANCE_INFO**</br>
    Fired when data from RequestRaidInfo is available.

- **UPDATE_TICKET**</br>
    Fires when information about an active GM ticket changes or becomes available.
    - arg1 - ?
    - arg2 - ?
    - arg3 - ?
    - arg4 - ?
    - arg5 - ?
    - arg6 - ?
    - arg7 - ?

- **UPDATE_PENDING_MAIL**</br>
    Fires when there is pending mail.

- **MEETINGSTONE_CHANGED**</br>

- **CURSOR_UPDATE**</br>
    Fired when the player right-clicks terrain, and on mouseover before UPDATE_MOUSEOVER_UNIT and on mouseout after UPDATE_MOUSEOVER_UNIT. This excludes doodads, player characters, and NPCs that lack interaction.

- **SPELL_UPDATE_USABLE**</br>
    This event is fired when a spell becomes "useable" or "unusable".

- **CHAT_MSG_SPELL_FAILED_LOCALPLAYER**</br>
    Fired when you fail to successfully cast a spell, for one of several reasons. arg1 is the full combat chat text and includes the reason. Examples: You fail to cast Heal: Interrupted. You fail to perform Bear Form: Not enough mana.
    - arg1 - full combat chat text
    - arg2 - ?
    - arg3 - ?
    - arg4 - ?
    - arg5 - ?
    - arg6 - ?
    - arg7 - ?
    - arg8 - ?
    - arg9 - ?

- **ACTIONBAR_UPDATE_COOLDOWN**</br>
    Fired when the cooldown for an actionbar or inventory slot starts or stops. Also fires when you log into a new area.
    
- **SPELL_UPDATE_COOLDOWN**</br>
    This event is fired immediately whenever you cast a spell, as well as every second while you channel spells.</br>
    The spell you cast doesn't need to have any explicit "cooldown" of its own, since this event also triggers off of anything that incurs a GCD (global cooldown). In other words, it basically fires whenever you cast any spell or channel any spell.</br>
    (It may possibly even trigger from spells that are "off the GCD" and which don't have any cooldown of their own; but there's no way to verify that, since all spells in game that are "off the GCD" are special class "burst" abilities with long cooldowns.)</br>
    It's worth noting that this event does NOT fire when spells finish their cooldown!</br>

- **ITEM_LOCK_CHANGED**</br>
    Fires when the "locked" status on a container or inventory item changes, usually from but not limited to Pickup functions to move items.
    - arg1 - when changing items in bags it's "LeftButton" (?)

- **PLAYER_REGEN_DISABLED**</br>
    Fired whenever you enter combat, as normal regen rates are disabled during combat. This means that either you are in the hate list of a NPC or that you've been taking part in a pvp action (either as attacker or victim).
    
- **PLAYER_REGEN_ENABLED**</br>
    Fired after ending combat, as regen rates return to normal. Useful for determining when a player has left combat. This occurs when you are not on the hate list of any NPC, or a few seconds after the latest pvp attack that you were involved with.

- **UNIT_FLAGS**</br>
    Fired when entering combat. Need more info.
    - arg1 - unitTarget ("player", "target", ...)

- **CHAT_MSG_COMBAT_HOSTILE_DEATH**</br>
    Fired when you fail to successfully cast a spell, for one of several reasons. arg1 is the full combat chat text and includes the reason. Examples: You fail to cast Heal: Interrupted. You fail to perform Bear Form: Not enough mana.
    - arg1 - Chat message (eg: "Snowshow Rabbit dies." or "You have slain Snowshow Rabbit!")
    - arg2 - ?
    - arg3 - ?
    - arg4 - ?
    - arg5 - ?
    - arg6 - ?
    - arg7 - ?
    - arg8 - ?
    - arg9 - ?

- **CHAT_MSG_SPELL_SELF_DAMAGE**</br>
    Fired whenever you cast a harmful spell, be it direct damage or debuff of any kind, be it Warrior's taunt or DoT. arg1 holds the exact same string that is posted to the Combat Log. (eg. "Your Fireball hits Snivvle for 842.")</br>
    Also fired when your spell does not actually take effect; ie. if it is resisted, or if the target is immune. A "resist" message is "Your Banish was resisted by Felguard Elite." while an "immune" message is of the format "Your Fire Blast failed. Firelord is immune."
    - arg1 - Chat message (eg: "Your Fireball hits Snivvle for 842.")
    - arg2 - ?
    - arg3 - ?
    - arg4 - ?
    - arg5 - ?
    - arg6 - ?
    - arg7 - ?
    - arg8 - ?
    - arg9 - ?

- **CHAT_MSG_SYSTEM**</br>
    Fired when a system chat message (they are displayed in yellow) is received.</br>
    Be very careful with assuming when the event is actually sent. For example, "Quest accepted: Quest Title" is sent before the quest log updates, so at the time of the event the player's quest log does not yet contain the quest. Similarly, "Quest Title completed." is sent before the quest is removed from the quest log, so at the time of the event the player's quest log still contains the quest.
    - arg1 - text
    - arg2 - ?
    - arg3 - ?
    - arg4 - ?
    - arg5 - ?
    - arg6 - ?
    - arg7 - ?
    - arg8 - ?
    - arg9 - ?

- **CHAT_MSG_CHANNEL_NOTICE**</br>
    Fired when you enter or leave a chat channel (or a channel was recently throttled).
    - arg1 - text ("YOU_CHANGED" if you joined a channel, or "YOU_LEFT" if you left, or "THROTTLED" if channel was throttled.)
    - arg2 - ?
    - arg3 - ?
    - arg4 - channelName (Channel name with channelIndex, e.g. "2. Trade - City")
    - arg5 - ?
    - arg6 - ?
    - arg7 - zoneChannelID (The static ID of the zone channel, e.g. 1 for General, 2 for Trade and 22 for LocalDefense. [4])
    - arg8 - channelIndex (Channel index, this usually is related to the order in which you joined each channel.)
    - arg9 - channelBaseName (Channel name without the number, e.g. "Trade - City")

- **CHAT_MSG_COMBAT_SELF_HITS**</br>
    Fired when a you hit a creature. Also called when you hurt yourself by falling, drowning or burning on a campfire.
    - arg1 - text (Chat message)
    - arg2 - ?
    - arg3 - ?
    - arg4 - ?
    - arg5 - ?
    - arg6 - ?
    - arg7 - ?
    - arg8 - ?
    - arg9 - ?

- **CHAT_MSG_COMBAT_XP_GAIN**</br>
    Fires when you gain XP from killing a creature or finishing a quest. Does not fire if you gain no XP from killing a creature.
    - arg1 - text (e.g. "Rockjaw Invader dies, you gain 44 experience.")
    - arg2 - ?
    - arg3 - ?
    - arg4 - ?
    - arg5 - ?
    - arg6 - ?
    - arg7 - ?
    - arg8 - ?
    - arg9 - ?

- **CHAT_MSG_SPELL_DAMAGESHIELDS_ON_SELF**</br>
    Fired when a buff (or possibly item) damages an opponent in response to an action... IE Thorns.

- **WORLD_MAP_UPDATE**</br>
    Fired when the world map should be updated.</br>
    When entering a battleground, this event won't fire until the zone is changed (i.e. in WSG when you walk outside of Warsong Lumber Mill or Silverwing Hold) --Salanex

- **ZONE_CHANGED**</br>
    Fires when the player enters an outdoors subzone.

- **MINIMAP_ZONE_CHANGED**</br>
    Fired whenever the text above the Minimap changes, i.e. each time the player changes the area or the zone in the current area.

- **UNIT_ENERGY**</br>
    Fired whenever a units energy is affected.
    - arg1 - unitId ("player")

- **ACTIONBAR_UPDATE_USABLE**</br>
    Fired when something in the actionbar or your inventory becomes usable (after eating or drinking a potion, or entering/leaving stealth; for example). This is affected by rage/mana/energy available, but not by range.

- **TABARD_CANSAVE_CHANGED**</br>
    Fired when it is possible to save a tabard.

- **PLAYER_GUILD_UPDATE**</br>
    This appears to be fired when a player is gkicked, gquits, etc.
    - arg1 - unitId ("player")

- **LANGUAGE_LIST_CHANGED**</br>

- **UPDATE_SHAPESHIFT_FORMS**</br>
    Fired when the available set of forms changes (i.e. on skill gain)

- **SPELLS_CHANGED**</br>
    Fires when spells in the spellbook change in any way. Can be trivial (e.g.: icon changes only), or substantial (e.g.: learning or unlearning spells/skills).</br>
    In prior game versions, this event also fired every time the player navigated the spellbook (swapped pages/tabs), since that caused UpdateSpells to be called which in turn always triggered a SPELLS_CHANGED event. However, that API has been removed since Patch 4.0.1.

- **UNIT_NAME_UPDATE**</br>
    Fired when a unit's name changes.
    - arg1 - unitId ("player")

- **PLAYER_FLAGS_CHANGED**</br>
    This event fires when a Unit's flags change (e.g. /afk, /dnd).</br>
    WoW condenses simultaneous flag changes into a single event. If you are currently AFK and not(DND) but you type /dnd you'll see two Chat Log messages ("You are no longer AFK" and "You are now DND: Do Not Disturb") but you'll only see a single PLAYER_FLAGS_CHANGED event.
    - arg1 - unitId ("player")

- **PLAYER_UPDATE_RESTING**</br>
    Fired when the player starts or stops resting, i.e. when entering/leaving inns/major towns.

- **ZONE_CHANGED_NEW_AREA**</br>
    Fires when the player enters a new zone.</br>
    For example this fires when moving from Duskwood to Stranglethorn Vale or Durotar into Orgrimmar.</br>
    In interface terms, this is anytime you get a new set of channels.

- **UPDATE_WORLD_STATES**</br>
    Fired within Battlefields when certain things occur such as a flag being captured.</br>
    Also seen in the outdoor world occasionlly, but it's not clear what triggers it.

- **FRIENDLIST_UPDATE**</br>
    Fired when...
    - You log in
    - Open the friends window (twice)
    - Switch from the ignore list to the friend's list
    - Switch from the guild, raid, or who tab back to the friends tab (twice)
    - Add a friend
    - Remove a friend
    - Friend comes online
    - Friend goes offline

- **IGNORELIST_UPDATE**</br>
    Fires twice when a player is added or removed from the ignore list.

- **PLAYER_ALIVE**</br>
    Fired when the player releases from death to a graveyard; or accepts a resurrect before releasing their spirit.</br>
    Does not fire when the player is alive after being a ghost. PLAYER_UNGHOST is triggered in that case.

- **PLAYER_UNGHOST**</br>
    Fired when the player is alive after being a ghost.</br>
    The player is alive when this event happens. Does not fire when the player is resurrected before releasing. PLAYER_ALIVE is triggered in that case.</br>
    Fires after one of:
    - Performing a successful corpse run and the player accepts the 'Resurrect Now' box.
    - Accepting a resurrect from another player after releasing from a death.
    - Zoning into an instance where the player is dead.
    - When the player accept a resurrect from a Spirit Healer.
    - You log in

- **ACTIONBAR_SLOT_CHANGED**</br>
    Fired when any actionbar slot's contents change; typically the picking up and dropping of buttons.M</br>
    On 4/24/2006, Slouken stated "ACTIONBAR_SLOT_CHANGED is also sent whenever something changes whether or not the button should be dimmed. The first argument is the slot which changed." This means actions that affect the internal fields of action bar buttons also generate this event for the affected button(s). Examples include the Start and End of casting channeled spells, casting a new buff on yourself, and the cancellation or expiration of a buff on yourself.
    - arg1 - slot (number)

- **UNIT_PORTRAIT_UPDATE**</br>
    Fired when a units portrait changes.
    - arg1 - unitId ("player")

- **UNIT_MODEL_CHANGED**</br>
    Fired when the unit's 3d model changes.
    - arg1 - unitId ("player")

- **PLAYER_XP_UPDATE**</br>
    Fired when the player's XP is updated (due quest completion or killing).
    - arg1 - unitTarget ("player")

- **PLAYER_COMBO_POINTS**</br>
    Fired when your combo points change.

- **UNIT_AURA**</br>
    Fires when a buff, debuff, status, or item bonus was gained by or faded from an entity (player, pet, NPC, or mob.)
    - arg1 - unitTarget ("player")

- **UPDATE_EXHAUSTION**</br>
    Fired when your character's XP exhaustion (i.e. the amount of your character's rested bonus) changes. Use GetXPExhaustion to query the current value.

- **COMBAT_TEXT_UPDATE**</br>
    Fired when the currently watched entity (as set by the CombatTextSetActiveUnit function) takes or avoids damage, receives heals, gains mana/energy/rage, etc. This event is used by Blizzard's floating combat text addon.
    - arg1 - combatTextType ("DAMAGE", "SPELL_DAMAGE", "DAMAGE_CRIT"...)
    - arg2 - amount (number)

- **PLAYER_MONEY**</br>
    Fired whenever the player gains or loses money.</br>
    To get the amount of money earned/lost, you'll need to save the return value from GetMoney from the last time PLAYER_MONEY fired and compare it to the new return value from GetMoney.

- **MERCHANT_SHOW**</br>
    Fired when the merchant frame is shown.

- **MERCHANT_UPDATE**</br>
    Fired when a merchant updates.

- **MERCHANT_CLOSED**</br>
    Fired when a merchant frame closes. (Called twice).

- **LOOT_CLOSED**</br>
    Fired when a player ceases looting a corpse. Note that this will fire before the last CHAT_MSG_LOOT event for that loot.

## Widgets

## API