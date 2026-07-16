# GDX — A Primer

- **Status:** Draft v0.4.0 (companion to the GDX Specification, pre-1.0)
- **Date:** 2026-05-25
- **Authors:** Julian Mussi

> **This document is non-normative.** It introduces GDX (Graph Data eXchange) by example and explains the thinking behind its design. It teaches; it does not define. Where a precise rule matters, the **GDX Specification** is the authority. A reader new to GDX should start here and move to the Specification to build or to settle exact questions.

---

## Author's Note

Why an author's note in a primer for a data-exchange model? Because, instead of writing the novel I've talked about, my only literary contribution will be this primer. So indulge me.

GDX started in my home office about ten years ago. I'd taken on a project for a client, a web app due in about three weeks (spoiler: it took much longer than three weeks), and early in development I realized I would need a way to get data out of my desktop app's database and into the new web app. A few tables, to start. But scope did what scope does, and before long I had a lot of tables with tricky references. Rejecting the obvious fix (dump the tables and just rebuild them with a script) as far too practical, I decided to take "the afternoon" and create a file format for exchanging color data.

That afternoon lasted about three days. My app data needed context, carried metadata, and had both self-referential and external links. Quickly blowing past a simple CSV-type format and not having the patience to commit to a binary format, I loaded up my trusty JSON library. But JSON wasn't doing me any favors as my metadata requirements grew. Most of these issues were solvable in XML, but the XML rabbit hole is deep. When the requirements grew to include exchange with third-party systems, I felt obliged to produce something coherent enough that *another* system could make sense of it, and I soon found myself building schemas. The quest for self-description and typing had begun, and it was unpleasant.

The thing I shipped worked, but its inelegance (obvious to me and perhaps three other people on the planet) kept me awake at night, knowing that I had sent out into the world a monstrosity like Frankenstein's monster. The feeling was too much, and I decided to try to build something really good to replace it. Starting from a practical point, I decided that a cleaned-up XML format would deliver what I wanted. Sketching out various schemas, I found a nice tight approach that met my needs perfectly. And then I immediately decided that it would be much better if this nice valid solution also worked for a more general problem set. Why fix one target problem well when I could fix many problems nearly as well? It was at this point that I had the brilliant revelation: why restrict myself to a mature, well-supported file format when I could reinvent the wheel and this time do it better! Best of all, I now had a way to fill my spare time, my afternoons, my nights, my weekends, and my vacations. I now had a justification for reading RFCs at coffee shops. I now had opinions about XML schema approaches, joked about YAML bugs, and spent time crawling through JSON parser code. With my hard-earned perspective, I sat down to draft my new file format. Everything was fine for a while. And then one day, while minding my own business, I stumbled onto an article about the semantic web, which led me to RDF, and OWL, and down the ontology rabbit hole I went, emerging a few years later using phrases like *semantically orthogonal* at children's birthday parties. Somewhere during this time there was a lost weekend, waking up next to a book on Gellish with no memory of how I got there. There was the month of graphs that I still can't speak about. And each time I looked back at my project, I saw it through a different lens: a serialization format, an object model, a schema language, an ontology, a knowledge base... a serialization format, an object model, a schema language... are you getting it? Not five separate concerns, one stubborn, quixotic quest to fit all of them into a single coherent thing.

After another couple of years of refining, throwing the whole thing out, writing a truly awful implementation that revealed all the ways to do it wrong, starting over, pruning, and then rewriting it from scratch one more time, I started circling the shape laid out below.

Whether GDX shall turn out to provide a benefit, or whether that station will be held by another attempt, these pages will show. So what is GDX? First, it's a way of thinking about how data is organized, labeled, and exchanged so that it becomes *information*. The goal, hopefully realized below, is to let one small unit of data work on several planes at once. Within a GDX element we have a typed value; a name, so we can speak about it; an address, so we can point at it; a shared meaning; and a node in a graph, linked to whatever it relates to. Lots of approaches can do that. GDX does it in a small space, using position and shape to carry meaning. That's what lets a single GDX line say so much.

The risk, of course, was complexity, and yes, GDX can get complex. It hands you tools for doing some genuinely neat things. But it also lets you forget, ignore, shun, dismiss, and divorce that complexity in favor of simple statements: a key-value pair with a type. In its simple form, GDX is like BASIC, or, for the younger and ever-growing cohort, Python. At the other end, it's a line of C# crammed with lambdas and attributes. And it tries very, very hard to *tell you how to use it*: the simple, repeating form means you can only go so far off the rails, while quietly removing hard questions of morality like "is this an attribute or a value?"

GDX is opinionated about how things should be done. It treats bytes differently from strings. It's clear about what a reference is. It won't let you invent new building blocks when the ones it gives you compose just fine. And it thinks a value should be allowed to be just a value, right up until the moment you need it to be more.

