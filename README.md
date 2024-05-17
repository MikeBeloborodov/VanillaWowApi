# Vanilla World of Warcraft 1.12.1 API Documentation

It's hard to come by any old WoW API documentation, only bits and pieces here and there. So I've decided to slowly fill this README with all the things that I come across in Vanilla WoW API.

It is by far no the best or exhaustive documentation, and you should not refer to it as an official one. If you have any qustions or if you want to contribute, please be welcome and open an issue or a pull request.

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

- **UNIT_HEALTH**
    Fires when the health of a unit changes.
    - arg1 - source of health change ("target", "player", "mouseover")
- **BAG_UPDATE**
    Fired when a bags inventory changes.
    - arg1 - bagID (default bag is 0)
- **UNIT_INVENTORY_CHANGED**
    Fired when the player equips or unequips an item.
    - arg1 - unitTarget ("player", "target")
- **PLAYER_TARGET_CHANGED**
    This event is fired whenever the player's target is changed, including when the target is lost.
- **UPDATE_MOUSEOVER_UNIT**
    Fired when the mouseover object needs to be updated.
    Fired when the target of the "mouseover" UnitId has changed and is a 3d model. (Does not fire when UnitExists("mouseover") becomes nil, or if you mouse over a unitframe.)
- **UI_ERROR_MESSAGE**
    Fired when the interface creates an error message. These are the red messages that show in the top middle of the screen. "Your inventory is full." is one example.
    - arg1 - message
- **SPELLCAST_FAILED**
    Fired when a spell fails.
- **SPELLCAST_STOP**
    Fired when a spell cast stops.
- **ACTIONBAR_UPDATE_STATE**
    Fired when the state of anything on the actionbar changes. This includes cooldown and disabling.
- **CURRENT_SPELL_CAST_CHANGED**
    Fired when the spell being cast is changed.
- **UNIT_COMBAT**
    Fired when an npc or player participates in combat and takes damage
    - arg1 - unitTarget ("player", "target", ...)
    - arg2 - event (Action, Damage, etc (e.g. HEAL, DODGE, BLOCK, WOUND, MISS, PARRY, RESIST, ...))
    - arg3 - flagText (Critical/Glancing indicator (e.g. CRITICAL, CRUSHING, GLANCING))
    - arg4 - amount (The numeric damage)
    - arg5 - schoolMask (0 - physical; 1 - holy; 2 - fire; 3 - nature; 4 - frost; 5 - shadow; 6 - arcane)
- **PLAYER_ENTER_COMBAT**
    Fired when a player engages auto-attack. Note that firing a gun or a spell, or getting aggro, does NOT trigger this event.
    PLAYER_ENTER_COMBAT and PLAYER_LEAVE_COMBAT are for *MELEE* combat only. They fire when you initiate autoattack and when you turn it off. However, any spell or ability that does not turn on autoattack does not trigger it. Nor does it trigger when you get aggro.
- **PLAYER_LEAVE_COMBAT**
    Fired when the player leaves combat through death, defeat of opponents, or an ability. Does not fire if a player flees from combat on foot.



## Widgets

## API