# GDX — A Primer

- **Status:** Draft v0.5.0 (companion to the GDX Specification Core v0.30.0, pre-1.0)
- **Date:** 2026-10-19
- **Authors:** Julian Mussi

> **This document is non-normative.** It introduces GDX by example and explains the thinking behind its design. It teaches; it does not define. Where a precise rule matters, the **GDX Specification** is the authority.

---

## Author's Note


GDX came out of trying to reconcile conflicting goals: a format readable enough to write by hand, structured enough for machines, explicit enough to carry type and meaning, and small enough that the simple case stayed simple. It is opinionated about that balance. It treats values, references, relationships, and schemas as parts of one model rather than as separate systems stitched together after the fact.

This primer is the attempt to explain that model from the ground up: first the concepts GDX is trying to make visible, then the model itself.

---

## Abstract

This primer introduces GDX (Graph Data eXchange) by example and explains the thinking behind its design. It starts with the conceptual problems GDX is trying to make visible, then walks through the model one construct at a time. A reader new to GDX should start here and then move to the **GDX Specification**, which is the normative guide.



# Part I — Conceptual Foundation

Part I surfaces those distinctions so that Part II can move more directly through the model. If you already know why identity, type, meaning, structure, linking, and encoding are easy to blur together, you can skip ahead to the guided tour.

## Information

Rather than supply a glossary, this section explains the foundational concepts that will be used throughout this document and the core spec. The definitions are not meant to be canonical. Many of these terms shift depending on context, so the goal is to introduce them in the context in which they are used here.

A useful way to start is with the concept of **data**. For the purposes of this document, data can be understood as raw, unorganized values; facts, measurements, and observations, considered apart from any meaning we might assign them. The set of numbers 1,**46**,42,68,11 absent any other framing is data. Data are the **values** that are often recorded in variables within applications or written in documents.

Data becomes **information** when we give it order, context, and structure. When data carries meaning it becomes information. Information may carry shallow or deep meaning. Meaning is often **implicitly** conveyed through labeling or organization of information. In other cases meaning is made explicit through **ontology** or shared **semantic** definition.

For an exploration of how meaning is conveyed through information we'll use the value `46` from the list above. This value can be transformed from a data point into information by changing its structure and **labeling** the data so it reads as follows: `age : 46`. It's intuitive that the age label is adding something to the data. What may not be clear is what its exact purpose is: is it transforming it, contextualizing it, naming it? What should be stated, however, is that `age` and `46` are associated, and that the two values now have a dependency. Removing one alters the interpretation of the other.



## Form and syntax

If we were to write the above pair as `age = 46`, the set of values is easily recognized as a mathematical **statement**. In this form the terms say something. In this specific case the statement claims a fact, creating an **assertion** that the statement is true. 

Visually there is not much of a difference between writing `age:46` and `age = 46`; however, the `=` symbol is a more explicit way to express the relationship between the two values.

When discussing the way in which we arrange and present the label and the value, we are talking about its **form**. 

The same concept can be expressed in many forms:

- age = 46
- age : 46
- Their age is 46.
- <age>46</age> 
- {age : 46}
- Aetas est XLVI 

As long as we share a way to interpret it, every form above assigns the same meaning to the value. The set of rules that govern how the parts of the statement may be composed is its **syntax**.

Form does not inherently carry meaning. The relative position of information often does. Consider the following forms: `46 = age` and `Forty six is his age`. In the first case, the transposition creates a statement expressing something entirely different, while in the second we are running into a grammatical convention with more flexibility.

## Value types

Using our current example statement, if we were to ask a person (or agent) to convert this to a code snippet they might return something like `int age = 46;`. While most employable developers would probably interpret age as holding a number, an English major turned application architect might provide `age = 'forty six';` and would have a defensible argument. Indeed, both statements encode data that, when interpreted by a human, would produce the same meaning. However, to a machine these values have no inherent similarity. So as we consider the reader, or interpreter, of information, the way in which data is represented becomes increasingly important.

Defining a **data type** can be tricky. From a mathematical perspective the focus is on what values "are" and what they can hold. The computer science view is weighted toward how they are represented. The perspectives have overlap, as well as some real distinctions. For example, we might say that the age needs to be a **numeric** type. We might add the constraint that it be an **integer** (whole numbers), or perhaps we want precision and will allow **real numbers** (fractional values). At a conceptual level value types are likely to fall into categories of numeric, textual, and **Boolean**. Other categories like **Bytes**, **Null**, and **Reference** types are less representative of universal concepts and more oriented around functional concerns. The guiding principle is that each of these categories expresses a range of attributes or constraints about the type of information they contain.

