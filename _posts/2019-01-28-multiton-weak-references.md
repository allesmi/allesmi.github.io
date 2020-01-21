---
layout: post
title: "The Multiton Design Pattern with Weak References in C#"
---

Recently I came across an issue in one of our data access object classes.
While fixing the bug I also added code to prevent a potential memory leak which I will describe in this post.

## The Multiton Pattern

[Wikipedia][1] defines the Multiton as follows:

>  [T]he multiton pattern is a design pattern which generalizes the singleton pattern. Whereas the singleton allows only one instance of a class to be created, the multiton pattern allows for the controlled creation of multiple instances, which it manages through the use a map.

The multiton in our program uses a dictionary that maps object ids as string to actual objects: `Dictionary<string, MyObject> dictionary;`.

References to these objects are then requested by the client code via a static method on the MyObject class:

```csharp
public static MyObject getMyObject(string id)
{
        if (MyObject.dictionary.ContainsKey(id))
        {
                return MyObject.dictionary[id];
        }
        else {
                MyObject myObject = new MyObject(id);
                MyObject.dictionary[id] = myObject;
                return myObject;
        }
}
```

This makes sense for example when creations of instances of MyObject are costly. 

## Problem

C# is a [garbage-collected language][2].
At runtime the garbage collector (GC) periodically frees allocated objects that are no longer reachable from the code.
Note the word "reachable": the GC cannot free objects that are still referenced somewhere in the program.
A common source of memory leaks in garbage-collected languages are references to objects in dictionaries, exactly the data structure the multiton pattern uses.
While the caller of `getMyObject` may no longer use the object there is still a reference to it in `dictionary` preventing it from being collected and freed.
Over time the heap grows until eventually the program runs out of free memory and crashes.

## Solution

While we could add a method to explicitly remove the objects from our dictionary this contradicts the philosophy of automatic memory management and is easy to forget.

The [.NET API][3] offers another solution:
C# (and also Java) distinguishes between strong and weak references.
A strong reference is a reference to an object that prevents it from being collected.
A weak reference alone is not enough for the GC to consider an object as reachable.
Exactly what we need to prevent a memory leak in our program.

## Implementation

We change our dictionary to map from string ids to weak references: `Dictionary<string, WeakReference<MyObject>> dictionary`.

The method `getMyObject` gets an additional check if the weakly referenced object is still available:

```csharp
public static MyObject getMyObject(string id)
{
        WeakReference<MyObject> myObjectRef = null;
        MyObject myObject = null;

        if (MyObject.dictionary.ContainsKey(id)) {
                myObjectRef = MyObject.dictionary[id];

                if (myObjectRef.TryGetTarget(out myObject))
                {
                        // object is still alive
                        return myObject;
                }
                else
                {
                        // object has been collected in the meantime,
                        // so we have to re-create it.
                        myObject = new MyObject(id);
                        myObjectRef.SetTarget(myObject);

                        return myObject;
                }
        }
        else
        {
                // Create the object for the first time
                myObject = new MyObject(id);
                myObjectRef = new WeakReference<MyObject>(myObject);
                MyObject.dictionary[id] = myObjectRef;

                return myObject;
        }
}
```

Note that a strong reference is returned to the caller while the private dictionary member only stores weak references.
Thus if the calling code no longer holds that reference the object is freed at the next collection and the memory leak is avoided.

## Summary

Use the multiton pattern with weak references in dictionaries to allow automatic memory management and prevent memory leaks.

[1]: https://en.wikipedia.org/wiki/Multiton_pattern "Multiton pattern"
[2]: https://docs.microsoft.com/en-us/dotnet/standard/garbage-collection/index ".NET Garbage Collection"
[3]: https://docs.microsoft.com/en-us/dotnet/api/system.weakreference?view=netframework-4.7.2 ".NET API: WeakReference Class"
