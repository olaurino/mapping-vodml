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

General Remarks
---------------

This specification provides a standardized mechanism
for annotating VOTables according to data models. Thus, it enables:

  * Data providers to annotate VOTables so to faithfully map VOTable contents to one or more
  data models, as long as such Data Models are expressed according to the
  VO-DML standard [@vodml]. In other terms, they can *serialize* data model
  *instances* in a standard, interoperable way. Some examples are provided below
  as concrete use cases.
  * Service clients to faithfully reconstruct the semantics of the data model instances
  they consume in VOTable format. Some concrete examples are also provided below.

One of the main goals of this specification is also to alleviate data modelers
from the burden of defining special serialization strategies for their data models, at least
in most cases. Specialized serializations might be defined if there are special
constraints in term of efficiency or effectiveness which require
specialized serialization schemes.

As a corollary to the above paragraph, client applications can also be implemented
on top of standardized Input/Output libraries that implement the present specification,
as the serialization mechanisms are standardized across data models. Without this specification
clients would need to be coded against specialized serializations for each data model.
Similarly, data providers can now serialize instances of any data model to VOTable using
the same annotation mechanism.

As a result, this mapping specification should enable a large number of concrete use cases
to be implemented, by reducing the annotation burden on both data providers and data consumers,
improving the overall interoperability, at least for what VOTable is concerned.

This document also represents a template for mapping data model instances to other
formats (see [@sec:other-formats]).

Concrete Use Cases
------------------

### STC clients ###
A typical usage scenario may be a VOTable client
that is sensitive to certain models only, say Space Time Coordinates (STC).
Such a tool may be
written to understand annotations for instance of STC types, manipulate such instances, and write them back
to disk.

By finding an element
mapped to a type definition from the STC model, the application may infer for example that it
represents a coordinate on the sky and use this information according to
its requirements. For example, the client may convert all positions
expressed in a certain coordinate frame to a different coordinate frame
through some specific transform.

Note that the client may never parse any VO-DML description files,
as the knowledge about the data model may be assumed when the client code is developed.

Also, the STC annotations are the same in all the contexts in which an STC type is used.
So, for instance, if a VOTable describes catalogues of sources, and each source
has a position attribute describing its coordinates in a specific reference
frame, the *instance* describing the source's position would be annotated in the
exact same way as, say, the central position of a cone search query.

So, the STC tool may not necessarily understand other models where
STC types are *used*, but it may still be able to find the instances
of those types.

Note that the same file may contain multiple tables and in any case
STC model instances defined in multiple coordinate systems or frames.

### VO-enabled plotting and fitting applications ###
An application whose
main requirement is to display, plot, and/or fit data cannot be required
to be aware of *all* astronomical data models. However, if these data models shared
some common representation of quantities, their errors, and their units,
the application may discover these pieces of information and structure a
plot, or perform a fit, with minimal user input: each point may be
associated with an error bar, upper/lower limits, and other metadata.
The application remains mostly model-agnostic: it is not required to
*understand* high-level concepts like Spectrum, or Photometry.

Also, knowledge about the basic building-block types like quantities
and coordinates, may be hard-coded during development.

### Data discovery portals ###
Data discovery portals allow users to query VO services, display metadata,
filter responses, and fetch datasets.

While these applications may not be particularly interested in specific data models,
standardized annotation mechanisms may allow their developers to dynamically capture
the structure of the metadata, and provide better exploratory tools rather than flat
tables.

Consider for example filtering tables according to an arbitrary physical quantity, say
for instance the spectral coverage of a spectrum, the filter with which an image was taken, or
the luminosity of a source in a certain band. The portal may provide a friendly interface that allows
users to select the physical quantities using standardized representations. It may do
so dynamically for all the pieces of metadata present in the dataset, rather
than limiting this functionality to a set of hard-coded metadata properties.

Also, the application may group data and metadata in a tree of concepts the user is familiar with,
rather than flattening the instrinsic structure of the data.

By allowing such applications to faithfully represent data model instances the way they
were curated and annotated by data providers, and according to the scientific domain
with which users are familiar, this specification may enable users to easily make sense of
the file contents, even if the application lacks any knowledge of high-level data models.

Parsing VO-DML description files may be useful to provide the user with even more, and more
accurate, information.

