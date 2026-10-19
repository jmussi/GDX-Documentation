# GDX — A Primer

- **Status:** Draft v0.5.0 (companion to GDX Core Specification v0.30.0, pre-1.0)
- **Date:** 2026-10-19
- **Authors:** Julian Mussi

> **This document is non-normative.** I

---

#### Abstract

The primer introduces GDX (Graph Data eXchange) by example and explains the thinking behind its design. It introduces the core terms and concepts necessary to understand and use GDX. A reader new to GDX should start here and then move to the **GDX Specification**, which is the normative guide.



#### Introduction  

Before introducing GDX it will be helpful to understand the motivations behind GDX's development. GDX is an attempt to address a set of challenges that are commonly encountered when information is persisted, exchanged, organized and contextualized. This first section attempts to surface and name these concepts so that GDX can be more easily understood, so that the reader can evaluate how well GDX achieves its goal, and so they can understand its position relative to other options.

###### Information

Rather than supply terms and their definitions, this section attempts to explain the foundational concepts that will be used throughout this document and the core spec. The definitions are not designed to be canonical. In many cases the terms have meaning which can vary considerably depending on context, so the goal is to introduce them in the context in which they are likely to be used in this document.

A useful way to start is with the concept of **data**. For the purposes of this document, data can be understood as raw, unorganized values; facts, measurements, and observations, considered apart from any meaning we might assign them. The set of numbers 1,**46**,42,68,11 absent any other framing is data. Data are the **values** that are often recorded in variables within applications or written in documents.

Data becomes **information** when we give it order, context, and structure. When data carries meaning it becomes information. Information may carry shallow or deep meaning. Meaning is often **implicitly** conveyed through labeling or organization of information. In other cases meaning is made explicit through **ontology** or shared **semantic** definition.

For an exploration of how meaning is conveyed through information we'll use the value `46` from the list above. This value can be transformed from a data point into information by changing its structure and **labeling** the data so it reads as follows: `Age : 46`. It's intuitive that the age label is adding something to the data. What may not be clear is what its exact purpose is; is it transforming it, contextualizing it, naming it? What should be made explicit, however, is that `age` and `46` are associated, that the two values have a dependency. Removing one alters the interpretation of the other.



###### Statements

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

While for does not inherently carry meaning. The relative position of information often does. Consider the following forms; `46 = age` and `Forty six is his age`.  In the first case the transposition creates a statement expressing an entirely different different statement, while in the second we're running into grammatical convention with more flexibility.

###### Value types

Using our current example statement, if we were to ask a programmer to convert this to a code snippet they might return something like `int age = 46;`. While most employable developers would probably interpret age as holding a digit, an English major turned application architect might provide `age = 'forty six';` and would have a defensible argument. Indeed, both statements encode data that, when interpreted by a human, would produce the same meaning. However, to a machine these values have no inherent similarity. So as we consider the reader, or interpreter, of information, the way in which data is represented becomes increasingly important.

Defining a **data type** can be tricky. From a mathematical perspective the focus is on what values "are" and what they can hold. The computer science view is weighted towards how they are represented. The perspectives have overlap, as well as some real distinctions. For example we might say that the age needs to be a **numeric** type. We might add the constraint that it be an **integer** (whole numbers), or perhaps we want precision and will allow **real numbers** (fractional values). At a conceptual level value types are likely to fall into categories of numeric, textual, and **Boolean**. Others categories like **Bytes** **Null** and **Reference** types are less representative of universal concepts and more oriented around functional concerns. The guiding principle is that each of these categories expresses a range of attributes or constraints about the type of information they contain.

All of the types identified above can be communicated conceptually. We can understand the difference between the concept of the whole numbers between 0 and 255 vs. the real numbers in that range. However, when that value moves from the conceptual form to a concrete representation, a linked but distinct concern emerges. Using the example value `46`, when this value is stored in memory its representation is now not just conceptual but concrete.

The number of bits required to **encode** a value can vary; we can use 8 bits or 32 bits to encode the integer value 255. Both hold the same conceptual value, but the way in which that data is encoded varies. This conceptual and representational split is important. It allows the conceptual constraint to focus on the class of information being stored, and the encoding to decide how that information is stored on a given medium.



###### Identifiers 

Looking at the statement `age = 46` and asked to infer some meaning the term "age" is going to provide more context than 46.  We reading "age" followed by the number 46 readers will likely infer the same meaning from the statement. Going back to our first example we started with 46 and added "age" to give it meaning, but agreed that we could have "46" as an independent datapoint. In this form "age" functions to **label** the data. However converting this to code the variable holding the data point "46" needs an **identifier**. 

It's possible to look at the labeled string or the variable identifier and say that it is providing the meaning for this statement, and in this example there's a strong case to support that. A term like age has clear **semantics**; its meaning is shared and has less room for interpretation. But what happens if the developer changes the name of the variable to just "a". Have we changed the meaning of the information, or just what we use to refer to it? What if the identifier was `name`? How certain are we now about the meanining; it could represent first name, last name, or both separated by a comma. What meaning the label or identifier conveys can often being conflated or misinterpreted.