All of the types identified above can be communicated conceptually. We can understand the difference between the concept of the whole numbers between 0 and 255 vs. the real numbers in that range. However, when that value moves from the conceptual form to a concrete representation, a linked but distinct concern emerges. Using the example value `46`, when this value is stored in memory its representation is now not just conceptual but concrete.

The number of bits required to **encode** a value can vary; we can use 8 bits or 32 bits to encode the integer value 255. Both hold the same conceptual value, but the way in which that data is encoded varies. This conceptual and representational split is important. It allows the conceptual constraint to focus on the class of information being stored, and the encoding to decide how that information is stored on a given medium.



## Identifiers 

Looking at the statement `age = 46` and asked to infer some meaning, the term "age" is going to provide more context than `46`. Reading "age" followed by the number `46`, readers will likely infer the same meaning from the statement. Going back to our first example, we started with `46` and added "age" to give it meaning, but agreed that we could have `46` as an independent data point. In this form "age" functions to **label** the data. However, converting this to code means the variable holding the data point `46` needs an **identifier**. 

It's possible to look at the labeled string or the variable identifier and say that it is providing the meaning for this statement, and in this example there's a strong case to support that. A term like age has clear **semantics**; its meaning is shared and has less room for interpretation. But what happens if the developer changes the name of the variable to just `a`? Have we changed the meaning of the information, or just what we use to refer to it? What if the identifier was `name`? How certain are we now about the meaning? It could represent first name, last name, or both separated by a comma. The meaning conveyed by a label or identifier can often be conflated or misinterpreted.

Identifiers also differ in scope. Some names only need to work locally, like a property name inside a record. Others need to be stable enough that something elsewhere can point at them. If a field moves, or if a document is reorganized, a purely positional name may not be enough. This is the seed of a distinction that will matter in the model: labels help readers, identifiers let us refer, and stable identifiers let other information safely point.

## Nested relationships

Let's pause here and reflect on what else the "age" label might be communicating. In this example specifically, the idea of an "age" does not make much sense without a subject, something that "has" an age. Without stating it, when we saw age we probably instinctively understood there was a relationship between some subject and the aspect of them we call an "age". If we refined the statement to include the hinted-at owner of the age, it might say `Bob's age is 46`. By doing so we've provided a **subject**. The subject is the owner of the age. And just like "age" and `46` are linked, so are "age" and "Bob".

## Properties

This aspect of Bob is a **property** of his. In this case his "age". This is interesting because it can now be said that there's an aspect of Bob, his age, that exists even if its value is undefined. Without knowing someone's age we can still be confident they have one. This is an obvious deduction for a subject that we're familiar with, like people, but not for something that we're not familiar with or that is abstract.

Asked to model the statement `Bob's age is 46`, a programmer is likely thinking "this looks like a 'person' class with an age property, of which Bob is an instance" — a snippet of code something like `class person { uint8 age; }`. If we don't know Bob's age we still know he has a property. When his age is known it can be assigned, `Bob.age = 46;`, updated or removed, knowing that the system will constrain the value to an integer type that can be no more than 255.

## Meaning

By this point it should be clear that the statement "Bob's age is 46" contains layers of meaning. We have some numeric data with the value of 46. We've labeled that data with a term, "age". With the inclusion of Bob, he emerges as our subject, the entity our property is addressing. 

At this point it should be clear that how we interpret age is critical to understanding how all the terms in this statement should be understood. If age is just a label, then "age" can be substituted with `a` and `46` should be interpreted the same. However, if "age" explicitly means something, then it's more than a label. 

In this example "age" could be viewed as taking on the shape of a **predicate**, a thing that holds true *of* something. Predicates are more than just a string; they carry meaning that is shared within a given context. In this way, if "name" means "someone's given name," that meaning is consistent.

Borrowing from logic, we could write this as `age(bob, 46)`: a predicate named `age` applied to two arguments, `bob` and `46`. Read it as the claim "the age of Bob is 46." The same relationship is the basic unit of **RDF** (the Resource Description Framework). In that form our example looks like this: `ex:bob schema:age 46.` Here `ex:bob` is the subject, `schema:age` is the predicate, and `46` is the object.