Below is the attempt to make something useful. It's what I built instead of writing that novel.

> **A note on this draft.** The Primer is in active revision ahead of GDX 1.0. Its structure — an Author's Note, a guided tour of the model, the capabilities that follow from it, and an honest comparison to neighboring formats — is settled, and the prose throughout is written to teach rather than to specify. Details may still shift as the Specification stabilizes; where the two ever disagree, the Specification governs.

---

# Part I — An Introduction to GDX

**GDX (Graph Data eXchange) is a model for structured data in which each piece of data describes itself** — its type, its meaning, and how it relates to everything else. It addresses the same family of problems as JSON, YAML, and XML, and has several concrete forms: two text serializations (one verbose and explicit, one compact), a binary encoding, and an in-memory representation for working with GDX in a program. What that self-description buys you is the rest of this primer's subject: a value states its own type, can state what it *means* in terms other systems will recognize, and can point to the other data it relates to — all on the line itself, rather than leaving you to infer it or look it up elsewhere. This primer builds that idea up from a single line of GDX, then steps back to why it's shaped this way and when to reach for it. A reference implementation, with a full object model and API, exists separately; this primer and the Specification describe GDX itself, independent of any one implementation.

Most structured-data work runs into the same wall. You want data that is all of these at once: easy for a person to read and edit, unambiguous about its types, able to carry what its fields *mean* and not just what they're named, checkable against a schema, and able to express relationships between things — not just nested boxes. No common format gives you all of that. You pick one and give up the rest.

Here is the same small record in three familiar formats. Watch what each one *doesn't* say:

**JSON:**

```json
{ "customer": { "firstName": "Alice", "age": 24, "active": true } }
```

**YAML:**

```yaml
customer:
  firstName: Alice
  age: 24
  active: true
```

**XML:**

```xml
<customer>
  <firstName>Alice</firstName>
  <age>24</age>
  <active>true</active>
</customer>
```

All three are readable. But none of them actually says that `age` is a whole number rather than the text `"24"`, or that `active` is a true/false value rather than the word `"true"`. None of them says that `firstName` is "the same thing other systems call a *given name*." And to check any of this against a schema, you reach for a *second* language — a separate schema file in a different syntax. The format carries the shapes; the meaning lives somewhere else, in your head or in documentation or in code that already knows what to expect.

GDX is a way to write data so that the meaning travels *with* it. Here is roughly the same data in GDX:

```
<OBJ Alice
    <STR firstName : "Alice">
    <INT age : 24>
    <BOOL active : true>
    <@OBJ #knows : @Bob>
>
```

Read it left to right. Each line starts with a short tag — `STR`, `INT`, `BOOL` — that says what kind of thing the line is: a string, a whole number, a true/false value. You don't have to guess that `age` is a number; the line says so. The last line is different: `@OBJ #knows : @Bob` isn't a value at all, it's a *relationship* — it says this Alice "knows" another thing called Bob, somewhere else in the data. So a GDX document is at once a readable document and a graph of connected things, in one form, with the types and the meaning stated on each line rather than assumed.

That is the whole idea: **one format that carries types, meaning, structure, relationships, and its own schema — without making you choose which two you get.** GDX uses a small, fixed set of building blocks that you combine, rather than a format you extend with new syntax; that fixedness is what lets any GDX tool read any GDX document without being told the rules in advance. The rest of this primer builds that picture up from the smallest possible piece.

**How to read the rest.** Part II is a guided tour: it starts with a single line of GDX and adds one idea at a time until the example above makes complete sense. Part III steps back to lay out the capabilities that come with all of it — what you can do once your data is shaped this way. Part IV sets GDX beside JSON, YAML, XML, RDF, and binary formats, to see honestly where it sits among them. If you want exact rules at any point, they live in the GDX Specification; this primer will point you there only where it genuinely helps.

---

# Part II — A Guided Tour of the Model

> *This part walks through GDX one concept at a time, building a single example up from its smallest piece. Each step assumes only what came before it. By the end you should be able to read any GDX element and know which construct to reach for when you write your own.*

Before the first element, two quick orientations.

What you are about to read is the GDX *model*, written down. GDX is a model of how data is shaped; text is the form you author it in, the way source code is the form you write a program in. When this tour shows you a line of GDX, it is showing the model in the one form you can see on a page — so we'll talk about the text, but the thing we're really pointing at is the model underneath it.

And you'll meet a handful of small punctuation marks — `:`, `#`, `@`, `%`, and a couple more. None of them are concepts to memorize. They're recognition aids: they let you, and a parser, pick out the parts of a crowded line at a glance. The meaning of an element always lives in its parts and where they sit, never in the punctuation. We'll name each mark as it appears and move on.

