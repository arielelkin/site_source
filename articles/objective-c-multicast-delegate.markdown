---
layout: page
title: "Easy and Robust Multicast Delegation in Objective-C"
date: 2014-03-10 00:45
comments: true
sharing: false
footer: false
---

Here's a simple and reliable way to implement one-to-many delegation in Objective-C:

```objective-c
@protocol CoolDelegate <NSObject>
@required
- (void)eventHappened;
@end

@interface Delegator : NSObject
- (void)addDelegate:(id<CoolDelegate>)delegate;
- (void)removeDelegate:(id<CoolDelegate>)delegate;
@end

@implementation Delegator {
    NSHashTable *delegates;
}

- (id)init {
    self = [super init];
    if (self) {
        delegates = [NSHashTable weakObjectsHashTable];
    }
    return self;
}

- (void)addDelegate:(id<SomeDelegate>)delegate {
    [delegates addObject:delegate];
}

- (void)removeDelegate:(id<SomeDelegate>)delegate {
    [delegates removeObject:delegate];
}

- (void)somethingHappened {
    for (id delegate in delegates) {
        [delegate eventHappened];
    }
}
```

Here are the main benefits of this implementation: 

 * Any number of objects that add themselves as delegates of a `Delegator` object can expect `eventHappened` to be called. 
 * If a delegate object is deallocated, you don't need to worry about anything. The `delegates` hash table stores a weak reference to it, so the delegator won't call it in `somethingHappened`. This also means that there are no strong reference cycles incurred. 
 * You get compile-time checks for protocol conformance.
 * It doesn't matter if you accidentally add the same object as a delegate several times, it will only appear once inside `delegates`. 
 * You can add and remove delegate objects dynamically.
 * No extraneous abstractions required, i.e. you avoid using KVO and `NSNotificationCenter`.
 
### When would I want to use this?

Whenever you need an object to talk to several other objects, and you want to use the delegate pattern. 
 
### Delegate methods with return values

Consider this method in `UITableViewDelegate`: 
 
```objective-c
- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath;
```

The tableview object calls this method when it needs to know what height its rows should have. If it has more than one delegate returning row heights, the delegates could potentially return contradicting values and leave the tableview in an inconsistent state. 

Therefore, don't use one-to-many delegation for delegate methods whose return value could leave a delegator object in an inconsistent state. Use traditional one-to-one delegation instead: declare a new formal protocol specifically for a one-to-one delegative relationship, and declare an adequately named `delegate` property.

You *can* combine one-to-many and one-to-one delegate methods in the same protocol, but it's not good practice as the contract is not clear. The compiler will think that any class that conforms to the protocol should implement all its non-optional methods. 

### Discussion
 
Traditionally, delegators have only one delegate in Objectice-C. If they need to talk to more than one object, then we're told to either:

 * Set the delegate to a manager object which will be responsible for updating other objects.
 * Use The Observer Pattern (via Key-Value Observing), or the Publish-Subscribe pattern (via `NSNotificationCenter`)
 
The first option can be a good choice, but can turn manager objects into complex control centers. It may also add an unecessary layer of abstraction if the manager is created just for the purpose of handling those delegate messages. So this option is often chosen at the expense of simplicity and modularity. 

The second option is chosen at the expense of delegation, i.e. having an object talk in your terms to another object. 

Theres's two problems with these two alternatives:

#### Problem 1: They bring in a set of problems that are not worth solving if all you need is a one-to-many delegative relationship

 * You're adding unnecessary complexity by having to deal with new abstractions *you* never needed: `NSKeyValueObservingOptions`, key paths, notification names, `NSNotification`,etc. 
  * There's no type-checking. I can shove whatever object I want as the `value` in `setValue:forKeyPath:`, and I can shove anything I want in an `NSNotification`'s `userInfo` dictionary. Which means that *you* need to ensure that your listeners are doing all the type-checking they can.
  * If you register yourself multiple times for the exact specific notification, NSNotificationCenter will NOT recognize the redundancy and instead will fire off as many notifications as you've registered an observation for.
  * Neither KVO nor `NSNotificationCenter` provide a way for you to know who the listeners are. 
 * They don't clean up after themselves. *You* are responsible for ensuring that listeners are alive when either KVO or `NSNotificationCenter` fires up a notification. Otherwise they... crash your app. 
 * The list goes on (for more in-depth analyses (of people smarter than me who agree with me), check out the relevant articles in the References section below)