Looking back now we can see that the age field, without being explicit, could have been doing double duty as a predicate and a label. This identity vs meaning ambiguity is so common as to be often overlooked.



## Links

 When we wrote `age = 46`, `46` *is* the thing the statement is about. We call resources of this type **literals**. The object is the text, number, or bytes that are written on the page or stored in memory.

But consider a statement like `Bob knows Alice`. 

If "Bob" the subject is not just a string, but represents an entity of some sort, then it follows that "Alice" does too. Again, as the reader of this statement we can infer that Alice is likely a person, but to a machine parsing the string, "Alice" has no more inherent meaning than "cat" or "124H34a".

If Bob and Alice are entries on a wikipedia page, "bob" might appear as a blue text to the right of the "Spouse : *Bob*" field. In this case our expectation would be that clicking this text would open the "Bob" page. 

"Bob" is not just text, it carries a **reference** that has to be *resolved* before it means anything. This is the difference between a **value type** and a **reference type**: a value *is* the data, while a reference *points at* data that lives elsewhere. `"Alice"` the literal string and `Alice` are not the same kind of thing at all, one is held, the other is pointed at.

When information expands beyond the limits of literals and enables the creation of links between different entities, we have created the components necessary to construct a **graph**. At this point Bob and Alice may each represent a **node** containing features, properties, and other information that refine how we understand them. When one node references another node we have created an **edge**, a directed link that says *this* relates to *that* in some named way. `Bob knows Alice` is an edge from the Bob node to the Alice node, labeled `knows`. 

This is one of the central tensions in structured data. Some information is naturally shaped like a **tree**: a whole made of nested parts, such as an order containing line items. Other information is naturally shaped like a **graph**: independent things connected by named relationships, such as people, accounts, products, and organizations. Most formats lean strongly toward one shape or the other. GDX is designed to let both shapes live in the same model: containment for parts, links for relationships.

## Shape and schema

Once information has structure, we often want to know whether it has the *right* structure. An address should probably have a street and a city. A customer may be expected to carry a name, an account id, and a billing address. This expected arrangement is its **shape**.

A **schema** is a way of naming and checking that shape. It says, "an object or record of this kind should have these parts, with these value types, and perhaps these additional constraints." Without a schema, a reader can still parse a document, but cannot know whether the document satisfies some agreement outside itself.

The important distinction is between carrying data and claiming a shape. The statement `Bob's age is 46` carries data. A separate claim like "this is a valid person record" says the data conforms to a known pattern. GDX keeps those concerns close together: the data can carry the shape it claims, and the schema that defines the shape can be written in the same model.


## Summary

Having explored the example statement from different perspectives, the above shows the hidden depth that often lurks just below the surface. In many cases the aspects of the information that supply the meaning are understood implicitly, but when exchanged between individuals, systems, or organizations with divergent perspectives, they can quickly produce ambiguous interpretations. Rather than looking at information as representative of its form, its more honest expression is one in which orthogonal aspects of identity, meaning, syntax, type, linking, and encoding are often conflated and obfuscated. GDX strives to make explicit the facets of meaning that information implicitly carries, and by doing so, to improve how information carries its meaning.

To be clear, every aspect mentioned above can be clearly expressed using a range of existing technologies. JSON and YAML allow objects and properties to be represented in a human-editable format. XML supports typing through XSD, a robust schema language. Type and data encoding is built into every coding language, and many of them support reflection and attribution. RDF triples in a range of formats are used to build ontologies and add a semantic layer to data. Graph databases support explicit creation of nodes and edges. Not only is there a solution for every implied challenge, but there are myriad options for doing so. However, very few approaches address all of these layered concerns within a single framework. Far fewer offer a solution that is both compact and relatively simple.

The crux of this work has been trying to reconcile often antithetical objectives like completeness and simplicity. To this end, much of what defines GDX is not what it chose to include, but a ruthless refinement to leave out as much as possible while still meeting the core objectives.



# Part II — The GDX Model

## Model definition

**GDX (Graph Data eXchange) is a model for structuring data so that layered identity, meaning, and form can be explicitly communicated**. 

GDX is the model, separate from any concrete representation. Just as `2*2`, `2 X 2`, and `multiply(2,2)` are different forms conveying the same conceptual idea, different GDX representations should be viewed as different ways of specifying the state of a GDX model.