### Color-color diagrams ###
The creation of a color-color diagram requires knowledge of the semantics
of the rows and columns in a table, e.g. a source catalog. Also, some columns
may be grouped together in a structured way, e.g. a luminosity measurement
that goes with its error and a reference to the photometry filter it is associated
with, although such columns might not be adjacent in the file.

Usually, plotting applications are not aware of the semantics of the columns in a table,
and users may have to select all the relevant columns in order to produce a
meaningful plot. They may also have to convert between units, and so on.

With a standardized annotation and knowledge of the annotations for basic quantities
a plotting application may find that there are luminosity measurements in
a VOTable and allow users to display color-color diagrams, with error bars,
and possibly with domain-specific actions like unit conversions, seamlessly and requiring minimal input.

There are many examples of very specific use cases that may be implemented by
science applications that are aware of the semantics of specific models and their annotations.

### Validators ###
The existence of an explicit data model representation
language and of a precise, unambiguous mapping specification
enables the creation of universal validators, just as it happens
for XML and XSD: the validator may parse the data model descriptions
imported by the VOTable and check that the file represents valid
instances of one or more data models.

### VO Publishing Helper ###
There is some complexity involved in understanding how to publish data in the Virtual
Observatory. The availability of a standard for serializing data model instances
can provide tools for publishers to build templates of their responses.

Such application may help
data providers in interactively mapping Data Models elements to their
files or DB tables, either producing a VOTable template with the
appropriate annotations, or by creating a DAL service on the fly.

The application is not required to be model-aware, since it
may get all the information from the standardized description files and the mapping
specification.

### VO Importer ###
Users and Data Providers may have non-compliant files
that they want to convert to a VO-compliant format according to some
data models: a generic, model-unaware importer application may allow them to do so
for any standard data model with a proper description file.

Generic Use Cases
-----------------
This section generalizes the previous one by stating the same use cases
in a more abstract, generic formulation. It also adds some more generic use cases
that this specification addresses or enables.

### Serialize and de-serialize instances according to a data model ###
Data providers may want to serialize data and metadata in VOTable
according to a specific data model with a standardized description.

A client may build an in-memory faithful representation of that instance
according to the data provider's annotations, assuming the knowledge of
a finite set of data model identifiers, of a full data model, or by parsing
standard VO-DML data model specifications.

### Annotate files according to multiple models ###
Data providers may find it useful to annotate a file according to different
data models for different classes of data consumers. Also, they may decide to provide
annotations according to different versions of a specific data model, to favor
backward compatibility with older clients.

### Representing cross matches and linking files together ###
It is often the case that two or more files or tables represent different
pieces of information regarding the same astronomical sources of objects.
In these cases, one or more columns
usually are used as keys to identify instances in such tables.

For instance, the output of a cross-matching service may provide the IDs of the
cross matched sources along with some data regarding the cross-matching process,
while most of the data about the sources themselves may
be stored in different tables.

This is a very common relational pattern. A standardized annotation with
Object-Relational Mapping capabilities may be used to connect different tables
and provide users with a unified view according to data models
the user is familiar with.

It is then possible to link different views of the datasets, with an additional
layer of semantics. For instance, an application may display the image of a region
of the sky, and a catalogue may contain information about sources in the catalogue.

A user may want to link the image to the catalogue. When no a-priori link between
the image coordinates and the positions in the catalogue is known, users need to set such
links themselves, by selecting the relevant columns.
With standardized annotations according to specific data models
applications can figure out the links themselves, and ask the user to intervene only
when there is ambiguity or lack of information, or when the user wants to make a custom link.

Similarly, one may produce a color-color diagram from magnitudes stored in different tables
as long as there is
a known mapping from source identifiers among tables, and a standardized annotation of
all the tables involved.

### Mission-specific data model extensions ###
One may identify data and metadata features that are common to a certain
astronomical domain, e.g. catalogues of astronomical sources. These are the models
the IVOA sanctions in standards.

However, different missions, or archives, or applications will most certainly
have specific additional features that are not captured by such common, standardized
data models.

One may express such extensions and their instances in such a way that:
  * data providers may easily annotate specialized instances, including the additional information, in a
  standardized, interoperable fashion.
  * clients of the common, standard data models may still find instances of these parent models in files
  serializing specialized instanced.
  * a model annotation that is valid according to a specialized data model
  is also valid according to the parent model.

