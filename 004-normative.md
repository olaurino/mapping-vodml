Patterns for annotating VOTable [NORMATIVE] { #sec:normative }
===========================================

In this section we list all legal mapping patterns that can be used to
express how instances of VO-DML-defined types are represented in a
VOTable and the possible roles they play. It defines the VOTable
annotation syntax, what restrictions there are, and how to interpret the
annotation semantically.

The organization of the following sections is based on the XML types introduced
in the new VOTable schema. The entire VO-DML annotation is included in a new `VODML`
element and its descendents. The following sections introduce each element and how
it is used to map the VO-DML concepts to VOTable documents.

In particular, [@sec:norm-vodml;@sec:norm-relations] describe
the schema elements that provide data model annotations, while [@sec:norm-types] inverts
this approach and shows how to map VO-DML concepts to the VOTable elements that have been
introduced.

Some comments on how we refer to VOTable and VO-DML elements:

-   When referring to VOTable elements we will use the notation by which
    these elements will occur in VOTable documents, i.e. in general "all
    caps", E.g. `GROUP`, `FIELD`, (though `FIELDref`).

-   When referring to an XML attribute on a VOTable element we will
    prefix it with a ‘@’, e.g. `@id`, `@ref`.

-   References to VO-DML elements will be CamelCase and in **`bold face`**,
    using their VO-DML/XSD type definitions. E.g. **`ObjectType`**, **`Attribute`**.

The following list defines some shorthand phrases (*italicized*), which we
use in the descriptions below:

-   Generally when using the phrase *meta-type* we mean a "kind of" type
    as defined in VO-DML. These are **`PrimitiveType`**, **`Enumeration`**,
    **`DataType`** and **`ObjectType`**.

-   With *atomic type* we will mean a **`PrimitiveType`** or an
    **`Enumeration`** as defined in VO-DML.

-   A *structured type* will refer to an **`ObjectType`** or **`DataType`**
    as defined in VO-DML.

-   With a property *available on* or *defined on* a (structured) type
    we will mean an **`Attribute`** or **`Reference`**, or (in the case
    of **`ObjectType`**s) a **`Composition`** defined on that type itself, or
    inherited from one of its base class ancestors.

-   A VO-DML **`Type`**  *plays a role* in the definition of
    another (structured) type if the former is the declared data type of
    a property available on the latter.

-   When writing that a VOTable element *represents* a certain VO-DML
    type, we mean that the VOTable element is mapped either directly to the
    type, or that it identifies a role played by the type in another type's
    definition.

-   A *descendant* of a VOTable element is an element contained in that
    element, or in a descendant of that element. This is a standard
    recursive definition and can go up the hierarchy as well: an
    *ancestor* of an element is the direct container of that element, or
    an ancestor of that container.

When we say this section is NORMATIVE we mean that:

  1. when a client finds an annotation pattern conforming to one defined
     here, that client is justified in interpreting it as described in
     the comments for that pattern. It is an ANNOTATION ERROR if that
     were to lead to inconsistencies[^15].
  1. when a client encounters a pattern not in this list, the client
     SHOULD ignore it. Interpreting it as a mapping to a data model MAY
     work, but is not mandated and other clients need not conform
     to this.

[^15]: E.g. when interpreting an `<INSTANCE>` as a certain **`ObjectType`**, if one of
 its children is not annotated or identifies a child element that is
 not available on the type, this is an error. For each pattern there
 is a set of rules that, if broken, are annotation errors. (**TODO** we
 better strive to make these comprehensive. OL thinks this is too strict of a default rule.
 We should probably be more lenient by default and add strictness where required.
 A lot is already mandated by the new schema.)

The `VODML` element { #sec:norm-vodml }
-------------------

This specification introduces a new `VODML` element to `VOTABLE`.

The element is a direct child of `VOTABLE` and only one `VODML` element is allowed
in each file. This element contains all the data model annotations for the entire file [^vodml-resource].

There are three sections making up the contents of the `VODML` annotation:

  * model declarations (`MODEL` elements)
  * global instances (`GLOBALS` elements)
  * tabular instance templates (`TEMPLATES`).
  
The model declarations introduce the models used in the annotation. Global instances are direct instances,
i.e. instances whose values are completely determined by the annotation, as opposed to instance templates,
which annotates multiple instances that have some of their values represented in tabular format inside the
annotated file, as described in more detiail in the following sections.

[^vodml-resource]: Earlier versions of this specification had the `VODML` element defined as
a potential child of several existing elements, including  `RESOURCE` and `TABLE`. While this
achieved a form of locality by bringing the annotations close to the elements it was annotating,
it introduced two different, valid ways of annotating files. Also, the locality it achieved was
completely lost in the more complex cases with multiple `TABLE` elements, where this kind of
locality matters the most. A single instance of `VODML` is much simpler to implement for data
providers and consumers alike, and it provides only one obvious way of annotating a file.
On the other hand, the single `VODML` element might pose challenges to providers streaming
VOTable instances without a knowledge of the semantics of the contents upfront. This case
seems more like a theoretical possibility than an actual implementation at this point, so it
was ignored. It could always be possible to add the `VODML` element as a child of other elements
in future versions of this specification, keeping into account actual requirements from actual
implementations rather than just a theoretical possibility.

### Example ###

~~~~ vodsl
file: instances/simple.vodsl
caption: |
    Example VOTable with basic annotations. This VOTable illustrates the basic
    annotation elements in the `VODML` section. More complex examples will be provided
    in the following sections.
id: lst:simple
classes:
  - xml
  - numberLines
----
~~~~

The example in [@lst:simple] uses sample models that are simple enough to be useful for exploring the mapping
patterns, but also complete as they exercise the entirety of the mapping schema. These models
are not IVOA standard models themselves, but were defined for illustrating the
mapping specification only.

### Models Declaration: `MODEL` ### { #sec:norm-model }

A VOTable can provide serializations for an arbitrary number of data model types.
In order to declare which models are represented in the file, data providers **must**
declare them through the `MODEL` elements.

Only models that are used in the file must be declared. A model is used if at least
one element in the file has a `vodml-ref` with the model's prefix/name. In other terms, only models that
define `vodml-ids` used in the annotation (see [@sec:vodmlref]) must be declared.

A `MODEL` is uniquely and globally identified by its `NAME`, which refers to the latest available
minor version of a specific model. According to the VO-DML specification [@vodml], all minor versions
of a model are compatible with each other, and major versions have different names (e.g. `stc` vs `stc2`).

Clients should not be sensible to the specific minor version of a model and can simply ignore `vodml-ref`s
they are not familiar with.

The `NAME` is also the prefix of the `vodml-ref`s pointing to elements of a specific model.

For clients that need to parse the model descriptions, i.e. the VODML/XML file of a model, data providers
**must** include the URL of the VODML/XML document in the `MODEL/URL` element. if the model is defined in an IVOA
recommendation, then this URL **must** be persistent,
and the same URL that is registered for that model in the IVOA registries.
If the model is a custom extensions, it **should** also be registered in IVOA registries. However,
since this is not a requirement, for custom extensions that are not registered data providers **must** still
provide the URL of the VODML/XML description file, and care should be taken into ensuring that the URL is
persistent, otherwise clients will not be able to read the model definition. For more information on
registering and identifying data models, please refer to the VO-DML specification ([@vodml])

Note that because of the above procurements, clients are not required to resolve the `NAME` of the model to
its VODML/XML document, but can, if necessary, rely on the `URL` element.

In [@lst:simple] three models are declared, with the names `ivoa`, `filter`, and `sample`,
which are the prefixes used in all the `vodml-refs` in the file.

### Instance Annotation: `INSTANCE` ### { #sec:norm-instance }

VO-DML structured types are annotated by using the new `INSTANCE` XML element. Note that there is no difference,
from a schema point of view, between `**ObjectType**`s and `**DataType**`s. However, some restrictions apply and
are enforced through schematron rules. More details are provided in [@sec:norm-object;@sec:norm-data].

Instances **must** a *type* (`@dmtype` attribute). Note that instances are not annotated with a role, as roles are
mapped to specific XML elements (see [@sec:norm-relations]). This is compatible with the intuitive fact that the same
instance can have different roles in relationships with other instances.

Instances **may** be provided with an `@ID` value which allows other objects to refer to the instance itself
(see [@sec:norm-idref]).

Instances usually have a number of *roles*, which can be themselves instances through a composition relationship
([@sec:norm-composition]), references to other instances ([@sec:norm-reference]), or attributes
([@sec:norm-attribute]).

As explained in more detail in [@sec:norm-composition] and related sections, a composition relationship is
annotated so that parents contain or refer to children instances and children point to their parent instances
through the `CONTAINER` element ([@sec:norm-container]).

### Global Instances: `GLOBALS` ### { #sec:norm-globals }

Some annotations may map the VOTable contents to instances of data model types that are global in the file,
possibly because such instances are referenced by other intances that annotate specific tables.
More generally, some annotations will define instances that are completely defined in terms of constant
value, i.e. they are not represented in tabular form. Rather, they are completely and directly represented
by an `XML` element.

Such instances should be included in the `GLOBALS` element.

An annotation can have multiple `GLOBALS` sections so to group together global instances that are related
to each other.

In particular, `GLOBALS` sections can be identifier by an `@ID` attribute. This is particularly useful
when referencing global instances from table elements, see for instance [@sec:norm-reference-foreignkey].

`GLOBALS` **must** only contain direct representations of instances,
i.e. `INSTANCE` elements that do not have any `COLUMN` elements directly or in any of their descendants.
This rule is not enforces via the XSD schema but by schematron rules.

Also, `GLOBALS` **should not** contain any `INSTANCE`s with `REFERENCE`s to indirect `INSTANCE`s, unless
the model allows for multiple references with the same role, which is however deprecated by the `VO-DML`
specification.

As an example, [@lst:simple] contains one `GLOBALS` section with one instance representing a coordinate
frame in the rather simplistic *sample* model. The coordinate frame instance is completely determined by
the global `INSTANCE`.

### Tabular Instances: `TEMPLATES` ### { #sec:norm-templates }

Most data in VOTable is expressed in tabular form, each row representing one individual instance.
When this is the case, instances need to be represented through *templates* in the `TEMPLATES` section
of the annotation. An `INSTANCE` in a template is identical to `INSTANCE`s in the `GLOBALS` section,
but they are also allowed to contain `COLUMN` elements (see [@sec:norm-column]), which are references to
regular VOTable `FIELD`s. Rather than being directly and completely represented by its `INSTANCE` annotation,
instances are thus materialized for each row in a table, according to their annotation. Note that a `COLUMN` may
be present either directly in an `INSTANCE` or in any of its descendent `INSTANCE`s.

`TEMPLATES` **must** refer to a `TABLE` through the `tableref` attribute. `COLUMN` elements inside a template
`INSTANCE` **must** be references to `FIELD`s in the `TABLE` identified by `tableref`.

In [@lst:simple] table `_table1` contains simple astronomical sources with columns for the sources
names, Right Ascension, and Declination in the ICRS reference frame. Thus, the sources are annotated using an
`INSTANCE` element inside the `TEMPLATES` section of the `VODML` element. The instance and its attributes refer
to the three columns in the table and map their contents to the attributes of sources and positions defined
in the respective data model.

### Direct vs Indirect Instances ###

As an illustration of the difference between direct and indirect instances, consider the following VOTable:

~~~~ vodsl
file: instances/direct_sources.vodsl
caption: |
    VOTable where all instances are direct rather than indirect.
id: lst:direct_sources
classes:
  - xml
  - numberLines
----
~~~~

This is a representation of the very same instances as in [@lst:simple]. However, while [@lst:simple] uses
template instances in `TEMPLATES` and a `TABLE` element to represent the values of the sources,
[@lst:direct_sources] uses direct representations for all three sources in the example.

Generally speaking, this specification is used to annotate data products according to one or more data models.
In this sense, the choice on whether to use direct or indirect representations is driven by the data being
produced. In principle, for each indirect representation there is an equivalent indirect representation where
template instances and one or more tables are turned into a number of explicit, direct instances.

### Schema Constraints ###

In order to be valid, the `VODML` **must** contain at least one `MODEL` instance and at least one
of either `GLOBALS` or `TEMPLATES`, which in turn **must** contain at least one `INSTANCE`.

Relations { #sec:norm-relations }
---------

All instances have a type, and some instances also play a *role* in another instance. For example, if a **Source**
has a **position** of type **Coordinate**, then an instance of type **Coordinate** will also have the role
of **position** in the enclosing **Source** instance.

The role corresponds to a relationship between two types.

The following roles are mapped from VO-DML to VOTable:

  * Attributes ([@sec:norm-attribute]).
  * Compositions ([@sec:norm-composition]).
  * References ([@sec:norm-reference]).

### Attributes: `ATTRIBUTE` ### { #sec:norm-attribute }

An attribute may be represented by another instance ([@sec:norm-instance]),
by a primitive or enumerated value ([@sec:norm-literal;@sec:norm-constant]), or by a column ([@sec:norm-column]) if
the parent instance is a template, i.e. an indirect representation.

An `ATTRIBUTE` **must** have a `@dmrole` attribute indicating the role of the attribute, and such `@dmrole` **must**
be a valid vodml-ref as defined in one of the VO-DML models declared in the models section ([@sec:norm-model]). The
vodml-ref **must** also identify a VO-DML Attribute.

### Attributes: `LITERAL` ### { #sec:norm-literal }

A `LITERAL` **must** represent an attribute with a primitive or enumerated type. It **must** have a `@value` attribute
and optionally a `@unit`. It **must** also have a `@dmrole` attribute, whose value **must** be a valid vodml-ref defined
in one of the declared models ([@sec:norm-model]) and identify a VO-DML role with a primitive type.

### Attributes: `CONSTANT` ### { #sec:norm-constant }

A `CONSTANT` is a reference to a VOTable `PARAM`. This element can be used in place of a `LITERAL` to point to an
existing `PARAM` in the same VOTable, avoiding duplicated values.

Note that the target of the reference **must** be a `PARAM`.[^why-not-param]

[^why-not-param]: There are a few design considerations behind the choice of not including `PARAM`s, `PARAMref`s and
`FIELDref`s directly in this specification. One is that the VO-DML mapping schema might be used outside of is VOTable
context to describe and map data model instances in XML. The other is to keep the VOTable and VO-DML schemata as much
decoupled as possible, in order to facilitate future revisions. Also, `PARAM` is too rich of an element with its own
attributes and semantics. Such semantics are rather different than the ones in `VO-DML`, in particular regarding the
different kinds of types (`@utype`, `@xtype`, `@datatype`) and annotations (`@arraysize`, `@ucd`) that a `PARAM` is
composed of, which do not have an equivalent in VO-DML.

### Template Attributes: `COLUMN` ### { #sec:norm-column }

A `COLUMN` is a reference to a VOTable `FIELD`. Instances representing attributes as `COLUMN`s **must** be defined
inside a `TEMPLATES` element. A `COLUMN` **must** have a `@dmtype` attribute whose value **must** be a valid vodml-ref
defined by one of the declared models ([@sec:norm-model]). Such vodml-ref **must** identify the VO-DML type
corresponding to the enclosing attribute's role.

### References: `REFERENCE` ### { #sec:norm-reference }

A reference is a relationship between a referring instance and a referred instance. While the referring instance can
be both an **`ObjectType`** and a **`DataType`** the referred instance **must** be of an **`ObjectType`**.

The reference can be to an identified direct instance in the same file ([@sec:norm-idref]), to a remote instance
identified by a URI ([@sec:norm-remote-reference]), or to an instance indirectly represented by a template, through
a foreign key pointing at the referred instance's primary key ([@sec:norm-reference-foreignkey]).

### References: `IDREF` ### { #sec:norm-idref }

In order to use `IDREF` the referred instance **must** be an instance of an **`ObjectType`**, it **must** be serialized
as a direct instance, and it **must** be located in the same document as the referrer instance. Also, there **must**
be a correspondent relationship defined in the VO-DML description of the model in which the referrer type is defined.

### References: `REMOTEREFERENCE` ### { #sec:norm-remote-reference }

In order to use `REMOTEREFERENCE` the referred instance **must** be an instance of an **`ObjectType`**,
it **must** be serialized as a direct instance, and it **must** be located in a different document than the referrer
instance. Also, there **must** be a correspondent relationship defined in the VO-DML description of the model in which
the referrer type is defined.

`REMOTEREFERENCE` is of type `xs:anyURI` an **must** identify a globally unique instance through a specific URI (e.g. an
IVOA Resource Name), **or** a URL to the serialization of the instance, but the URL **must** be persistent.

### References: `FOREIGNKEY` ### { #sec:norm-reference-foreignkey }

The referred instance may be serialized as a row in a table different than the one serializing the referrer (if any).
In this case the reference is annotated through a `FOREIGNKEY` ([@sec:norm-foreign]).

### Composition: `COMPOSITION` ### { #sec:norm-composition }

A composition is a whole-part relationship between **`ObjectTypes`**, where
one instance is said to be the container, or parent, or whole, and the other is said to be the contained, or child, or
part. In a VOTable composed instances may be serialized directly within the parent instance
([@sec:norm-instance]) if the parent is directly representated or if the part is serialized in the same table
as the parent, or externally ([@sec:norm-extinstances]) if the part is indirectly represented and serialized in a
different table. Children refer back to their parent through an implicit container relationship
([@sec:norm-container]). The reference is achieved through a foreign key ([@sec:norm-foreign]) referencing the
container's primary key ([@sec:norm-primary]).

In order to be valid, the composition **must** link two **`ObjectType`**s and there must be a correspondent relationship
defined in the VO-DML description of the model. The number of contained instances must be compatible with the
relationship's multiplicity.

The `COMPOSITION` element **must** have a `@dmrole` attribute, whose value **must** be a valid vodml-ref identifying a
composition relationship in one of the models declared ([@sec:norm-model]), and types of the parent and children
**must** be compatible with the relationship defined in that model.

### Composition: `EXTINTANCES` ### { #sec:norm-extinstances }

A composition relationship can delegate the definition of some of the children to an external `INSTANCE` declaration
elsewhere in the file, for example if the instances are defined in a different table.

The `EXTINSTANCES` element is useful for clients in that it links instances from the parent to the children. In
relational data bases, the composition relatioship is usually implementing by referencing objects from the parts to
the whole, not the other way around (see @[sec:norm-container]). This specification provides a way for containers to
declare where the contained objects are located and described.

(**TODO** this leaves the door open to the possibility that children refer to their parents but not the other way
around. Should clients be prepared to this possibility? If so, then the `EXTINSTANCES` element is redundant. Otherwise
what seems to be redundant is the `CONTAINER` element).

### Composition: `CONTAINER` ### { #sec:norm-container }

On the part side of a composition relationship, especially in complex Object Relational Mapping applications, parts
include a reference to their containers. This is consistent with the relational implementation of the composition
relationship. For more details see [@sec:norm-foreign].

### `PRIMARYKEY` ### { #sec:norm-primary }
Instances **may** be identified by an object identifier through the `PRIMARYKEY` element.

An object identifier can have any number of `PKFIELD` fields. Each `PKFIELD` is a choice among a `LITERAL`,
i.e. a local value ([@sec:norm-literal]),
a `COLUMN`, i.e. a reference to a cell value ([@sec:norm-column]), or a `CONSTANT`, i.e. a reference to a `PARAM`
([@sec:norm-constant]).

Fields are values that make up the object identifier. Generally only one
value is used, but it is not uncommon for objects to be identified by a tuple of values.

Primary keys **must** be unique within object types. In other terms there can be no two instances with the same primary
key in a single file. If two instances appear to have the same primary key then client's behavior is undefined.

### `FOREIGNKEY` ### { #sec:norm-foreign }

In Object Relational Mapping a foreign key is the mechanism by which instances in a table identify other instances in
many-to-one and many-to-many relationships with itself.

In a simple model where a source can be observed in an arbitrary number of photometric filters, one would usually have
a table for sources, with IDs and general metadata, a table for photometric filters, and a table for luminosity
measurements of a source. In this implementation, the Luminosity entity is in a many-to-one relationship with both
the Source and the Filter tables. This relationship is implemented by providing the Luminosity table with two foreign
key columns to the Source and Luminosity tables. The value of the foreign keys is the ID of the Source and Filter
instances to which each luminosity measurements refer to.

A query might query the data base for "all the luminosity measurements of Source 3c273", implying the ownership
relationship of a Source with its Luminosities, although there is no actual reference from the Source table to the
Luminosity table.

Providing the parent table with a column for each reference to an arbitrary number of potential children is clearly
unsustainable, if at all possible.

A `FOREIGNKEY` can provide a `TARGETID` to point to an identified element in the same file.
(**TODO** Be more specific here on how to use `TARGETID`)

A `FOREIGNKEY` **must** contain at least one `PKFIELD`, and the structure of the key fields must be compatible with
the one of the primary keys of the referenced instance.


Representing Types { #sec:norm-types }
------------------

### **`PrimitiveType`** ### { #sec:norm-map-primitive }

Primitive Types are types without structure, and so instances are represented by a simple value. They can be mapped to:

  * a `LITERAL` element if the value is provided in the file's header ([@sec:norm-literal]);
  * a `CONSTANT`, i.e. a reference to a `PARAM`, if the value is mapped to the value of 
  an existing `PARAM` ([@sec:norm-constant]);
  * a `COLUMN`, i.e. a reference to a `FIELD`, if the value is in a table cell. In this case we say that the enclosing
  instance is indirectly represented by an `INSTANCE` template, with actual instances serialized in tabular format
  ([@sec:norm-column]).

### **`Enumeration`** and **`EnumerationLiteral`** ### { #sec:norm-map-enumeration }

Enumerations are primitive types with a limited number of possible, enumerated values. Enumerations are mapped to the
same elements as primitive types ([@sec:norm-map-primitive]), with the limitation that values **must** be valid
enumeration literals compatible with the type declared by the enclosing `ATTRIBUTE` element.

### **`ObjectType`** ### { #sec:norm-object }

Object types are mapped to the `INSTANCE` element ([@sec:norm-instance]). Object types can have **`DataType`**d
attributes, which are mapped to the `ATTRIBUTE` element ([@sec:norm-attribute]). The `ATTRIBUTE` **must** have a
`@dmrole` corresponding to the model-defined role identifier. In turn, the `ATTRIBUTE` **must** contain an `INSTANCE`
with a type compatible with the attribute's role. Note that the type can be the type declared directly in the model
or any specialization of that type defined in any model declared via the `MODEL` element.
 
Object types can also have attributes with a `PrimitiveType` type or with an `Enumeration` type. In this case
attributes are mapped following the patterns described in [@sec:norm-map-primitive] and [@sec:norm-map-enumeration] 
respectively.

Object types can hold references to instances of other object types. This roles are mapped to the `REFERENCE` element
([@sec:norm-reference]). In this case the `REFERENCE` **must** have a `@dmrole` corresponding to the model-defined
reference relationship.

Finally, object types can be in composition relationships with other object types. An object type can be both the
*parent*, or *whole*, and *contain* its *children*, or *parts* ([@sec:norm-composition]).

A parent `INSTANCE` will have a `COMPOSITION` element with the `@dmrole` of the composition relationship defined in the
model.

In addition to `INSTANCE`s, parents can also point to instances described elsewhere in the document through the
`EXTINSTANCE` elements.
  
For each composition relationship an arbitrary number of instances may be present, with two limitations:

  * the number of instances must be compatible with the multiplicity of the relationship;
  * the instances must have `@dmtype`s compatible with the data type of the relationship defined in the model, i.e.
  the exact type declared there or any of its subtypes.

A children `INSTANCE` will have a `CONTAINER` element pointing at the 

As described in [@sec:norm-globals] and [@sec:norm-templates] an **`ObjectType`** can be represented both directly or
indirectly through standalone instances or instance templates respectively.

### **`DataType`** ### { #sec:norm-data }

Data types are mostly mapped like object types. This removes a lot of burden from data providers and clients that do not
need to validate datasets, simplifying the syntax.

Some constaints apply to **`DataType`**s. They are not enforced through XSD but datasets **must** meet such constraints
in order to validate.

VO-DML **`DataType`**s **cannot** be in composition relationship with any other types, so `COMPOSITION` and `CONTAINER`
**cannot** be used inside an `INSTANCE` representing a **`DataType`**. Similarly, **`DataType`**s cannot have a
`PRIMARYKEY`.

Also, there **cannot** be references pointing to a **`DataType`** instance.

### Type Generalization and Inheritance ###

(**TODO This needs to be expanded and completed with examples)

VO-DML allows types to *extend* or *specialize* other types. As it happens in object oriented languages, an instance of
a specialization, or *subtype* of a more general type, or *supertype*, is also a valid instance of the supertype, and
can be used wherever an instance of a supertype is expected.

This means that clients of the supertype must be able to recognize instances of the supertype even though a specialized
subtype might have been instantiated. This case includes all kinds of clients discussed in [@sec:clients]. What we
called *guru* clients can read the VO-DML description files and can figure out the generalization relationships.
*Advanced* clients might or might not care about generalizations, but generalization must be mapped with particular care
so that *simple* clients can find the information they are seeking, even when a type they are interested in is present
as an instance of one of its subtypes.

Since we currently provide only one `@dmtype` per `INSTANCE`, only one type can be expressed for each instance. This
type **must** always be the actual type of the instance. This means that clients **cannot** rely on the instance's
type in order to recognize instances, unless they rely on VO-DML-aware specialized software libraries.

In VO-DML specialized types inherit all of the supertype's vodml-id descriptors, including the prefix. This means that
`@dmrole`s can always be matched, whether or not an instance represents a supertype or one of its subtypes.

### Quantities ###

(**TODO** This is a placeholder for specific mapping of ivoa quantities, but we need the ivoa model to settle)

### Comparison of `EXTINSTANCES` and `INSTANCE` ###

The distinction between `INSTANCE` and `EXTINSTANCES` as children of the `COMPOSITION` element allows for common object
relational mappings to be used. In a completely normalized implementation each type may be serialized in its own table.
However, in practice astronomical archives and datasets show some form of flattening, i.e. some types are serialized
as part of the table that represent the parent type, or multiple instances of the part type are serialized in the same
table. This is the case, for instance, of sources and their luminosities.

In a mission's archive, and in the data sets they serve, usually a single table represents a number of photometric
measurements in different filters. This corresponds to a VO-DML annotation with an indirect `INSTANCE` for the Source
type with a `COMPOSITION` element containing a number of indirect `INSTANCE`s for the luminosity measurements.
Such instances correpond to the finite set of filters (and errors, and other kinds of metadata) specific to a particular
mission. All the `INSTANCE`s, in this example, would point to columns in the same table.

However, consider now the case of a data set that contains a number of sources, each with an arbitrary number of
photometric measurements coming from different missions and archives. In this case a single flattened table is not an
efficient or effective way of serializing such instances, and multiple tables can be used for sources and luminosities.
The `EXTINSTANCES` mechanism provides a mechanism for representing this pattern.
(**TODO** However, the safest and most efficient way is to just have children refer to their containers.)

Full Example
------------

~~~~ vodsl
file: instances/complex.vodsl
caption: |
    Example covering all of the XML elements and mapping patterns, including multi-table Object Relational Mapping.
id: lst:complex
classes:
  - xml
  - numberLines
----
~~~~

