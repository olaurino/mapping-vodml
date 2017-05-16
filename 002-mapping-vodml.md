
Mapping with the `VODML` element.
=================================

This section summarizes the technical basis of this reccomendation.

The present specification, in conjunction with with VO-DML recommendation
[@vodml] provides a formal mechanism for using such data model identifiers,
although different from the original `@utype` attribute definition and its
usages [@usages].

VOTable 1.2 introduced the `@utype` attribute, which was intended to
represent "pointers into a data model". A precise and formal definition
on how this *pointing* was to be achieved and a description of its
meaning was missing though.

First, a formal definition of the target of the pointers was missing. To
solve this, data models were usually accompanied by a list of *utypes*
(**TODO** [TBD refer to STC, Characterization, Spectrum?]), and these could be
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

VO-DML provides a formal target for these pointers in the data model
itself and formally defines how models can be reused in the definition
of other, dependent models. Precisely *how* to use these pointers in a
VOTable to provide a complete annotation useful for interoperability
requires more work though.

The current specification provides such a definition. It shows how data
publishers can identify also the more complex data model elements such
as structured types and relationships inside some published data source,
be it a VOTable or relational database published through the TAP
protocol.

This specification defines various *mapping patterns* from VOTable to
VO-DML. Such a pattern identifies a VOTable element with a VO-DML
element. The VO-DML element is said to be *represented* by the VOTable
element. The mapping pattern indicates that instances of identified
VO-DML types are present in the VOTable. These may be atomic *values*
(instances of VO-DML ValueTypes [@vodml]) or represented by
cells in a table column identified by a `FIELD`. Alternatively they may
be instances of structured types.

The extension to the VOTable schema is reproduced in [@sec:schema].

VODMLReference { #sec:vodmlref }
--------------

This XML type represents a reference to a single element in a VO-DML/XML
document. It takea over the role of the `@utype` attribute in this regards.
Whenever we wish to refer to instances of the VODMLReference type we
will call them **vodmlref**-s. A vodmlref is a string with the following
syntax:

````
vodmlref ::= prefix ‘:’ vodml-id
````

The prefix identifies the model in which the element identified by the
suffix is defined.

**vodml-ids** are always considered opaque, meaning that clients have no
reason to parse them. They are identifiers mapping VOTable elements to
VO-DML elements in the identified data model. Thus, they must follow the
same syntax rules defined in the VO-DML/Schema document.

Prefixes MUST be exactly the same as the **name** attribute of the model
in the VO-DML/XML document that defines it. They are sequences of
\[A-Za-z0-9\_-\], and they are case sensitive.

For new models, that are not (yet) standardized or for custom data
models used in a smaller community, it is recommended to form DM
prefixes as `<author-acronym>_<dm-name>`, where the
`<dm-name>` is the name of a standard data model; thus, NED's
derivation of spec could have `ned_spec` as a prefix, CDS's derivation
`cds_spec`.

Prefixes correspond to major versions for the corresponding data models.
Thus, **vodmlrefs** remain constant over "compatible" changes in the
sense of [@vodml]. In consequence, clients must assume a compatible
extension when encountering an unknown **vodmlref** with a known prefix
(and should in general not fail).

Another consequence of this rule is that there may be several VO-DML
URLs for a given prefix.  To identify a data model, use the prefix, not
the VO-DML URL, which is intended for retrieval of the data model
definition exclusively.  In case a client requires the exact minor
version of the data model, it must inspect the models declarations
as described in [@sec:normative]

(**TODO** OL: This doesn't feel right. I believe minor versions should be uniquely
identified by a URI and without having to parse the descriptor, especially since
we have started talking about registering models in the Registry.)

#### How to look for a vodmlref in a document

(**TODO** to fill out once the syntax has settled)

General information about this spec {#sec:info}
===================================

Sample model and instances
--------------------------

(**TODO** This needs to be filled with the designated sample model)

<!--

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
-->

Single-table representations and Object-Relational Mapping
----------------------------------------------------------

Broadly speaking, this specification is all about Object-Relational
Mapping (ORM). Data Models are represented in VO-DML according to an
Entity-Relationship paradigm, in a fashion that is implementable
by relational databases, object oriented languages, and possibly
by certain document oriented data bases as well.

As VOTable can represent several tables in the same file with rich
metadata, one can look at VOTable as a data base that can store instances of
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

In any case, the examples in [@sec:normative] are focused on the
single-table,
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

