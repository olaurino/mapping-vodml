History of this document {-}
------------------------

**TODO** migrate document's history

Introduction
============

Data providers put a lot of effort in organizing and maintaining
metadata that precisely describes their data files. This information is
invaluable for users and for software developers that provide users with
user-friendly VO-enabled applications. For example, such metadata can
characterize the different axes of the reference system in which the
data is expressed, or the history of a measurement, like the publication
where the measurement was drawn from, the calibration type, and so
forth. In order to be interoperable, this metadata must refer to some
Data Model that is known to all parties: the IVOA defines and maintains
such standardized Data Models that describe astronomical data in an
abstract, interoperable way.

In order to enable such interoperable, extensible, portable annotation
of data files, one needs:

* A language to unambiguously and efficiently describe Data Models and
    their elements’ identifiers (VO-DML, [@vodml]).

* Pointers linking a specific piece of information (data or metadata)
    to the Data Model element it represents[^2].

* A mapping specification that unambiguously describes the mapping
    strategies that lead to faithful representations of Data Model
    instances in a specific format.

[^2]: This used to be the assumed role of the `@utype` attribute in
    VOTable and for example TAP. This document is based on the new
    `VODML` element in `VOTable` 1.4 [@votable].

Without a consistent language for describing Data Models there can be no
interoperability among them, through reuse of models by models, or in
their use in other specifications. Such a language must be expressive
and formal enough to enable the serialization of data types of growing
complexity and the development of reusable, extensible software
components and libraries that can make the technological uptake of the
VO standards seamless and scalable.

For serializations to non-standard representations one needs to map the
abstract Data Model to a particular format meta-model. For instance, the
VOTable format defines `RESOURCE`s, `TABLE`s, `PARAM`s, `FIELD`s, and so forth,
and provides explicit attributes such as `units`, `UCD`s, and `utypes`: in
order to represent instances of a Data Model, one needs to define an
unambiguous mapping between these meta-model elements and the Data Model
language, so to make it possible for software to be able to parse a file
according to its Data Model and to Data Providers to mark up their data
products.

While one might argue that a standard for portable, interoperable Data
Model representation would have been required before one could think
about such a mapping, we are specifying it only at a later stage. In
particular several different interpretations of `UTYPE`s have been
proposed and used [@usages]. This specification aims to resolve this
ambiguity.

As a matter of fact, existing files and services can be made compliant
according to this specification by simply *adding* annotations and
keeping the old ones. So they do not need to *change* them in such a way
that would necessarily make them incompatible with existing software.

Several sections of this document are utterly informative: in
particular, the appendices provide more information about the impact of
this specification to the current and future IVOA practices.

This specification describes how to represent Data Model instances using
the VOTable schema. This representation uses the `<VODML>` element
introduced for this purpose in VOTable v1.4 [@votable] and the structure of
the VOTable meta-model elements to indicate how instances of data models
are stored in VOTable documents. We show many examples and give a
complete listing of allowed mapping patterns.

In sections [-@sec:usecases] to [-@sec:info] we give an introduction to why and
how the VODML elements can be used to hold pointers into the data models.

Section [-@sec:normative] is a rigorous listing of all valid annotations, and the
normative part of the specification.

The appendices contain additional material. [Section @sec:schema] describes the
VODML annotation element that was added to the VOTable schema to support
this mapping specification. [Section @sec:clients] describes different types of
client software and how they could deal with VOTables annotated
according to the current specification.

**Throughout the document we will refer to some real or example Data
Models. Please remember that such models have been designed to be fairly
simple, yet complex enough to illustrate all the possible constructs
that this specification covers. They are not to be intended as actual
DMs, nor, by any means, this specification suggests their adoption by
the IVOA or by users and or Data Providers. In some cases we refer to
actual DMs in order to provide an idea of how this specification relates
to real life cases involving actual DMs.**