## A value that knows what it is

Here is a complete, legal piece of GDX:

```
<STR firstName : "Bob">
```

The first thing on the line — and the first thing in every GDX element — is the **prefix**: here, `STR`. The prefix is the most distinctive part of GDX, and it's worth meeting first.

`STR` is a **Property** — an element that holds a single value — and specifically the *string* Property. There are others of the same family: `INT` holds an integer, `BOOL` holds a true-or-false, `FLOAT` a decimal number. Each is a Property; the prefix says which kind. So before you've read anything else on the line, `STR` has already told you two things: that this element is a Property, and that the value it holds is a string.

Next is `firstName` — the element's **Name**. It's how we refer to this particular element: its handle within the document. Naming is worth a step of its own, so we'll come back to it.

Last is `"Bob"` — the content the element actually holds. The slot it sits in is called the **Resource**. It's a deliberately broad word: a Resource can hold a plain value like the string `"Bob"`, and later we'll see it can also hold a pointer to another element. For now it holds a literal — the string `"Bob"`.

Put the three together and read the line back: *a string Property, identified as `firstName`, holding `"Bob"`.* That sentence is the whole element, and every part of it is stated openly, on the data itself — what kind of element it is, what it's called, and what it carries. Nothing is inferred from a schema you loaded elsewhere or guessed from the shape of the value. This is the first and most basic sense in which GDX data **describes itself**: change the content and the prefix changes to match it — `<INT age : 24>`, `<BOOL active : true>` — but the shape never changes. A prefix saying what the element is, a name for it, a Resource holding what it carries. A program reading the line knows all three without help; so do you.

One mark before we move on. The `:` between `firstName` and `"Bob"` is punctuation, not a concept — it marks where the Name ends and the Resource begins, so you and the parser can find the boundary at a glance. A different mark would do the job just as well; nothing about the element would change.

## Naming it so we can refer to it

We deferred one part of that first line. Here it is again:

```
<STR firstName : "Bob">
```

The `firstName` is the element's **Name** — the name we give it so we can refer to it. Every element can carry one. Without it, the element still holds its value perfectly well, but we'd have no handle for it: no way to say *this* property as opposed to that one, no way to point at it from elsewhere. The Name is that handle.

For a Property like this one, the Name is a **local** name. It means something *within the context the property sits in* — it names the role this value plays among its neighbors, the way `firstName` is a meaningful field among the others you'd expect when describing a person. It isn't a globally unique name, and it doesn't need to be: two different people can each have a `firstName`, with no conflict, because each is named only within its own surroundings. (Later, when we meet elements that *do* need to be findable across a whole document, we'll see identity reach further. For now, local is exactly enough.)

The Name is the only kind of name we need yet, but it isn't the only kind an element can carry — GDX has a couple of other identity fields for jobs that haven't come up. We'll meet them when something actually calls for them, rather than listing them now.

With the Name named, we can see the whole element for what it is — and notice it has two distinct halves. On the left of the `:` is everything about *what the element is and how we refer to it*: its role (`STR`) and its name (`firstName`). On the right is *what it actually holds*: the Resource, `"Bob"`. GDX draws this line through every element — one side identity, the other side value. It's a small thing to notice now, with only a name and a string to divide, but it's a division the whole model is built on: it's why an element can tell you everything about its place in the data before you ever read its contents.

## Grouping values: the Record

A single property is rarely the whole story. Bob has a first name, but also an age, and probably more. In GDX you collect related properties by placing them inside a **Record**:

```
<REC person
    <STR firstName : "Bob">
    <INT age : 24>
>
```

A Record is a named thing made of properties — the struct, in a programming-language analogy. `REC` is its prefix; `person` is its Name, naming this record the same local way a property's Name names a property. And inside it sit the two properties, each exactly the element you already know how to read: a string `firstName` holding `"Bob"`, an integer `age` holding `24`.

Notice what *isn't* there. Nothing on the `firstName` line says "I belong to `person`." There's no joining word, no pointer back to the parent, no mark connecting child to container. The relationship is carried entirely by **position**: `firstName` is inside `person`, so `firstName` is one of `person`'s properties. GDX leaves parent-child containment implicit because it doesn't need stating — the nesting already says it, and the child's role already says *what kind* of thing it contributes. Spelling it out on every line would only repeat what the structure makes plain.

And here is the first quiet thing worth noticing about that arrangement. Putting `firstName` inside `person` isn't only filing it there — it's a small claim: *person has a firstName, and it's "Bob".* Each property nested in the record is the record saying it *has* that attribute, with that value. You don't write the "has" anywhere; it's what the containment means. For now that's just a nice way to read the structure. Later it turns out to matter more than that — but we'll get there one piece at a time.