###### Nested relationships

Let's pause here and reflect on what else the "age" label might be communicating. In this example specifically the idea of an "age", doesn't make doesn't make much sense without a subject, something that "has" and age.  Without stating it when we saw age we probably Instinctively understood there was a relationship between some subject and the aspect of them we call an "age". If we refined the statement to include the hinted-at owner of the age, it might say `Bob's age is 46`. By doing so we've provided a **subject**. The subject is the owner of the age. And just like "age" and `46` are linked, so are "age" and "Bob".

###### Properties

This aspect of Bob is a **property** of his. In this case his "age". This is interesting because it can now be said that there's an aspect of Bob, his age, that exists even if its value is undefined. Without knowing someone's age we can still be confident they have one. This is an obvious deduction for a subject that we're familiar with, like people, but not for something that we're not familiar with or that is abstract.

Asked to model the statement `Bob's age is 46`, a programmer is likely thinking "this looks like a 'person' class with an age property, of which Bob is an instance" — a snippet of code something like `class person { uint8 age; }`. If we don't know Bob's age we still know he has a property. When his age is known it can be assigned, `Bob.age = 46;`, updated or removed, knowing that the system will constrain the value to an integer type that can be no more than 255.

**Meaning**

By this point it should be clear that the statement "Bob's age is 46" contains layers of meaning. We have some numeric data with the value of 46. We've labeled that data with a term, "age". With the inclusion of Bob, he emerges as our subject,  who our property is addressing. 

At this point it should be clear that how we interpret age is critical to understanding how all the terms in this statement should be understood. If age is just a label, then"age" can be substituted with "a" then 46  should be interpreted the same. However if "age" explicitly means something then it's more than a label. 

In this example "age" could be viewed as taking on the shape of a **predicate**,  a thing that holds true *of* something. Predicates are more than just a string, they carry meaning that is shared within a given context. In this way If "name" means "someone's given name" that meaning is consistent.

Borrowing from logic, we could write this as `age(bob, 46)`: a predicate named `age` applied to two arguments, `bob` and `46`. Read it as the claim "the age of Bob is 46".The same relationship is the basic unit of **RDF** (the Resource Description Framework).  In this form our example looks like this`ex:bob schema:age 46.` . In this form `ex:bob` is the subject, `schema:age` is the predicate, `46` is the object, e.g 

Looking back now we can see that the age field, without being explicit, could have been doing double duty as a predicate and a label. This identity vs meaning ambiguity is so common as to be often overlooked.



###### Links

 When we wrote `age = 46`, `46` *is* the thing the statement is about . We call resources of this type  **literals**. The object is the text, or number or bytes that are written on the page or stored in memory.

But consider a statement like `Bob knows Alice`. 

If "Bob" the subject is not just a string, but represents an entity of some sort, then it follow that "Alice" is too. Again, as the reader of this statement we can infer that Alice is likely a person, but to a machine parsing the string, "Alice" has no more inherent meaning then "cat" or "124H34a".

If Bob and Alice are entries on a wikipedia page, "bob" might appear as a blue text to the right of the "Spouse : *Bob*" field. In this case our expectation would be that clicking this text would open the "Bob" page. 

"Bob" is not just text, it carries a **reference** that has to be *resolved* before it means anything. This is the difference between a **value type** and a **reference type**: a value *is* the data, while a reference *points at* data that lives elsewhere. `"Alice"` the literal string and `Alice` are not the same kind of thing at all, one is held, the other is pointed at.

When information expands beyond the limits of literal and enables creation of links between different entities we have created the components necessary to construct a **graph**. At this point Bob & Alice may represent **node** which contains features, the properties and other information that refine how we understand Bob. When one node references another node we have created and **edge**, a directed link that says *this* relates to *that* in some named way. `Bob knows Alice` is an edge from the Bob node to the Alice node, labeled `knows`. 



###### Summary

Having explored the example statement from different perspectives, the above shows the hidden depth that often lurks just below the surface. In many cases the aspects of the information that supply the meaning are understood implicitly, but when exchanged between individuals, systems, or organizations with divergent perspectives, they can quickly produce ambiguous interpretations. Rather than looking at information as representative of its form, its more honest expression is one in which orthogonal aspects of identity, meaning, syntax, type, linking, and encoding are often conflated and obfuscated. GDX strives to make explicit the facets of meaning that information implicitly carries, and by doing so, to improve how information carries its meaning.

To be clear, every aspect mentioned above can be clearly expressed using a range of existing technologies. JSON and YAML allow objects and properties to be represented in a human-editable format. XML supports typing through XSD, a robust schema language. Type and data encoding is built into every coding language, and many of them support reflection and attribution. RDF triples in a range of formats are used to build ontologies and add a semantic layer to data. Graph databases support explicit creation of nodes and edges. Not only is there a solution for every implied challenge, but there are myriad options for doing so. However, very few approaches address all of these layered concerns within a single framework. Far fewer offer a solution that is both compact and relatively simple.

