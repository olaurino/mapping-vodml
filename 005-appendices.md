The VODML annotation elements in the VOTable schema {#sec:schema .AppendixA}
===================================================

The VOTable’s *Param*, *Paramref*, *Fieldref* and *Group* definitions
obtain an extra element named `<VODML>` with type *VODMLAnnotation*.
That type is defined by the next fragment extracted from the proposal
[VOTable-1.3\_vodml.xsd](https://volute.googlecode.com/svn/trunk/projects/dm/vo-dml/doc/samples/votable/VOTable-1.3_vodml.xsd)
(as of volute 2873).

``` {.xml .numberLines}
<!-- VODMLAnnotation section -->

  <xs:simpleType name="VODMLReference">
    <xs:annotation>
      <xs:documentation>
      The valid format of a reference to a VO-DML element. (Used to be 'UTYPE').
      MUST have a prefix that elsewhere in the VOTable is defined to correspond to a VO-DML model defining the referenced element.
            Suffix, separated from the prefix by a ':', MUST correspond to the vodml-id of the referenced element in the VO-DML/XML representation
      of that model.
      </xs:documentation>
    </xs:annotation>
    <xs:restriction base="xs:string">
      <xs:pattern value="[\w_-]+:[\w_\-/\./*]+" />
    </xs:restriction>
  </xs:simpleType>

	<xs:complexType name="VODMLOptionMapping">
	<xs:annotation>
	  <xs:documentation>
	  Allows one to map particular values defined in a VALUES/OPTION list to enumerayion literals
	  in the VO-DML model.
	  </xs:documentation>
	</xs:annotation>
		<xs:sequence>
			<xs:element name="OPTION" type="xs:string" minOccurs="1" maxOccurs="1" />
			<xs:element name="LITERAL" type="VODMLReference" minOccurs="1" maxOccurs="1" />
		</xs:sequence>
	</xs:complexType>
<!--
-->
	<xs:complexType name="VODMLAnnotation">
		<xs:sequence>
			<xs:element name="ROLE" type="VODMLReference" minOccurs="0">
				<xs:annotation>
					<xs:documentation>
						MUST be provided on annotations of FIELDref and PARAMref and, if contained in a VO-DML annotated GROUP or PARAM.
						If not provided on a root GROUP, indicates that it represents a stand-alone object.
					</xs:documentation>
				</xs:annotation>
			</xs:element>
			<xs:element name="TYPE" type="VODMLReference" minOccurs="0" maxOccurs="unbounded">
				<xs:annotation>
					<xs:documentation>
						MUST reference the exact VO-DML type of the instance represented by the container of the annotation element.
						Q: should we make minOccurs="1"?
					</xs:documentation>
				</xs:annotation>
			</xs:element>
			<xs:element name="OPTION" type="VODMLOptionMapping" minOccurs="0" maxOccurs="unbounded">
				<xs:annotation>
					<xs:documentation>
						Identifies a mapping between an OPTION value and an EnumLiteral.
						Only to be used in PARAM/PARAMref/FIELDref.
						TODO make new subtype of VODMLAnnotation for these three contexts, extending ?
					</xs:documentation>
				</xs:annotation>
			</xs:element>
		</xs:sequence>
	</xs:complexType>
```

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

Regular expressions for mapping  {#sec:regexp .AppendixA}
===============================

Expressions for VOTable elements:

-   ELEMENT : any VOTable element that can be annotated, i.e. a GROUP,
    PARAM, FIELDref or PARAMref

-   E $\in$ F : VOTable element E is directly contained under F

-   E $\not\in$ F : E is not directly contained under F

-   E $\subset$ F : E has F in ancestry

-   E $\not\subset$ F : E has no F in ancestry

-   $\neg$E/VODML/ROLE : E’s VODML element has no ROLE

-   G[R] : an element G for which R is true

-   ...

-   $\supset$, $\not\supset$

-   ...

For VO-DML elements

-   **R**$\in$**T** : VO-DML Role **R** is defined on **T**

-   **R**$\subset$**T** : VO-DML Role **R** is „available“ on **T**, i.e. **R**
    is defined on **T** or a supertype of **T**

-   **R**$\not\subset$**T** : VO-DML Role **R** is not available“ on **T**, i.e.
    **R** is not defined on **T** or any supertype of **T**

-   **...**

Mixed:

-   E/VODML/TYPE $\Rightarrow$ **D** : VOTable element E identifies VO-DML concept
    **D** as its VODML/TYPE

-   E/VODML/TYPE = “a:b.c” : VOTable element E’s VODML element has a
    TYPE with value “a:b.c”

-   ...

-   {GROUP G| G $\in$ VOTABLE & G/VODML/TYPE=”*vo-dml:Model*” &
    $\neg$G/VODML/ROLE} :

    -   Model declaration

    -   A GROUP G is contained directly under the VOTABLE element, it
        has a VODML element with TYPE set to “vo-dml:Model” and
        without ROLE.

-   {GROUP G | G $\not\subset$ TABLE & G $\not\subset$ GROUP[VODML &
    VODML/TYPE$\Rightarrow$**ObjectType** & $\neg$G/VODML/ROLE} :

    -   standalone/singleton **ObjectType** instance

    -   A GROUP G that is not contained in a TABLE, nor in a GROUP with
        a VODML annotation. It has a VODML element with TYPE indicating
        an **ObjectType** and no ROLE element

-   {GROUP G | G $\not\subset$ TABLE & VODML/TYPE$\Rightarrow$**DataType** & $\neg$G/VODML/ROLE} :
    standalone/singleton object type instance

-   -   {ELEMENT E| E $\in$ GROUP[VODML & $\neg$E/VODML/ROLE}

    -   Any element inside a GROUP that has a VODML annotation, MUST
        have a VODML/ROLE

-   {ELEMENT E| E[VODML & E $\not\subset$ GROUP[VODML & $\neg$E/VODML/TYPE},\
    {ELEMENT E| E[VODML & E $\not\subset$ GROUP[VODML & E/VODML/ROLE}

    -   Any element with VODML annotation, that does NOT have an
        ancestor GROUP with a VODML annotation, MUST have a TYPE and
        must NOT have a ROLE MUST have a VODML/ROLE

-   {ELEMENT E| E[VODML & E/VODML/ROLE & E/VODML/TYPE\
    & E/VODML/ROLE $\not\subset$ E/VODML/TYPE}

    -   For an element with both a TYPE and ROLE annotation, the ROLE
        MUST be available on the declared TYPE.

-   

Frequently Asked Questions {#sec:FAQ .AppendixA}
==========================

**Q: I was used to discover elements in a VOTable by trivially matching
strings: can I still do it?**

A: Yes, actually even more so: in fact, according to this specification,
the very same string for the very same element will be present in any
context that includes such element. For example, if you want to find the
error bar of an element you can simply look for a VO-DML-ref like
Accuracy.StatError, which does not depend on whether the error refers to
a Flux or a Wavelength. Consider the real case of SPLAT, a science
application that deals with spectra: in order to use the “old” UTYPEs,
SPLAT cannot trivially match strings, but uses a regular expression
(even though very simple) to match the \*.Accuracy.\* portion of a
UTYPE. With the new specification in place, SPLAT could just trivially
match strings by checking that they are equal.

**Q: Will this specification increase the number of UTYPEs?

(**TODO** [TBD as UTYPEs are no longer relevant in this context, remove this
question?)

A: No. On the contrary, it will dramatically reduce them: consider for
instance an Accuracy type, which basically encodes error bars. It is
composed of few attributes that express symmetric and asymmetric error
bars, systematic errors, upper and lower limits. In the new
specification they can be univocally referenced by less than ten UTYPEs.
In the “old” scheme, each of these elements was bound to other elements
(e.g. the spectral axis, the flux axis, and so on): this means that for
expressing the same concepts you needed more than one hundred UTYPEs
only in the Spectrum Data Model!

Consider IRIS, an application that deals with SEDs and spectra: in this
application about a thousand UTYPEs needed to be hard-coded in order to
fully represent instances and provide a Java library for their I/O and
manipulation. With this specification IRIS would need one order of
magnitude less UTYPEs to be hard-coded, and could actually even make
without them, by simply using the VO-DML description and a set of
intelligently designed Java interfaces and objects.

**Q: Will this specification increase the number of Data Models?**

A: The simple answer is: No. This spec is agnostic about which data
models there are. However, a whole lot of Data Models are already out
there, since basically each astronomical instrument needs an ad-hoc Data
Model. This specification allows such Data Models to be expressed in a
standard, machine-readable format *in terms* of common, more generic
Data Models defined and maintained by the IVOA. So, the extended answer
is: No, but it will effectively increase the number of *interoperable*
Data Models, allowing Data Providers to register their own Models in an
IVOA registry.

**Q: Am I now forced to use UML for using this specification?**

A: No. However, part of the larger picture includes UML as a convenient
tool to design, display, document, and register Data Models. The larger
picture of which this specification is part includes a VO-UML
specification in the form of a UML profile that can make the creation of
Data Models more standard and easier. However, using VO-UML is not a
requirement.

**Q: Am I now forced to use specific software for using this
specification?**

A: No. However, this specification enables the development of software
that can make the creation and use of Data Models easier. For example,
such software might allow the automatic generation of documentation,
code, web applications, and services from a simple set of UML diagrams.
Some software was already created as part of this specification effort,
or other IVOA efforts from which this specification was derived.

**Q: Does this specification break the current standards and
protocols?**

A: No. This specification does not require any changes in services and
applications that implement current IVOA standards and protocols.
However, it also enables new, more complex standards to be designed and
implemented. Files complying with old standards can be made compliant
with this specification by simply adding metadata to them.

**Q: Can I use UTYPEs in a customized way without violating this
specification?**

A: Yes. This specification has nothing to say about UTYPEs.

**Q:** **It looks like this specification needs clients to recursively
concatenate strings to build the actual UTYPE. Is this the case? **

A: No. UTYPEs are defined so to be opaque strings that do not need to be
parsed or to be concatenated in order to be used. They are simply
references that map the VOTable elements to a standard “model of
models”.

**Q: This specification is so complicated! Do I need to implement it
all, even just for extracting a flux from a VOTable?**

A: No. And it is not that complicated, actually. As a DNA strand is a
simple concatenation of four basic components, but it is used in nature
to build complex organisms, this specification shows how simple
identifiers can be used to enable complex scientific applications. How
complex an implementation is depends on the implementation requirements:
if they are simple, the specification enables you to meet them in a very
simple way. For example, you can extract a flux by simply looking for a
`@utype` attribute like src:source.LuminosityMeasurement.value. If the
requirements for your application are more complex, then this
specification makes the best effort to enable them straightforwardly. If
you are a Data Provider and you want to publish compliant files and
databases, the complexity of the task depends pretty much on the
complexity of your files: they might be very simple structures of
VOTable groups, or they could require complicated Object-Relational
Mapping patterns. However, software can be built using this
specification in order to make the data publishing process easier.

**Q: Should I parse UTYPEs?**

A: No. [It is still to be discussed whether prefixes are fixed or not.
In the second case, some strategies, but not all, might require clients
to strip the prefix off the ‘localname’ in order to look it up in the
VODML/XML, in a way similar to what happens now, e.g. obs:Target.Name vs
sdm:Target.Name.


References
==========

1.  <span id="_Ref412877457" class="anchor"></span>VO-DML document

2.  <span id="_Ref417881604" class="anchor"></span>VOTable document(s)

3.  <span id="_Ref417883681" class="anchor"></span>Simulation DM spec

4.  Characterization spec

5.  Spectrum DM spec

6.  STC spec

7.  <span id="_Ref417880637" class="anchor"></span>ORM mapping tools:
    Hibernate, JPA spec

8.  <span id="_Ref413820347" class="anchor"></span>UCDs

9.  <span id="_Ref413820452" class="anchor"></span>Investigation into
    existing UTYPE usages, document form UTYPEs tiger team.

10. <span id="_Ref413820652" class="anchor"></span>Note on STC in
    VOTable (M. Dempleitner)

11. <span id="_Ref414176449" class="anchor"></span>PR on use of SKOS
    vocabularies in IVOA

12. <span id="_Ref414949253" class="anchor"></span>IVOA Identifiers

13. <span id="_Ref417883709" class="anchor"></span>VO-URP