## A thing that stands on its own: the Object

The `person` record from the last step was a *compound property* — a thing that belongs to something else, the way an address belongs to a customer or a shipping detail belongs to an order. But often what you're describing isn't a part of something else. It's a thing in its own right: a person, an order, a server, a document. For that, GDX has the **Object**:

```
<OBJ bob
    <STR firstName : "Bob">
    <INT age : 24>
>
```

An Object represents an **entity** — a thing that exists on its own and can be referred to. Structurally this is almost identical to the record you just saw: a named container holding properties. The prefix is `OBJ` instead of `REC`, and that one change carries the whole difference. An Object is **self-contained** — it doesn't belong to a parent the way a record does; it stands on its own as a complete, addressable entity. It's the unit GDX treats as a whole: the thing it stores, the thing it sends across a wire, the thing the root of a document is built from. If a Record is a *part*, an Object is a *whole*.

Being a whole is exactly what makes an Object referable. Its Name — `bob` — isn't just a local label among siblings; it's an address. A Record's name only locates it inside its immediate parent; an Object's name locates it across the whole document, so anything elsewhere in the data can name `bob` to point back at this entity, and find it. That difference is the foundation of what comes next: once an entity has an address others can use, those others can link to it — and that is where GDX stops being a tree and starts being a graph.

## Linking entities: edges and the graph

Everything so far has been one entity described in isolation — Bob, with his properties. But data is rarely a set of isolated things; the interesting part is usually how things *relate*. Bob knows someone. To say so, we need a second entity to point at, so let Alice into the document:

```
<OBJ alice
    <STR firstName : "Alice">
>
<OBJ bob
    <STR firstName : "Bob">
    <INT age : 24>
    <@OBJ #knows : @alice>
>
```

The new line inside `bob` is an **edge** — a link from one entity to another. Read it left to right like any element. The prefix `@OBJ` says this element points at an Object. The `@alice` on the right is the target: not a value Bob holds, but the *address* of another entity — Alice, whom we just gave a name precisely so she could be pointed at. And between them, `#knows` is the **label** on the link: it says *what kind* of relationship this is. The whole line reads: *bob knows alice.*

Two marks earn a word here, since this is the first time you've seen them. The `@` is how GDX flags "this points at an entity" — for you, scanning, and for the parser. The `#` flags the label that names the relationship. Like the colon, they're recognition marks, not concepts; the meaning is in what they point at. The label `knows` has more behind it than first appears — it's the kind of thing two different systems can agree on — but that's a later step; for now, read it as the name of the relationship.

Notice how this differs from a property. When we nested `firstName` inside `bob`, the relationship was silent — *bob has a firstName* — carried by position, with no "has" written anywhere. An edge makes the relationship *explicit and named*: `#knows` states the link out loud. That's the difference between an attribute an entity carries and a relationship it stands in — and it's why the edge gets a label where the property didn't need one.

And with that one line, the shape of the data changes. Until now everything nested neatly into trees: properties inside records, records inside objects. But `bob` now points *sideways*, to `alice`, who sits in her own part of the document. The data is no longer just a tree of containment — it's a **graph**: entities connected by labeled links, in any direction. This is the dimension the name *Graph Data eXchange* points at, and it falls out of one idea — that an entity, once it has an address, can be linked to.

## Pointing without linking: references

The edge gave us one way for an element to point at another: assert a *relationship*. But sometimes you don't want to assert a relationship — you just want a slot to *hold a handle to something else* rather than hold a literal value. That's a **reference**, and it's the second of the two kinds of pointing in GDX.

Recall the Resource — the value slot on the right of an element. So far it's held literals: `"Bob"`, `24`. A reference is what you put there when the slot should point at another element instead:

```
<OBJ order-1001
    <%OBJ customer : @bob>
>
```

The `%OBJ` is a **reference**: the slot `customer` doesn't contain a customer, it holds a handle to one — the Object `bob`, living elsewhere. Read left to right: the prefix `%OBJ` says "a handle to an Object goes here," and `@bob` is the address it resolves to. The order's `customer` slot is *filled by* a pointer to Bob.

This looks a lot like the edge from the last step — both end in `@bob`, both point at an entity. But they do genuinely different jobs, and GDX keeps them firmly apart:

- An **edge** (`@`) *asserts a relationship*: bob **knows** alice. It's a statement about how two entities relate, and it carries a label (`#knows`) saying what the relationship is.
- A **reference** (`%`) *fills a slot*: this order's customer **is** bob. It isn't asserting a relationship about Bob — it's saying "the thing that goes in this slot is over there."