The canonical serialization format defined in the spec and used in this primer is designed to be human- and machine-editable. GDX's serialization has several concrete forms: two text serializations (one verbose and explicit, one compact) and a binary encoding. The primer uses the canonical verbose form for easier readability.

The central idea is that the conceptual concerns from Part I are represented as fields on a small, fixed set of model constructs. GDX does not require every field to be filled for an element to be valid; a simple typed value is still a fully formed piece of GDX, and the other fields are available when the data needs them.



## Element anatomy

The model GDX uses to organize and contextualize data is structurally simple.  The building block of the GDX system is the **element**. 

Elements package data so that its associations can express layered meaning. Every element has 6 data slots, or **fields**. Every bit of data lives in a field, and the element groups the fields.

The fields that make up an element are the **Prefix** (which states the element's **Role**), **Id**, **Predicate**, **Name**, **Resource** and **Type**.

The majority of the primer will be spent explaining the function of each of these fields and how they work together.



## Text form

Framing GDX as a model is correct, but without a way to represent the model it's mostly just a philosophical exercise, and GDX was designed to be used. Its primary format is text that can be read and written by humans, as well as easily parsed.

Before we get into the details, here is a small GDX object:

```gdx
<OBJ Alice
    <STR firstName : "Alice">
    <INT age : 24>
    <BOOL active : true>
>
```

GDX's standard form encloses every element between an opening `<` and a closing `>`. Structurally, its syntax is similar to XML except that it eliminates the closing tag. XML's tag structure is excellent for its purpose as a markup language. GDX, however, is not a markup language and does not compete with XML in that role.

In the text form a GDX element is what's between the matched angle brackets. Data in GDX can't exist outside of an element.

GDX also makes use of **sigils**; these are punctuation marks that tell the reader, be it a human or a machine parser, how to interpret the text in each element. The common ones are `:`, `&`, `#`, `@`, `%`, and `--`. None of them are concepts to memorize. The meaning of an element always lives in its fields, never in the punctuation.

Nested bracket pairs form **child** elements of the enclosing element. 

## Role

Looking back to the earlier example in this primer, let's now map that statement to GDX.

The following statement is a well formed GDX element:

```
<INT age : 46>
```

The first thing that will likely stand out is the leading **prefix**. In the example above the `INT` prefix is used. The first field in every GDX element is the prefix, and it must be present for the element to be considered valid. Every GDX element contains a prefix which states the element's **Role**.

GDX uses a fixed set of roles which make up the core GDX **vocabulary**. The role terms in the vocabulary enable GDX documents to represent, organize, and contextualize a wide range of information. In many ways the role is the most important conceptual and functional part of GDX, because it allows similarly formed elements to take on different interpretations. It is the Role that determines how an element should be interpreted, and how it relates to its parent and child elements.

Having only seen the above GDX element, it would be reasonable to assume that `INT` is only a type assertion. That is close, but incomplete. A literal property role like `INT` does constrain the kind of value in the Resource field, but role does more than that. It says how the whole element participates in the model.

```
<OBJ acme:bob 
    <INT age : 46>
>
```

In the example above the second line defines the element that carries the age property, but we've now also added a parent element.

The `INT` role is part of a family of literal value properties. The `OBJ` role defines a different kind of element, one that can contain child properties. Because the `INT` element is nested inside the `OBJ`, it is not only saying "the Resource is an integer." It is also saying "this integer property is an aspect of this object." The parent-child relationship that was only implied in Part I is a rule in GDX: containment is part of interpretation.

This is where the representation-vs-model distinction comes into play. GDX's written form is designed to represent the model's form, but the model defines the rules that control how each element is interpreted.

## Resource

In the element we've been looking at, the value after the `:` is `46`. The field that corresponds to this position in the element is called the **Resource**. It's a deliberately broad word: a Resource can hold a plain value literal like the integer `46`, hold a pointer to another element, or remain empty when a role gets its value from structure instead.

In this example the value of the Resource is the integer `46`.

```
<INT age : 46>
```

The Resource is the field that holds the element's value when the role uses a direct value. The role defines how that Resource is interpreted: a `STR` Resource is read as text, an `INT` Resource as an integer, and a `BOOL` Resource as a truth value. As discussed in Part I, this is a conceptual value class, not a storage encoding. `INT` says "integer"; it does not say "signed 64-bit integer."

That remains useful when the value is absent. These are still different fields:

```
<STR firstName>
<INT age>
<BOOL active>
```

Even without Resources, the roles tell the reader what kind of value each field accepts.

Put the pieces together and the line reads naturally: *an integer Property, named `age`, holding the integer value `46`.* That sentence is the element. It says what kind of element it is, what it is called, and what it carries. Nothing depends on a schema loaded elsewhere or on a reader guessing from the spelling of a field name. This is the first and most basic sense in which GDX data describes itself.

One mark before we move on. The `:` between `age` and `46` is punctuation, not a concept. It marks where the name ends and the Resource begins so the reader and parser can find the boundary quickly. A different mark could have served the same job. The meaning lives in the fields. The one rule to remember is the spacing: the `:` always has whitespace in front of it. `<INT age : 46>` is right; `<INT age:46>` is not, because a `:` glued to a word is read as part of that word (that is what lets names like `schema:age` contain a colon).

## Name

Let's look at the line again:

```
<STR firstName : "Bob">
```

In this example the element has three defined fields. The first is always the prefix. The field to the right of the `:` is the Resource. The field after the prefix, `firstName`, is the element's **Name**: the label we give it so a reader can tell *this* property from that one. Every element can carry one. Without it, the element still holds its value perfectly well, but we'd have no way to say which of its parent's properties it is. The Name is that label.

For a Property like this one, the Name is a **local** name. It means something *within the context the property sits in* — it names the role this value plays among its neighbors, the way `firstName` is a meaningful field among the others you'd expect when describing a person. It isn't a globally unique name, and it doesn't need to be: two different people can each have a `firstName`, with no conflict, because each is named only within its own surroundings. (Later, when we meet elements that *do* need to be found and pointed at, we'll see a second field take on that job. For now, local is exactly enough.)

