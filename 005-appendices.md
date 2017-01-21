The VODML annotation elements in the VOTable schema {#sec:schema .AppendixA}
===================================================

<!--
``` {include=utils/votable-additions.xml .xml .numberLines}
VODML additions to the VOTable schema.
```
-->

Growing complexity: naïve, advanced, and guru clients {#sec:clients .AppendixA}
=====================================================

This document defines a complete, unambiguous, standard specification
that can be used to serialize and de-serialize instances of Data Model
types. It was designed to be simple and straightforward to implement by
Data Providers and by different kind of clients. We can classify clients
in terms of how they parse the VOTable in order to harvest its content.
Of course, in the real word such distinction is somewhat fuzzy, but this
section tries and describe the different levels of usage of this
specification.

Naïve clients
-------------

We say that a client is naïve if:

  * it does not parse the VO-DML description file

  * it assumes the a priori knowledge of one or more Data Models

  * it discovers information by looking for a set of predefined
    VO-DML-refs in the VOTable

In other terms, a naïve client has knowledge of the Data Model it is
sensitive to, and simply discovers information useful to its own use
cases by traversing the document, seeking the elements it needs by
looking at their `<VODML>` element.

Examples of such clients are the DAL service clients that allow users to
discover and fetch datasets. They will just inspect the response of a
service and present the user with a subset of its metadata. They do not
*reason* on the content, and they are not interested in the structure of
the serialized objects.

If such clients allow users to download the files that they load into
memory, they should make sure to preserve the structure of the metadata,
so to be interoperable with other applications that might ingest the
same file at a later stage.

Advanced clients
----------------

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

Guru clients
------------

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

Java guru clients could automatically generate interfaces for
representing Data Models and dynamically implement those interfaces at
runtime, maybe building different views of the same file in different
contexts.

Notice that Guru frameworks and libraries can be used to build Advanced
or even Naïve applications in a user-friendly way, abstracting the
developers from the technical details of the standards and using first
class scientific objects instead.

References
==========

