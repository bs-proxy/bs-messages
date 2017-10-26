Protocol Definition Files
=========================

Brawl Stars messages are passed between client and server in PDUs containing an integer message identifier
and a binary payload. The files in this package are used to map the identifier to a message
structure definition so that the binary payload can be parsed.

Each message structure is defined in an individual JSON file. The name of the file is not
significant, but generally it corresponds to the type name of the message.

Based on https://github.com/royale-proxy/cr-messages 

Type System
===========

Each file defines a type. Types are referred to by their *name* attribute or, for top level
messages, by their *id* attribute (the identifier passed in the PDU).

Using just the *name* and *id* attributes we can define simple structures such as the
[KeepAlive](client/KeepAlive.json) message:

```json
{
  "id": 10108,
  "name": "KeepAlive"
}
```

To read more complex messages we need to define the field list that makes up the payload. For
example, the [EndClientTurn](client/EndClientTurn.json) message comprises four fields; *unkBool* (a bool that is always false, so far.) , *tick* (the
time that the command was issued), *an unknown value*, and *commands*, an array of
[CommandComponent](component/CommandComponent.json) structures:


```json
{
  "id": 14102,
  "name": "EndClientTurn",
  "fields": [
    {"name": "unkBool", "type": "BOOLEAN"},
    {"name": "tick", "type": "RRSINT32"},
    {"name":" unk", "type": "RRSINT32"},
    {"name": "commands", "type": "CommandComponent[]", "lengthType": "RRSINT32"},
    {"type": "BYTE"},
    {"type": "BYTE"},
    {"type": "BYTE"},
    {"type": "BYTE"}
  ]
}
```

Each field has a *name* attribute that is used to refer to it and a *type* attribute that defines
its serialized structure. The *type* can refer to another structure or a built in primitive, or
an array of structures or primitives.

Primitives
----------

Five primitive types are defined.

- **BOOLEAN:** A single bit, read in little endian order. All other primitives are read from byte
  boundaries.

- **BYTE:** One byte.

- **INT:** Four bytes, big-endian

- **INT32:** [Variable length integer](https://developers.google.com/protocol-buffers/docs/encoding)

- **SINT32:** [Signed variable length integer](https://developers.google.com/protocol-buffers/docs/encoding)

- **RRSINT32:** Obfuscated signed variable length integer. The encoding takes the first byte of the varint, rotates the 7 bits after MSB to the right by 1. The LSB is wrapped around and get put in the 7th bit. MSB value is preserved.

- **RRSLONG:** Two RRSINT32 representing high/low.

- **SCID:** Brawl Stars uses ID like 60000000 for achievements. In packets, they often appear as two RRSINT32 (in this case, 0x3C00). You will find this in Regions, Cards and Arenas, etc. (Usually defined in a csv file -- row 0 is 60000000, and so on.)

- **LONG:** Eight bytes, big-endian

- **STRING:** An integer length, then a UTF-8 encoded string.

- **ZIP_STRING:** An integer length, then a little-endian integer unzipped length, then a zlib compressed UTF-8 encoded string.

Arrays
------

Arrays types are defined by appending square brackets to a type name, e.g. "INT[5]" defines an array
of five ints. If the size specifier is omitted the size of the array will be read/written as an
integer before the array contents. E.g. a field of type "CommandComponent[]" would be written as an
INT (the array size), followed by the contents of the array, a list of command components.

Optionals
---------

There is one other specialized structure, the optional field. Optionals are defined by prefixing
a type name with "?", e.g. "?LONG". They are serialized as a BOOLEAN (indicating the presence of the
value), followed by the value if it is present.

Extensions
----------

Some structures have additional trailing fields that are conditionally included. Extension fields
are read when their *id* attribute matches the *id* field of the structure that they extend (note:
the field with the *name* "id" is matched against, not the *id* attribute of the structure). For
example, the CommandComponent has extensions that are conditional on the command id:


```json
{
  "name": "CommandComponent",
  "fields": [
    {"name": "id", "type": "RRSINT32"}
  ],
  "extensions": [
    {
		"id": 203,
		"name": "OpenBox (203)",
		"fields": [
			{"name":"boxID", "type": "RRSINT32"},
			{"name":"boxCount", "type": "RRSINT32"},
			{"name":"itemType", "type": "RRSINT32"},
			{"name":"itemCount", "type": "RRSINT32"},
			{"type": "RRSINT32"},
			{"type": "RRSINT32"},
			{"type": "RRSINT32"}		  
		]
    },
    {
      "id": 504,
      "name": "CollectNewEventCoins (504)",
      "fields": [
        {"name": "tickStart", "type": "RRSINT32"},
        {"name": "tickEnd", "type": "RRSINT32"},
        {"type": "RRSLONG"},
        {"name": "challengeID", "type": "RRSINT32"},
        {"type": "RRSINT32"}
      ]
    }
  ]
}

```

Default Values
--------------

Fields with primitive values may have a *default* attribute to indicate the value that should be
used if the field is not set.

Comments
--------

Messages and Fields may have a *comment* attribute to document their use.