The crux of this work has been trying to reconcile often antithetical objectives like completeness and simplicity. To this end, much of what defines GDX is not what it chose to include, but a ruthless refinement to leave out as much as possible while still meeting the core objectives.



### The GDX Model

###### Definition

**GDX (Graph Data eXchange) is a model for structuring data so that layered identity, meaning, and form can be explicitly communicated**. 

GDX is a conceptual approach to data modeling. Formally, GDX is the model, separate from any concrete representation. Just as `2*2`, `2 X 2`, and `multiply(2,2)` are different forms conveying the same conceptual idea, different GDX representations should be viewed as different ways of specifying the state of a GDX model.

The canonical serialization format defined in the spec and used in this primer is designed to be human- and machine-editable. GDX's serialization has several concrete forms: two text serializations (one verbose and explicit, one compact) and a binary encoding. The primer uses the canonical verbose form for easier readability.

So the central idea behind GDX is that **with one model, information can carry meaning in a set of explicit fields**. To accomplish this, GDX uses a small, fixed set of building blocks that you combine, rather than a format you extend with new syntax. That fixedness is what lets any GDX tool read any GDX document without being told the rules in advance.

Another important feature of GDX is that it does not require that every one of these fields be known to be valid. GDX holding a key-value pair is still considered a fully formed GDX model. This progressive expansion of data specificity makes GDX adaptable and scalable.



###### GDX element

The model GDX uses to organize and contextualize data is structurally simple.  The building block of the GDX system is the **element**. 

Elements package data so that its associations can express layered meaning. Every element has 6 data slots, or **fields**. Every bit of data lives in a field, and the element groups the fields.

The fields that make up an element are **Prefix** (which declares the element's role), **Id**, **Predicate**, **Name**, **Resource** and **Type**. (An element can also carry a small block of metadata about how it is stored or sent, but that sits outside the model and isn't one of the six.)

The majority of the primer will be spent explaining the function of each of these fields and how they work together.



###### Representing GDX  

Framing GDX as a model is correct, but without a way to represent the model it's mostly just a philosophical exercise, and GDX was designed to be used. Its primary format is text that can be read and written by humans, as well as easily parsed.

Before we get into the details, let's look at a snippet of information represented in GDX as well as JSON, YAML and XML formats.

**GDX**

```GDX
<OBJ Alice
    <STR firstName : "Alice">
    <INT age : 24>
    <BOOL active : true>
>
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

**JSON:**

```json
{ "customer": 
 			{ "firstName": "Alice", 
             "age": 24, 
             "active": true } 
}
```

What should be apparent is that GDX looks very similar to the three other formats, and for good reason. GDX addresses the same family of problems as JSON, YAML, and XML, all of which are mature and well understood. GDX isn't trying to reinvent the wheel when it doesn't need to. For this reason, GDX should be reasonably easy to interpret even if you haven't seen it before.

GDX's standard form encloses everything between an opening `<` and a closing `>`. Structurally, its syntax is similar to XML except that it eliminates the closing tag. XML's tag structure is excellent for its purpose of being a markup language. GDX, however, is not a markup language and doesn't compete with XML in this role.

 In the text form a GDX element is what's between the matched angle brackets. Data in GDX can't exist outside of an element.

GDX also makes use of **sigils**; these are punctuation marks that tell the reader, be it a human or a machine parser, how to interpret the text in each element. The common ones are `:`, `&`, `#`, `@`, `%`, and `--`. None of them are concepts to memorize. The meaning of an element always lives in its fields, never in the punctuation.

Nested bracket pairs form **child** elements of the enclosing element. 

###### Roles

Looking back to the earlier example in this primer, let's now map that statement to GDX.

The following statement is a well formed GDX element:

```
<INT Age : 46>
```

The first thing that will likely stand out is the leading **PREFIX**. In the example above the `INT` prefix is used. The first field in every GDX element is the prefix, and it must be present for the element to be considered valid. Every GDX element contains a prefix which explicitly asserts the element's **ROLE**.

GDX uses a fixed set of roles which make up the core GDX **vocabulary**. The role terms in the vocabulary enable GDX documents to represent, organize and contextualize a wide range of information. In many ways the role is the most important conceptual and functional part of GDX, because it allows similarly formed elements to take on different interpretations. It is the ROLE that determines how an element should be interpreted, and how it relates to its parent and child elements.

Having only seen the above GDX element, it would be reasonable to assume that this is a type assertion... this would almost be right. An expanded example, however, reveals that the role is expressing more than a value class.

```
<OBJ acme:bob 
	<INT Age : 46>
>
```

