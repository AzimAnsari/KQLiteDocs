# KQLite

![Logo](https://raw.githubusercontent.com/AzimAnsari/KQLiteDocs/refs/heads/main/logo.png)

- A Lightweight, Kotlin Multiplatform DSL to write typesafe SQL queries.
- Features Object-relational Mapping (ORM) style interaction with tables.
- Returns typesafe cursor which executes lazily on read and auto-closed when fully iterated.
- Typesafe values binding and reading.
- Entity binding and mapping.
- Cursor result as Flow which emits on every INSERT, UPDATE and DELETE on table.
- Quick convenience API.
- Builtin SQLite functions. Scalar, Aggregate, Math, Date & Time, JSON.
- SQLite familiar DSL.

## Kotlin Multiplatform Dependency

```kotlin
sourceSets {
    commonMain.dependencies {
        implementation("com.kqlite:kqlite:0.2.1")
    }
}
```

## TABLE

```kotlin
object TblContact : KQLiteTable("contacts"), KQLiteAdapter<Contact> {
    val id = intColumn("id").notNull().primaryKey().autoIncrement()
    val firstName = textColumn("first_name").notNull()
    val lastName = textColumn("last_name")
    val phone = jsonArrayColumn("phone").notNull()
    val birthDate = dateColumn("birth_date")
    val email = textColumn("email").check { (it IS null) OR (it LIKE "%_@__%.__%") }
    val image = blobColumn("image")
    val type = enumColumn("type", ContactType.entries).notNull()
    val deleted = booleanColumn("deleted").notNull().default(false)
}
```

## INSERT

```kotlin
val rowId: Int =
    TblContact
        .insert()
        .bind {
            it.firstName.bind("John")
            it.lastName.bind("Doe")
            it.phone.bind(JSON_ARRAY("1234567890", "0987654321"))
            it.birthDate.bind(DATE("2000-01-01"))
            it.email.bind("john.doe@example.com")
            it.type.bind(ContactType.Friend)
        }
        .executeReturning(TblContact.id)
```

## SELECT

```kotlin
val cursor =
    TblContact
        .select()
        .where {
            it.deleted NOT_EQ true
        }.execute()

cursor.forEach {
    val name = it[TblContact.firstName]
    val type = it[TblContact.type]
    println("Name: $name, Type: $type")
}
```

## UPDATE

```kotlin
TblContact
    .update {
        it.firstName.bind("Joker")
    }
    .where {
        it.id EQ 1
    }.execute()
```

## DELETE

```kotlin
TblContact
    .delete()
    .where {
        it.id EQ 1
    }.execute()
```

## Callback Flow

```kotlin
val flow =
    TblContact
        .select()
        .where { it.deleted NOT_EQ true }
        .orderBy(TblContact.firstName)
        .execute()
        .asCallbackFlow()
        .mapToList(mapper = TblContact::mapper)
```

## Quick Convenience API

```kotlin
// Quick INSERT
TblContact.quickInsert {
    it.firstName.bind("John")
    it.lastName.bind("Doe")
    it.phone.bind(JSON_ARRAY("1234567890", "0987654321"))
    it.type.bind(ContactType.Friend)
}

TblContact.quickInsert {
    it.binder(this, contact)
}

// Quick SELECT
val all = TblContact.quickSelect()

val deleted = TblContact.quickSelect {
    it.deleted EQ true
}

val names = TblContact.quickSelect(TblContact.firstName)

// Quick UPDATE
TblContact.quickUpdate(
    binding = {
        it.firstName.bind("Joker")
    },
    where = {
        it.id EQ 1
    }
)

TblContact.quickUpdate(
    binding = {
        it.deleted.bind(true)
    },
    where = null
)

// Quick DELETE
TblContact.quickDelete {
    it.type EQ ContactType.Other
}

TblContact.quickDelete(where = null)
```

## KQLiteCursor Usage

```kotlin
// Obtaining KQLiteCursor, it is also an Iterator. 
val cursor: KQLiteCursor = TblContact.quickSelect()

// Converting KQLiteCursor to Sequence for more features
val sequence: Sequence<KQLiteCursor> = cursor.asSequence()


// Getting row count from KQLiteCursor
val count: Int = cursor.asSequence().count()


// Mapping KQLiteCursor to list of entities, TblContact implements KQLiteAdapter
val contactList: List<Contact> = cursor.mapToList(TblContact::mapper)


// Reading single row. Throws Exception if count() != 1
val singleContact: Contact = cursor.mapToSingle(TblContact::mapper)


// Reading single item or null
val nullableContact: Contact? = cursor.mapToSingleOrNull(TblContact::mapper)


// KQLiteCursor custom mapping
val nameList: List<String> = cursor.mapToList { it[TblContact.firstName] }


// Converting KQLiteCursor to Flow. It does not observe table changes.
val flow: Flow<KQLiteCursor> = cursor.asFlow()


// Mapping Flow to other types
val nameFlow: Flow<String> = cursor.asFlow().map { it[TblContact.firstName] }


// Converting KQLiteCursor to CallbackFlow. Observe table changes INSERT, UPDATE and DELETE.
val callbackFlow: Flow<KQLiteCursor> = cursor.asCallbackFlow()


// Mapping Flow to List of observable entities.
val contactsFlow: Flow<List<Contact>> =
    cursor.asCallbackFlow().mapToList(mapper = TblContact::mapper)


// Custom dispatcher, default is IO, if not given.[dokka_home.md](../KQLiteLibrary/kqlite/dokka/dokka_home.md)
val dispatcher: Flow<List<Contact>> =
    cursor.asCallbackFlow().mapToList(Dispatchers.Main, TblContact::mapper)


// Observable Flow of nullable item.
val singleFlow: Flow<Contact?> =
    cursor.asCallbackFlow().mapToSingleOrNull(mapper = TblContact::mapper)
```

## Documentation

[kqlite.com](kqlite.com)

## Samples

- [KQLite Contacts Official Sample](https://github.com/AzimAnsari/KQLiteContacts).
- [PeopleInSpace KQLite Sample](https://github.com/AzimAnsari/PeopleInSpaceKQLite)
- [Droidcon KQLite Sample](https://github.com/AzimAnsari/DroidconKQLite)
- [D-KMP KQLite sample](https://github.com/AzimAnsari/D-KMP-sample-KQLite)

## Authors

- [AzimAnsari](https://github.com/AzimAnsari)

## License

Copyright © 2026 MOHAMMAD AZIM ANSARI. All Rights Reserved.

Permission is hereby granted, free of charge, to any person obtaining a
copy of this software and associated documentation files (the
"Software"), to use the Software for personal, educational, or other
non-commercial purposes, subject to the following conditions:

- The Software must not be used for any commercial purpose, including
  but not limited to selling, licensing, or incorporating it into a
  product or service for which payment or compensation is received.
- The Software may be redistributed only in its original, unmodified,
  and complete form, provided that this copyright and license notice
  is included.
- The Software must not be modified, sublicensed, or sold without
  prior written permission from the copyright holder.
- The Software is provided solely in binary or compiled form unless
  otherwise explicitly stated.
- All copyright and proprietary notices must be retained.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS
OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER
DEALINGS IN THE SOFTWARE.