A Name is a label, not an address. It can be changed without changing what the element is, and links do not find an element through its Name (the one exception, a path that walks the tree by Names, comes later). That is why the Name isn't the only kind of identifier an element can carry. GDX has two other identity fields, the **Id** and the **Predicate**, for jobs that haven't come up yet. We'll meet them when something actually calls for them, rather than listing them now.

With the Name in place, we can see the whole element for what it is. This is also a good time to point out that the element has two distinct halves. On the left of the `:` is everything about *what the element is and how we refer to it*: its role (`STR`) and its name (`firstName`). On the right is *what it actually holds*: the Resource, `"Bob"`. GDX draws this line through every element. One side identity, the other side value. It's a small thing to notice now, with only a name and a string to divide, but it's a division the whole model is built on: it's why an element can tell you everything about its place in the data before you ever read its contents.

## Record

In GDX, related properties can be grouped inside a **Record**:

```
<REC person
    <STR firstName : "Bob">
    <INT age : 24>
>
```

A Record is a named thing made of properties, the struct, in a programming-language analogy. `REC` is its prefix; `person` is its Name. Unlike literal properties, a `REC` does not put a literal value in the Resource position. Its value is composed from its children.

And inside it sit the two properties. The child properties should now be easy to read: an `STR` element with a **Name** field `firstName` and a **Resource** holding the value `"Bob"`, followed by an `INT` element with a **Name** field `age` and a **Resource** holding `24`.

The relationship is carried by position: `firstName` is inside `person`, so it is one of `person`'s properties. GDX does not add a separate joining word or back-pointer because containment already supplies the model relationship.

A Record is always a part of something larger; it never stands alone as a document of its own. The next role is the whole it belongs to.

## Object

For things that stand on their own, GDX has the **Object** role:

```
<OBJ &bob
    <STR firstName : "Bob">
    <INT age : 24>
>
```

An Object represents an **entity**, a thing that exists on its own and can be referred to. An Object is **self-contained**, standing on its own as a complete, addressable entity. If a Record is a component, an Object is a whole.

Notice the `&` in front of `bob`. This is the second identity field, the **Id**, and the `&` sigil is what introduces it. Where a Name is a label, an Id is the element's **durable identity**: the token every link and every reference binds to. `bob` here is not a local label among siblings; it is an address other elements can use. Renaming an element (changing its Name) leaves it the same element; changing its Id makes it a different one.

An element can carry both, conventionally written Id first and then Name: the Id says which thing this is, the Name says what to call it. In real-world applications, the Id is more likely to be locally or globally unique, perhaps in the form of a GUID or a fully qualified URI, with a friendlier Name beside it:

`<OBJ &com.acme:people/robert bob>`

Once an Object has an Id, other elements can link to it. An Object with only a Name (`<OBJ bob>`) is still a perfectly good Object, but no link can bind to it through that Name.

## Edge

An **edge** is a GDX element that links one entity to another. To say Bob knows Alice, we need a second Object to point at:

```
<OBJ &alice
    <STR firstName : "Alice">
>
<OBJ &bob
    <STR firstName : "Bob">
    <INT age : 24>
    <@OBJ #knows : @alice>
>
```

The new line inside `bob` is the edge. Read it left to right like any element. The prefix `@OBJ` says this element points at an Object. The `@alice` on the right is the target: not a value Bob holds, but the Object whose Id is `alice`. Between them, `#knows` is the **Predicate** field: it says what kind of relationship this is. The whole line reads: *bob knows alice.*

Two marks earn a word here, since this is the first time you've seen them. The `@` means "another root": a top-level element such as an Object, found by its Id, whether it sits in this document or in another one. In the prefix (`@OBJ`) it marks the element as an edge to such a root; in the Resource (`@alice`) it gives that root's Id. The `#` flags the relationship name. Like the colon, they are recognition marks; the model fields carry the meaning.

An edge always joins whole entities, so its target is always written as a plain `@` followed by an Id. An edge never points into the middle of another element; when you need to reach inside one, that is the job of the next construct.

The important model difference is that containment supplies an unnamed parent-child relationship, while an edge states a named relationship between entities.

## Reference

The edge gave us one way for an element to point at another: assert a *relationship*. But sometimes you don't want to assert a relationship — you just want a slot to *hold a handle to something else* rather than hold a literal value. That's a **reference**, and it's the second of the two kinds of pointing in GDX.

Recall the Resource — the value slot on the right of an element. So far it's held literals: `"Bob"`, `24`. A reference is what you put there when the slot should point at another element instead:

```
<OBJ &order-1001
    <%OBJ customer : @bob>
>
```

The `%OBJ` is a **reference**: the slot `customer` doesn't contain a customer, it holds a handle to one — the Object whose Id is `bob`, living elsewhere. Read left to right: the prefix `%OBJ` says "a handle to an Object goes here," and `@bob` is the address it resolves to. The order's `customer` slot is *filled by* a pointer to Bob.

This looks a lot like the edge from the last step — both end in `@bob`, both point at an entity. But they do genuinely different jobs, and GDX keeps them firmly apart:

- An **edge** (`@`) *asserts a relationship*: bob **knows** alice. It's a statement about how two entities relate, and it carries a label (`#knows`) saying what the relationship is.
- A **reference** (`%`) *fills a slot*: this order's customer **is** bob. It isn't asserting a relationship about Bob — it's saying "the thing that goes in this slot is over there."

The prefix keeps the two from blurring: a prefix starting with `@` means "asserts a relationship," a prefix starting with `%` means "fills a slot with a handle." They're never interchangeable, and once you read them that way, a dense line tells you immediately which kind of pointing you're looking at. (The `@bob` in the Resource means the same thing under both: the root whose Id is `bob`. What differs is what the element does with it.) The question to ask is simply: *am I stating how two things relate (edge), or saying what goes in this slot (reference)?*

One affordance falls out here. For `order-1001` to point at `bob`, Bob has to be findable. An Object's Id already provides that. But sometimes the thing you want to point at is not a whole Object. It may be an element tucked inside one, and its position might shift as the document is edited. A reference can reach inside, which an edge never does, and there are two ways to do it.

The first is by Id. The Id field is not only for Objects: any element can carry one, and an Id on an element inside an Object is a stable handle that references can target even when nothing else about the element's location holds still. Imagine the order has a shipping address with an Id, and a shipment that refers to it:

```
<OBJ &order-1001
    <REC &shippingAddress shippingAddress
        <STR city : "Oakland">
    >
>
<OBJ &shipment-77
    <%REC shipTo : @order-1001&shippingAddress>
>
```

The reference `@order-1001&shippingAddress` reads in two steps: `@order-1001` finds the root whose Id is `order-1001`, and `&shippingAddress` then looks up the element with that Id inside it. (Within a single document, a bare `&shippingAddress` would also work: an `&` on its own looks for the Id anywhere in the current document, and never outside it. The `@order-1001` in front pins the search to one root, and lets it cross to another document.)