This use case can be formalized in terms of inheritance and polymorphism.

Inheritance allows
models to specialize types defined in other data models. Polymorphism is
the common object-oriented design concept that says that the value of a
property may be an instance of a *subtype* of the declared type.

Typed languages such as Java support a casting operation, which provides
more information to the interpreter about the type it may expect a
certain instance to be.

A client must be able to identify
the information about a standard type, even if the serialization includes
instances of its subtypes. Similarly, a client should have enough information to *cast* an instance
from the declared type to the actual subtype.

Growing complexity: naïve, advanced, and guru clients {#sec:clients}
-----------------------------------------------------

We can classify clients
in terms of how they parse the VOTable in order to harvest its content.
Of course, in the real word such distinction is somewhat fuzzy, but this
section tries and describe the different levels of usage of this
specification.

### Naïve clients ###

We say that a client is naïve if:

  * it does not parse the VO-DML description file

  * it assumes the a priori knowledge of one or more Data Models

  * it discovers information by looking for a set of predefined
    vodml-refs in the VOTable

In other terms, a naïve client has knowledge of the Data Model it is
sensitive to, and simply discovers information useful to its own use
cases by traversing the `VODML` element.

Examples of such clients are the DAL service clients that allow users to
discover and fetch datasets. They will just inspect the response of a
service and present the user with a subset of its metadata. They do not
*reason* on the content, and they are not interested in the structure of
the serialized objects.

If such clients allow users to download the files that they load into
memory, they should make sure to preserve the structure of the metadata,
so to be interoperable with other applications that might ingest the
same file at a later stage.

### Advanced clients ###

We say that a client is advanced if:

  * it does not parse the VO-DML description file

  * it is interested in the structure of the serialized instances

  * can follow the mapping patterns defined in this specification, for
    example collections, references, and inheritance

Examples of such clients are science applications that display
information to the user in a structured way (e.g. by plotting it, or by
displaying its metadata in a user-friendly format), that *reason* on the
serialized instances, perform operations on those instances, and
possibly allow the users to save the manipulated version of the
serialization.

Notice that the fact that an application does not directly use some
elements that are out of the scope of its requirements does not mean
that the application cannot provide them to the user in a useful way.
For example, an application might allow users to build Boolean filters
on a table, using a user-friendly tree representing the whole metadata.
This exposes all the metadata provided by the Data Provider in a way
that might not be meaningful for the application, but that may be
meaningful for the user.

Notice, also, that advanced clients *may* be DM-agnostic: for instance,
an Advanced Data Discovery application may allow the user to filter the
results of a query by using a structured view of its metadata, even
though it does not possess any knowledge of Data Models.

### Guru clients ###

We say that a client falls into this category if:

  * it parses the VO-DML descriptions

  * it does not assume any a priori knowledge of any Data Models.

Such applications can, for example, dynamically allow users and Data
Providers to map their files or databases to the IVOA Data Models in
order to make them compliant, or display the content of any file
annotated according to this standard.

This specification allows the creation of universal validators
equivalent to the XML/XSD ones.

It also allows the creation of VO-enabled frameworks and universal
libraries. For instance, a Python universal I/O library can parse any
VOTable according to the Data Models it uses, and dynamically build
objects on the fly, so that users can directly interact with those
objects or use them in their scripts or in science applications, and
then save the results in a VO-compliant format.

Java and Python guru clients could automatically generate interfaces for
representing Data Models and dynamically implement those interfaces at
runtime, maybe building different views of the same file in different
contexts.

Notice that Guru frameworks and libraries can be used to build Advanced
or even Naïve applications in a user-friendly way, abstracting the
developers from the technical details of the standards and using
scientific concepts as first class citizens instead.

Formats other than VOTable { #sec:other-formats }
--------------------------
We want to explicitly note that this specification covers the VOTable format only.

Other mapping specifications
can and will provide standardized strategies for mapping Data Models to formats other
than VOTable.

Part of the implementation efforts related to the present specification was to validate
the standard against prototype serializations in JSON and YAML formats.

Mapping specifications targeting additional formats can use this document as a template.

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