Use Cases {#sec:usecases}
=========

This specification provides a standardized mechanism
for annotating VOTables. In this sense, it enables an indefinite number
of use cases, each specific to the Data Model instances being represented
in VOTable format.

However, to give a sense of what it is possible to accomplish with this
specification, we provide some explicit use cases relative to the VO
domain.

A typical usage scenario may be a VOTable naïve (see [@sec:clients]) client
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
the common object-oriented design concept that says that the value of a
property may be an instance of a *subtype* of the declared type. This is
a common scenario, since different missions, instruments, or services may
represent specializations of standard types. A client must be able to identify
the information about a standard type, even if the serialization includes
instances of its subtypes, as it usually happens in object-oriented languages
and patterns.

Typed languages such as Java support a casting operation, which provides
more information to the interpreter about the type it may expect a
certain instance to be.

**Serialize and de-serialize instances according to a data model.**
Using this specification (or software implementing it), a data provider
can serialize the metadata for a dataset according to a data model. A
client can build in memory a faithful representation of that instance
according to the data provider's annotations, assuming the knowledge of
a finite set of Data Model identifiers. For example, the client can find all the
information about a Source by looking at a `INSTANCE` annotated with the
with type `src:source.Source`, and interpret its components as the attributes of
the object, identified by their identifier strings.

**Model-unaware serialization and de-serialization.** Model-unaware
readers and writers can serialize and de-serialize instances according
to specific data models by mapping the contents of a VOTable to model
description files. This may include file browsing, code generation, data
integration, etc.

**VO-enabled plotting and fitting applications.** An application whose
main requirement is to display, plot, and/or fit data cannot be required
to be aware of *all* data models. However, if these data models share
some common representation of quantities, their errors, and their units,
the application can discover these pieces of information and structure a
plot, or perform a fit, with minimal user input: each point will be
associated with an error bar, upper/lower limits, and other metadata.
The application remains mostly Data Model-agnostic: it wouldn’t need to
*understand* high-level concepts like Spectrum, or Photometry.

**Validators.** The existence of an explicit Data Model representation
language and of a precise, unambiguous mapping specification
enables the creation of universal validators, just as it happens
for XML and XSD: the validator can parse the Data Model descriptions
imported by the VOTable and check that the file represents valid
instances of the Data Model.

**VO Publishing Helper.** A universal publisher application may help
data providers in interactively mapping Data Models elements to their
files or DB tables, either producing a VOTable template with the
appropriate annotations, or by creating a DAL service on the fly.
The VO Publisher application is not required to be DM-aware, since it
can get all the information from the standardized description files.

**VO Importer.** Users and Data Providers may have non-compliant files
that they want to convert to a VO-compliant format according to some
data model: a DM-unaware Importer application may allow them to do so
for any standard Data Model.

**Extensibility.** Most often each astronomical facility, instrument, or
mission needs to express measurements and metadata attributes that are
unique to the facility, instrument, or mission. A data provider may want
to *extend* a Data Model, adding to the common information about
astronomical sources and data products the metadata that is specific for
their instruments or domain. The added metadata can be serialized in a
standardized fashion so that the user can take advantage of the
information.

The need for a mapping language
===============================

When encountering a data container, i.e. a file or database containing
data, one may wish to interpret its contents according to some external,
predefined data model. That is, one may want to try to identify and
extract instances of the data model from amongst the information. For
example in the *global as view* approach to information integration, one
identifies elements (e.g. tables) defined in a global schema with views
defined on the distributed databases[^4].

[^4]: See, for example,
    http://logic.stanford.edu/dataintegration/chapters/chap01.html

If one is told that the data container is structured according to some
standard serialization format of the data model, one is done. I.e. if
the local database is an exact *implementation* of the global schema,
one needs no special annotation mechanism to identify these instances.
An example of this is an XML document conforming to an XML schema that
is an exact physical *representation* of the data model. We call such
representations *faithful*.

But in an information integration project like the IVOA, which aims to
homogenize access to many distributed heterogeneous data sets, databases
and documents are in general *not* structured according to a standard
representation of some predefined, global data model. The best one may
hope for is to obtain an *interpretation* of the data set, defining it
as a *custom serialization* of the result of a *transformation* of the
global data model[^5]. For example, even if databases themselves are
exact replications of a global data model, results of general queries
will be such custom serializations.

[^5]: Or alternatively as a transformation of a (standard) serialization
    of the data model.

To interpret such a custom serialization one generally needs extra
information that can provide a *mapping* of the serialization to the
original model. If the serialization *format* is known, this mapping may
be given in phrases containing elements both from the serialization
format and the data model. For example if our serialization contains
data stored in ‘rows’ in one or more ‘tables’ that each have a unique
‘name’ and contain ‘columns’ also with a ‘name’, you might be able to
say things like:

-   The rows in this table named SOURCE contain *instances* of *object
    type* ‘Source’ as defined in *data model* ‘SourceDM’.

-   The *type*’s ‘name’ *attribute* (having *datatype* ‘string’, a
    *primitive type*) also acts as the *identifier* of the Source
    *instances* and is stored in the single column with ID 'name'.

-   The *type’s* ‘classification’ *attribute* is stored in the table
    column CLASSIFICATION (from the *data model* we know its *datatype*
    is an *enumeration* with certain *values*, e.g. ‘star’,
    ‘galaxy’, ‘agn’).

-   The *type’s* ‘position’ *attribute* (being of *structured data type*
    ‘SkyCoordinate’ defined in *model* ‘SourceDM’) is stored over the
    two columns RA and DEC, where RA stores the SkyCoordinate’s
    *attribute* ‘longitude’, DEC stores the ‘latitude‘ *attribute*. Both
    must be interpreted using an *instance* of the SkyCoordinateSystem
    *type*, This *instance* is stored in 1) another document elsewhere,
    referenced by a *reference* to a URI, or 2) in this document, by
    means of an *identifier.*

