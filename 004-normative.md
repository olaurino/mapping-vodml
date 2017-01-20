Patterns for annotating VOTable [NORMATIVE] { #sec:normative }
===========================================

In this section we list all legal mapping patterns that can be used to
express how instances of VO-DML-defined types are represented in a
VOTable and the possible roles they play. It defines which VOTable
elements can be annotated with `<VODML>` elements described in
section 4, what restrictions there are and how to interpret the
annotation.

The organization of the following sections is based on the different
VO-DML concepts that can be represented. Each of these subsections
contains sub-subsections which represent the different possible ways the
concept may be encountered in a VOTable and discuss rules and
constraints on those annotations. We start with **Model**, and then we
discuss value types (**PrimitiveType**, **Enumeration** and
**DataType**) and **Attributes**. Then **ObjectType** and the
relationships, **Composition** (collection), **Reference** and
**Inheritance** (extends). **Package** is not mapped: none of the use
cases required this element to be actually mapped to a VOTable instance.

See whether we should define a list of all the legal mapping patterns in
an Appendix.

Some comments on how we refer to VOTable and VO-DML elements (**TODO** TBD
redundant with text above in section ...)

-   When referring to VOTable elements we will use the notation by which
    these elements will occur in VOTable documents, i.e. in general “all
    caps”, E.g. GROUP, FIELD, (though FIELDref).

-   When referring to rows in a TABLE element in a VOTable, we will use
    TR, when referring to individual cells, TD. Even though such
    elements only appear in the TABLEDATA serialization of a TABLE. When
    referring to a column in the TABLE we will use FIELD, also if we do
    not intend the actual FIELD element annotating the column.

-   When referring to an XML attribute on a VOTable element we will
    prefix it with a ‘@’, e.g. `@id`, `@ref`.

-   References to VO-DML elements will be capitalized and in **courier
    bold**, using their VO-DML/XSD type definitions. E.g.
    ObjectType, Attribute.

-   Some mapping solutions require a reference to a GROUP defined
    elsewhere in the same VOTable. We refer to such a construct as a
    “GROUPref”, which is not an element of the current VOTable
    standard (v1.3). It refers to a GROUP with a `@ref` attribute, which
    must *always* identify another GROUP in the same document. The
    target GROUP must have an `@id` attribute. In cases where this is
    important we will indicate that this combination is to be
    interpreted as a “GROUPref”, including the quotes.

The following list defines some shorthand phrases (underlined), which we
use in the descriptions below:

-   Generally when using the phrase *meta-type* we mean a "kind of" type
    as defined in VO-DML. These are **PrimitiveType**, **Enumeration**,
    **DataType** and **ObjectType**.

-   With *atomic type* we will mean a **PrimitiveType** or an
    **Enumeration** as defined in VO-DML.

-   A *structured type* will refer to an **ObjectType** or **DataType**
    as defined in VO-DML.

-   With a property *available on* or *defined on* a (structured) type
    we will mean an Attribute or Reference, or (in the case
    of ObjectTypes) a Collection defined on that type itself, or
    inherited from one of its base class ancestors.

-   A VO-DML **Type**  *plays a role* in the definition of
    another (structured) type if the former is the declared data type of
    a property available on the latter.

-   When writing that a VOTable element *represents* a certain VO-DML
    type, we mean that the VOTable element is mapped through its
    `<VODML>` annotation either directly to the type, or that it
    identifies a role played by the type in another type’s definition.

-   A *descendant* of a VOTable element is an element contained in that
    element, or in a descendant of that element. This is a standard
    recursive definition and can go up the hierarchy as well: an
    *ancestor* of an element is the direct container of that element, or
    an ancestor of that container.

**Regarding the *normative* aspects of this specification**

When we say this section is NORMATIVE we mean that:

  1. when a client finds an annotation pattern conforming to one defined
     here, that client is justified in interpreting it as described in
     the comments for that pattern. It is an ANNOTATION ERROR if that
     were to lead to inconsistencies[^15].
  1. when a client encounters a pattern not in this list, the client
     SHOULD ignore it. Interpreting it as a mapping to a data model MAY
     work, but is not mandated and other clients need not conform
     to this.

[^15]: E.g. when interpreting an `<INSTANCE>` as a certain ObjectType, if one of
 its children is not annotated or identifies a child element that is
 not available on the type, this is an error. Fr each pattern there
 is a set of rules that, if broken, are annotation errors. (**TODO** we
 better strive to make these comprehensive.)

Model
-----

### Model declaration ###

#### Description ####

#### Example ####

ObjectType
----------

### Individual, global ObjectType instance ###

#### Description ####

#### Example ####

### Root ObjectType instance in Table I: not identified ###

#### Description ####

#### Example ####

### Root ObjectType instance in Table II: identified ###

#### Description ####

#### Example ####

DataType
--------

A DataType instance (also a *value*) is structured; it consists of
values assigned to each of its attributes and possibly references. To
represent the complete instance of a DataType the various attributes
(and references) must be grouped together; in VOTable this is done using
a GROUP element. GROUPs in fact can play two different roles, depending
on where the instance’s data is really stored. If all values are
eventually stored exclusively in PARAMs in the GROUP, or possibly
outside the GROUP but accessed through PARAMrefs, the GROUP *directly*
represents a complete instance.

If even only one of the attributes is stored in a FIELD and accessed
through a FIELDref, the GROUP *indirectly* represents possibly multiple
instances, one for each TR. This is also true in any child GROUP
containing a FIELDref and so on. We will use these terms, *direct* and
*indirect* representation all through the document. And note that this
same classification holds for ObjectTypes discussed in 7.2, though with
a twist related to possible child GROUPs representing Collections.

The attribute values of a DataType are stored according to the
prescription for storing instances of their data type. For attributes
with declared data type a PrimitiveType or Enumeration section provides
details. If the attribute’s data type is itself a DataType the
prescription in the current section should be used recursively.
DataTypes can also have References to ObjectTypes, but a discussion of
how to store References is deferred to section 7.4, after the discussion
of storing ObjectType instances, which is provided in section 7.2.

### Individual, global DataType instance ###

#### Description ####

(TODO OL: Should this be supported at all? It could just be mentioned that a
DataType instantiated as root is not supported by the standard, because they
should always be enclosed in ObjectType instances, but clients should try and
interpret them anyway).

Comments:

The possible role this instance plays in the VOTable document must have been
defined outside of any mapping to a data model. After all, in contrast to
instances of ObjectTypes, the existence of instances of value types need not be
explicitly stated (see the description of value types in [1]). An example could
be a VOTable containing the result of a simple cone search. A RESOURCE in that
document might contain the position of the cone, which may be mapped through its
to a DataType “src:source.SkyCoordinate”.

#### Example ####

(TODO Again: is this required? It could just be a special case clients should be
encouraged to support).

Note that the a **DataType** instance is allowed
to be contained in an `INSTANCE` without a type. There are some restrictions. NONE of its ancestor
`INSTANCE`s MAY be mapped to a data model element. This would conflict with
rules of mapping such an ancestor `INSTANCE` that state that children of such
`INSTANCE`s MUST be mapped to **Roles** of the structured type it represents.

An example of this pattern is the case of an assumed DAL protocol, say
SCS that defines *implicitly* some data structure that has components
that may be represented by elements from a data model as I the following
example.

### Root DataType values in TABLE records: Template Instance ###

#### Description ####

#### Example ####

### Attributes ###

#### Description ####

Restrictions:

  * The **Attribute** identified by the nested `INSTANCE` **MUST** be
    available to the structured type represented by the containing
    `INSTANCE`.

  * The attribute `INSTANCE` **SHOULD** also declare the actual structured type
    it represents, through its `VODML/TYPE`. This type **MUST** be a valid type
    for the **Attribute**, i.e. the **Attribute’s** declared type or one
    of its subtypes. If the type in the serialization is a subtype, the
    `VODML/TYPE` **MUST** declare that.

#### Example ####

PrimitiveType and Enumeration
-----------------------------

**PrimitiveTypes** and **Enumerations** represent atomic values. They
appear in serializations of data models predominantly in their role as
attributes defined on structured types (**ObjectType** or **DataType**)
and this specification focuses on that role. VO-DML assumes
the existence of a standard data model, namely the `ivoa` model
defined in [@vodml], defining primitive types such as *ivoa:integer*,
*ivoa:string* etc. But there is no new semantics implied by these types
beyond the ones their counterparts in the VOTable schema carry[^16].

[^16]: *ivoa* types that do not exist in VOTable may there be
    represented using appropriate xtypes, e.g. adql:timestamp for
    *ivoa:datetime.*

(**TODO** much of the above is probably obsoleted by the new mapping strategy)

### Individual primitive value ###

#### Description ####

This indicates a PARAM that is *not* contained in a VODML-annotated
`INSTANCE`, has a `VODML/TYPE` identifying an atomic type and no `VODML/ROLE`.

(**TODO** TBD do we want to support this? For if so, what about FIELDs? OL: NO!)


### EnumLiteral mapping

#### Description ####

#### Example ####

**TODO**

  * VODML/OPTION
  * value to literal
  * value to SKOS concept

### SKOSConcept mapping

**TODO**

Attributes
----------

### Attribute values in table

#### Description ####

  * A FIELDref contained in a GROUP represents an Attribute serialized
    by the FIELD it refers to.

Restrictions

  * The **Attribute** identified by the element MUST be
    available to the structured type represented by the containing
    INSTANCE.

  * The reference MUST identify a FIELD in a TABLE. The FIELD is guaranteed
  to be unique by the XML/VOTable rules for IDs.

  * The **Attribute** MUST have an atomic datatype, or a **DataType**
    from the set of special cases described in section 7.9.

  * The `@datatype` of the referenced FIELD SHOULD be compatible with the
    declared **datatype** of the **Attribute**. E.g. an integer
    (*ivoa:integer*) **Attribute** SHOULD be represented by VOTable’s
    int or long.

#### Example ####

### Attribute value { #sec:attribute-value }

#### Description ####

Restrictions

  * The **Attribute**’s **datatype** MUST identify an atomic type, i.e.
    a **PrimitiveType** or **Enumeration**, or a **DataType** from the
    set of special cases described in be section 7.9.
  * The PARAM `@datatype` SHOULD be a compatible, valid serialization type
    for the **datatype** of the **Attribute**.

#### Example ####

### Attribute value as reference to PARAM

#### Description ####

Inside a GROUP a PARAMref identifies a PARAM defined inside the same
RESOURCE or TABLE where the GROUP is defined. Using its VODML/ROLEthe
PARAMref can annotate the PARAM as holding the value of an
**Attribute**.

Restrictions

  * The **Attribute** must be available to the structured type
    represented by the containing INSTANCE.

  * The PARAM referencd by the reference MUST obey the restrictions
    defined for the annotation of a PARAM by the **Attribute’s** type in
    {@sec:attribute-value}.

#### Example ####

### DataType as Attribute

(**TODO** Should we support references to INSTANCEs as DataType attributes?)

#### Description ####

#### Example ####

References
----------

A **Reference** is a relation between a structured type (the “referrer”,
an **ObjectType** or **DataType**) and an **ObjectType**, the “target
object”, or “referenced object”. The reference is a property defined on
the referrer; it is one of the roles an **ObjectType** can play in the
definition of another structured type. **Reference** is a many-to-1
relation, many referrers can reference the same target object. The
implication of a Reference relation is that the referenced object is
somehow “used” by the referrer. Often, especially when the referrer is a
**DataType**, it defines reference data that helps interpret the values
its attributes assume. See the VO-DML document [@vodml] for more
information.

In explicit serialization languages [TBD must define and discuss this
term earlier in this document, say in section 3] such as XML Schema or
the relational model there is support for representing this relation.
E.g. XML schema allows one to define an attribute of type ID, which must
have a value unique in the document. Another element can “refer” to this
element using an attribute of type IDREF. It does so by assigning a
value to the IDERF attribute and that same value MUST have been assigned
to the ID attribute of another element for the document to be valid.
This equality of values implies the relationship between the two
elements, but there is in the definition no restriction on the type of
the referenced element.

If a restriction on the referenced element is desired, XML Schema also
allows one to define a “key/keyref” combination. A *key* has a unique
name and is defined using an XPath [REF] “selector”, identifying a set
of elements in a document and one or more fields that define the unique
values distinguishing between these elements. A *keyref* can similarly
be defined by the name of the target key, a selector identifying the
referrer elements and the fields used for identifying the specific
referenced elements among the ones defined by the key.

In a relational database a column on one table can be defined as a
(primary) key, its values must be unique in that table[^17]. Another
table (or even the same one) can declare a column to be a “foreign key”,
which “references” the first table through its key column.

[^17]: One can use multiple columns, that together must be unique (and
    NOT NULL!).

In all three cases the reference relationship is implemented using pairs
of elements which take up the same value, the ID or key, being
referenced using the IDREF and keyref/foreign key column(s). These
elements can be seen as addresses and pointers, the way by which similar
relationships can be made in programming languages, sometimes explicit,
as in C/C++, sometimes implicit as in Java/C\#.

The mapping prescription laid out in this specification follows the same
approach. It identifies three different patterns for representing
references between instances of **Types**. Their difference lies in the
way the target object is represented and identified, which puts demands
on the way to identify them on the referrer objects. If the target
object is serialized in the same document as the referrer it may be
either represented as an individual object represented by an INSTANCE or an
object in a table.

The specification also allows for the case where the target object is
*not* represented in the same VOTable document as the referrer. It may
exist in another VOTable, or in some alternative, possibly standardized
serialized form. An example of the latter could be an XML document
following a standardized XML schema explicitly defined to represent the
data model, or possibly a location in some publicly accessible database
with a relational schema mapped to the model in a standardized
Object-Relational manner.

This *remote reference pattern* is potentially very useful. By locating
certain standard, reference data objects in some well-defined place,
possibly a registry, different documents can reference the same instance
in an explicit and interoperable manner. In this case, instead of
providing a serialization of (part of) the referenced object with every
file that uses it, a reference to the external serialization could be
sufficient.

As an example consider defining a set of standard space-time coordinate
reference frames and registering these in an IVOA sanctioned registry,
with a well-defined and unique IVOA Identifier. Client tools could be
coded to understand references to such standard data sets, so that their
identifier might suffice to express the reference. Alternatively they
might cache such globally persistent instances so to minimize
de-referencing. Proper support for such standard reference data requires
some standardized form for their registration and storage, an effort
that should be part of the creation of standard data models and should
have maintenance responsibilities similar to that of the UCD vocabulary
[@ucd].

### Reference to individual Object

#### Description ####

Restrictions

  * The **Reference** MUST be *available* on
    the **StructuredType** identified by the referred instance.

  * The **OjectType** identified MUST be compatible with the
    **datatype** of the **Reference**, i.e. must either be the same type
    or one of its subtypes.

#### Example ####

### Reference to object(s) in a TABLE I: objects in same row

Pattern Expression:

#### Description ####

This pattern indicates that each row in a TABLE contains (at least)
two objects S and T, and that S references T
through the reference. R has no elements apart
from the VODML element.

#### Example ####

### Reference to object(s) in a TABLE II: objects in different rows, possibly different TABLE { #sec:reference-table-ii }

#### Description ####

This is the most complex pattern in this specification. It is the
analogue of a foreign key constraint in a relational database. The
pattern represents a reference R, contained in a source object S
representing a structured type, referencing a target instance T. S and T
may be contained in the same TABLE, but in general this need not be the
case.

This pattern indicates that each row in the TABLE contains a corresponding
structured object, which has a reference to an object represented by T,
located in some row in its TABLE. The reference is represented by R.

<!--
It is clear that in contrast to the
previous two cases the combination of `S/@ref` and `T/@ID` is not sufficient
to identify which objects are related. To make the connection this
specification prescribes that the relation must be based on a kind of
foreign key pattern.\
First, the objects T MUST be explicitly identified as described in
7.2.3. So T MUST contain a GROUP PK with
VODML/ROLE=”*vo-dml:ObjectTypeInstance.ID*” and I must contain
components which allow one to distinguish between different instances of
type T in table T2. This may be a unique[^18] identifier, but this is
not mandatory.

To identify which T is referenced, the reference GROUP R MUST contain a
“foreign key GROUP” FK that also has
VODML/ROLE=”*vo-dml:ObjectTypeInstance.ID*”. It’s structure MUST be
compatible with the structure of the GROUP PK. In general this means
that if PK contains *N* FIELDrefs, also FK must contain *N* FIELDrefs.
In general N=1, but if not, this specification prescribes that the order
is important. I.e. FK’s FIELDref 1 must be matched to PKs FIELDref 1
etc. to find the referenced objects. Note also that it is allowed that
FIELDref-s in PK are matched by PARAMs in FK. [TBD other way as well?]

-->

#### Example ####

### Reference to Object in external data store

#### Description ####

This pattern identifies a reference to an object serialized elsewhere.
Such an instance MUST be identified using an IVOA Identifier [@ivoid] if
it is stored in an IVOA Registry. However, a URL might be provided as a
convenient hook for the client.

#### Example ####

Here it is assumed that the DM working group maintains a set of data
model instance documents for various instruments and stores these in
some registry.

Composition
-----------

A **Composition** is defined in VO-DML as a “whole-parts” relation
between parent and child **ObjectTypes**. The child can be interpreted
as defining a *part* of the parent, the parent is *composed of* the
child objects. Generally a parent can have a collection of 0..N children
of a certain type, where N may be unbounded, but the multiplicity can be
more constrained. The relation between the child and its parent is much
stronger than the relation between a source and a target in a
**Reference** relation. In particular the child’s life-cycle is tightly
coupled to its parent: a child only has one parent and when the
parent is deleted, so is the child. Because of this also the
serialization of the relationship may in many cases be more direct than
the indirect reference mapping patterns of the previous section.

For example, in faithful XML serializations of a data model one will
generally map child objects as elements directly contained by the parent
element. It is possible to do so in VOTable as well, having a child
object represented by an INSTANCE that is contained within the INSTANCE
representing the parent. This pattern is the most natural for this
relationship because a collection element is really to be considered as
a *part* of the parent object.

This containment of INSTANCEs can be used for individual objects in an
obvious manner. But it may also be used to represent a flattening of the
parent-child relation in a TABLE. One or more child objects may be
stored together with the parent object in the same row in a TABLE. Such
a case is actually very common, for example when interpreting tables in
typical source catalogues. These generally contain information of a
source together with one or more magnitudes. The latter can be seen as
elements from a collection of photometry points contained by the
sources. (See the sample data model).

Faithful object-relational mapping of composition relations does
generally not allow for such a containment. The typical pattern is that
the table representing the child **ObjectType** has a foreign key to the
table representing the parent. This specification allows for this
mapping pattern as well, using the ORMReference pattern from section
[@sec:reference-table-ii].

### Composition I: child instance inside parent instance

#### Description ####

INSTANCE C represents a child object in a **collection** on some
**ObjectTupe** identified by its ROLE, and of **datatype**
possibly identified by its TYPE. In this pattern C is contained
inside of an INSTANCE P that represents an **ObjectType** that is a valid
**Container** of the **Collection**.

No statement is made about the location of the parent INSTANCE. Both parent and
child may represent individual, global objects. If their instances refer to
TABLEs, each row stores both parent and child objects.

Note also that there may be multiple child INSTANCEs identifying the same
composition relation in the same parent INSTANCE.

#### Example ####

### Composition II: object-relational reference from child to parent, both in TABLEs


#### Description ####

Every instance of an **ObjectType** inherits from its implicit ultimate
super type *vo-dml:ObjectTypeInstance* its
*vo-dml:ObjectTypeInstance.container* property. In the *vo-dml* model
this is represented as a collection of type *vo-dml:Reference*, with
multiplicity 0..1. This should be interpreted as the possibility of
adding on a serialized **ObjectType** instance a reference to its
container/parent object.

In the case of serializing to a VOTable, this allows one to implement an
object relational mapping pattern for the parent-child relation as
described in the introduction to this section.

The INSTANCE CH represents a collection of child objects that are contained in
parent objects represented by INSTANCE P. CH and P are generally stored in
different tables. CH references P following the ORM reference pattern described
in section [@sec:reference-table-ii] above.

<!--
For clients it is useful if an identification is made *which* collection
on the parent object the type represented by GROUP CH belongs to. The
VO-DML spec restricts types to be the child in at most one composition
relation, where inheritance/polymorphism is taken into account[^19].
Hence that information is in principle available through the VO-DML
model and an explicit indication would be redundant, but it may make
implementation of clients simpler.

One way to make this reference is to allow the GROUP CH to have a
VODML/ROLE identifying the collection as well as the VODML/TYPE. An
objection to this could be if we want to keep the invariant that only
those GROUPs can have a VODML/ROLE that are contained inside another
GROUP defining a type on which the role is available [TBD ref to
statement along these lines in this spec, *if* we impose this rule].
The following pattern is a possible implementation that preserves this
invariant:

--------------------------------------------------------------------------------
{GROUP CH, GROUP R, PARAM P | P $\in$ R & P/VODML/ROLE =
”*vo-dml:ORReference.container*” & `P/@value` $\Rightarrow$ **collection** &
`P/@datatype` = ”string” [ & P/VODML/TYPE = ”*vo-dml:ref*”] }

--------------------------------------------------------------------------------

The PARAM on the GROUP R identifies the collection, which MUST be
available on the **ObjectType** represented by the GROUP P.

Annotators SHOULD add a reference from the parent group to the group
representing the collection of child objects according to the following
pattern:

--------------------------------------------------------------------------------
{GROUP P, GROUP C, GROUP CH|
C $\in$ P & C/VODML/ROLE = **collection**
& C/VODML/TYPE = “*vo-dml:CollectionReference*”
& `C/@ref` = `CH@ID` }

--------------------------------------------------------------------------------

It indicates that the GROUP representing the container P contains a
GROUP C representing a “collection reference” to the GROUP with the
child objects C. It does *not* indicate that all objects in C are
contained in P. And the child objects MUST have the container reference
as defined in the first pattern above.

TBD Do we want these extra patterns? They imply a redundancy with the
first. That one is sufficient to define the relation and necessary as it
defines how to identify the parent on the child. The third pattern does
allow parsers to start looking for the children as soon as the
collection-reference is encountered.

**TODO** should we merge these patterns into one? Would make the pattern even
more complex.

-->

#### Example ####

Extends, inheritance
--------------------

The inheritance relation between two **Type**-s might seem not to be
relevant for serializations and mapping. After all the relation implies
no relation between objects, the target of serializations, only between
types. In serializations, any inherited **Role** can appear on a
sub-type and, as described above, all properties inherited from a
super-type are identified with the *vodml-id* defined on the super-type.

There are some special cases though where inheritance makes a
non-trivial appearance. The first we have seen already, in that we can
explicitly *cast* a **Role’s** value to a sub-type of a declared
datatype. See Appendix B for a discussion how different
types of clients can deal with this.

There are however subtler issues related to inheritance mapping. An
example comes from a typical object-relational mapping pattern for
inheritance hierarchies. When creating a relational model for an object
model with inheritance hierarchies, usually there are three patterns
people will follow.

One may choose to map each concrete (i.e. non-abstract) type to its own
table. But one may also choose to map all sup-types of a given
(ultimate) super type to a single table. Different rows may store
different sub-types, and certain columns defined for storing an
attribute defined on one sub-type will in general be NULL on its
siblings. Or one may choose a mixture of these where there is a table
for each type, but these only store elements defined on that type;
sub-types have a foreign key to their parent and to retrieve a complete
subtype this pointer must be followed, possibly recursively until the
ultimate super-type. Tools such as Hibernate [7] provide explicit
support for these inheritance mapping patterns, and we point the reader
to their documentation for more information.

In the latter two patterns there will generally be a table, either the
single one storing the whole hierarchy, or the one representing the
ultimate super-type, which defines a special column that allows one to
decide which concrete sub-type is stored in a particular row. This may
be through some pre-agreed code, or by storing the complete name of the
type.

This pattern can be supported as well.

(**TODO** The above description assumes one wants to serialize instances of
  multiple types in the same table or set of tables. Do we have a use case for
  this? If this is not required, this section should just highlight that
  the `vodml-ref`s for an instance of a subtype are inherited from the
  supertype.)

The extending type inherits all of the parent type’s Attributes and
their VO-DML-refs (including the prefixes), and adds its own Attributes
and their VO-DML-refs (including the prefixes).

The extending type also inherits all of the parent’s ancestors,
recursively.

#### Example ####

Serializing to other file formats {#sec:other}
=================================

VOTable is expressive enough to allow the mapping patterns described in
this specification. Other formats (notably FITS) cannot support such
annotations. In particular FITS does not support the GROUP-ing of
columns, which is of prime importance for the current specification. A
solution to this, which is even one of the motivations behind the
VOTable specification, is to annotate other format’s tables with a
VOTable *wrapper*. VOTable (see [2], section 5.2) already supports
wrapping FITS tables using a TABLE/DATA/FITS. This has a `STREAM/@href`
URL to specify which FITS file is represented and an `@extnum` attribute
to indicate which FITS extension is used.

Some FITS formats also allow a VOTable header to be included in an HDU.
Whilst this would allow data and meta-data to be combined in a single
file, this solution seems to be less portable, since some FITS readers
do not support these files.

A similar solution could be used to annotate the tables in the schema of
a TAP service. Here no specific support yet exists, but could be easily
added to the TAP specification. I fact a VOTable representing all TAP
tables as “empty” TABLE elements with only header, no DATA information
has been proposed in the past as one of the ways by which a TAP schema’s
metadata could be declared. If this were to be a formalized option it
would automatically allow the mapping patterns from the current
specification to be included. The main challenge for TAP is whether that
annotation can be carried over to the results of queries against its
schema. For the simple, column-only annotations such as UCDs or
descriptions, this is already a non-trivial task, and not always
possible[^20]. General ADQL queries can easily unravel the GROUP-ing of
columns and their nesting and produce results that cannot be annotated
with concepts of the original data models. However we believe this is a
90-10 problem, and for the most common queries it should be possible to
identify the data model concepts with not much more work than is already
required to carry along the UCDs.

This should in particular not be a problem for the results of standard
protocols such as the simple cone search. There the service provider is
in charge of designing the result set, which, being tabular, can be
easily annotated using the current specification. In fact, in the
absence of a TAP\_SCHEMA-like mechanism, it could be argued that the
current specification presents a perfect mechanism by which an SCS
service can declare the contents of its holdings.

Whereas similar approaches can likely be used for other tabular formats,
an interesting question is whether mapping patterns can be defined also
for annotating serialization formats other than tables. In particular
the more structured formats such as XML or JSON, or more modern ones
such as Google Protocol Buffers[^21] or Apache Avro[^22], would pose
different problems.

The first approach would be to see if and how such formats can be used
to provide *faithful* representations as defined in section 3. For XML
it has been shown in a similar effort (see [3] and [13]) that one
can derive an XML schema that represents the types and relations of a
well-defined data model in a 1-1 fashion. It is even possible to do so
in an automated manner using code generation and we believe it would be
a useful effort to define such . As long as a well-defined meta-model
exists for the target serialization, and as long as it is rich enough we
believe this will carry over to other serialization formats.

The problem is when one want to annotate existing XML documents that
have not been designed as such. It has as yet not been explore whether a
simple annotation mechanism such as the current one will work there as
well. We believe that *if* the concepts of a data model can be
identified in an informal manner, it may be possible to at least
partially reproduce the target serialization as a *view* on a faithful
serialization. For example one might define an XSLT document that, when
working on a faithfully serialized data model would produce the desired
one.

More useful would likely be the opposite, i.e. a document that would
produce a faithful serialization of a non-faithful one. Such a document
would allow clients to first transform the document to a known
serialization for further processing. This brings one close to the
global-as-view approach, in that the global schema, the data model, is
represented as a view on the source.

More discussion will be needed to address these issues.

[^18]: As discussed earlier, it is allowed that the same object is
    represented multiple times in the same table.

[^19]: I.e. if a super-type is declared as the child in a composition
    relation, this “property” is inherited by its sub-types.

[^20]: E.g. consider SELECT a-b FROM foo. Even if columns a and b have a
    UCD, what is the UCD of their difference?

[^21]: https://developers.google.com/protocol-buffers/

[^22]: https://avro.apache.org/