#### Problem 2: They force you to use their built-in event listeners

KVO forces you to implement at least these two monstrosities:
```objective-c
- (void)addObserver:(NSObject *)observer forKeyPath:(NSString *)keyPath options:(NSKeyValueObservingOptions)options context:(void *)context;

- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context;
```

`NSNotificationCenter`, on the other hand, lets you specify which selector will be called when the listener receives the notification:
```objective-c
- (void)addObserver:(id)observer selector:(SEL)aSelector name:(NSString *)aName object:(id)anObject;
```

But this method can have one and only one argument, and that argument must be of type `NSNotification`, and you have to fit your data into its `userInfo` dictionary.

## Other implementations

There are other implementations of multicast delegates in Objective-C:

 * [Colin Eberhardt's](http://www.scottlogic.com/blog/2012/11/19/a-multicast-delegate-pattern-for-ios-controls.html)
 * [GCDMulticastDelegate](https://code.google.com/r/riky-adsfasfasf/source/browse/Utilities/GCDMulticastDelegate.h)
 * [LBDelegateMatrioska](https://github.com/lukabernardi/LBDelegateMatrioska)

They all offer a class with comprehensive multicast delegate functionality, that you'd use thus:

```objective-c
LBDelegateMatrioska *multicastDelegate = [[LBDelegateMatrioska alloc] init];
[multicastDelegate addDelegate:scrollViewDelegate];
[multicastDelegate addDelegate:anotherScrollViewDelegate];

scrollView.delegate = multicastDelegate;
```

These libraries are useful if you need to give multicasting capabilities to a Cocoa or Cocoa Touch class. But note that they don't support compile-time checks for protocol conformance.

I liked the idea of the `addDelegate` and `removeDelegate:` methods, but wanted an implementation that wasn't as complex, and which did not require a separate class. 

## Demo

Visit https://github.com/arielelkin/Obj-C-Multicast-Delegate for a demo project.

## References aka Further Reading

### Articles

* "Objective-c multicasting delegates" question on Stackoverflow. [Link](http://stackoverflow.com/a/14792617/1072846)
 * Colin Eberhardt, "A Multicast Delegate Pattern for iOS Controls". *Scott Logic*. [Link](http://www.scottlogic.com/blog/2012/11/19/a-multicast-delegate-pattern-for-ios-controls.html)
 * Robbie Hanson, `GCDMulticastDelegate`. [Link](https://code.google.com/r/riky-adsfasfasf/source/browse/Utilities/GCDMulticastDelegate.h)
 * Luka Bernardi, `LBDelegateMatrioska`. [Link](https://github.com/lukabernardi/LBDelegateMatrioska)
 * Martin Rybak, "Why NSNotification is bad". *ObjectiveC#*. [Link](http://objcsharp.wordpress.com/2013/08/28/why-nsnotificationcenter-is-bad/)
 * Mattt Thompson, "NSHash​Table & NSMap​Table". *NSHipster*. [Link](http://nshipster.com/nshashtable-and-nsmaptable/)
 * Mike Ash, "Key-Value Observing Done Right". *mikeash.com*. [Link](https://www.mikeash.com/pyblog/key-value-observing-done-right.html).
 * Soroush Khanlou, "KVO Considered Harmful". *khanlou.com*. [Link](http://khanlou.com/2013/12/kvo-considered-harmful/)

### Documentation

* "Protocol". *iOS Developer Library. Cocoa Core Competencies*. [Link](https://developer.apple.com/library/mac/documentation/general/conceptual/devpedia-cocoacore/Protocol.html)
* "Notification and Delegation". *iOS Developer Library. Notification Programming Topics*. [Link](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/Notifications/Articles/Notifications.html)
* "Delegates and Data Sources". *iOS Developer Library. Concepts in Objective-C Programming*. [Link](https://developer.apple.com/library/ios/documentation/general/conceptual/CocoaEncyclopedia/DelegatesandDataSources/DelegatesandDataSources.html#//apple_ref/doc/uid/TP40010810-CH11-SW3)
* "Key-Value Observing Programming Guide". *iOS Developer Library*. [Link](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/KeyValueObserving/KeyValueObserving.html)