In the example above the second line defines the element that carries the Age property, but we've now also added a parent element.

 The `INT` role is part of a family of literal value properties. The `OBJ` role defines a different type of element, one that can contain child properties. The parent-child relationship is explicitly saying that the parent of this element (the OBJ element) has a property whose resource type may be fulfilled by an integer value. The role also asserts that the element is an aspect of the parent, and that this relationship is fundamental to its interpretation. This relationship, which appeared in the introduction as either a suggested or implicit relationship between parts of a statement, is a RULE in GDX. This is where the representation-vs-model distinction comes into play. *GDX's written form is designed to represent the model's form, but the GDX model defines and communicates explicit rules that control the interpretation of each GDX element.*

###### Resources

In the element we've been looking at, the string after the `:` is `46`. The field that corresponds to this position in the element is called the **Resource**. It's a deliberately broad word: a Resource can hold a plain value literal like the integer `46`, hold a pointer to another element, or in some cases be empty.

In this example the value of the resource is the integer `46`.

```
<INT Age : 46>
```

The resource is the field that holds a data value. Another one of GDX's strengths is that, without having to look at a schema, the reader or parser always knows how to interpret the resource field. The role defines how the data in the resource field is interpreted. A STR role tells the parser to interpret the resource as a string. In some cases, such as the `OBJ` role, the resource field will always be empty. While this element doesn't hold a value in the field, the OBJ role interprets the children as the resource, a compound data type where each part is defined by a child element.

 The name here is important. and this is another one of those nuanced aspects of the GDX model that is critical to it's use. The property type roles all have resources that may contain a literal value. In this example it's the string "Bob". This is conceptually straight forward for the primitives `INT`, `STR`, `FLOAT`. 

`BOOL` is where the nuance starts to become important. While the GDX resource for a `BOOL` role may contain the strings "true", "TrUe", or "T" internally,  all of these resolve to the same conceptual interpretations of **TRUE**. So 't' and 'TRUE' are conceptually equivalent , even though their string representations are different.

//Start here

 So formally the second assertion made by this role is TypeOf(this, valuetype). Going back to the start of the primer, I stated that the role was not a type. And that's because we need to be clear about what EXACTLY role is asserting. In this case  the STRING type has to be understood as a "text literal". `INT`  is a whole number that can be positive, negative, or zero, without any fractional or decimal parts. What the role does not say is how the value is encoded. It an 64 bit signed int, or a unsigned 8 bit representation. The anser is it's doesnt say. That's because at this level the role is restricting the value to the abstract conceptual understanding. Boolean, strings, int and real (floating point) values all occupy a clear conceptual space. BYTES is the one role that gets closer to implementation rather than concept.  GDX is intentionally seperated from the encoding because at this level the role is on the conceptual level. We're explicitly saying the restriction is about the conceptual INTEGER. 8 or 800000. If it's an integer it fits the roles assertion. Later on we'll look at the practical aspects of how we actually persist the value, and good news, GDX has thought of that too. 

So the element's prefix`STR` has now told you two things: that this element is a Property, and that the value it holds is a string.

Before wrapping up, I want to make sure that the reason behind the role is clear. Let's take a step back and look at our example element as if it was a key value pair `firstName : "Bob"`. Could I infer meaning and the value type from this? Sure. But let's let make it a bit harder and remove the value. So given just the key `firstName`, could I infer the expected type? Again probably...as long as I'm a human, who also took the time to read and think about the field AND who speaks English. That's asking a lot more. Let's say the author is Welsh, do you now know that '**enw** ' means string?  And what about a field named 'color'? Is this a string 'blue'. Or the hex '#ff00ff', maybe a compound data type RGB(2550,0,255). All are equally valid. All might be wrong. 

But given a GDX document with a null value, one that hasn't been filled in YET, the receiver know what type of data the field accepts. Explicit interpretation is a core tenant. With GDX the writer, reader and parser, all know from just reading the prefix that any value in the resource field is interpreted in one way. 

Put the three together and read the line back: *a string Property, identified as `firstName`, holding a string`"Bob"`.* That sentence is the whole element, and every part of it is stated explicitly; what kind of element it is, what it's called, and what it carries. Nothing is inferred from a schema you loaded elsewhere or guessed from the shape of the value. This is the first and most basic sense in which GDX data **describes itself**: change the the prefix and the element changes what it says `<INT age : 24>`, `<BOOL active : true>`, but the shape never changes. A prefix saying what the element is, a name for it, a Resource holding what it carries. A program reading the line knows all three without help. And now so do you.

One mark before we move on. The `:` between `firstName` and `"Bob"` is punctuation, not a concept. It marks where the element's identity ends (here, its name `firstName`) and the Resource begins, so you and the parser can find the boundary at a glance. It is always written with a space before it: `firstName : "Bob"`, not `firstName:"Bob"`, because a `:` glued to a word is read as part of that word (names like `schema:givenName` carry one inside them). A different mark would do the job just as well; nothing about the element would change.



//In addition to establishing a parent child relationship, it's also stating that this element has a **RESOURCE** field. 

## Naming it so we can refer to it

Let's look at the line again:

```
<STR firstName : "Bob">
```

 In this example the element has 3 defined fields. The first is always the PREFIX. The field to the right of the `:` is the resource. The field after the prefix, `firstName`, is the element's **Name** field. It's how we talk about this particular element among its neighbours. The `firstName` is the element's **Name** — the label we give it so we can tell it apart. Every element can carry one. Without it, the element still holds its value perfectly well, but we'd have no label for it: no way to say *this* property as opposed to that one. The Name is that label.

For a Property like this one, the Name is a **local** name. It means something *within the context the property sits in* — it names the role this value plays among its neighbors, the way `firstName` is a meaningful field among the others you'd expect when describing a person. It isn't a globally unique name, and it doesn't need to be: two different people can each have a `firstName`, with no conflict, because each is named only within its own surroundings. (Later, when we meet elements that *do* need to be findable and linked to, we'll meet a second identity field that does that job. For now, local is exactly enough.)

The Name is the only kind of identity we need yet, but it isn't the only kind an element can carry. GDX has two other identity fields for jobs that haven't come up. We'll meet them when something actually calls for them, rather than listing them now.

With the Name in place, we can see the whole element for what it is!  this is alos a good time to point out that the element it has two distinct halves. On the left of the `:` is everything about *what the element is and how we refer to it*: its role (`STR`) and its name (`firstName`). On the right is *what it actually holds*: the Resource, `"Bob"`. GDX draws this line through every element. One side identity, the other side value. It's a small thing to notice now, with only a name and a string to divide, but it's a division the whole model is built on: it's why an element can tell you everything about its place in the data before you ever read its contents.

## Grouping values: the Record

A single property is rarely the whole story. Bob has a first name, but also an age, and probably more. In GDX you collect related properties by placing them inside a **Record**:

```
<REC person
    <STR firstName : "Bob">
    <INT age : 24>
>
```

A Record is a named thing made of properties, the struct, in a programming-language analogy. `REC` is its prefix; `person` is its Name, labelling this record the same local way a property's Name labels a property. Unlike properties, `REC` has no resource field. Instead it gets its value from it's children. And like a property, a Record is always part of something larger: it is never a document on its own. We'll give it something to belong to in the next step.

And inside it sit the two properties. The child properties should now be easy to read. An `STR` element with a **Name** field `firstName` with a **resource** holding the value "Bob". 

Notice what *isn't* there. Nothing on the `firstName` line says "I belong to `person`." There's no joining word, no pointer back to the parent, no mark connecting child to container. The relationship is carried entirely by **position**: `firstName` is inside `person`, so `firstName` is one of `person`'s properties. GDX leaves parent-child containment implicit because it doesn't need stating — the nesting already says it, and the child's role already says *what kind* of thing it contributes. Spelling it out on every line would only repeat what the structure makes plain.

And here is the first quiet thing worth noticing about that arrangement. Putting `firstName` inside `person` isn't only filing it there — it's a small claim: *person has a firstName, and it's "Bob".* Each property nested in the record is the record saying it *has* that attribute, with that value. You don't write the "has" anywhere; it's what the containment means. For now that's just a nice way to read the structure. Later it turns out to matter more than that — but we'll get there one piece at a time.

## A thing that stands on its own: the Object

The `person` record from the last step was a *compound property* but as was stated earlier properties must belong to something. For things that are distinct from other things, something that has a unique identity like a person, an order, a server, or a document GDX has the **Object** role.

```
<OBJ &bob
    <STR firstName : "Bob">
    <INT age : 24>
>
```

An Object represents an **entity**, a thing that exists on its own and can be referred to by its Id. An Object is **self-contained** standing on its own as a complete, addressable entity.  If a Record is a *component*, an Object is a *whole*.

This is where the second identity field comes in. The `&` in `&bob` marks the element's **Id**: its durable identity, the token that every link and reference to it binds to. A Name is a label: it describes the element, it can be changed freely, and no link or reference binds to it. (A path through the tree can step through Names, but a path names a place, not an identity.) An Id is an address: it stays fixed while labels, values and position change, and changing it makes the element a different element as far as anything pointing at it is concerned. An element can carry both, Id first and Name after it:

```
<OBJ &bob Bob
    <STR firstName : "Bob">
>
```

Here `bob` is what other elements point at, and `Bob` is the label a person reads; renaming the label to `Robert` breaks nothing.

Being a whole is exactly what makes an Object referable. Its Id `bob` isn't a local label among siblings; it's an address. A Record's Name only labels it inside its immediate parent. In real world applications "bob" is more likely to need an Id that is unique beyond this one document. This may be in the form of a GUID or a namespaced identifier:

`<OBJ &com.acme:people/robert>`

That difference is the foundation of what comes next: once an entity has an address others can use, those others can link to it — and that is where GDX stops being a tree and starts being a graph.

## Linking entities: edges and the graph

Everything so far has been one entity described in isolation — Bob, with his properties. But data is rarely a set of isolated things; the interesting part is usually how things *relate*. Bob knows someone. To say so, we need a second entity to point at, so let Alice into the document:

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