The sigils keep the two from blurring: `@` always means "asserts a relationship," `%` always means "fills a slot with a handle." They're never interchangeable, and once you read them that way, a dense line tells you immediately which kind of pointing you're looking at. The question to ask is simply: *am I stating how two things relate (edge), or saying what goes in this slot (reference)?*

There's a useful way to say what a reference *means* when you read one: **the value here is the value of the resolved target** — "my content comes from there." The document stores the address (that's what travels and round-trips); following it gets you the real thing (the spec's §6.16 calls this *value-resolution*). That reading also settles the borderline cases. "This field's value is that whole object" — a reference, because you're sourcing a value, even though the target is a whole entity. "These two entities are related" — an edge, because you're stating a relationship. And it explains a capability difference you'd otherwise have to memorize: a reference can reach *inside* an element (pull one field's value out of a record somewhere — sourcing a value from an interior is perfectly sensible), while an edge never can — you don't draw a graph edge to half of an object.

One affordance falls out here. For `order-1001` to point at `bob`, Bob has to be findable — he needs a stable address. An Object's Name already provides that. But sometimes the thing you want to point at isn't a whole Object with a global name — it's an element tucked inside one, and its position might shift as the document is edited. For those cases an element can carry a **Anchor**: a stable anchor, independent of where the element sits, that references can target even when nothing else about its location holds still. You don't need an Anchor for most pointing — an Object's Name is enough — so we'll leave it there: it's identity's way of saying "here's a fixed handle on me," available the moment a reference needs one.

There is also a third way to fill a slot, used less often: a **VAL**. Where a reference points at something that lives elsewhere, a VAL holds a structured value *right there in the slot* — a composite that belongs to this element and nowhere else, rather than a handle to an independent entity. It's the slot-filler for "the value here is itself a structured thing, owned by this element." We mention it for completeness; most slots hold either a literal or a reference, and VAL is there for the case that needs it.

## Naming the relationship: predicates

Two threads have been left deliberately loose, and this step ties them together.

The first was when we nested a property inside a record: *bob has a firstName.* The relationship — "has" — was never written down; it lived in the position. The second was the edge: `#knows`, which we read as "the label on the link," with a quiet note that there was more behind it. Those two loose ends are the same thing, and it has a name: a **predicate**.

A predicate is the *name of a relationship.* When `bob` knows `alice`, the relationship is `knows`. When a record has a `firstName`, there is a relationship there too — it just went unnamed because it didn't need naming yet. The `#knows` you saw on the edge is a predicate written out in the open; the silent "has" of a nested property is a relationship you simply chose not to name. Predicates are how GDX lets you name a relationship when naming it earns its keep.

And the reason to name one is the payoff GDX is built for. A bare label like `knows` is just a word — but a *predicate* is a shared, agreed-upon name. Two systems that store their data differently, that call their fields different things, can still both point at the predicate `knows` and mean the same relationship by it. That is what the introduction promised when it said a value can state what it *means* in terms other systems recognize: the meaning rides on the predicate, not on whatever the field happens to be called locally. A field named `pals` in one system and `acquaintances` in another can each carry the predicate `knows` — and now they line up, with no translation table, because the shared meaning travels on the data itself.

Because a predicate is a shared name, it helps to be able to *define* it — to say, in one place, what `knows` is. GDX does this with a **relation declaration**, written `REL`:

```
<REL gdx:knows
    <@REL #gdx:inverse : @gdx:knownBy>
    <@OBJ #gdx:domain  : @gdx:Person>
    <@OBJ #gdx:range   : @gdx:Person>
>
```

This declares the predicate `knows` and says a few things about it: its inverse is `knownBy`, and it relates a `Person` to a `Person`. (The `gdx:` in front of each name just says which shared vocabulary the name comes from — more on vocabularies in a moment.) Notice what this *is*, though: it's just GDX. A predicate is declared with the same elements, edges, and references you already know how to read; `gdx:Person` is simply an Object with a conceptual name, pointed at like any other. GDX has no separate language for "defining vocabulary" — the things that give your data shared meaning, the predicates and the concepts they relate, are themselves ordinary GDX. A vocabulary is just a GDX document whose contents happen to be definitions.

That is the quiet payoff the earlier steps were building toward. Each line you wrote was already a small statement about how things relate — *bob has a firstName, bob knows alice.* Predicates are what let those relationships carry meaning past the edge of your own document, into any system that shares the names. The relationships were there all along; this is the step that lets them travel.

## Claiming a shape: Type and schema

There is one field of an element we haven't used yet, and one last idea it unlocks.

So far every element has been valid on its own terms — a `STR` holds a string, a `REC` groups properties — but nothing has *checked* that an element is shaped the way some agreement expects. Often you want that check: an address should have a street and a city; a customer record should carry the fields a customer is supposed to carry. In GDX, an element can claim to conform to a named shape by carrying a **Type**:

```
<REC address --schema:address
    <STR street : "Main St">
    <STR city : "Oakland">
>
```

The `--schema:address` is the Type. It's the element saying *I claim to be a `schema:address`* — that this record matches the shape that name stands for. A validator can take that claim and check it: does this record actually carry what a `schema:address` is supposed to have? The claim travels on the data, like everything else; the element announces not only what it is and what it holds, but what shape it asserts it conforms to.

The shape itself — the thing `schema:address` names — is called a **Template**, and here is the move you've now seen once already. A Template is written in GDX:

```
<DEFREC schema:address
    <STR street>
    <STR city>
>
```

That is the definition of `schema:address`: a record that should have a `street` and a `city`. It's an ordinary GDX element — the `DEFREC` prefix marks it as a *definition* of a record rather than an instance of one, but its body is just properties, read exactly as you've read properties all along. The properties have names but no values, because a Template describes the *shape* a conforming element must fill, not the data that fills it.

This is the same lesson predicates taught, now for schemas: GDX has no separate language for defining structure. Just as a vocabulary was a GDX document full of predicate definitions, a schema is a GDX document full of Templates — written in the same form, read by the same parser, as the data they describe. The schema *is* data. There is nothing new to learn to read one; you already can.

## Everything together — and how little you need

You've now met every part of a GDX element, one at a time. Here they all are on a single line:

```
<%OBJ &primary #schema:customer customer : @bob --schema:customerRef>
```

At the start of this tour, that would have been noise. Now you can read it. It's a **reference** (`%OBJ`, a handle to an Object) filling a slot named `customer`; it carries a stable anchor (`&primary`) so other elements can point at it; it aligns to a shared predicate (`#schema:customer`) so its meaning travels; the handle resolves to `@bob`; and it claims to conform to a Template (`--schema:customerRef`). Every field doing its job at once. You can take it left to right and say what each part is — which is the whole skill the tour set out to give you.

A line that dense is not something to avoid. If you're using everything GDX offers — typed values, shared meaning, stable anchors, schema claims — a line like that is simply what it looks like when you need all of it, and that's fine. The point isn't to write less; it's that **you choose what you need, and you don't pay for what you don't.** Look back at where the tour started:

```
<STR firstName : "Bob">
```

That was a complete, legal element too. So was every element in between. Each step added something — a name, a record around it, an edge, a predicate, a type — but nothing along the way was *forced*. An element is valid with just its prefix; a property can be a bare `<STR firstName>` with no value at all. The packed line above and the plain one here are the same model, asked for different amounts. Neither is more "correct" — each carries exactly what its situation calls for.

That is the discipline underneath everything you've read. GDX gives you a lot of room — typed values, named relationships, shared meaning, schemas, a graph — and charges you for none of the room you leave empty. You reach for each piece when the data needs it, the way you reached for a record only when one value wasn't enough, or a predicate only when a relationship needed naming. Simple stays simple; complex is there when you want it; and it's all the same handful of moves, composed.

One last thing, now that you can read all of it. Everything in this tour has been written one way — bracketed, with full three-letter prefixes — but that is only one of GDX's surface forms, and the model underneath doesn't depend on it. Two independent choices vary the surface. The **prefix tokens** can be written in full (`STR`, `OBJ`, `REC`) or **minified** to single letters (`S`, `O`, `R`). And the structure can be drawn with **brackets** — what you've seen — or in **Compact Form**, which drops the brackets and lets indentation carry containment instead, closing each element when the indentation steps back out. Here is the same `bob`, written four ways:

```
Standard form, full prefixes        Standard form, minified
<REC bob                            <R bob
    <STR firstName : "Bob">             <S firstName : "Bob">
    <INT age : 24>                      <I age : 24>
>                                   >

Compact form, full prefixes         Compact form, minified
REC bob                             R bob
    STR firstName : "Bob"               S firstName : "Bob"
    INT age : 24                        I age : 24
```

Four renderings, one element. The prefixes are longer or shorter; the brackets are present or replaced by indentation — but the model is identical in all four: the same record named `bob`, holding the same string and the same integer. This is what it means for GDX to be a model with several realizations rather than a syntax. The text is how the model is written down, and it can be written down more than one way. Reading any of them is the same skill — which is now yours.

The tour ends here. What follows steps back from *how* GDX works to *why* it's shaped this way, how it compares to the formats it sits beside, and where it's the right tool — and where it isn't.

---

# Part III — What GDX Gives You

> *The tour showed you how GDX works. This is the payoff: the capabilities that come with it, now that you can see what each one rests on. Each header names a capability; the prose makes the case — the problem it ends, and what it does for you.*

**Explicit types.** A value's type is fixed the moment you write it, and every tool reads it the same way — no inference, no loading a schema first to find out whether a field is a number or the text of a number. If you've wrangled config in YAML, you know the other way: a `NO` that parses as `false`, a version `1.10` that turns into a float and loses its trailing zero, a value that's a string on Tuesday and an integer on Wednesday depending on what's in it. Those bugs come from types left to inference. In GDX the prefix carries the type — `<INT age : 24>` is an integer, `<STR version : "1.10">` stays a string — so there's nothing to infer and nothing to get wrong downstream.

**Inline semantic linking.** Names are local: one system's `firstName` is another's `given_name` is another's `fname`, and reconciling them is the work mapping layers and integration glue exist to do. GDX lets a field carry a shared *predicate* beside its local name, so `firstName` and `fname` can both declare themselves `schema:givenName` and align with no table in between — the meaning rides on the data, not in glue between systems. This is the multi-tenant integration problem dissolved: tenants keep their own field names and still agree on what the fields *mean*.

**First-class predicates.** If you've reached for RDF or OWL to model a domain, you know the power is real and so is the cost: triple stores, blank nodes, several serializations, a query language and a reasoner before you can do anything. GDX gives you the load-bearing parts — named relationships you can define and share, concepts you can relate, axioms like inverse and domain and range — using nothing but the elements you already write. A predicate is declared with a `REL`, a concept is just an Object with a conceptual name, and a whole vocabulary is a GDX document full of definitions, read by the same parser as your data. You get shared, machine-checkable meaning without a second modeling stack to stand up and operate.

**Schema as data.** In most stacks the schema is a separate language — XSD is not XML, JSON Schema is a convention bolted onto JSON — so you maintain two artifacts in two grammars and keep them in sync by hand. In GDX a schema is a GDX document: a Template is an ordinary element, read by the same parser, and it can travel in the same file as the data it describes. There's nothing new to learn to read or write one, and no impedance between the shape and the thing it shapes. For config especially, this is the missing middle: readable like YAML, but with a real schema in the same readable form, not a typed language you switched to just to get safety.

**Layered validation.** Most formats give you one flat verdict — valid or not — and leave you to work out why. GDX's validation comes in distinct layers with distinct owners, so a failure tells you which layer it lives in. GDX itself checks structure: is this actually an integer, do these fields match the Template's shape. Finer rules — numeric bounds, string patterns, semantic constraints — are carried by constraint vocabularies that ride on top, each interpreted by the consumer that understands it. The same element can satisfy one vocabulary's rules and not another's, and you can host several on the same data at once. You get a precise, layered answer instead of a single opaque "invalid," and you always know whether a failure is GDX's to report or a vocabulary's.

**Materialized instances.** When a schema fills in values an instance leaves out, the instance is only whole *with the schema in hand* — pull it into storage or hit it with a query on its own, and the schema-supplied parts are missing. GDX can write those values into the element before it's stored, so the stored thing is complete by itself: searchable, queryable, and parseable with no schema loaded. You author against the schema; your data doesn't depend on it afterward.

**References as data.** Anything worth pointing at can be pointed at — not just top-level objects with global names, but elements tucked inside them, reached by a stable anchor that survives the document being rearranged. And the pointers themselves are ordinary GDX: an edge linking two entities, a reference filling a slot with a handle, are elements like any other, with their own identity and attributes. Configurations point at objects, vocabularies point at schemas, records point at each other — one uniform model, nothing special-cased about the act of pointing, and references you can attach data to like anything else.

Underneath all of these is the property the tour ended on: you reach for each capability only when the data needs it. An element is valid with nothing but its prefix; type, meaning, schema, links, and anchors are there when a situation calls for them and silent when it doesn't. None of the powers above is mandatory — which is what lets the same model carry a one-line typed config value and a fully-aligned, schema-validated, graph-connected entity, and ask no ceremony of the simple case.

---

# Part IV — How GDX Compares

> *This part is a map, not a pitch. GDX sits in a long-settled neighborhood of data formats, each built for a real purpose and good at it. The point here is to see the terrain honestly: what each format was designed to do, where its design draws a line, and how a single idea takes different shapes — or fails to take shape at all — as you move across them. GDX is one territory on this map, with its own good ground and its own edges.*

Every format here exists because it solved a real problem well, and each carries the marks of the problem it was built for.

**JSON** was built to serialize JavaScript objects, and its great virtue is that it does almost nothing else: a handful of types, two container shapes, supported everywhere. That minimalism is why it won, and it's also where its design draws lines. JSON has no place to record that a number is an integer rather than a float, no native binary, and — structurally — no slot for a field's meaning or a claim about its type; those live outside the document, in a schema written in a separate language or in convention the receiver is expected to know. JSON is the right answer when reach matters more than what the data can carry on its own.

**YAML** was built for humans to write, especially configuration, and it's pleasant for that. Its design choice to *infer* types from how a value looks is what makes it terse — and the same choice is its sharpest edge: `yes`, `no`, `on`, `off` may read as booleans, `1.10` may read as a float and lose a digit, the country code `NO` may read as false. The information is recoverable only by quoting discipline the author has to remember. YAML is ideal where a person is the primary author and the document stays small enough to hold in the head.

**XML** was built for marked-up documents — text with structure woven through it — and for that mixed content it remains well-suited, with mature namespaces and a real schema language in XSD. Its design has two edges for pure data: verbosity, and the unresolved question of whether a given datum belongs in an attribute or a child element, which two XML documents will answer differently with no principled rule to settle it. Excellent for documents; heavier than necessary for data interchange.

**RDF and OWL** were built for shared meaning at web scale — the ontology problem — and within that purpose they are the most powerful tools here: a uniform triple model, predicates as first-class things, formal inference. The design pays for that power in operational weight: data decomposed into triples that must be reassembled to recover a whole entity, several serializations, blank nodes, named graphs, and a reasoning stack. The capability is real; the cost is why many application developers reach past it.

**Binary protocols** — Protobuf, FlatBuffers, Cap'n Proto, Avro — were built for the wire: minimum bytes, maximum parse speed, schema-defined and compiled ahead of time. Where every byte and microsecond counts, they are simply the right tool, and nothing readable will beat them. The design trade is deliberate: the bytes are not meant for a human to read, and the schema lives in a separate definition compiled into code.

Each of these traces to a single motivating purpose — markup for XML, human authoring for YAML, ontology for RDF, object serialization for JSON, wire efficiency for binary — and each is strongest squarely inside that purpose. The shape of the space GDX occupies is different: not a sixth single-purpose point on the map, but the region in the middle, for data that needs *several* of these things at once — explicit types, inline meaning, in-document schema, a graph, and human readability — where today you would otherwise pick one format and rebuild the rest by convention. GDX's own edges are equally a matter of design: it is a text model first, so on the raw wire a compiled binary protocol will beat it; and for a job that genuinely needs only what JSON does, GDX is more machinery than the task asks for.

## One concept, several shapes

The clearest way to read the map is to watch a single record take shape across it. Here is the same small entity — a person with a typed age, and a `knows` relationship to another person — written four ways. Watch two things in particular: whether `age` is committed to a type, and whether the `knows` relationship can be stated *in the document at all*.

**GDX (Standard Form):**

```
<OBJ bob
    <INT age : 24>
    <@OBJ #schema:knows : @alice>
>
```

**JSON:**

```json
{
  "bob": {
    "age": 24,
    "knows": "alice"
  }
}
```

**YAML:**

```yaml
bob:
  age: 24
  knows: alice
```

**XML:**

```xml
<bob>
  <age>24</age>
  <knows>alice</knows>
</bob>
```

Read for the type first. In GDX, `INT` commits `age` to an integer on the data itself. In the other three, `24` is an integer only by the reader's assumption — JSON happens to render it bare, YAML will infer it, XML can't distinguish it from the string `"24"` — and none of them records the commitment; it lives in a schema or a convention elsewhere.

Now read for the relationship. In GDX, `<@OBJ #schema:knows : @alice>` says three things in the open: this is a link to another entity (`@`), the relationship is the shared predicate `schema:knows` (`#`), and the target is the entity `alice`. In the other three, `knows: "alice"` is just a string whose value happens to be the word "alice." Nothing marks it as a *reference* rather than a literal, nothing says it points at the `bob`-level entity named `alice` rather than holding the text, and nothing carries the predicate that would let another system recognize this as the same `knows` it uses. The relationship is representable only as a convention layered on top — a rule, kept outside the document, that a field named `knows` should be dereferenced as a pointer and aligned to some agreed meaning.

That is the structural difference the map is meant to show. It is not that the other formats are weaker at their work; it is that the type-commitment and the relationship are, in those formats, things the document *cannot hold on its own* and must defer to an external layer — and in GDX they are stated on the element. Whether that property is worth reaching for is a question only your own data answers: where you need more than one of the things GDX carries, held in one readable place, it earns its keep; where a single-purpose format already does the one thing you need, that format is the honest choice.

---

*To build with GDX from worked, runnable patterns, see the GDX Cookbook. To settle any exact question of grammar, composition, or conformance, the GDX Specification is the authority.*
