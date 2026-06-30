---
layout:     post
title:      "How to decode/encode custom containers with different objects types in with Swift"
date:       2019-06-17 00:00:00
author:     "Daniel Vela"
og_image: "/img/post-bg-14.jpg"
locale:       en
lang-ref:   parse-swift-json-sub-object-different-types
---

# How to decode/encode Github API event feed

- Reference: [Apple deocumentation](https://developer.apple.com/documentation/foundation/archives_and_serialization/encoding_and_decoding_custom_types)
- [Github API: Events](https://developer.github.com/v3/activity/events/types/)
- [Sample feed with github events](https://api.github.com/events)

## Decoding automatically a custom object from json

When decoding an object like:

```json
{
    "type": "foo",
    "payload": {
        "word": "hello"
        "push_id": 3711282988,
        "size": 1,
    }
}
```

With a struct implementing `Codable` can be parsed easyly.

```swift
struct Payload: Codable {
    var word: String
    var push_id: Int
	var size: Int
}

struct Foo: Codable {
    var type: String
	var payload: Payload	
}

let decoder = JSONDecoder()
let product = try decoder.decode(Foo.self, from: json)
```

But what happens when one of the inner properties of an object, includes a type that can be different.

## Decoding manualy  a custom object from json

```json
[{
    "type": "foo",
    "payload": {
        "word": "hello"
        "push_id": 3711282988,
        "size": 1,
    }
},
{
    "type": "bar",
    "payload": {
        "size": 1,
		"radious": 0.5
    }
}]
```

In this case, the object cannot be parsed directly because the type of the payload object depends in the "type" property.

To solve this, the object must be decoded manually, this way.

```swift
struct FooPayload: Codable {
    var word: String
    var push_id: Int
	var size: Int
}

struct BarPayload: Codable {
    var size: Int
	var radious: Double
}

struct Foo {
    var type: String
    var payload: Payload?
    enum CodingKeys : CodingKey {
        case type, payload
    }

    init(from decoder: Decoder) throws {
        let values = try decoder.container(keyedBy: CodingKeys.self)
        type = try values.decode(String.self, forKey: .type)
        swith type {
        case "foo":
            payload = values.decode(FooPayload.self, forKey: .payload)
        case "bar":
            payload = values.decode(BarPayload.self, forKey: .payload)
        default:
            payload = nil
        }
    }
}

```