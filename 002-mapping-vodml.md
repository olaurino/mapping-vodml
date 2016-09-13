Mapping with the `VODML` element.
=================================

Here we discuss the technicalities of *how* to annotate VOTable with
VO-DML metadata. The later sections will provide details on the implied
semantics of these annotations and rules on their application.

VOTable 1.2 introduced the `@utype` attribute, which was intended to
represent "pointers into a data model". A precise and formal definition
on how this “pointing” was to be achieved and a description of its
meaning was missing though.

**TODO** Missing Votable reference

First, a formal definition of the target of the pointers was missing. To
solve this, data models were usually accompanied by a list of “utypes”
(**TODO** [TBD refer to STC, Characterization, Spectrum]), and these could be
used as values for said `@utype`, be it in VOTable or for example in the
Table Access Protocol metadata. These were not linked in any formal,
machine readable way to the underlying data model.

Basically it means that the data model is represented solely by a list
of attributes, which does not do justice to the complexity of data
models describing complex data products like Data Cubes or the
provenance of Simulations. These contain complex object hierarchies
organized in graphs with various types of relations between individual
objects. It also proved difficult to express the relationship among
different, but overlapping, data models, with much discussion centred on
the question how to reuse utypes from one model in the definition of
another.

The approach is basically not much more than another vocabulary, similar
to UCDs [@ucd], or SKOS vocabularies [@skos], obtained by different means.
Efforts were made to provide some structure to these values that might
provide some hints of their location in a model, but there was no formal
mechanism on how to derive that structure and it was unclear whether it
could truly represent the richness of the existing and future data
models. In particular there was no standard defined how this could be
achieved and no common usage patterns were discovered [@usages].

As described above (**TODO** TBD check, true?) VO-DML has solved the problem
that there was no formal target for these pointers in the data model
itself and formally defines how models can be reused in the definition
of other, dependent models. Precisely *how* to use these pointers in a
VOTable to provide a complete annotation useful for interoperability
requires more work though. It requires restrictions on which VOTable
elements can point to which concepts and how they are organized.

The current specification provides such a definition. It shows how data
publishers can identify also the more complex data model elements such
as structured types and relationships inside some published data source,
be it a VOTable or relational database published through the TAP
protocol. It notes that in fact the meta-model defined by the VOTable
schema is perfectly suited for supporting such a *mapping* in an almost
1-1 and completely logical fashion.

This specification defines various *mapping patterns* from VOTable to
VO-DML. Such a pattern identifies a VOTable element with a VO-DML
element. The VO-DML element is said to be *represented* by the VOTable
element. The mapping pattern indicates that instances of identified
VO-DML types are present in the VOTable. These may be atomic *values*
(instances of VO-DML ValueTypes [@vodml]), represented by `PARAM@value` or by
cells in a table column identified by a FIELDref. Alternatively they may
be instances of structured types (**TODO** ref to location of VO-DML doc)
represented by GROUPS consisting of multiple PARAMs, PARAMrefs, or
FIELDrefs. Especially this use of the GROUP element, introduced already
in version 1.10 of the VOTable specification, that distinguishes this
approach from the previous ones. The GROUP element can be directly
mapped to the structured types defied in VO-DM and by mapping its
components to corresponding components in the VO-DML type definition a
1-1 mapping is achieved almost trivially[^6]. The complete set of
mapping patterns supported by this specification will be defined
formally in section 6.7.3 (**TODO** fix reference).

