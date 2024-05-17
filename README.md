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

**self** referes to the frame on which this event was registered. In the example above **self** would refer to exampleFrame.</br>
**event** is the name of the event that was fired.

Some events have other arguments that are available. They have percise  names **arg1**, **arg2**, ... **arg9**, so the maximum is 9 arguments.

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
In the list of events below I will not mention basic 2 arguments (self and event), but they are always there.

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

## Widgets

## API