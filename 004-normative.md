Patterns for annotating VOTable [NORMATIVE] {#sec:normative}
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

Each subsection contains a concise, formal description of a mapping
pattern, according to a simple “grammar” described in Appendix C.
(**TODO** TBC)

For example the pattern expression

--------------------------------------------------------------------------------
{GROUP G| G $\in$ TABLE & G/VODML/TYPE $\Rightarrow$ **ObjectType**
& G/VODML/ROLE = NULL}

--------------------------------------------------------------------------------

Defines the pattern: GROUP directly under a TABLE, with VODML/TYPE
element identifying an **ObjectType** and without VODML/ROLE element.

For example `GROUP|VODML/TYPE=”vo-dml:Model”] $\Rightarrow$ Model` implies the
GROUP with VODML/TYE set to *vo-dml:Model* is mapped to a Model.

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

[^15]: E.g. when interpreting a GROUP as a certain ObjectType, if one of
 its children is not annotated or identifies a child element that is
 not available on the type, this is an error. Fr each pattern there
 is a set of rules that, if broken, are annotation errors. (**TODO** we
 better strive to make these comprehensive.)

Model
-----

### Model declaration: GROUP in VOTABLE ###

Pattern expression:

--------------------------------------------------------------------------------
{GROUP G| G $in$ VOTABLE & G/VODML/TYPE=”*vo-dml:Model*”}

--------------------------------------------------------------------------------

A GROUP element with VODML element identifying a Model and placed
directly under the root VOTABLE element indicates that the corresponding
VO-DML model is used in VODML associations.

**Restrictions**

  - GROUP element representing a VO-DML Model must exist directly under
    VOTABLE
  - Each such GROUP MUST have a VODML element with
    VODML/TYPE=”vo-dml:Model”, *no* VODML/ROLE
  - MUST have child PARAM element with VODML/ROLE=”vo-dml:Model.uri” and
    `@value` the URI of the VO-DML document representing the model. `@name`
    is irrelevant, `@datatype=”char”` and `@arraysize=”*”`. This annotation
    allows clients to *discover* whether a particular model is used in
    the document, the prefix of its `@utype` in the document, and to
    resolve the Model to its VO-DML description. The URI MUST be a IVORN
    registered in the VO registries.
  - SHOULD have child PARAM element with VODML/ROLE=”vo-dml:Model.url”
    and `@value` the url of the VO-DML document representing the model.
    `@name` is irrelevant, `@datatype=”char”` and `arraysize=”*”`. This is a
    convenient shortcut for the resolution of the URI. Data providers
    should make sure the URL is not broken. Clients should make sure
    that they fall back to resolving the URI if the URL is broken.
  -   MUST have child PARAM element with VODML/ROLE=”vo-dml:Model.name”
    and `@value` the name of the Model, which also works as the
    **vodmlref** prefix (see Section 4). `@name` is irrelevant,
    `@datatype=”char”` and `arraysize=”\*”`.

**Example**

~~~~ {.xml .numberLines}
<GROUP>
  <VODML><TYPE>vo-dml:Model</TYPE></VODML>
  <PARAM name="name" datatype="char" arraysize="*" value="src">
    <VODML><ROLE>vo-dml:Model.name</ROLE></VODML>
  </PARAM>  
  <PARAM name="version" datatype="char" arraysize="*" value="1.0" >
    <VODML><ROLE>vo-dml:Model.version</ROLE></VODML>
  </PARAM>  
  <PARAM name="url" datatype="char" arraysize="*"
      value="https://volute.googlecode.com/svn/trunk/projects/dm/vo-dml/models/sample/Source.vo-dml.xml" >
    <VODML><ROLE>vo-dml:Model.url</ROLE></VODML>
  </PARAM>  
</GROUP>
~~~~

ObjectType
----------

See also section 7.7 on composition for more patterns regarding
serialisation of **ObjectType** instances.

### Singleton root ObjectType: direct GROUP ###

**Pattern expression**:

--------------------------------------------------------------------------------
{GROUP G | G $subset$ RESOURCE & G $\not\subset$ TABLE & G $\not\subset$
  GROUP[VODML] & G/VODML/TYPE $\Rightarrow$ **ObjectType**
  & G/VODML/ROLE = NULL}

--------------------------------------------------------------------------------


Singleton **ObjctType** represented by “direct GROUP”. Not in a TABLE,
in a RESOURCE. MUST have VODML/TYPE, and no VODML/ROLE

Notice that there is a formal difference in VO-DML between DataType and
ObjectType. From a practical point of view, these differences can be
summarized as follows:

  * DataType does not inherit the vo-dml:ObjectType.ID attribute

  * You cannot Reference a DataType instance

  * You cannot have a Collection of DataType instances

### Root ObjectType instances in Table I: not identified ###

Pattern expression:

--------------------------------------------------------------------------------
{GROUP G | G $\subset$ TABLE & G $\not\subset$ GROUP[VODML] & G/VODML/TYPE
  $\Rightarrow$ **ObjectType** & G/VODML/ROLE = NULL}

--------------------------------------------------------------------------------

### Root ObjectType instances in Table II: identified ###

Pattern expression:

--------------------------------------------------------------------------------
{GROUP G, GROUP I, FIELDref F | I $\in$ G & G $\subset$ TABLE & G $\not\subset$
   GROUP[VODML] & G/VODML/TYPE $\Rightarrow$ **ObjectType** & G/VODML/ROLE =
   NULL & I/VODML/ROLE = “*vo-dml:ObjectTypeInstance.ID*”
   & F $\in$ I & F/VODML/ROLE = “*vodml:Identifier.field*”}

--------------------------------------------------------------------------------

When the VOTable document stores the result of an ADQL query it may be
common to find multiple copies of the same **ObjectType** instance
represented in a table. [TBD example of query joining parent to child
replicating parent information.] If one wants to indicate this is the
case one MUST use an Identifier pattern. Here one maps one or more
columns to fields in the *vo-dml:ObjectTypeInstance.ID* property that
every **ObjectType** instance inherits form its implicit super-type,
*vo-dml:ObjectTypeInstance*. TBC

This same identifier also allows the creation of Object-Relational
Mapping patterns for references, this is described in section 7.6.3.

If such an explicitidentifier mapping does not exist, as is the case in
7.2.27.3.2, there is formally no way for clients to infer that instances
in different rows are the same, even if all their properties have the
same value. After all, **ObjectTypes** are *not* identified by the
values of their properties, but *only* by an identifier (**TODO* TBD ref to
VO-DML definition).

**Restrictions/conditions/rules**:

  * Clients SHOULD consider it an error if they encounter two objects
    with identical IDs, that otherwise have distinct values for the
    same properties.

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

### Root DataType as singleton: direct GROUP ###

Pattern expression:

--------------------------------------------------------------------------------
{GROUP G | G $\subset$ RESOURCE & G $\not\subset$ TABLE & G $\not\subset$
  GROUP[VODML]& G/VODML/TYPE $\Rightarrow$
**DataType** & G/VODML/ROLE = NULL}

--------------------------------------------------------------------------------

  * Singleton **DataType** value, represented by “direct GROUP”. Not in
    a TABLE, in a RESOURCE. MUST have VODML/TYPE, and MUST NOT
    have VODML/ROLE.

Restrictions:

  * A GROUP in a RESOURCE cannot have FIELDrefs, only PARAMs, PARAMrefs
    and GROUPs. The GROUP represents therefore a single instance of the
    DataType directly as defined in 8.2, with the PARAMs etc. providing
    the values of the components of the DataType.

Comments:

The possible role this instance plays in the VOTable document must have
been defined outside of any mapping to a data model. After all, in
contrast to instances of ObjectTypes, the existence of instances of
value types need not be explicitly stated (see the description of value
types in [1]). An example could be a VOTable containing the result of
a simple cone search. A RESOURCE in that document might contain a GROUP
representing the position of the cone, which may be mapped through its
`@utype` to a DataType “src:source.SkyCoordinate” (if such a type existed:
our sample model contains a type like it).

Example:

**A GROUP defined as a child of RESOURCE representing a singleton
datatype instance **

~~~~ {.xml .numberLines}
<RESOURCE>
<GROUP>
  <VODML><TYPE>src:source.SkyCoordinate</TYPE></VODML>
  <INFO value="The center coordinate of the simple cone search"/>
  <PARAM name="ra" value="123.00000" ...>
    <VODML><ROLE>src:source.SkyCoordinate.longitude</ROLE></VODML>
  </PARAM>
  <PARAM name="dec" value="-2.10000" ...>
    <VODML><ROLE>src:source.SkyCoordinate.longitude</ROLE></VODML>
  </PARAM>
  <GROUP ref="_icrs">
    <VODML><ROLE>src:SkyCoordinate.frame</ROLE></VODML>
  </GROUP>
</GROUP>
~~~~
Example:

**A GROUP defined as a child of a GROUP without annotation**

Note that the GROUP representing the singleton **DataType** is allowed
to be contained in another GROUP, as long as that GROUP does not declare
a VODML mapping. There are some restrictions. NONE of its ancestor
GROUPs MAY be mapped to a data model element. This would conflict with
rules of mapping such an ancestor GROUP that state that children of such
GROUPs MUST be mapped to **Roles** of the structured type it represents.
Whether the GROUP represents the **DataType** instance directly or
indirectly is independent of this embedding in a parent GROUP. That is
purely dependent on whether the GROUP has an ancestor TABLE or not.

An example of this pattern is the case of an assumed DAL protocol, say
SCS that defines *implicitly* some data structure that has components
that may be represented by elements from a data model as I the following
example.

Example

A GROUP representing parameters of a Simple Cone Search with components
mapped to data model elements.

~~~~ {.xml .numberLines}
<RESOURCE>
<GROUP>
 <INFO value="The simple cone search request"/>
 <PARAM name="serchRadius" value="3" unit="arcsec" .../>
 <GROUP name="center">
  <VODML><TYPE>src:source.SkyCoordinate</TYPE></VODML>
  <INFO value="The center coordinate of the simple cone search"/>
  <PARAM name="ra" value="123.00000" ...>
    <VODML><ROLE>src:source.SkyCoordinate.longitude</ROLE></VODML>
  </PARAM>
  <PARAM name="dec" value="-2.10000" ...>
    <VODML><ROLE>src:source.SkyCoordinate.longitude</ROLE></VODML>
  </PARAM>
  <GROUP ref="_icrs">
    <VODML><ROLE>src:SkyCoordinate.frame</ROLE></VODML>
  </GROUP>
 </GROUP>
~~~~

### Root DataType values in TABLE records: indirect GROUP ###

Pattern expression:

--------------------------------------------------------------------------------
{GROUP G | G $\subset$ TABLE & G $\not\subset$ GROUP[VODML] & G/VODML/TYPE
$\Rightarrow$ **DataType** & G/VODML/ROLE = NULL}

--------------------------------------------------------------------------------

Restrictions:

  * ...

Comments:

A GROUP defined on a TABLE, annotated with a **DataType**, represents a
**DataType** instance indirectly.

Example

~~~~ {.xml .numberLines}
<TABLE>
  <GROUP>
    <VODML><TYPE>src:source.SkyCoordinate</TYPE></VODML>
    <FIELDref ref="_ra">
      <VODML><ROLE>bSTC:SkyCoordinate.longitude</ROLE></VODML>
    </FIELDref>
    <FIELDref ref="_dec">
      <VODML><ROLE>bSTC:SkyCoordinate.latitude</ROLE></VODML>
    </FIELDref>
    <GROUP ref="_icrs">
      <VODML><ROLE>bSTC:SkyCoordinate.frame</ROLE></VODML>
    </GROUP>
  </GROUP>
...
<FIELD name="ra" ID="_ra" unit="deg">
<DESCRIPTION>right ascension (J2000 decimal deg)</DESCRIPTION>
</FIELD>
<FIELD name="dec" ID="_dec"  unit="deg">
<DESCRIPTION>declination (J2000 decimal deg)</DESCRIPTION>
</FIELD>
...
~~~~

### DataType as Attribute: GROUP in GROUP ###

Pattern Expression:

--------------------------------------------------------------------------------
{GROUP A, GROUP G | A $\in$ G & A/VODML/ROLE $\Rightarrow$ **Attribute**
  & G/VODML $\Rightarrow$ **StructuredType**}

--------------------------------------------------------------------------------

  * A GROUP representing a structured type can have Attributes that are
    of a structured type as well.
  * However, this time the nested GROUP has a VODML/ROLE identifying the
    Attribute that the GROUP represents.

Restrictions:

  * The **Attribute** identified by GROUP A’s VODML/ROLE MUST be
    available to the structured type represented by the containing
    GROUP (G).
  * GROUP A SHOULD also declare the actual structured type it
    represents, through its VODML/TYPE. This type MUST be a valid type
    for the **Attribute**, i.e. the **Attribute’s** declared type or one
    of its subtypes. If the type in the serialization is a subtype, the
    VODML/TYPE MUST declare that.

**Comments:**

Example

~~~~ {.xml .numberLines}
<GROUP>
  <VODML><TYPE>src:source.Source</TYPE></VODML>
...
  <GROUP>
    <VODML>
      <ROLE>src:source.Source.position</ROLE>
      <TYPE>src:source.SkyCoordinate</TYPE>
    </VODML>
    <FIELDref ref="_ra">
      <VODML><ROLE>src:source.SkyCoordinate.longitude</ROLE></VODML>
    </FIELDref>
    <FIELDref ref="_dec">
      <VODML><ROLE>src:source.SkyCoordinate.latitude</ROLE></VODML>
    </FIELDref>
    <GROUP ref="_icrs">
      <VODML><ROLE>src:source.SkyCoordinate.frame</ROLE></VODML>
    </GROUP>
  </GROUP>
...
</GROUP>
~~~~

PrimitiveType and Enumeration
-----------------------------

**PrimitiveTypes** and **Enumerations** represent atomic values. They
appear in serializations of data models predominantly in their role as
attributes defined on structured types (**ObjectType** or **DataType**)
and this specification focuses on that role. There is only limited extra
information in identifying an individual FIELD or PARAM as representing
some primitive type or enumeration from a data model, without indicating
the role the FIELD or PARAM plays in some complex type. VO-DML assumes
the existence of a standard data model (**TODO** *ivoa,* see [1] TBD refer to
actual model) defining primitive types such as *ivoa:integer*,
*ivoa:string* etc. But there is no new semantics implied by these types
beyond the ones their counterparts in the VOTable schema carry[^16].

[^16]: *ivoa* types that do not exist in VOTable may there be
    represented using appropriate xtypes, e.g. adql:timestamp for
    *ivoa:datetime.*

### Singleton primitive value as PARAM

Pattern expression:

--------------------------------------------------------------------------------
{PARAM P| P $\not\in$ G[VODML] & P/VODML/ROLE = NULL & G/VODML/TYPE $\Rightarrow$
**PrimitiveType**|**Enumeration**}

--------------------------------------------------------------------------------

This indicates a PARAM that is *not* contained in a VODML-annotated
GROUP, has a VODML/TYPE identifying an atomic type and no VODML/ROLE.

**TODO** TBD do we want to support this? For if so, what about FIELDs?

### EnumLiteral as OPTION

Pattern expression:

--------------------------------------------------------------------------------
{FIELDref FR, FIELD F, OPTION O| `FR/@ref`=`F/@ID` & O $\subset$ F
& FR/VODML $\Rightarrow$ Enumeration & FR/VODML/OPTION/NAME = `O/@value`
& FR/VODML/OPTION/LITERAL $\Rightarrow$ EnumLiteral}

--------------------------------------------------------------------------------

**TODO**

  * VODML/OPTION
  * value to literal
  * value to SKOS concept

### SKOSConcept as OPTION

**TODO**

Attributes
----------

### Attribute values in FIELD: FIELDref in GROUP in TABLE

Pattern expression:

--------------------------------------------------------------------------------
{FIELDref F, GROUP G| F $\in$ G & G $\subset$ TABLE & F/VODML/ROLE $\Rightarrow$
**Attribute ** & G/VODML $\Rightarrow$ **StructuredType** }

--------------------------------------------------------------------------------

  * A FIELDref contained in a GROUP represents an Attribute serialized
    by the FIELD it refers to.

Restrictions

  * The **Attribute** identified by the FIELDref/VODML/ROLE MUST be
    available to the structured type represented by the containing
    GROUP (G).

  * FIELDref MUST reference a FIELD in the TABLE that contains its
    parent GROUP.

  * The **Attribute** MUST have an atomic datatype, or a **DataType**
    from the set of special cases described in be section 7.9.

  * The `@datatype` of the referenced FIELD SHOULD be compatible with the
    declared **datatype** of the **Attribute**. E.g. an integer
    (*ivoa:integer*) **Attribute** SHOULD be represented by VOTable’s
    int or long.

Example

See example in 6.2.

### Attribute to PARAM in GROUP

Pattern Expression:

--------------------------------------------------------------------------------
{PARAM P, GROUP G | P $\in$ G & G/VODML/ROLE $\Rightarrow$ **Attribute** &
G/VODML $\Rightarrow$ **StructuredType}**

--------------------------------------------------------------------------------

  * When defined inside of a GROUP that represents a structured type, a
    PARAM MAY represent an Attribute of the type itself. The Attribute
    is declared by the `@utype`.

Restrictions

  * The **Attribute**’s **datatype** MUST identify an atomic type, i.e.
    a **PrimitiveType** or **Enumeration**, or a **DataType** from the
    set of special cases described in be section 7.9.
  * The PARAM `@datatype` SHOULD be a compatible, valid serialization type
    for the **datatype** of the **Attribute**.

Example

**TODO** See example in TBD

### Attribute to PARAMref in GROUP

Pattern expression:

--------------------------------------------------------------------------------
{PARAMref P, GROUP G | P $\in$ G & P/VODML/ROLE $\Rightarrow$ Attribute &
GROUP/VODML $\Rightarrow$ **StructuredType**]

--------------------------------------------------------------------------------

Inside a GROUP a PARAMref identifies a PARAM defined inside the same
RESOURCE or TABLE where the GROUP is defined. Using its VODML/ROLEthe
PARAMref can annotate the PARAM as holding the value of an
**Attribute**.

Restrictions

  * The **Attribute** must be available to the structured type
    represented by the containing GROUP G.

  * The PARAM referencd by the PARAMref MUST obey the restrictions
    defined for the annotation of a PARAM by the **Attribute’s** type in
    section 7.5.2.

Example

See example in 6.2

### DataType as Attribute: “GROUPref” in GROUP

A PARAMref identifyin some PARAMin a GROUP can represent an
**Attribute** with an atomic type on a **StructuredType**, so we can to
a GROUP representing a structured **DataType** to represent
**Attributes**.

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
its attributes assume. See the VO-DML document [1] for more
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
either represented as a singleton object represented by a direct Group
(7.2.1) or as an object in a table (7.2.2 – 7.2.3).

The specification also allows for the case where the target object is
*not* represented in the same VOTable document as the referrer. It may
exist in another VOTable, or in some alternative, possibly standardized
serialized form. An example of the latter could be an XML document
following a standardized XML schema explicitly defined to represent the
data model, or possibly a location in some publicly accessible database
with a relational schema mapped to the model in a standardized
Object-Relational manner.

This “remote reference pattern” is potentially very useful. By locating
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
[8].

In all patterns below the actual **Reference** is represented by a GROUP
contained by the GROUP representing the referrer, and with structure
that allows the identification of the target object. We do this for
uniformity: there may be cases where a single reference PARAM for
example might be sufficient, just as an IDREF can identify a target with
an ID in XML Schema, but more generic cases require more structure. And
in fact our solution for this simplest case, using what we will refer to
as a “GROUPref” (including the quotes), is more lightweight even that a
solution with a PARAM as will be shown next.

### Reference to singleton Object in direct GROUP

Pattern Expression:

--------------------------------------------------------------------------------
{GROUP S, GROUP R, GROUP T| S/VODML $\Rightarrow$ **StructuredType** & R $\in$
S & `R/@ref` = `T@ID` & T/VODML $\Rightarrow$ **ObjectType** & R/VODML/ROLE
$\Rightarrow$ **Reference** & T $\not\in$ TABLE & T $\subset$ RESOURCE [&
R/VODML/TYPE = *“vo-dml:GROUPref”*]}

--------------------------------------------------------------------------------

This pattern represents a source GROUP R (the *reference*), contained in
a GROUP S (the *referrer*) representing a **StructuredType**,
referencing a *target* GROUP T, representing a singleton **ObjectType**
instance. The reference R uses its `@ref` attribute to identify the target
GROUP T which is identified by a unique `@ID`. The GROUP R has no elements
apart from the VODML element which MUST contain a ROLE identifying a
particular **reference**; it MAY explicitly identify its VODML/TYPE as
being *vo-dml:GROUPref*.

We refer to this pattern represented by GROUP R as a “GROUPref”, it is a
reference to an identified GROUP. But we use the quotes as we do not
need an explicit GROUPref element in the sense of VOTABLE’s FIELDref and
PARAMref to represent this concept. It can also be used in other
patterns as shown in the next section.

Restrictions

  * The **Reference** identified by R/VODML/ROLE MUST be *available* on
    the **StructuredType** identified by O/VODML.

  * **OjectType** identified by T/VODML MUST be compatible with the
    **datatype** of the **Reference**, i.e. must either be the same type
    or one of its subtypes.

  * The GROUP R representing the **Reference** must not have any child
    elements apart from the VODML element.

Example

~~~~ {.xml .numberLines}
<GROUP ID="_2massJ">
  <VODML><TYPE>phot:PhotometryFilter</TYPE></VODML>
  <PARAM name="name" datatype="char" value="2mass:J">
    <VODML><ROLE>phot:PhotometryFilter.name</ROLE></VODML>
  </PARAM>
...
</GROUP>
...
<GROUP>
  <VODML><TYPE>src:source/Source</TYPE></VODML>
  <GROUP>
    <VODML>
      <ROLE>src:source/Source.luminosity</ROLE>
      <TYPE>src:source/LuminosityMeasurement</TYPE>
    </VODML>
  	<FIELDref ref="_magJ" >
      <VODML><ROLE>src:source.LuminosityMeasurement.value</ROLE></VODML>
    </FIELDref>
   	<FIELDref ref="_errJ" >
      <VODML><ROLE>src:source.LuminosityMeasurement.error</ROLE></VODML>
    </FIELDref>
   	<GROUP  ref="_2massJ">
      <VODML><ROLE>src:source/LuminosityMeasurement.filter</ROLE></VODML>
  </GROUP>
...
~~~~

### Reference to object(s) in a TABLE I: objects in same row

Pattern Expression:

--------------------------------------------------------------------------------
{GROUP S, GROUP R, GROUP T, TABLE TB| S/VODML $\Rightarrow$ **StructuredType** &
R $\in$ S & `R/@ref` = `T@ID` & TB/VODML $\Rightarrow$ **ObjectType** & R/VODML/ROLE
$\Rightarrow$ **Reference** & S $\subset$ TB & T $\subset$ TB [& R/VODML/TYPE =
*“vo-dml:GROUPref”*] }

--------------------------------------------------------------------------------

This pattern represents a “reference GROUP” R, contained in “source
GROUP” S representing a structured type, referencing a “target GROUP T”.
Both S and T are contained in the same TABLE TB and a “GROUPref” is used
to represent the reference.

This pattern indicates that each row in the TABLE TB contains (at least)
two objects, represented by GROUPS S and T, and that S references T
through the reference represented by GROUP R. R has no elements apart
from the VODML element; it MUST identify the actual **reference** in its
VODML/ROLE element; it SHOULD explicitly identify its VODML/TYPE as
being v*o-dml:GROUPref*; it MUST identify which reference it represents
through its VODML/ROLE.

**Example**:

TBD

### Reference to object(s) in a TABLE II: objects in different rows, possibly different TABLE

Pattern Expression:

--------------------------------------------------------------------------------
{GROUP S, GROUP R, GROUP T, GROUP PK, GROUP FK | S $\subset$ TABLE & T $\subset$
TABLE & S/VODML $\Rightarrow$ **StructuredType** & T/VODML $\Rightarrow$
**ObjectType** & PK $\in$ T & PK/VODML/ROLE=*”vo-dml:ObjectTypeInstance.ID”* & R
$\in$ S & R/VODML/ROLE $\Rightarrow$ **reference** & R/VODML/TYPE =
*“vo-dml:ORMReference”* & FK $\in$ R &
FK/VODML/ROLE=*”vo-dml:ObjectTypeInstance.ID”* [& `R/@ref` = `T@ID`] }

--------------------------------------------------------------------------------

This is the most complex pattern in this specification. It is the
analogue of a foreign key constraint in a relational database. The
pattern represents a “reference GROUP” R, contained in “source GROUP” S
representing a structured type, referencing a “target GROUP T”. S and T
may be contained in the same TABLE, but in general this need not be the
case.

This pattern indicates that each row in the TABLE annotated by GROUP S
contains a corresponding structured object, which has a reference to an
object represented by GROUP T, located in some row in its TABLE. The
reference is represented by the GROUP R. The GROUP R MUST identify which
reference it represents through its VODML/ROLE. R MUST explicitly
identify its VODML/TYPE as being *vo-dml:ORMReference*.

This latter requirement indicates that this pattern represents an
object-relational mapping pattern. It is clear that in contrast to the
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

Example:

TBD

### Reference to Object in external data store

**Pattern Expression**:

--------------------------------------------------------------------------------
{GROUP O, GROUP S, PARAM P| O/VODML $\Rightarrow$ **StructuredType** & S $\in$ O &
`S/@ref` = NULL & S/VODML/ROLE $\Rightarrow$ **Reference** & S/VODML/TYPE =
*“vo-dml:RemoteReference”* & P $\in$ S & (P/VODML/ROLE =
“*vo-dml:RemoteReference.ivoid”* | P/VODML/ROLE =
“*vo-dml:RemoteReference.uri”*) }

--------------------------------------------------------------------------------

This pattern identifies a reference to an object serialized elsewhere.
Such an instance MUST be identified using an IVOA Identifier [12] if
it is stored in an IVOA Registry. However, a URL might be provided as a
convenient hook for the client. The reference is again represented by a
GROUP, but now with PARAM whose `@value` is an IVOA Identifier. As
alternative one can use a URI, but exactly *how* that should be resolved
is outside of the scope of this document.

**Example:**

For example the previous example from the previous sub-section could be
written as follows:

``` {.xml .numberLines}
<GROUP>
  <VODML><TYPE>src:source/Source</TYPE></VODML>
  <GROUP>
    <VODML>
      <ROLE>src:source/Source.luminosity</ROLE>
      <TYPE>src:source/LuminosityMeasurement</TYPE>
    </VODML>
  	<FIELDref ref="_magJ" >
      <VODML><ROLE>src:source.LuminosityMeasurement.value</ROLE></VODML>
    </FIELDref>
   	<FIELDref ref="_errJ"  >
      <VODML><ROLE>src:source.LuminosityMeasurement.error</ROLE></VODML>
    </FIELDref>
   	<GROUP >
      <VODML>
        <ROLE>src:source/LuminosityMeasurement.filter</ROLE>
        <TYPE>vo-dml:RemoteReference</TYPE>
      </VODML>
      <PARAM name="ivoid" datatype="char" arraysize="*"
             value="ivo://ivoa.registry/dm/phot/instances/2mass.J">
        <VODML><ROLE>vo-dml:RemoteReference.ivoid</ROLE></VODML>
      </PARAM>
      <PARAM name="ivoid" datatype="char" arraysize="*"
             value="http://ivoa.registry/dm/phot/instances/2mass#J">
        <VODML><ROLE>vo-dml:RemoteReference.uri</ROLE></VODML>
      </PARAM>
  </GROUP>
...
```

Here it is assumed that the DM working group maintains a set of data
model instance documents for various instruments and stores these in
some registry. [TBD check how realistic the ivoId is, add URL.]

Composition
-----------

A **Composition** is defined in VO-DML as a “whole-parts” relation
between parent and child **ObjectTypes**. The child can be interpreted
as defining a *part* of the parent, the parent is “composed of” the
child objects. Generally a parent can have a collection of 0..N children
of a certain type, where N may be unbounded, but the multiplicity can be
more constrained. The relation between the child and its parent is much
stronger than the relation between a source and a target in a
**Reference** relation. In particular the child’s life-cycle is tightly
coupled to its parent: a child only has one parent and when when the
parent is deleted, so is the child. Because of this also the
serialization of the relationship may in many cases be more direct than
the indirect reference mapping patterns of the previous section.

For example, in faithful XML serializations of a data model one will
generally map child objects as elements directly contained by the parent
element. It is possible to do so in VOTable as well, having a child
object represented by a GROUP that is contained within the GROUP
representing the parent. This pattern is the most natural for this
relationship because a collection element is really to be considered as
a *part* of the parent object.

This containment of GROUPs can be used for singleton patterns in an
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
7.6.3 in combination with the implicit *vo-dml:ObjectType.container*
**Reference**.

### Composition I: child GROUP inside parent GROUP

**Pattern Expression**:

--------------------------------------------------------------------------------
{GROUP P, GROUP C | C $\in$ P & P/VODML $\Rightarrow$ **ObjectType**
  & C/VODML/ROLE $\Rightarrow$ **Composition**
[& C/VODML/TYPE $\Rightarrow$ **ObjectType**] }

--------------------------------------------------------------------------------

GROUP C represents a child object in a **collection** on some
**ObjectTupe** identified by its VODML/ROLE, and of **datatype**
possibly identified by its VODML/TYPE. In this pattern C is contained
inside of a GROUP P that represents an **ObjectType** that is a valid
**Container** of the **Collection**.

No statement is made about the location of the parent GROUP. If it is
inside a RESOURCE, *not* in a TABLE, both parent and child represent
singleton objects. If it is inside a TABLE, each row stores both parent
and child objects.

Note also that there may be multiple child GROUPs identifying the same
composition relation in the same parent GROUP.

Example

See example in 6.4

### Composition II: object-relational reference from child to parent, both in TABLEs

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

Pattern Expression:

--------------------------------------------------------------------------------
{GROUP CH, GROUP R, GROUP P, GROUP PK, GROUP FK | CH $\subset$ TABLE & P
$\subset$ TABLE & CH/VODML $\Rightarrow$ **ObjectType** & P/VODML $\Rightarrow$
**ObjectType** & PK $\in$ P & PK/VODML/ROLE=”*vo-dml:ObjectTypeInstance.ID*” & R
$\in$ CH & R/VODML/ROLE = “*vo-dml:ObjectTypeInstance.container*” & R/VODML/TYPE
= “*vo-dml:ORMReference*” & FK $\in$ R &
FK/VODML/ROLE=”*vo-dml:ObjectTypeInstance.ID*” [& `CH/@ref` = `P@ID`] }

--------------------------------------------------------------------------------

The GROUP CH represents a collection of child objects that are contained
in parent objects represented by GROUP P. CH and P are generally stored
in different tables. CH references P following the ORM reference pattern
described in section 7.6.3 above. The only constraint is that the
VODML/ROLE on the GROUP representing the reference MUST be
*vo-dml:ObjectTypeInstance.container*.

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

**Example**:

TBD

Extends, inheritance (**TODO** needs work!)
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
datatype. Supporting this type of polymorphism at the serialization
level is one main motivation for adding `<TYPE>` to the
`<VODML>` element. See Appendix B for a discussion how different
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

Pattern Expression:

(**TODO** Pattern missing?)

In this case the $\Uparrow$ symbol means that the leftmost ObjectType *extends*
the rightmost ObjectType, adding some Attributes to it. Attributes can
be structured or unstructured, direct or indirect, recursively. This
mapping pattern leverages the patterns introduced in the previous
sections.

The extending type inherits all of the parent type’s Attributes and
their VO-DML-refs (including the prefixes), and adds its own Attributes
and their VO-DML-refs (including the prefixes).

The extending type also inherits all of the parent’s ancestors,
recursively. Thus, the serialization of the child type MUST include all
the ancestors’ declarations of *vo-dml:Instance.type*. This makes it
possible for clients of any ancestor to recognize the instance and to
correctly apply polymorphism.

It is possible that types in a Model extend types in the same Model. In
this case one can already tell them apart from the a priori knowledge of
a Model, or by parsing the VO-DML description of the Model, and
**vodml-id**-s cannot clash by definition. So, the pattern described
above, where the child representation carries over the
vo-dml:Instance.type from its ancestors only applies to inter-Model
extensions, UNLESS the extended type is abstract. In this case, clients
may want to look for the abstract type before interpreting the actual
concrete type. For instance, a client may look for all the
“*src:source.SkyError*” instances before interpreting them as a circle
error, an elliple error, and so on.

For example, assuming that the ExtentedSource Model extends the Source
Model (Attribute source.Source.redshift), and that the
ExtendedExtendedSource Model extends the ExtendedSource Model (Attribute
source.Source.profile), a serialization will look like this:

**Example**

``` {.xml .numberLines hl_lines="1 2"}
<TABLE>
<GROUP>
  <VODML>
    <TYPE>xxsrc:Source</TYPE>
    <TYPE>xsrc:Source</TYPE>
    <TYPE>src:source.Source</TYPE>
  </VODML>
  <FIELDref ref="_designation">
    <VODML><ROLE>vo-dml:ObjectTypeInstance.ID</ROLE></VODML>
  </FIELDref>
  <FIELDref ref="_designation">
    <VODML><ROLE>src:source.Source.name</ROLE></VODML>
  </FIELDref>
  <FIELDref ref="_z">
    <VODML><ROLE>xsrc:source.Source.redshift</ROLE></VODML>
  </FIELDref>
  <FIELDref ref="_profile">
    <VODML><ROLE>xxsrc:source.Source.profile</ROLE></VODML>
  </FIELDref>
  <PARAM name="type" value="galaxy">
    <VODML><ROLE>src:source.Source.classification</ROLE></VODML>
  </PARAM>
  <GROUP>
    <VODML>
      <TYPE>src:source.SkyCoordinate</TYPE>
      <ROLE>src:source.Source.position</ROLE>
    </VODML>
    <FIELDref ref="_ra">
      <VODML><ROLE>src:source.SkyCoordinate.longitude</ROLE></VODMLL>
    </FIELDref>
    <FIELDref ref="_dec">
    <VODML><ROLE>src:source.SkyCoordinate.latitude</ROLE></VODMLL>
    </FIELDref>
    <GROUP ref="_icrs">
      <VODML><ROLE>src:source.SkyCoordinate.frame</ROLE></VODMLL>
    </GROUP>
  </GROUP>
</GROUP>
<FIELD name="designation" ID="_designation" .../>
<FIELD name="ra" ID="_ra" unit="deg" .../>
<FIELD name="dec" ID="_dec"  unit="deg" .../>
<FIELD name="z" ID="_z" .../>
<FIELD name="profile" ID="_profile" .../>
<TR><TD>08120809-0206132</TD><TD>123.033734</TD><TD>-2.103671</TD><TD>0.123</TD><TD>devaucoulers</TD></TR>
...
</TABLE>
```

In this example we have assumed that the VO-DML preamble declared the
xsrc prefix for the ExtendedSource Model and the xxsrc prefix for the
ExtendedExtendedSource Model.

Value, Unit, UCD
----------------

Some DataTypes may have attribute names that can be naturally mapped to
the VOTable PARAM and FIELD attributes. In this case, the following
rules apply:

  * the value of the DataType's 'value' attribute is stored in the
    `@value` attribute of the PARAM, or the TD corresponding to the
    annotated FIELD.
  * the value of the DataType’s 'unit' attribute is represented by the
    `@unit` attribute of the PARAM, or the annotated FIELD.
  * the value of the 'ucd' attribute is mapped to the `@ucd` attribute of
    the annotated FIELD or PARAM

(**TODO** Extend this pattern to datatype, arraysize? ivoa:Quantity DM.)

Notable absences {#sec:absences}
================

The VOTable schema allows for redundancy in meta-data assignment. For
example it allows assigning a UCD or UTYPE to FIELDrefs, but also to the
FIELD it references. How is one to interpret or use this? Our approach
is to try to avoid this redundancy.

The design laid out in the previous sections focuses UTYPE assignments
on the GROUP element and its components. The main reason is that in all
but the most simplistic use cases we will not be able to void the use of
GROUPs, and that at the same time they provide all functionality (and
more) that TABLE and FIELD could provide. Choosing this approach implies
client coders do not need to take the possibly conflicting assignments
into account, they only need consider GROUPs.

Here we list a few possible assignments that we avoid, though they might
seem valid.

Atomic Types: support for custom and legacy UTYPEs
--------------------------------------------------

It is worth stressing explicitly that some VOTable elements are not
covered by this specification (e.g. TABLE, RESOURCE, INFO, FIELD, and
standalone PARAM). Also, according to this specification some elements
will be ignored in the de-serialization of DM instances if their UTYPEs
do not have a prefix declared in the VO-DML preamble.

This is intentional, and its purpose is to achieve full backward
compatibility of this standard with the current non-standardized usages,
while enabling new, complex Data Models to be effectively serialized in
a standardized way.

Atomic Types are types referenced by `@utype` attributes of FIELDs and
standalone PARAMs (i.e. PARAMs not included in GROUPs). Usually these
UTYPEs are path-like strings pointing to some implicit and unspecified
meta-model in an atomic fashion. Such UTYPEs can happily coexist with
UTYPEs used according to this specification.

Not only this means that this standard does not break any of the
existing standards. Not only this means that it enables customized use
of the `@utype` attribute in local implementations. This also means that
in order to make a legacy VOTable file compliant with the new specs,
Data Providers will only need, if willing to do so, to add some GROUP
definitions to its header. Old clients of that file will still be able
to parse them, while new generation clients will be able to perform
their more advanced usage of the new specification.

Packages
--------

No use cases require the serialization of Package instances in VOTable.
Package names are encoded in the UTYPE syntax of VO-DML for avoiding
name clashes when two classes in different packages of the same model
have the same name (other than that, UTYPEs are effectively opawue). The
use of ‘.’ as the separator for the Package->Type relationship (e.g.
source.Source) might cause name clashes when a package contains a Type
and a Package with the same name. However, packages cannot be directly
referenced by UTYPEs, so this is not an issue.

ObjectType to TABLE
-------------------

Pattern expression:

--------------------------------------------------------------------------------
{ TABLE $\Rightarrow$ **ObjectType** }

--------------------------------------------------------------------------------

There might be some cases where a TABLE could be said to represent a
structured type completely, and where the TABLE could be annotated with
a `<VODML>` element to indicate this. However in probably most cases
only part of the TABLE will correspond to the type, or it will contain
instances of multiple types stored inside a single row (e.g. photometry
catalogs).

In all of these cases one can (and MUST) use one or more GROUP elements
contained by the TABLE to make the precise assignment. Hence we extend
this to the rare case where the TABLE might have been annotated to
simplify the spec. By leaving this mapping pattern out we do not lose
any information content. Also, this way client code only has to deal
with GROUPs, with no need to inspect the TABLE.

Attribute to FIELD|PARAM in TABLE
---------------------------------

Pattern expression:

--------------------------------------------------------------------------------
[Field $\Rightarrow$ Attribute] $\in$ [TABLE $\Rightarrow$ ObjectType]

--------------------------------------------------------------------------------

Pattern expression:

--------------------------------------------------------------------------------
[PARAM $\Rightarrow$ Attribute] $\in$ [TABLE $\Rightarrow$ ObjectType]

--------------------------------------------------------------------------------

Assigning an Attribute to a FIELD would only make sense in the context
of a structured container, which can only be TABLE. But as we propose
not to use TABLE to represent a DM element directly, consequently FIELD
need not have this capability. We use FIELDref for that.

For the same reason that makes us avoid the assignment of Attribute to
FIELD, we avoid assigning an attribute to a standalone PARAM, i.e. a
PARAM *that is directly contained in a TABLE or RESOURCE*. The context
(TABLE) is not used to indicate the type containing the Attribute. For
this element a PARAMref or a PARAM inside a GROUP is to be used. This
again enables backward compatibility.

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