[^6]: This was foreseen for example in the note on referencing STC in
    VOTable [@stc_in_vot], as well as in a presentation by S.
    Derriere in Naples: [http://wiki.ivoa.net/internal/IVOA/InterOpMay2011SED/PPDMDesc_Naples.pdf].
    Lacking a formal target modelling language these developments
    remained ad hoc.

In the rest of this chapter we discuss *how* the VOTable elements are to
identify the corresponding VO-DML element. Originally it was assumed
that the `@utype` attribute of these VOTable elements (PARAM, PARAMref,
FIELDref and GROUP) would be used to identify which data model element
is represented. But it was decided *not* to use `@utype` after all[^7].
Instead it was agreed that the VOTable schema would be extended with a
new type elements, `<VODML>` to be added to these VOTable elements.
This element would take the task that was originally intended for
`@utype`, namely to point into a data model. The requirement of extending
the VOTable schema is unfortunate and the existing schema would have
sufficed to represent the mapping patterns[^8], on the other hand it
allows us to design a much more explicit annotation mechanism than would
be possible when using `@utype`-s only.

[^7]: During various interoperability meetings, finalized in Banff 2014.

[^8]: See older versions of this document on volute, mentioned in the
    history above.

The extension to the VOTable schema is reproduced in Appendix A. (**TODO** fix
reference) In the next few sub-sections we discuss this element in detail.

**TODO** TBD see if we need next paragraphs here or in other parts of text

A typical usage scenario may be a VOTable naïve (see Appendix B) client
that is sensitive to certain models only, say STC. Such a tool can be
written to understand annotation with STC types. Finding an element
mapped to a type definition from STC it might infer for example that it
represents a coordinate on the sky and use this information according to
its requirements.

Such a tool would not necessarily understand other models where such an
STC type is *used* as a role. So, if the annotation refers to both the
attribute’s role *and* type, even a naïve client can trivially find the
information it needs. A more advanced client may want to read the Data
Model Description File that describes the Data Model in a standardized,
machine readable, fashion.

Other scenarios involve inheritance and polymorphism. Inheritance allows
models to extend classes defined in other data models. Polymorphism is
the common object-oriented design concept that says that the declared
type of a property may not be the same as the type of an instance of
that property that is actually serialized. In particular, the value of a
property may be an instance of a *subtype* of the declared type. So in
general it is not enough to know the declared datatype type of the
attribute (for example) to uniquely know which type of instance to
expect. And it may also not be possible to infer the instance type
uniquely from the contents of the element representing the attribute.
Hence only annotating with the VO-DML **Role** may not be sufficient to
infer all DM information about a VOTable element. Hence we include the
possibility to annotate VOTable elements with the exact VO-DML **Type**
as well.

Typed languages such as Java support a casting operation, which provides
more information to the interpreter about the type it may expect a
certain instance to be.

complexType: VODMLAnnotation
----------------------------

According to the schema fragment in Appendix A, the VODML element
consists of elements `<TYPE>`, `<ROLE>`; and possibly one or more
`<OPTION>` elements. We ignore the latter element here, see the
detailed definition in section 6.7.3 for their use. Note, at least one
of TYPE or ROLE must be present.

A VODML/TYPE element MUST have as value a valid **vodmlref** (see next
subsection) ,which formally indicates that the VOTable element it
belongs to represents an instance of the identified VO-DML **Type**. The
VO-DML/XML document that the **vodmlref** identifies through its prefix
will give the formal definition of that type at the element identified
by the suffix **vodml-id**.

Most instances of types are actually used in the definition of a larger,
structured type: they play a role in its definition \[TBD ref to
location in VO-DML element\]. In VOTable this structured parent type
will be represented by a GROUP (see 7.2 or 7.2). A VOTable element that
is contained in such a group MUST identify which precise role it
represents through the VODML/ROLE element. This element is again a
vodmlref that MUST identify a **Role** (i.e. **Attribute**,
**Reference** or **Composition** relation) available on the **Type**
identified by the parent GROUP.

These elements VODML/TYPE and VODML/ROLE

simpleType: VODMLReference
--------------------------

This type represents a reference to a single element in a VO-DML/XML
document. It take over the role of the `@utype` attribute in this regards.
Whenever we wish to refer to instances of the VODMLReference type we
will call them **vodmlref**-s. A vodmlref is a string with the following
syntax:

**vodmlref ::= prefix ‘:’ vodml-id.**

The prefix identifies the model in which the element identified by the
suffix is defined.

How this is to be done is still under some discussion and we describe
both versions next. (**TODO** TBD version 1 is the currently accepted approach;
needs rewrite once we settle on one of these)

**vodml-ids** are always considered opaque, meaning that clients have no
reason to parse them. They are identifiers mapping VOTable elements to
VO-DML elements in the identified data model. Thus, they must follow the
same syntax rules defined in the VO-DML/Schema document.

The following two sections describe two different scenarios for the
UTYPEs format: they both have pros and cons, and we need more discussion
and feedback from the community in order to adopt only one of the two.

### Version 1:

Prefixes MUST be exactly the same as the **name** attribute of the model
in the VO-DML/XML document that defines it. They are sequences of
\[A-Za-z0-9\_-\], and they are case sensitive.

For new models, that are not (yet) standardized or for custom data
models used in a smaller community, It is recommended to form DM
prefixes as `<author-acronym>_<dm-name>`, where the
`<dm-name>` is the name of a standard data model; thus, NED's
derivation of spec could have `ned_spec` as a prefix, CDS's derivation
`cds_spec`.

Prefixes correspond to major versions for the corresponding data models.
Thus, **vodmlrefs** remain constant over "compatible" changes in the
sense of [@vodml].  In consequence, clients must assume a compatible
extension when encountering an unknown **vodmlref** with a known prefix
(and should in general not fail).

Another consequence of this rule is that there may be several VO-DML
URLs for a given prefix.  To identify a data model, use the prefix, not
the VO-DML URL, which is intended for retrieval of the data model
definition exclusively.  In case a client requires the exact minor
version of the data model, it must inspect the GROUPs identifying which
models are used in a VOTable as described in section 7.1 below.

### Version 2

Prefixes are sequences of \[A-Za-z0-9\_-\], and they are case sensitive.

#### The Namespace URI

The **vodmlref** prefix is a reference to a namespace URI defined in a
VO-DML preamble (see 4.3, also 7.1).

The namespace URI MUST be an IVOA Resource Name (IVORN) in the form
ivo://authorityID/DM-ID

#### How to look for a vodmlref in a document

Clients may match **vdmlrefs** by the simple string comparison of the
`<ROLE&>` or `<TYPE>` elementsin a VOTable with **vodml-ids**
defined in the Data Models descriptions.

Clients need to look for the Data Models they are interested in by
parsing the VO-DML preamble (see 4.3) and matching the Model’s URI with
the ones declared in the preamble. The preamble maps the Model to a
prefix string. This string must be attached to the id part according to
the **prefix:vodml-id** syntax before it can be compared to the
**vdmlrefs** in the document.

#### Extended notation for portable use: vdmlrefs as URIs

In order to make **vdmlrefs** portable outside of the VOTable semantics,
we define an extended URI notation for **vdmlrefs** by stringing
together the namespace URI and the local name of their QName compact
notation: the **vodml-id** will be the fragment part of the URI.

Thus, the extended notation for a **vdmlref** with **vodml-id**
‘Foo.bar’ will be:

ivo://authorityID/DM-ID#Foo.bar

Such IDs can be referenced in any context and can be resolved to the
VO-DML document description using the standard mechanism for resolving
IVORNs in Resource Registries.

The VO-DML preamble
-------------------

This section assumes Alternative 1 in section 4.2

In order to signal to the reader that a VOTable document falls under
this specification, a VOTable instance MUST declare all the Data Models
it includes, their versions, and the **vodmlref** prefixes for this
model. They MUST also declare the actual URL of the VO-DML/XML
description as a shortcut[^9].

[^9]: If a model is registered in an IVOA compliant registry, its IVOID
    may be used instead of the URL to identify it. (**TODO** This will
    require a Registry extension and should be handled by VO-DML spec).

Any number of such declarations can be included in a document.

``` {.xml .numberLines}
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
```

The above example introduces the Source Data Model and declares the name
“src” which is to be used as prefix to **vodmlref**-s pointing to its
elementsIt also identifies the url referring to the VO-DML/XML
description of the Data Model. Notice the “vo-dml:Model” special
VODML/TYPE that annotates the GROUP element to introduce the
declaration. No role should be used here.

The preamble GROUPs MUST be direct children of the VOTABLE element.

Note that the VO-DML preamble MUST include the models corresponding to
ALL the different prefixes used for the **vodmlrefs** in the VOTable. As
a matter of fact, the preamble can be seen as a way to declare such
prefixes and map them to models. Note furthermore that the **vo-dml**
prefix used in the above annotations itself refers to an explicitly
defined data model. It will be described in the next section.

Special annotations from the VO-DML Mapping model
-------------------------------------------------

There are some special **vodmlrefs** with prefix *vo-dml* (**TODO** TBD should
we use different name? E.g. vodml-i or so? To identify this models deals
with mapping instances?). These can be used to create specialized
mapping patterns. We have already encountered some examples in the
previous section to identify Models represented in a VOTable.

We actually use the same mapping patterns defined in this specification
for this. This implies we need a special data model, which we name
*VO-DML Instance Mapping* and whose prefix is *vo-dml* (**TODO** should
maybe change this to *vodml-i*). The model[^10] can be found
[https://volute.googlecode.com/svn/trunk/projects/dm/vo-dml/models/vo-dml/VO-DML.vo-dml.xml] here.

[^10]: Note that this model does **not yet** obey all of the data
    modelling rules imposed on real data models. It is defined to keep
    up the invariant that each vodmlref MUST refer to an element in a
    formal VO-DML/XML element.

General information about this spec {#sec:info}
===================================

Sample model and instances
--------------------------

For examples we use a highly simplified version of a possible Source
data model, illustrated by its UML representation in [@fig:model].

![Data model used in
examples. It represents a simplified Source data model, containing
luminosities that refer to the imported PhotDM. It also defines a
simplistic version of an STC model with some types for defining
coordinates on the sky, for the sake of simplicity and just for example
purposes.](media/image3.png){#fig:model}

The model defines some types allowing one to define a Source with
position on the sky and a collection of luminosities. The position is
modeled as a DataType, ‘SkyCoordinate’. SkyCoordinate has a reference to
a coordinate frame that is required to interpret its longitude and
latitude attributes. The luminosities are really *measurements* of
luminosities in a given filter that is indicated by a reference to a
PhotometryFilter, which is imported from the PhotometryDM; hence they
have a value *and* an error. A Quantity DataType is introduced that
provides a real value and a unit.

The models are *by no means* meant to be comprehensive and include some
admittedly artificial elements such as an Equinox PrimitiveType, which
is supposed to be a simple string and might carry enough semantic value
of its own to use it as an annotation on
PARAM elements for example.

Note that this sample model defines a Package that contains all the
types. This package shows up in the values of the **vodml-ids** we use
to identify the different elements. The values we use for these
**vodml-id** identifiers are generated from the VO-DML using a
particular grammar: they are path-like expressions that are guaranteed
to be unique and give some indication of the location of the element
they point to in the data model.

We also use some sample instances of the models. These are here
illustrated by UML instance diagrams. The diagram in {#fig:instance} represents
the first two lines returned from a query[^query] to the SDSS DR7 database.

![Figure 2 Instance diagram representing SDSS
objetcs as sources in the sample data model. The first few results are
represented from the default radial SDSS query at
<http://skyserver.sdss.org/dr7/en/tools/search/radial.asp>](media/sdss_instance.jpg){#fig:instance}

[^query]: Specifically:
``` sql
SELECT top 10 p.objID, p.run, p.rerun, p.camcol, p.field, p.obj,
p.type, p.ra, p.dec, p.u,p.g,p.r,p.i,p.z,
p.Err\_u, p.Err\_g, p.Err\_r,p.Err\_i,p.Err\_z
FROM fGetNearbyObjEq(195,2.5,3) n, PhotoPrimary p
WHERE n.objID=p.objID
```

Data carriers in VOTable
------------------------

VO-DML describes four different kinds of types: PrimitiveType (PT),
Enumeration (E), DataType (DT) and ObjectType (OT). PT, E and DT are
value types; OT is an object - or reference type. PT and E are atomic,
their values consist of a single value; DT and OT are structured, they
are built from multiple values, organized as attributes, and possibly of
reference relations to OTs. An OT can also have composition relations,
or collections of other OTs, and can have an identifier, an attribute of
undetermined type that is implicitly defined for ObjectTypes.

To store instances (*values* and *objects*) of these types in a VOTable
various options are available. Atomic values (i.e. instances of PT and
E) are stored in cells in a row in a table (i.e. a TD) or in the value
attribute of a PARAM (`@value`). To store an instance of a structured type
one must store its components. To identify the structured instance in a
serialization one must be able to identify the individual components and
how they are to be combined. This aggregation is done using a GROUP
element. It is the main representation of structured types, both
ObjectType and DataType. It is also used to represent relations to
object types. These may be stored using foreign-key-like mechanisms or
through some kind of hierarchy.

In fact in the approach described here virtually *all* mapping of
VOTable to VO-DML is performed by GROUP elements and their components.
The VODML elements contained in them identifies which elements are to be
combined to create the object or DataType instance, what the roles are
that these components play (attribute, reference).

Single-table representations and Object-Relational Mapping
----------------------------------------------------------

Broadly speaking, this specification is all about Object-Relational
Mapping (ORM). Data Models are represented in VO-DML according to an
Object Oriented paradigm, although in a very limited fashion that is
also suited for use in Relational Databases.

(**TODO** OL: Actually, I think it's not OOP, rather it's Entity-Relationship,
which is also what's stated below.)

As VOTable can represent several tables in the same file with rich
metadata, one can look at VOTable as a database that can represent
complex relational models.

Such models are usually defined in terms of entities, with each table
representing each entity, and relationships that can be expressed as
tables themselves or as constraints on the values in the tables, and
most often with a combination of tables and constraints. For instance, a
Many-To-Many relationship between two Entities is usually represented in
the relational model as a table holding IDs of instances from the tables
representing the Entities, with Foreign Keys constraints.

Astronomers mainly work with single tables that hold flattened
representations of relatively simple models, although in some cases
complex data models are serialized in several tables inside the same
file.

This specification covers both requirements. Serializations of simple
models in a flattened table are easier to achieve than complex ORM
mappings where information is normalized into different tables, but they
are both achievable in VOTable. Moreover, the hybrid case of partly
de-normalized representations, where the model is only partly
normalized, is more challenging but should also be addressable in terms
of this specification.

In any case, the examples in section 6, and most of section 6.7.3 (the
normative part of this document) are focused on the single-table,
flattened representations of instances according to some data model.
Some of the patterns described in these sections are also applicable to
simple ORM cases. Especially the sections dealing with mapping reference
and composition relations also deal with the more complex cases of
proper ORM mappings, where data is partly or completely normalized into
different tables.

The simple and complex ORM patterns described by this specification
usually belong to very different concrete use cases, so it should be
acceptable in a broad range of cases that implementers, both on the
server and on the client side, focus on the single-table mappings. Data
providers requiring more complex patterns, more advanced applications,
or applications built on top of standard software libraries that
implement this specification as a whole will need to take advantage of
the ORM mapping patterns.
