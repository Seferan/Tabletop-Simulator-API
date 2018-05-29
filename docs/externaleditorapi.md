This page describes how our [Official Atom Plugin](atom) API works so that you can write your own plugin for your text editor of choice if Atom does not suit your needs.

Communication between the editor and TTS occurs via two localhost TCP connections: one where TTS listens for messages and one where the editor listens for messages. All communication messages are JSON.

##Atom as the Server

The following are messages the editor plugin listens for and handles.

!!!note ""
	Atom listens for incoming localhost TCP connections on port 39998.

###Pushing New Object

When clicking on "Scripting Editor" in the right click contextual menu in TTS for an object that doesn't have a Lua Script yet, TTS will send a JSON message with an ID of 0 and data for the object. The Atom plugin will open a new tab for this Object. If TTS is unable to connect to an external editor, e.g. if Atom is not running, then TTS falls back to the in-game editor.

```JSON
{
    "messageID": 0,
    "scriptStates": [
        {
            "name": "Chess Pawn",
            "guid": "db3f06",
            "script": ""
        }
    ]
}
```

###Loading a New Game

After loading a new game in TTS, TTS will send all the Lua scripts and UI XML from the new game to Atom. TTS sends a JSON message with an ID of 1 and an array of the Lua Scripts and UI XMLs. TTS also sends this message as a response to requests made by the editor, as described later under messages TTS listens for and handles.

```JSON
{
    "messageID": 1,
    "scriptStates": [
        {
            "name": "Global",
            "guid": "-1",
            "script": "...",
	    "ui": "..."
        },
        {
            "name": "BlackJack Dealer's Deck",
            "guid": "a0b2d5",
            "script": "..."
        },
    ]
}
```

Note that this message contains all objects in the TTS session that have Lua scripts or UI XML data. Any object not mentioned by this message does not have a Lua script or UI XML in TTS.

###Print/Debug Messages

TTS sends all print() messages to Atom to be displayed in Atom's console (ctrl + alt + i). TTS sends a JSON message with an ID of 2 and the message.

```JSON
{
    "messageID": 2,
    "message": "Hit player! White"
}
```


###Error Messages

TTS sends all Lua error messages to Atom to be displayed in Atom's console *(ctrl + alt + i)*. TTS sends a JSON message with an ID of 3 and the error message.

```JSON
{
    "messageID": 3,
    "error": "chunk_0:(36,4-8): unexpected symbol near 'deck'",
    "guid": "-1",
    "errorMessagePrefix": "Error in Global Script: "
}
```

##Tabletop Simulator as the Server

The following describes messages that TTS listens for and handles.

!!!note ""
	TTS listens for incoming localhost TCP connections on port 39999.


###Get Lua Scripts

TTS listens for a JSON message with an ID of 0, and responds by sending a JSON message containing scripts and UI XML. 

```JSON
{
    "messageID": 0
}
```

The response is not sent over the same connection as the original request, but rather is sent to a server listening on the localhost on port 39998. The response is a message sent by TTS with an ID of 1, and is the same as the message TTS sends when loading a new game.

###Save & Play

TTS listens for a JSON message with an ID of 1 containing an array of the Lua Scripts and UI XML.

```JSON
{
    "messageID": 1,
    "scriptStates": [
        {
            "guid": "-1",
            "script": "...",
	    "ui": "..."
        },
        {
            "guid": "a0b2d5",
            "script": "..."
        },
    ]
}
```

TTS updates the Lua scripts and UI XML for any objects listed in the message, and then reloads the save file, the same way it does when pressing "Save & Play" within the in-game editor. Objects not mentioned in the scriptStates array are not updated.

Any objects mentioned have both their Lua script and their UI XML updated. If no value is set for either the "script" or "ui" key then the corresponding Lua script or UI XML is deleted. That means if you want to update a Lua script for an object without changing its UI XML, then the message must contain both the updated Lua script and the unchanged UI XML.

After TTS reloads the game, it then also sends a message with an ID of 1 back to the editor, with content the same as the message TTS sends when loading a new game. 