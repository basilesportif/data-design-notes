This is about the data spec posted [here](https://gist.githubusercontent.com/loganallenc/447afd3298fcf6ed0628b3a80d636a67/raw/80774c10b85ff891506ea6011543d74b0f77d0d1/graph-store.hoon)

I’m writing this because it’s helpful for me to think about the possible ways to represent social networks.

It also a good test of Urbit’s primitives—this is exactly the type of use case they *should* support seamlessly. The spec is short, but it can be because identify, networking, and storage primitives are completely assumed.

I don’t have a background in writing social backends, so some of this is me figuring out on the fly. I am not affiliated with Tlon or in any close correspondence—this spec just got me hype about social, broadly defined, and I want to expand on and share that excitement, regardless of where this individual spec leads.

## Base Data Structures
It’s nice to know where the “bottom” is in any system that has one, since that gives a mental foothold from which to work up.

This system has 4 base pieces that are then lego’d together in most possible permutations to permit efficient-ish queries that start with any combination of the 4 as a given.

* node
* resource
* entity
* time

These structures are not circular: rather, each time one is added, all the relationships are indexed by each, in a manner that’s similar to adding to a db index upon writing.

### node
Nodes are content + metadata.

#### metadata
* `author`: an Urbit ship. We get identity verification for free
* `hash`: possibly a hash of the node and all its children. It could also be a hash of just this node.
* `signatures`: this allows other ships to make binary on/off attestations about this node. Most social networks would make these mean "likes", but you could also use it for email/messaging to mean "read"
* `index`: This is a `(list time)`, a type repeated both in the metadata and the content. My best guess here is that it's meant to record all updates (to `signatures` most likely)
* `replies`: a list of `node`, self-explanatory (think of all the reply trees branching off from a Twitter comment, or the replies to a given email).
* `contents`: a list of `content`, see below

#### content
This is a tagged union, which can be either:
* `text`
* `url`
* `reference`: a `uid`, which is a tuple of `[resource entity index]`, where `index` is a `(list time)`

`%reference` allows linking of content (and hence a node) to a resource and entity. We’ll get to those in a sec, but this can be used semantically in two main ways:
1. As an equivalent to a list of email recipients or `@`ing people in Twitter. To do this, give a blank or dummy value for `resource` and `index`--this lets `resource` say WHAT this is (maybe `%urtwitter.mention`) and `entity` represent WHO it's about. 
2. as a simple reference to another set of nodes, keyed by `resource`/`entity`, i.e. a very rich hyperlink

### resource
This is the simplest of the base data types, although it has some room for interpretation. It consists of:
* `ship`: an Urbit ship id
* `term`: a `@tas`. I'm not sure yet what this is used for.

### entity
A person or “corporation” (in the sense of an associated group of people treated as a "person")

## Possible Queries

## Possible Use Cases
I've touched on many of these in passing already, but it never hurts to be explicit. It's exciting how many things you get "for free" when you get the data structure right.

### Forum

### Chat

### Feed of Frens (Twitter, FB)

### Email

### Banking/Generalized Accounting
This is a bit more out there, but you definitely should be able to implement any banking/accounting action as a node with the corresponding signatures. I have to think more about whether replies are relevant here.

## Remaining Questions
* Why does a uid have multiple times? Is this for recording updates?
* Do entities reflect the participants for a whole chain of nodes, or just the node whose content they’re for? How is this a useful structure to index on when it 2^n possible values
* Why are the `reference`s in `content.node` not mandatory? What does it mean for a node to not have `reference`s for its content?

## Summary
I won't be sure that this data model holds up to all use cases until it's stress-tested; a lot of stuff tends to come out in the course of actually building applications, both wrt performance considerations and missing pieces of a model.

The goal is achieveable, inspiring, and righteous: a generalized database structure for all social graph interactions at any scale, both permissioned (private groups) and open. The rest is an implementation detail in the most positive and challenging sense of the word.
