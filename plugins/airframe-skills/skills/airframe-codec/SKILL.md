---
name: airframe-codec
description: Serializes and deserializes Scala objects to JSON and MessagePack using airframe-codec. Use when converting case classes to/from JSON, MessagePack serialization, or when the user mentions airframe-codec or object serialization.
---

# Using airframe-codec

Serialize Scala objects to JSON or MessagePack without annotations. Supports automatic schema-on-read type conversion.

## Basic Usage

```scala
import wvlet.airframe.codec.MessageCodec

case class User(id: Int, name: String, email: String)

val codec = MessageCodec.of[User]
val user = User(1, "alice", "alice@example.com")

// Serialize
val json = codec.toJson(user)       // {"id":1,"name":"alice","email":"alice@example.com"}
val msgpack = codec.toMsgPack(user) // Binary MessagePack

// Deserialize (returns Option)
codec.unpackJson(json)         // Some(User(1, "alice", "alice@example.com"))
codec.unpackMsgPack(msgpack)   // Some(User(1, "alice", "alice@example.com"))

// Deserialize (throws on failure)
codec.fromJson(json)           // User(1, "alice", "alice@example.com")
codec.fromMsgPack(msgpack)     // User(1, "alice", "alice@example.com")
```

## Nested Objects

```scala
case class Address(city: String, country: String)
case class Person(name: String, address: Address)

val codec = MessageCodec.of[Person]
val person = Person("Bob", Address("Tokyo", "Japan"))

codec.toJson(person)
// {"name":"Bob","address":{"city":"Tokyo","country":"Japan"}}
```

## Collections

```scala
case class Team(name: String, members: Seq[String])

val codec = MessageCodec.of[Team]
codec.toJson(Team("dev", Seq("alice", "bob")))
// {"name":"dev","members":["alice","bob"]}

// List of objects
val listCodec = MessageCodec.of[Seq[User]]
listCodec.toJson(Seq(User(1, "alice", "a@example.com"), User(2, "bob", "b@example.com")))
```

## Schema-On-Read Conversion

Automatic type conversion when deserializing:

```scala
case class Config(port: Int, debug: Boolean)

val codec = MessageCodec.of[Config]

// String "8080" automatically converts to Int 8080
// String "true" automatically converts to Boolean true
codec.fromJson("""{"port":"8080","debug":"true"}""")
// Config(8080, true)
```

## Default Values

Missing fields use type defaults:

| Type | Default Value |
|------|---------------|
| `Int`, `Long`, `Double` | `0` |
| `String` | `""` (empty string) |
| `Boolean` | `false` |
| `Option[X]` | `None` |
| `Seq[X]`, `List[X]`, `Array[X]` | empty collection |

```scala
case class Settings(host: String, port: Int, tags: Seq[String])

val codec = MessageCodec.of[Settings]
codec.fromJson("""{"host":"localhost"}""")
// Settings("localhost", 0, Seq())
```

## Required Fields

Use `@required` to enforce mandatory fields:

```scala
import wvlet.airframe.surface.required

case class User(
  @required id: Long,
  @required name: String,
  email: Option[String] = None
)

val codec = MessageCodec.of[User]

codec.fromJson("""{"id":1,"name":"alice"}""")           // OK
codec.fromJson("""{"id":1}""")                          // Throws MessageCodecException
```

## Querying Partial Data

Extract specific fields from larger JSON:

```scala
// Original JSON has many fields
val json = """[
  {"id":1,"name":"alice","email":"a@example.com","age":30,"role":"admin"},
  {"id":2,"name":"bob","email":"b@example.com","age":25,"role":"user"}
]"""

// Extract only what you need
case class UserSummary(id: Int, name: String)

val codec = MessageCodec.of[Seq[UserSummary]]
codec.fromJson(json)
// Seq(UserSummary(1, "alice"), UserSummary(2, "bob"))
```

## Map Types

```scala
case class Metadata(tags: Map[String, String])

val codec = MessageCodec.of[Metadata]
codec.toJson(Metadata(Map("env" -> "prod", "version" -> "1.0")))
// {"tags":{"env":"prod","version":"1.0"}}
```

## Option Types

```scala
case class Profile(name: String, bio: Option[String])

val codec = MessageCodec.of[Profile]

codec.toJson(Profile("alice", Some("Developer")))
// {"name":"alice","bio":"Developer"}

codec.toJson(Profile("bob", None))
// {"name":"bob"}

codec.fromJson("""{"name":"charlie"}""")
// Profile("charlie", None)
```
