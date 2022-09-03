# RST

_yes the acronym basically MVC don't @ me_

RST is a pattern I've accidentally developed over the last few (8!?) years of writing a _lot_ of Go code. It's kind of like MVC with a bit of DDD too. This is for large codebases and complex business domains. You shouldn't really _start_ a project with this. Start a project with a `main.go`. But this can be useful for scaling up a codebase without getting into a spaghetti nightmare 🍝 👻.

More often than not, I read about two approaches to structure: layer based and feature based. Layer based is what MVC is. You have some layers and this separates code that queries the database from code that renders UI. A timeless, classic 👌. Feature-based is kind of microservices or microfrontends where you group your code based on business domains, or features. So each "package" (or whatever your language's compilation grouping unit is) contains all the necessary bits to satisfy one business need. This one seems more modern but I'm honestly not sure when it appeared.

Now outside of these two, there are many more architectures. N-tier being the more "enterprise-y" (whatever that means nowadays...) and of course you come across that same separation: presentation, logic and data (hey, that looks familiar.) but these are often dismissed as "too complex". While that possibly _could_ be true, there's still a lot any engineer can learn from these alternative approaches.

But Golang is meant to be simple™ right? Well, we can take all the good ideas from these age-old paradigms, discard the ones that don't fit into Go's idioms and construct a nice way to architect applications. That left me, after many codebases, iterations and feedback from coworkers, with the following.

```
resources/
services/
transports/
internal/
```

That's it really. There's no huge folder tree like a lot of "Golang DDD" or "Clean architecture boilerplate" repositories out there. No, you have 3 layers to fill and internal packages for infrastructure.

One of the key ideas to practice when using this architecture is using abstraction wisely where it matters. Leaky abstractions are often worse than no abstraction. Sometimes it's better to just copy something a couple of times and learn about your use-cases before you spend time abstracting. Because, remember, engineering effort compounds over time and a bad design choice in the wrong place can cause many hundreds of man-hours over the lifetime of a company.

So what does that mean in practice? Well, after building a bunch of stuff I noticed a few things:

- Sometimes, you literally do not need a "service layer" and a HTTP handler can just talk to a repository. If you've written a service that does absolutely nothing other than proxy function calls to a database class, you're wasting time.
- Most of the time, database schemas do not map 1:1 to the business domain or the presentation layer and;
- Badly designed data structures will infect your application like a plague.

![https://docs.google.com/drawings/d/e/2PACX-1vQjEj4dKKUaQEUcNDq2UO58oIUu6pehqrE99q4gSRk0DY9KPIuhgG9Yg3qJGgW4ybrL5Ql8_Xo5z3yq/pub?w=960&h=720](https://docs.google.com/drawings/d/e/2PACX-1vQjEj4dKKUaQEUcNDq2UO58oIUu6pehqrE99q4gSRk0DY9KPIuhgG9Yg3qJGgW4ybrL5Ql8_Xo5z3yq/pub?w=960&h=720)

_Shamelessly stolen from [Herberto Graça](https://twitter.com/hgraca)_

## Resources

A "resource" in this architecture at the technical level is what you'd expect, a repository and some data structures. Boring stuff.

But conceptually, the way I've been using this is essentially using these as domain entities in a specific context. And this is where the whole "resources do not necessarily map to tables or collections" thing comes from. You may have a "user" table, in fact I'm willing to bet most applications on the planet have a "user" (or the regional equivalent translation) table in their database. This does not mean you create a resource called "user", give it the Create, Read, Update and Delete methods and call it a day. No, context matters. Let's take GitHub as an example, users have different contexts such as being members of an organisation or owners of a repository or followers of other people. The way I use resources is codifying these contexts.

What does that mean? Well, let's say I'm writing a service that invites people to an organisation. That service needs a few pieces of data, it needs an email address and a name, but that's pretty much it. It doesn't need to know my phone number or my payment plan or when my account was created. So we have a specific context in which a user is being referred to, let's give it a name! "Organisation Member". An organisation member only needs to have a handful of data, especially for sending them an invite.

In practice, this results in a tiny simple resource called "Organisation Member" which likely includes a couple of basic methods like "JoinOrg", "Get" and "LeaveOrg". These methods only query what they need. Which solves one of the common criticisms of DDD with which people will often comment that these architectures result in inadvertantly implementing application-side relational database joins because it's so easy to write `user.Get` followed by `orgs.Get(user.org[i])`. By writing use-case specific resources you write use-case specific queries. Queries that are optimised for that specific use-case. Of course it still takes careful and skillful understanding of your codebase to avoid but it certainly makes it easier.

Works well with generated code from Ent.

## Services

Services are simple. You have a business domain use-case, you write a service for it. There's really not much else to say here.

One key piece of advice I'd offer here based on past experience is to not map services 1:1 to resources. This is for a similar reason that you don't map resources 1:1 to database tables. Useful abstraction. A service responsible for inviting users to an organisation, as with the earlier example, would be named "Organisation Invite". There's no equivalent resource named this because "Organisation Member" is generic enough that it can be useful outside of the context of invitations, such as listing organisation members. The skill here is to know when to abstract and when not to. When your service demands its own resource or when it can re-use what you've already written.

## Transports

Now I've got to be honest here, I only named this "transports" because of the satisfyingly alphabetic properties of the acronym "RST". Don't @ me. (again)

Transports are just the entrypoint to your application. HTTP handlers, gRPC handlers, Command line handlers, just handlers basically.

All this layer is responsible for is validating input and serialising output. Pretty self explanatory that there's no business logic here.

Works well with generated code from OpenAPI, protocol buffers, etc.

## References:

https://herbertograca.com/2017/11/16/explicit-architecture-01-ddd-hexagonal-onion-clean-cqrs-how-i-put-it-all-together/

https://dzone.com/articles/package-component-and

https://twitter.com/simonbrown/status/969108890527903744
