---
title: "Hexagonal Architecture: What Is It and How Does It Work?"
source: "https://dzone.com/articles/hexagonal-architecture-what-is-it-and-how-does-it?fbclid=IwZXh0bgNhZW0CMTAAYnJpZBExb241UUJPNHliVnBnUnY4Z3NydGMGYXBwX2lkEDIyMjAzOTE3ODgyMDA4OTIAAR54NgM4ItPlsQxkFAdYwL14aNw_5TA-iHGe4rnmt5-jw3rgCaU1Ub4moxqjEQ_aem_D7xcCa4_3Q0YSallxgdMvw"
author:
  - "[[Phil Vuollet]]"
published: 2018-10-24
created: 2026-04-07
description: "Learn about the structure of the popular hexagonal software architecture, how it works, and how to set up the ports-and-adapters approach for your applications."
tags:
  - "clippings"
---
Join the DZone community and get the full member experience.

[Join For Free](https://dzone.com/static/registration.html)

Hexagonal architecture is a model or pattern for designing software applications. The idea behind it is to put inputs and outputs at the edges of your design. In doing so, you isolate the central logic (the core) of your application from outside concerns. Having inputs and outputs at the edge means you can swap out their handlers without changing the core code.

One major appeal of using hexagonal architecture is that it makes your code easier to test. You can swap in fakes for testing, which makes the tests more stable.

Hexagonal architecture was a departure from [layered architecture](https://blog.ndepend.com/layered-architecture-solid-approach/). It's possible to use dependency injection and other techniques in layered architecture to enable testing. But there's a key difference in the hexagonal model: the UI can be swapped out, too. And this was the primary motivation for the creation of hexagonal architecture in the first place. There's a bit of interesting trivia about its origins. The story goes a little like this...

<video aria-label="Connatix video player" role="application" title="" src="blob:https://dzone.com/046b0b13-3e92-480c-b4c5-311baf0ebd58" controls=""></video>1/1

## Hexagonal Architecture? More Like Ports and Adapters!

Hexagonal architecture was proposed by Alistair Cockburn in 2005. "Hexagonal architecture" was actually the working name for the " [ports and adapters pattern,](https://web.archive.org/web/20060711221010/http://alistair.cockburn.us:80/index.php/Hexagonal_architecture)" which is the term Cockburn settled on in the end. But the "hexagonal architecture" name stuck, and that's the name many people know it by today.

Cockburn had his "eureka moment" about ports and adapters after reading *[Object Design: Roles, Responsibilities, and Collaborations](https://www.amazon.com/exec/obidos/ISBN=0201379430/portlandpatternrA/)* by Rebecca Wirfs-Brock and Alan McKean. Cockburn [explains](http://wiki.c2.com/?HexagonalArchitecture) that the authors "call \[the adapter\] stereotype 'Interfacers,' but show examples of them using the GoF pattern 'Adapter.'" It's important to note the use of the term "Interfacers" in that wiki entry because that's really what it's all about!

Imagine a hexagon shape with another, larger hexagon shape around it. The center hexagon is the core of the application (the business and application logic). The layer between the core and the outer hexagon is the adapter layer. And each side of the hexagon represents the ports.

It's not as if there are six-and only six-ports in the hexagonal architecture. The analogy kind of falls apart there. The sides are simply representations in the model for ports. Cockburn chose this flat-sided shape instead of a circle to convey a specific intent about ports.

Here's my own interpretation of a hexagonal architecture diagram:

The model is balanced (Cockburn has proclaimed his affinity for symmetry) with some external services on the left and others on the right. Remember that hexagonal architecture is a model for how to structure certain aspects of the application. It's specifically about dealing with I/O. I/O goes on the outside of the model. Adapters are in the gray area. The sides of the hexagons are the ports. Finally, the center is the application and domain. There are no specific requirements about the core, just that all of the business/application/domain logic lives there.

So what are ports anyway? In C#, they're interfaces.

<iframe frameborder="0" src="https://0942ae57a663a2d07e2d8c0976b4f2df.safeframe.googlesyndication.com/safeframe/1-0-45/html/container.html" title="3rd party ad content" width="728" height="90" allow="private-state-token-redemption;attribution-reporting" aria-label="Advertisement"></iframe>

## Interfaces as Ports

Thirteen years after Cockburn's idea, we commonly use the term "interface" without remembering how the world was without one. But what is an interface really? An interface defines how one module communicates with another.

### For Example

Take two modules: "module A" and "module B." When module A needs to send a message to module B, it will do so through the interface. The interface is in the language of module A. (Let's not worry too much about module B just yet; that'll come later.)

In hexagonal architecture terms, the interface is the port. A port can represent things like the user, another app, a message bus, or a database. In hexagonal architecture, a port — much like an interface — is an abstraction.

### Ab-what?

Abstraction just means we don't know how something does what it does. With an abstraction, we only know the high-level details. For example, the instruction "Tell Johnny to meet me at the bank" is an abstraction. We don't care *how* you tell Johnny so long as he gets the message. If we wanted to be concrete about it, rather than abstract, we'd say "Call Johnny using the following procedure: Turn on your phone; tap the phone icon; now, tap the following numbers on the screen: 555-5555; tap send...." I won't bore you with any more details.

So, we have this interface (as a port). Our module can use the interface to send messages. Now we need to make that message talk to something else. Enter the adapter.

## Concretions as Adapters

Finally, the adapter is where we want to think about the concrete implementation. This is how the message is either handled or passed along. A message to "save the user record" might go to a database, a file, or a network call over HTTP. The adapter might even keep the message in memory. Anything goes so long as the adapter responds accordingly.

Here's a concrete example using some C# code. Notice how the *UserAdmin* only knows about the interface.

```
public class UserAdmin
```

```
{
```

```
private readonly IUserRepo _userRepo;
```

```
public UserAdmin(IUserRepo userRepo)
```

```
{
```

```
_userRepo = userRepo;
```

```
}
```

```
​
```

```
UserData _userData;
```

```
public void Save()
```

```
{
```

```
Validate(_userData);
```

```
_userRepo.Save(_userData);
```

```
}
```

```
...
```

```
}
```

In this example, the *Save* method uses the *IUserRepo* interface. The *UserAdmin* class has no idea how the *\_userRepo* does its thing. All our *UserAdmin* object knows about is the *Save* method on whatever the *\_userRepo* references via the interface.

Let's say the *\_userRepo* represents module B in our earlier example. Module A is the *UserAdmin* class. Module A sends a message to module B via the *IUserRepo* interface. Here's where our adapter comes in. The adapter is the implementing class of the *IUserRepo*. In our running application, this could write to a database, as in the following code:

```
class UserDatabaseRepository : IUserRepo
```

```
{
```

```
public void Save(UserData userData)
```

```
{
```

```
using(var db = GetDatabaseConnection())
```

```
{
```

```
db.ExecuteSave(userData);
```

```
}
```

```
}
```

```
}
```

Or, I could send the record over HTTP, as in this example:

```
class UserHttpRepository : IUserRepo
```

```
{
```

```
public void Save(UserData userData)
```

```
{
```

```
using(var http = GetHttpConnection(Connections.UserRepository))
```

```
{
```

```
http.Post(userData);
```

```
}
```

```
}
```

```
}
```

Either way, module A behaves the same: It interacts with the repository without any details about what that repository does with the message. This is the power of the adapter!

Our *UserDatabaseRepository* and *UserHttpRepository* classes are the adapters. Each adapts the message to the underlying I/O. The adapters aren't the database and the TCP port; rather, they adapt the message from the port to its destination(s).

But how does this help my code, you're wondering, and what does it have to do with hexagonal architecture?

## It's About Swappable Components

Module A can use the interface to send a message. It has no way of knowing how or what will actually receive the message. This is *the* biggest benefit of hexagonal architecture!

Hexagonal architecture is all about swapping components-specifically, external components. In the example above, the module host would inject the *IUserRepo* into the *UserAdmin* class. The host could be a web app, a console application, a test framework, or even another app. The point is to make the core independent of its inputs and outputs.

Cockburn stressed the importance of decoupling the application from the UI. This is where he really sought to differentiate his approach from layered architecture. After all, the main goal of decoupling through ports and adapters is to test-drive the application using software (a test harness).

## And About the Test Drive

I've already enumerated the advantages of using hexagonal architecture in your design. Now, let me clarify explicitly *why* you should use this pattern.

The bottom line is that you don't need to rely on external factors to test your application. Instead, just make the core of the system interact through ports. This way, your test framework will drive the application through those ports. You could even use files and scripts to drive it instead!

To give you a common scenario of how this works in practice, let's say we're using a.NET testing framework such as [xUnit](https://xunit.github.io/). The test runner, in this case, is the host.

The following four interactions between the tests and the application will happen via ports:

1. Tests send input to the application.
2. Test doubles receive output from the application.
3. Test doubles return input to the application.
4. Tests receive output from the application.

The tests and the [test doubles](https://martinfowler.com/bliki/TestDouble.html) (such as mocks, fakes, and stubs) drive the application through the ports. But what does *that* look like, you ask?

## The UserRepo Revisited

Let's look at that *UserRepo* again in light of testing. It's common to use a mock or fake during unit testing. A *FakeUserRepo* might look like this:

```
using UserDomain.Interfaces;
```

```
using UserDomain.Data;
```

```
using System.Collections.Generic;
```

```
using System.Linq;
```

```
​
```

```
namespace UserTests
```

```
{
```

```
public class FakeUserRepo: IUserRepo
```

```
{
```

```
List<UserData> _users = new List<UserData>();
```

```
​
```

```
public void Save(UserData user)
```

```
{
```

```
_users.Add(user);
```

```
}
```

```
​
```

```
public bool IsSaved(int userId)
```

```
{
```

```
return _users.Any(user => user.ID == userId);
```

```
}
```

```
}
```

```
}
```

This is a fake because it stores the data in memory in the List. Notice that it implements *IUserRepo*. I also added a very rudimentary way to check the List.

Now we need to pass this *FakeUserRepo* adapter to the *UserAdmin* in our test code like this:

```
public class UserAdminTests
```

```
{
```

```
[Fact]
```

```
public void SavesTheUser()
```

```
{
```

```
var fakeRepo = new FakeUserRepo();
```

```
var sut = new UserAdmin(fakeRepo);
```

```
var userData = new UserData { ID = 1 };
```

```
​
```

```
sut.Save(userData);
```

```
​
```

```
Assert.True(fakeRepo.IsSaved(1));
```

```
}
```

```
}
```

You can see that this test code is passing the *FakeUserRepo* into the *UserAdmin* class using its constructor.

In.NET-land, a web UI would interact with the *UserAdmin* class through an HTTP endpoint (the port) and [ASP.NET Web API](https://www.asp.net/web-api) (the adapter). Web API routes and adapts the HTTP message to the controller. Where it leaves you is to write the adapter code into the controller. That's where you'd adapt the action and message to the appropriate application class-in this case, *UserAdmin*.

## That's All There Is to It!

Although hexagonal architecture seems like some vague mystical concept from the ages, it's actually widespread in modern software development. The main theory behind it is decoupling the application logic from the inputs and outputs. The goal is to make the application easier to test. Alistair Cockburn changed the terminology from "hexagonal architecture" to "ports and adapters." Thankfully, hexagonal architecture sort of stuck. It just sounds a heck of a lot cooler!

Architecture application Testing Interface (computing) Database IT

Published at DZone with permission of Phil Vuollet. [See the original article here.](https://blog.ndepend.com/hexagonal-architecture/)

Opinions expressed by DZone contributors are their own.