The new line inside `bob` is an **edge** — a link from one entity to another. Read it left to right like any element. The prefix `@OBJ` says this element points at an Object. The `@alice` on the right is the target: not a value Bob holds, but the *address* of another entity — the Object whose Id is `alice`, whom we gave an Id precisely so she could be pointed at. A Name would not do: `<OBJ alice>` carries a label, not an address, and `@alice` would find nothing. And between them, `#knows` is the **label** on the link: it says *what kind* of relationship this is. The whole line reads: *bob knows alice.*

Two marks earn a word here, since this is the first time you've seen them. The `@` is how GDX flags "another entity" — for you, scanning, and for the parser. In the prefix (`@OBJ`) it marks the line as an edge; in front of `alice` it says the target is a whole, top-level entity found by its Id, in this document or another one loaded beside it. The `#` flags the label that names the relationship. Like the colon, they're recognition marks, not concepts; the meaning is in what they point at. The label `knows` has more behind it than first appears — it's the kind of thing two different systems can agree on — but that's a later step; for now, read it as the name of the relationship.

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

The `%OBJ` is a **reference**: the slot `customer` doesn't contain a customer, it holds a handle to one — the Object whose Id is `bob`, living elsewhere. Read left to right: the prefix `%OBJ` says "a handle to an Object goes here," and `@bob` is the address it resolves to. The order's `customer` slot is *filled by* a pointer to Bob.

This looks a lot like the edge from the last step — both end in `@bob`, both point at an entity. But they do genuinely different jobs, and GDX keeps them firmly apart:

- An **edge** (`@`) *asserts a relationship*: bob **knows** alice. It's a statement about how two entities relate, and it carries a label (`#knows`) saying what the relationship is.
- A **reference** (`%`) *fills a slot*: this order's customer **is** bob. It isn't asserting a relationship about Bob — it's saying "the thing that goes in this slot is over there."

The prefix sigils keep the two from blurring: a prefix starting with `@` (`@OBJ`) always means "asserts a relationship," a prefix starting with `%` (`%OBJ`) always means "fills a slot with a handle." The `@bob` on the right reads the same in both: the entity whose Id is `bob`. They're never interchangeable, and once you read them that way, a dense line tells you immediately which kind of pointing you're looking at. The question to ask is simply: *am I stating how two things relate (edge), or saying what goes in this slot (reference)?*

One affordance falls out here. For `order-1001` to point at `bob`, Bob has to be findable — he needs a stable address. An Object's Id already provides that. But sometimes the thing you want to point at isn't a whole Object — it's an element tucked inside one, and its position might shift as the document is edited. Any element can carry an Id, not only an Object, and a reference can reach it: `&home` finds the element with Id `home` anywhere in this document, and `@bob&home` finds it inside the entity `bob`, wherever that lives. (A reference can also follow a path through the tree, `$…`, which matches Names step by step; it works, but it breaks when the tree is rearranged, so prefer an Id when there is one.) This is the other half of the edge/reference split: an edge only ever joins whole, top-level entities, written as a plain `@x`, while reaching *inside* an entity is a reference's job. You don't need an Id on an interior element for most pointing, so we'll leave it there: it's identity's way of saying "here's a fixed handle on me," available the moment a reference needs one.

There is also a third way to fill a slot, used less often: a **VAL**. Where a reference points at something that lives elsewhere, a VAL holds a structured value *right there in the slot* — a composite that belongs to this element and nowhere else, rather than a handle to an independent entity. It's the slot-filler for "the value here is itself a structured thing, owned by this element." We mention it for completeness; most slots hold either a literal or a reference, and VAL is there for the case that needs it.

## Naming the relationship: predicates

Two threads have been left deliberately loose, and this step ties them together.

The first was when we nested a property inside a record: *bob has a firstName.* The relationship — "has" — was never written down; it lived in the position. The second was the edge: `#knows`, which we read as "the label on the link," with a quiet note that there was more behind it. Those two loose ends are the same thing, and it has a name: a **predicate**.

A predicate is the *name of a relationship.* When `bob` knows `alice`, the relationship is `knows`. When a record has a `firstName`, there is a relationship there too — it just went unnamed because it didn't need naming yet. The `#knows` you saw on the edge is a predicate written out in the open; the silent "has" of a nested property is a relationship you simply chose not to name. Predicates are how GDX lets you name a relationship when naming it earns its keep.

And the reason to name one is the payoff GDX is built for. A bare label like `knows` is just a word — but a *predicate* is a shared, agreed-upon name. Two systems that store their data differently, that call their fields different things, can still both point at the predicate `knows` and mean the same relationship by it. That is what the introduction promised when it said a value can state what it *means* in terms other systems recognize: the meaning rides on the predicate, not on whatever the field happens to be called locally. A field named `pals` in one system and `acquaintances` in another can each carry the predicate `knows` — and now they line up, with no translation table, because the shared meaning travels on the data itself.

Because a predicate is a shared name, it helps to be able to *define* it — to say, in one place, what `knows` is. GDX does this with a **relation declaration**, written `REL`:

```
<REL &gdx:knows
    <@REL #gdx:inverse : @gdx:knownBy>
    <@OBJ #gdx:domain  : @gdx:Person>
    <@OBJ #gdx:range   : @gdx:Person>
>
```

This declares the predicate `knows` and says a few things about it: its inverse is `knownBy`, and it relates a `Person` to a `Person`. (The `gdx:` in front of each identifier just says which shared vocabulary it comes from — more on vocabularies in a moment.) The REL's Id, `&gdx:knows`, is the predicate's identity: an edge's `#gdx:knows` is matched against it, never against a Name, so the declaration needs an Id just as Alice did. Notice what this *is*, though: it's just GDX. A predicate is declared with the same elements, edges, and references you already know how to read; `gdx:Person` is simply an Object with a conceptual Id, pointed at like any other. GDX has no separate language for "defining vocabulary" — the things that give your data shared meaning, the predicates and the concepts they relate, are themselves ordinary GDX. A vocabulary is just a GDX document whose contents happen to be definitions.

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

The `--schema:address` is the Type. It's the element saying *I claim to be a `schema:address`* — that this record matches the shape that identifier stands for. A validator can take that claim and check it: does this record actually carry what a `schema:address` is supposed to have? The claim travels on the data, like everything else; the element announces not only what it is and what it holds, but what shape it asserts it conforms to.

The shape itself — the thing `schema:address` identifies — is called a **Template**, and here is the move you've now seen once already. A Template is written in GDX:

```
<DEFREC &schema:address
    <STR street>
    <STR city>
>
```

That is the definition of `schema:address`: a record that should have a `street` and a `city`. Like a REL, a Template carries its identity as an Id (`&schema:address`), because that is what a `--schema:address` claim is matched against; the instance's children line up with the Template's slots by their Names (`street`, `city`). It's an ordinary GDX element — the `DEFREC` prefix marks it as a *definition* of a record rather than an instance of one, but its body is just properties, read exactly as you've read properties all along. The properties have names but no values, because a Template describes the *shape* a conforming element must fill, not the data that fills it.

This is the same lesson predicates taught, now for schemas: GDX has no separate language for defining structure. Just as a vocabulary was a GDX document full of predicate definitions, a schema is a GDX document full of Templates — written in the same form, read by the same parser, as the data they describe. The schema *is* data. There is nothing new to learn to read one; you already can.

## Everything together — and how little you need

You've now met every part of a GDX element, one at a time. Here they all are on a single line:

```
<%OBJ &primary #schema:customer customer : @bob --schema:customerRef>
```

At the start of this tour, that would have been noise. Now you can read it. It's a **reference** (`%OBJ`, a handle to an Object) filling a slot named `customer`; it carries its own Id (`&primary`), a durable identity other elements can point at; it aligns to a shared predicate (`#schema:customer`) so its meaning travels; the handle resolves to the entity whose Id is `bob` (`@bob`); and it claims to conform to a Template (`--schema:customerRef`). Every field doing its job at once. You can take it left to right and say what each part is — which is the whole skill the tour set out to give you.

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

**First-class predicates.** If you've reached for RDF or OWL to model a domain, you know the power is real and so is the cost: triple stores, blank nodes, several serializations, a query language and a reasoner before you can do anything. GDX gives you the load-bearing parts — named relationships you can define and share, concepts you can relate, axioms like inverse and domain and range — using nothing but the elements you already write. A predicate is declared with a `REL`, a concept is just an Object with a conceptual Id, and a whole vocabulary is a GDX document full of definitions, read by the same parser as your data. You get shared, machine-checkable meaning without a second modeling stack to stand up and operate.

**Schema as data.** In most stacks the schema is a separate language — XSD is not XML, JSON Schema is a convention bolted onto JSON — so you maintain two artifacts in two grammars and keep them in sync by hand. In GDX a schema is a GDX document: a Template is an ordinary element, read by the same parser, and it can travel in the same file as the data it describes. There's nothing new to learn to read or write one, and no impedance between the shape and the thing it shapes. For config especially, this is the missing middle: readable like YAML, but with a real schema in the same readable form, not a typed language you switched to just to get safety.

**Layered validation.** Most formats give you one flat verdict — valid or not — and leave you to work out why. GDX's validation comes in distinct layers with distinct owners, so a failure tells you which layer it lives in. GDX itself checks structure: is this actually an integer, do these fields match the Template's shape. Finer rules — numeric bounds, string patterns, semantic constraints — are carried by constraint vocabularies that ride on top, each interpreted by the consumer that understands it. The same element can satisfy one vocabulary's rules and not another's, and you can host several on the same data at once. You get a precise, layered answer instead of a single opaque "invalid," and you always know whether a failure is GDX's to report or a vocabulary's.

**Materialized instances.** When a schema fills in values an instance leaves out, the instance is only whole *with the schema in hand* — pull it into storage or hit it with a query on its own, and the schema-supplied parts are missing. GDX can write those values into the element before it's stored, so the stored thing is complete by itself: searchable, queryable, and parseable with no schema loaded. You author against the schema; your data doesn't depend on it afterward.