-   *Instances* from the *collection* of luminosities of the Source
    *instances* are stored in the same row as the source itself. Columns
    MAG\_U and ERR\_U give the ‘magnitude’ and ‘error’ *attributes* of
    *type* LuminosityMeasurement in the “u band”, an *instance* of the
    Filter *type*. (stored elsewhere in this document (‘a *reference* to
    this Filter instance is ...’). Columns MAG\_G and ERR\_G ... etc.

-   Luminosity *instances* also have a filter *relation* that points to
    instances of the PhotometryFilter *structured data type*, defined in
    the IVOA PhotDM model, whose *package* is imported by the SourceDM.

In this example the *emphasized* words refer to concepts defined in
VO-DML, a meta-model that is used as a formal language for expressing
data models. The use of such a modeling language lies in the fact that
it provides formal, simple, and implementation neutral definitions of the
possible structure, the ‘type’ and ‘role’ of the elements from the
actual data models that one may encounter in the serialization
(SourceDM). This can be used to constrain or validate the serialization,
but more importantly it allows us to formulate mapping rules between the
serialization format (itself a kind of meta-model) and the meta-model,
independent of the particular data models used; for example rules like:

-   An *object type* MUST be stored in an ‘INSTANCE’.

-   A ‘*primitive type*’ MUST be stored in a ‘COLUMN’.

-   A *reference* MUST identify an *object type* *instance* represented
    elsewhere, possibly in another ‘table’, possibly in the same table,
    possibly in another document.

-   An *attribute* SHOULD be stored in the same table as its containing
    *object type*.

-   etc

Clearly free-form English sentences as the ones in the example are not
what we are after. If we want to be able to identify how a data model is
represented in some custom serialization we need a formal, computer
readable mapping language.

One part of the mapping language should be anchored in a formally
defined serialization language. After all, for some tool to interpret a
serialization, it MUST understand its format. A completely freeform
serialization is not under consideration here. This document assumes
the target meta-model is VOTable.

The mapping language must support the interpretation of elements from
the serialization language in terms of elements from the data model. If
we want to define a generic mapping mechanism, one by which we can
describe how a general data model is serialized inside a VOTable, it is
necessary to use a general data model *language* as the target for the
mapping, such as the one described above. This language can give formal
and more explicit meaning to data modeling concepts, possibly
independent of specific languages representation languages such as XML
schema, Java, or the relational model.

This document uses VO-DML as the target language.

The final ingredient in the mapping language is a mechanism that ties
the components from the two different meta-models together into
*sentences*. This generally requires some kind of explicit annotation,
some meta-data elements that provide an identification of source to
target structure. This document uses an extension to VOTable with a
VODML element which can provide this link in a rather simple manner.

This solution is sufficient and it is in some sense the simplest and
most explicit approach for annotating a VOTable. It may *not* be the
most natural or suitable approach for other meta-models such as FITS
or TAP\_SCHEMA. We discuss this at the end of this document.