The second is a **path reference**, which uses `$` to walk the document shape instead:

```
<OBJ &shipment-78
    <%STR city : @order-1001$/shippingAddress/city>
>
```

The path steps through the Names of the elements on the way: from the root `order-1001`, into `shippingAddress`, to `city`. This is the one place a Name takes part in addressing.

Both forms can identify an interior element, but they make different promises. A path says "go through this structure to get there," which is useful when the structure itself is the address, but it breaks if something on the way is renamed or moved. An Id says "find the element with this stable handle," which survives renaming, reordering or reorganizing the surrounding tree. When both would work, prefer the Id form for references you expect to last.

There is also a third way to fill a slot, used less often: a **VAL**. Where a reference points at something that lives elsewhere, a VAL holds a structured value *right there in the slot* — a composite that belongs to this element and nowhere else, rather than a handle to an independent entity. It's the slot-filler for "the value here is itself a structured thing, owned by this element." We mention it for completeness; most slots hold either a literal or a reference, and VAL is there for the case that needs it.

## Predicate

Part I introduced predicates as shared names for relationships. In GDX, the field that carries that shared relationship name is the **Predicate**, marked in the text form with `#`.

In the edge below, `#knows` is the Predicate field:

```
<@OBJ #knows : @alice>
```

The Name, when present, labels the element locally. The Predicate names the relationship the element is claiming. The Id, when present, says which element this is. Those jobs can overlap in casual data modeling, but GDX keeps them in three separate fields.

Because a predicate is a shared name, it helps to be able to *define* it — to say, in one place, what `knows` is. GDX does this with a **relation declaration**, written `REL`:

```
<REL &gdx:knows
    <@REL #gdx:inverse : @gdx:knownBy>
    <@OBJ #gdx:domain  : @gdx:Person>
    <@OBJ #gdx:range   : @gdx:Person>
>
```

This declares the predicate `knows` and says a few things about it: its inverse is `knownBy`, and it relates a `Person` to a `Person`. The REL's Id, `gdx:knows`, is the predicate's identity: a `#gdx:knows` on any element is matched against it. (The `gdx:` in front of each name says which shared vocabulary the name comes from. The three targets here are themselves roots, a REL `gdx:knownBy` and an Object `gdx:Person`, declared elsewhere with those Ids.) Notice what this *is*, though: it's just GDX. The same elements, edges, and references used for ordinary data are also used to define vocabulary.

## Type and template

Part I introduced schema as the named shape an element can claim and be checked against. In GDX, that claim lives in the **Type** field:

```
<REC address --schema:address
    <STR street : "Main St">
    <STR city : "Oakland">
>
```

The `--schema:address` is the Type. It's the element saying *I claim to be a `schema:address`*. A validator can take that claim and check it: does this record actually carry what a `schema:address` is supposed to have?

The shape itself — the thing `schema:address` names — is called a **Template**, and here is the move you've now seen once already. A Template is written in GDX:

```
<DEFREC &schema:address
    <STR street>
    <STR city>
>
```

That is the definition of `schema:address`: a record that should have a `street` and a `city`. Like a REL, a Template carries an Id, and the `--schema:address` on the instance is matched against that Id. Once the Template is found, the instance's children are lined up with its slots by Name: the instance's `street` fills the Template's `street`. The `DEFREC` prefix marks it as a *definition* of a record rather than an instance of one. The properties have names but no values, because a Template describes the shape a conforming element must fill, not the data that fills it.

## Complete element

You've now met every part of a GDX element, one at a time. Here they all are on a single line:

```
<STR &email-1 #schema:email contact : "bob@example.com" --acme:emailAddress>
```

At the start of this tour, that would have been noise. Now you can read it. It's a string Property (`STR`); it carries an Id (`&email-1`) so other elements can point at it; it aligns to a shared predicate (`#schema:email`) so its meaning travels; its Name, `contact`, is the slot it fills among its neighbours; it holds the value `"bob@example.com"`; and it claims to conform to a Template (`--acme:emailAddress`, a `DEFSTR` that might constrain what an email address looks like). Every field doing its job at once. You can take it left to right and say what each part is — which is the whole skill the tour set out to give you.

