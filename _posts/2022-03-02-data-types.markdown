---
layout:     post
title:      "Data types"
date:       2022-03-02 01:00:00
author:     "Daniel Vela"
og_image: "/img/post-bg-09.jpg"
locale:       en
lang-ref:   data-types
---

Many times we make mistakes when designing simple data types. We usually have the habit of declaring a type and then adding the properties of that type. For example, the National ID class:

```swift
class NID {
    var id: String ❌

    func validate() -> Bool {...}

    func validity_character() -> Character {...}
}
```

We add a property called 'id' to this class, which represents a National Identity Document number, to contain the value of the class and its methods.

However, this should not be the case. We would have to apply the principle of object-oriented programming and understand that the NID is a specialization of the String class: because it is actually a string made up of numbers and letters, that holds a National Identity Document.

```swift
class DNI: String {
    func validate() -> Bool {...}

    func validity_character() -> Character {...}
}
```

Doing this way has un advantage: all the functions that allow strings, also will be able to treat NID class objects, such as assigning their value directly, without the need to access a property, such as:

```swift
    uiLabel.text = ID
```

... serialize, save in database, everything is easier. But the real advantage can be found when using generic functions, which can access all the functionality of strings, without the need to program any adapter.

### Another use: Dictionary

Another type of data that we can take advantage of to extend on many occasions is the dictionary.

```swift
struct Person: Dictionary {
    var name: String {
        get { self["name"] }
        set { self["name"] = newValue }
    }

    var id: NID {
        get { self["nid"] }
        set { self["nid"] = newValue }
    }
}
```

This class derived from Dictionary allows us to use all the dictionay functionalities, without the need to create an adapter. For example, there is the **iOS** `UserDefaults` functionality that allows you to save and retrieve data easily, making them persistent:

```swift
let person = Person(name: "Dani", dni: "12345678A")
UserDefaults.standard.set(person, forKey: "person")
```

### Other uses

There are many types of data that we can take advantage of to extend on many occasions. Classes like `String`, `Int`, `Double`, `Array`, `Dictionary`, `Set`, `Date`. All these types can be extended to add specialized properties and methods, but allowing to take advantage of all the functionalities that these classes and their helper functions already have. Mainly it is useful to us when it comes to serializing, persisting and saving in the database, but also to filter data, assign it, convert it.

### Labels

Another example would be the `Label` type that we could use to represent an internationalizable UI label.

```swift
class Label: String {
    func localized() -> String {
        return NSLocalizedString(self, comment: "")
    }
}
```

This class, programmed this way, would allow us to assign it directly to a UI label, without the need to create an adapter. But of course, the `localized` method would have to be called every time it was assigned to a UILabel; we could overload the '=' operator to do the localization automatically upon seeing that assignment. It would be like this:

```swift
let label = Label("VIEW_TITLE")
let uiLabel = UILabel()
uiLabel = label

func operator=(uiLabel: UILabel, label: Label) {
    uiLabel.text = label.localized()
}
```