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

`MODEL` instances declare what models have types that have been serialized in the file.

Only models that are used in the file must be declared. A model is used if at least
one element in the file has a `vodml-ref` with the model's prefix/name. In other terms, only models that
define `vodml-ids` used in the annotation (see [@sec:vodmlref]).

A `MODEL` is uniquely and globally identified by its `NAME`, which refers to the latest available
minor version of a specific model. According to the VO-DML specification [@vodml], all minor versions
of a model are compatible with each other, and major versions have different names (e.g. `stc` vs `stc2`).

Clients should not be sensible to the specific minor version of a model and can simply ignore `vodml-ref`s
they are not familiar with.

The `NAME` is also the prefix of the `vodml-ref`s pointing to elements of a specific model.

For clients that need to parse the model descriptions, i.e. the VODML/XML file of a model, data providers
**must** include the URL of the VODML/XML document in the `MODEL/URL` attribute. This URL **must** be persistent,
and the same URL that is registered for that model in the IVOA registries, if the model is defined in an IVOA
recommendation. If the model is a custom extensions, it should also be registered in IVOA registries. However,
since this is not a requirement, for custom extensions that are not registered, data providers **must** still
provide the URL of the VODML/XML description file, and care should be taken into ensuring that the URL is
persistent, otherwise clients will not be able to read the model definition.

Note that because of the above procurements, clients are not required to resolve the `NAME` of the model to
its VODML/XML document, but can, if necessary, rely on the `URL` element.

In [@lst:simple] three models are declared, with the names `ivoa`, `filter`, and `sample`,
which are the prefixes used in all the `vodml-refs` in the file.

### Instance Annotation: `INSTANCE` ###

VO-DML structured types are annotated by using the new `INSTANCE` XML element. Note that there is no difference,
from a schema point of view, between `**ObjectType**`s and `**DataType**`s. However, some restrictions apply and
are enforced through schematron rules. More details are provided in [@sec:norm-object;@sec:norm-data].

Instances **must** a *type* (`@dmtype` attribute). Most instances also have a *role* (`@dmtype` attribute), but
root instances, i.e. instances that are direct children of the `GLOBALS` and `TEMPLATE`s elements
(see [@sec:norm-globals;@sec:norm-templates]) do not have a role.

Instances **may** be provided with an `@ID` value which allows other objects to refer to the instance itself
(see [@sec:norm-idref]).

Instances usually have a number of *roles*, which can be themselves instances through a composition relationship
([@sec:norm-composition]), references to other instances ([@sec:norm-reference]), or attributes
([@sec:norm-attribute]).

As explained in more detail in [@sec:norm-composition] and related sections, a composition relationship is
annotated so that parents contain or refer to children instances and children point to their parent instances
through the `CONTAINER` element ([@sec:norm-container]).

#### `PRIMARYKEY` ####
Instances **may** be identified by an object identifier through the `PRIMARYKEY` element.

An object identifier can have any number of `PKFIELD` fields. Each `PKFIELD` is a choice among a `LITERAL`,
i.e. a local value,
a `COLUMN`, i.e. a reference to a cell value, or a `CONSTANT`, i.e. a reference to a `PARAM`.
See [@sec:norm-relations] for more information about these elements.

Fields are values that make up the object identifier. In most cases only one
value is used, but it is not uncommon for objects to be identified by a tuple of values.

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

### Attributes: `ATTRIBUTE` ### { #sec:norm-attribute }

### Attributes: `INSTANCE` ###

### Attributes: `LITERAL` ###

### Attributes: `CONSTANT` ###

### Template Attributes: `COLUMN` ### { #sec:norm-column }

### Containers: `CONTAINER` ### { #sec:norm-container }

### Containers: `PRIMARYKEY` ###

### Containers: `FOREIGNKEY` ###

### References: `REFERENCE` ### { #sec:norm-reference }

### References: `IDREF` ### { #sec:norm-idref }

### References: `REMOTEREFERENCE` ###

### References: `FOREIGNKEY` ### { #sec:norm-reference-foreignkey }

### Compositions: `COMPOSITION` ### { #sec:norm-composition }

### Compositions: `INSTANCE` ###

### Compositions: `EXTINTANCES` ###

Representing Types { #sec:norm-types }
------------------

### **`PrimitiveType`** ###

### **`Enumeration`** and **`EnumerationLiteral`** ###

### **`ObjectType`** ### { #sec:norm-object }

### **`DataType`** ### { #sec:norm-data }

### Type Extensions ###

### Quantities ###