A line that dense is not something to avoid. If you're using everything GDX offers — typed values, shared meaning, durable Ids, schema claims — a line like that is simply what it looks like when you need all of it, and that's fine. The point isn't to write less; it's that **you choose what you need, and you don't pay for what you don't.** Look back at where the tour started:

```
<STR firstName : "Bob">
```

That was a complete, legal element too. So was every element in between. Each step added something — a name, a record around it, an edge, a predicate, a type — but nothing along the way was *forced*. An element is valid with just its prefix; a property can be a bare `<STR firstName>` with no value at all. The packed line above and the plain one here are the same model, asked for different amounts. Neither is more "correct" — each carries exactly what its situation calls for.

That is the discipline underneath everything you've read. GDX gives you a lot of room — typed values, named relationships, shared meaning, schemas, a graph — and charges you for none of the room you leave empty. You reach for each piece when the data needs it, the way you reached for a record only when one value wasn't enough, or a predicate only when a relationship needed naming. Simple stays simple; complex is there when you want it; and it's all the same handful of moves, composed.

One last thing, now that you can read all of it. Everything in this tour has been written one way — bracketed, with full three-letter prefixes — but that is only one of GDX's surface forms, and the model underneath doesn't depend on it. Two independent choices vary the surface. The **prefix tokens** can be written in full (`STR`, `OBJ`, `REC`) or **minified** to single letters (`S`, `O`, `R`). And the structure can be drawn with **brackets** — what you've seen — or in **Compact Form**, which drops the brackets and lets indentation carry containment instead, closing each element when the indentation steps back out. Here is the same `bob`, written four ways:

```
Standard form, full prefixes        Standard form, minified
<OBJ &bob                           <O &bob
    <STR firstName : "Bob">             <S firstName : "Bob">
    <INT age : 24>                      <I age : 24>
>                                   >

Compact form, full prefixes         Compact form, minified
OBJ &bob                            O &bob
    STR firstName : "Bob"               S firstName : "Bob"
    INT age : 24                        I age : 24
```

Four renderings, one element. The prefixes are longer or shorter; the brackets are present or replaced by indentation — but the model is identical in all four: the same Object with the Id `bob`, holding the same string and the same integer. This is what it means for GDX to be a model with several realizations rather than a syntax. The text is how the model is written down, and it can be written down more than one way. Reading any of them is the same skill — which is now yours.

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

**References as data.** Anything worth pointing at can be pointed at — not just top-level objects, but elements tucked inside them, reached by a durable Id that survives the document being renamed or rearranged. And the pointers themselves are ordinary GDX: an edge linking two entities, a reference filling a slot with a handle, are elements like any other, with their own identity and attributes. Configurations point at objects, vocabularies point at schemas, records point at each other — one uniform model, nothing special-cased about the act of pointing, and references you can attach data to like anything else.

Underneath all of these is the property the tour ended on: you reach for each capability only when the data needs it. An element is valid with nothing but its prefix; type, meaning, schema, links, and Ids are there when a situation calls for them and silent when it doesn't. None of the powers above is mandatory — which is what lets the same model carry a one-line typed config value and a fully-aligned, schema-validated, graph-connected entity, and ask no ceremony of the simple case.

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
<OBJ &bob
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

Now read for the relationship. In GDX, `<@OBJ #schema:knows : @alice>` says three things in the open: this is a link to another entity (`@`), the relationship is the shared predicate `schema:knows` (`#`), and the target is the entity whose Id is `alice`. In the other three, `knows: "alice"` is just a string whose value happens to be the word "alice." Nothing marks it as a *reference* rather than a literal, nothing says it points at a `bob`-level entity identified as `alice` rather than holding the text, and nothing carries the predicate that would let another system recognize this as the same `knows` it uses. The relationship is representable only as a convention layered on top — a rule, kept outside the document, that a field named `knows` should be dereferenced as a pointer and aligned to some agreed meaning.

That is the structural difference the map is meant to show. It is not that the other formats are weaker at their work; it is that the type-commitment and the relationship are, in those formats, things the document *cannot hold on its own* and must defer to an external layer — and in GDX they are stated on the element. Whether that property is worth reaching for is a question only your own data answers: where you need more than one of the things GDX carries, held in one readable place, it earns its keep; where a single-purpose format already does the one thing you need, that format is the honest choice.

---

*To build with GDX from worked, runnable patterns, see the GDX Cookbook. To settle any exact question of grammar, composition, or conformance, the GDX Specification is the authority.*