**References as data.** Anything worth pointing at can be pointed at — not just top-level objects with global Ids, but elements tucked inside them, reached by an Id that survives the document being rearranged and the element being relabelled. And the pointers themselves are ordinary GDX: an edge linking two entities, a reference filling a slot with a handle, are elements like any other, with their own identity and attributes. Configurations point at objects, vocabularies point at schemas, records point at each other — one uniform model, nothing special-cased about the act of pointing, and references you can attach data to like anything else.

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

##### TODO

Consider introducing the idea of information generally being segmented into either a tree form or graph form, and that GDX merges the two concepts 

## Author's Note

Wait, what's an author's note doing at the start of a data format primer? 

Circa 2014 I'd taken on a development project for a client with a tight 3 month deadline. Early in development process I realized I would need a way to exchange data between the existing desktop app and the new web app. The initial case was solved with a CSV table dump. But scope did what scope does, and before long it crept to require more tables, references and context. Rejecting the obvious fix (dump the tables and just rebuild them with a script) as far too practical, I decided to take "the afternoon" and create a file format for exchanging color data.

As that afternoon dragged on into a week I conceded defeat, hacked something together and moved on. The thing I shipped worked, but  Like Frankenstein, my monstruous transfer format kept me awake at nigh. Its inelegance (obvious to me alone) need to be resolved.

Over the next few months I fiddled with approaches. My once focused requirements expanding from specific to a general case. My app data needed context, carried metadata, and had both self-referential and external links. Toiling in JSON wasn't doing me any favors as my metadata requirements grew. Most of these issues were solvable in XML, but the XML rabbit hole was deep. When the requirements grew to include exchange with third-party systems, I felt obliged to produce something coherent enough that *another* system could make sense of it, and I soon found myself building schemas. The quest for self-description and typing had begun, and it was unpleasant.

The feeling was too much, and I decided to try to build something really good to replace it. Starting from a practical point, I decided that a cleaned-up XML format would deliver what I wanted. Sketching out various schemas, I found a nice tight approach that met my needs perfectly. And then I immediately decided that it would be much better if this nice valid solution also worked for a more general problem set. Why fix one target problem well when I could fix many problems nearly as well? It was at this point that I had the brilliant revelation: why restrict myself to a mature, well-supported file format when I could reinvent the wheel and this time do it better! Best of all, I now had a way to fill my spare time, my afternoons, my nights, my weekends, and my vacations. I now had a justification for reading RFCs at coffee shops. I now had opinions about XML schema approaches, joked about YAML bugs, and spent time crawling through JSON parser code. With my hard-earned perspective, I sat down to draft my new file format. Everything was fine for a while. And then one day, while minding my own business, I stumbled onto an article about the semantic web, which led me to RDF, and OWL, and down the ontology rabbit hole I went, emerging a few years later using phrases like *semantically orthogonal* at children's birthday parties. Somewhere during this time there was a lost weekend, waking up next to a book on Gellish with no memory of how I got there. There was the month of graphs that I still can't speak about. And each time I looked back at my project, I saw it through a different lens: a serialization format, an object model, a schema language, an ontology, a knowledge base... a serialization format, an object model, a schema language... are you getting it? Not five separate concerns, one stubborn, quixotic quest to fit all of them into a single coherent thing.

After another couple of years of refining, throwing the whole thing out, writing a truly awful implementation that revealed all the ways to do it wrong, starting over, pruning, and then rewriting it from scratch one more time, I started circling the shape laid out below.

Whether GDX shall turn out to provide a benefit, or whether that station will be held by another attempt, these pages will show. So what is GDX? First, it's a way of thinking about how data is organized, labeled, and exchanged so that it becomes *information*. The goal, hopefully realized below, is to let one small unit of data work on several planes at once. Within a GDX element we have a typed value; a name, so we can speak about it; an address, so we can point at it; a shared meaning; and a node in a graph, linked to whatever it relates to. Lots of approaches can do that. GDX does it in a small space, using position and shape to carry meaning. That's what lets a single GDX line say so much.

The risk, of course, was complexity, and yes, GDX can get complex. It hands you tools for doing some genuinely neat things. But it also lets you forget, ignore, shun, dismiss, and divorce that complexity in favor of simple statements: a key-value pair with a type. In its simple form, GDX is like BASIC, or, for the younger and ever-growing cohort, Python. At the other end, it's a line of C# crammed with lambdas and attributes. And it tries very, very hard to *tell you how to use it*: the simple, repeating form means you can only go so far off the rails, while quietly removing hard questions of morality like "is this an attribute or a value?"

GDX is opinionated about how things should be done. It treats bytes differently from strings. It's clear about what a reference is. It won't let you invent new building blocks when the ones it gives you compose just fine. And it thinks a value should be allowed to be just a value, right up until the moment you need it to be more.

Below is the attempt to make something useful. It's what I built instead of writing that novel.
