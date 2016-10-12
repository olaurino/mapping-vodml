# Examples: Mapping VO-DML to VOTable #

This section shows examples of mappings as they may occur in realistic
VOTables, using example Data Models. We will describe how the different
VO-DML elements must be serialized and how annotations employing the
<VODML> element can help one interpret the VOTable. In the later
normative section we turn this around and explicitly list all legal
annotations, their constraints and interpretation.

In this section we list some mappings from VO-DML to VOTable. We use
examples extracted from a sample model with sample instances described
in the next section. These should be seen as an introduction to the
complete and formal specification in section \ref{6.7.3} on how one MUST use
VODML annotation in VOTable to indicate mapping to VO-DML.

In the next few subsections we provide some examples how this mapping
can be performed. It starts with the **`ObjectType`** and then discusses
its components, **`Attribute`**, **`Reference`** and **`Collection`**. The
mapping of the value types is discussed in the mapping of
**`Attribute`**s. The mapping of **`Reference`** and **`Collection`**
relations is the most complex part of the whole mapping story and
treated separately.

Notation:

-   Concepts from the VO-DML metamodel are in **`Courier boldface`**

-   When referring to elements form a concrete data model either by name
    or **`vodmlref`** we use in *italics*, e.g. *Source* or
    *src:source.Source*. We generally refer to data models by their
    short name that also should be used as prefix (but see the
    discussion in section \ref{4.2}). In this spec we rfer to the following
    models:

    -   *ivoa*: TBD

    -   *src*: TBD

    -   *photdm-alt*: TBD

    -   *vo-dml*: TBD

-   VOTable concepts are normal font, generally all-caps.

-   VODML/ROLE ad VODML/TYPE are used to refer to the corresponding
    elements of the <VODML> element.

-   XML attributes have an ‘@’ prefix

## Mapping ObjectType ##


**`ObjectTypes`** consist of **`Attributes`**, **`References`** and
**`Compositions`**. How these are mapped is described in more detail
below. But the important part for representing structured instance like
an object is that these components must be combined together to
construct a complete instance. In VOTable this is done using a GROUP
element.

In the representation of **`ObjectType`** instances, GROUPs can be used in
two different modes that are distinguished by the way the instance’s
data are ultimately stored. If all values are eventually stored
exclusively in @value attributes of PARAM elements in the GROUP (or
possibly outside the GROUP but accessed through PARAMrefs), the GROUP
represents a complete instance *directly*.

We generally refer to such stand-alone objects as *Singleton Objects*.
If even only one of the attributes is stored in a FIELD and accessed
through a FIELDref, the GROUP is said to *indirectly* represent possibly
multiple instances, one for each TR. We refer to these as *Table
Objects*. We will refer to the corresponding mapping as direct and
*indirect* all through the document. This freedom of choice *where* to
store objects actually complicates the mapping of *relations* between
objects, as many different referencing mechanisms must be taken into
account. This is particularly important when discussing how to represent
References in section ref{sect7.4}.

We illustrate the two modes of mapping by showing an example how each
mode may represent exactly the same object. For this we use the object
type in Figure \ref{fig3} and a corresponding instance in Figure \ref{fig4}.

\![ObjectType representing a Source. \[TBD Update to latest version of sample model\]](media/image5.png)



\![Instance of Source ObjectType in UML. \[TBD Update to latest version of sample model\]](media/image6.png)


### Indirect serialization to a TABLE ###

As an example of an indirect mapping (i.e. an **`ObjectType`** in a TABLE)
the *Source* instance from Figure \ref{fig4} is stored in the first TR in the
VOTable snippet below. The TABLE is annotated using a GROUP with a
VODML/TYPE indicating that it represents a *Source* from the *src*
model. Some atomic attributes are stored in FIELDs annotated by
FIELDref-s, some in PARAMs; the child GROUP with its attributes
annotated by FIELDref represents a structured attribute. Also, the
*src:source,SkyCoordinate.frame* identifies a VO-DML Reference and is
represented by a GROUP with a @ref attribute. When annotated with a
VODML element we will refer to this pattern as a “GROUPref” (generally
including the double quotes), which will be discussed in section \ref{sect6.3}:


| &lt;TABLE>
|   &lt;GROUP ID="_source">
|     **&lt;VODML>TYPE>src:source.Source&lt;/TYPE>&lt;/VODML>**
|     &lt;FIELDref ref="_designation">
|       **&lt;VODML>&lt;ROLE>vo-dml:ObjectType.ID&lt;/ROLE>&lt;/VODML>**
|     &lt;/FIELDref>
|     &lt;FIELDref ref="_designation">
|       **&lt;VODML>&lt;ROLE>src:source.Source.name&lt;/ROLE>&lt;/VODML>**
|     &lt;/FIELDref>
|     &lt;GROUP>
|       **&lt;VODML>**
|         **&lt;ROLE>src:source.Source.position&lt;/ROLE>**
|         **&lt;TYPE>src:source.SkyCoordinate&lt;/TYPE>**
|       **&lt;/VODML>**
|       &lt;FIELDref ref="_ra">
|         **&lt;VODML>&lt;ROLE>**src:source.SkyCoordinate.longitude**&lt;/ROLE>&lt;/VODML>**
|       &lt;/FIELDref>
|       &lt;FIELDref ref="_dec">
|          **&lt;VODML>&lt;ROLE>src:source.SkyCoordinate.latitude&lt;/ROLE>&lt;/VODML>**
|       &lt;/FIELDref>
|       &lt;GROUP ref="_icrs">
|         **&lt;VODML>&lt;ROLE>src:source.SkyCoordinate.frame&lt;/ROLE>&lt;/VODML>**
|       &lt;/GROUP>
|     &lt;/GROUP>
|    &lt;/GROUP>
|   &lt;FIELD name="designation" ID="_designation" .../>
|   &lt;FIELD name="ra" ID="_ra" unit="deg" .../>
|   &lt;FIELD name="dec" ID="_dec" unit="deg" .../>
|   &lt;TR>&lt;TD>08120809-0206132&lt;/TD>
|     &lt;TD>123.033734&lt;/TD>
|     &lt;TD>-2.103671&lt;/TD>&lt;/TR>
|   ...
| &lt;/TABLE>


Note the special representation of the identifier of the *Source*
object, itself identified by the *vo-dml:ObjectType.ID* **`vodmlref`**.
This attribute is not defined explicitly in the model, i.e. on *Source*;
hence no **`vodml-id`** exists for the attribute to identify it in the
*src* model itself. In this mapping spec, which describes how to
represent instances of VO-DML models in serializations, we interpret
each VO-DML **`ObjectType`** as ultimately being a sub-type of
*vo-dml:ObjectType* that is defined in the aforementioned *vo-dml*
model. This can be compared to the way in Java all classes ultimately
extend java.lang.Object, generally implicitly.

In serializations this implies we can add to any VO-DML **`ObjectType`**
attributes defined on the *vo-dml:ObjectType* type. For example, since
that type (which is itself a VO-DML **`ObjectType`**[^11]) defines an *ID*
attribute, this can be used on the representation of any **`ObjectType`**.

### Direct serialization to a GROUP ###

Here the instance is directly represented by a GROUP containing only
PARAMs for the atomic attributes, and a GROUP with PARAMs for the
structured attribute (and again a reference, see section \ref{sect6.3}).

~~~~~~
<GROUP>
**<VODML><TYPE>src:source.Source</TYPE></VODML>**
<PARAM value=”08120809-0206132” ...>
**<VODML><ROLE>vo-dml:ObjectType.ID</ROLE></VODML>**
</PARAM>
<PARAM value=”08120809-0206132” ... ...>
**<VODML><ROLE>src:source.Source.name</ROLE></VODML>**
</PARAM>
<PARAM value=”galaxy” ... ...>
**<VODML><ROLE>src:source.Source.classification</ROLE></VODML>**
</PARAM>
<GROUP>
**<VODML>**
**<ROLE>src:source.Source.position</ROLE>**
**<TYPE>src:source.SkyCoordinate</TYPE>**
**</VODML>**
<PARAM value=”123.033734” ...>
**<VODML><ROLE>src:source.SkyCoordinate.longitude</ROLE></VODML>**
</PARAM>
<PARAM value=”-2.103671” ...>
**<VODML><ROLE>src:source.SkyCoordinate.latitude</ROLE></VODML>**
</PARAM>
<GROUP ref="_icrs">
**<VODML><ROLE>src:source.SkyCoordinate.frame</ROLE></VODML>**
</GROUP>
</GROUP>
</GROUP>
~~~~~~

Details on the mapping of the components are discussed next.

## Mapping Attribute ##


An attribute is the role a value type plays in the definition of a
structured type. They may represent atomic or may represent structured
(Data)types themselves. These are represented differently.

\![The ObjectType *Source* and the DataType *SkyCoordinate* both define attributes. Attributes can represent a PrimitiveType (‘*name’* and ‘*description’* in *Source*, ‘*longitude’* and ‘*latitude’* in *SkyCoordinate*), an Enumeration (‘*classification’* in *Source*), or a structured DataType (‘*position’* in *Source*).](media/image5.png) \![](media/image7.png)


### PrimitiveType attribute as FIELDref, Enumeration as PARAM###

In the indirect representation of the **`ObjectType`** below a FIELDref
indicates that an attribute is stored in the FIELD with
`ID=”_designation”`. It does so using the **`vodmlref`**
*src:source.Source.name* in the VODML/ROLE element, identifying the
*name* attribute of the *Source* type. Note that in our example we use a
**`vodml-id`** syntax derived from the VO-DML model itself. From this
string one might here infer directly that some role with name ‘*name’*
defined on a type named *Source* is represented. But that is all one
might infer. In principle this could have been a string like
‘*src:123456789*’. In both cases one should inspect the formal data
model to find out precisely *what* kind of model element this
**`vodmlref`** represents.

Another attribute, ‘*classification’*, is represented by a PARAM as
indicated by its VODML/ROLE *src:source.Source.classification* and is
assigned the value ‘*galaxy’*. The attribute has as datatype an
**`Enumeration`**, *SourceClassification* and indeed the PARAM defines a
VALUES element with various OPTIONs (note that such a list is basically
useless for a PARAM that represents a single value directly).

In this case the set of OPTIONS indeed contains a value ‘*galaxy’*. In
general however, especially for existing “legacy” databases, one cannot
expect that enumerated values will exactly correspond to those in a data
model. Some type of mapping is required and this is supported using the
OPTION element on VODML, as explained further in section \ref{7.5.1}. The fact
that this attribute is stored in a PARAM in the GROUP indicates that all
*Source* instances stored in the TABLE are classified as galaxies. A
more realistic case would require the use of a FIELDref to assign a TD
value to each instance (row) in the table. \[TBD example of SDSS, with
OPTION mapping\]

~~~~~~
<TABLE>
<GROUP>
<VODML><TYPE>src:source.Source</TYPE></VODML>
<FIELDref ref="_designation" utype=" vo-dml:ObjectType.ID"/>
<FIELDref ref="_designation" utype="src:source.Source.name"/>
<PARAM name=”type” value=”galaxy”>
**<VODML><ROLE>src:source.Source.classification<ROLE>**
**<OPTION>**
**<VALUE>galaxy</VALUE>**
**<LITERAL>src:source.SourceClassification.galaxy</LITERAL>**
**</OPTION>**
**<OPTION>**
**<VALUE>star</VALUE>**
**<LITERAL>src:source.SourceClassification.star</LITERAL>**
**</OPTION>**
**</VODML>**
<VALUES><OPTION value=”galaxy”/><OPTION
value=”star”/>...</VALUES>
</PARAM>
<GROUP>
**<VODML>**
**<ROLE>src:source.Source.position</ROLE>**
**<TYPE>src:source.SkyCoordinate</TYPE>**
**</VODML>**
<FIELDref ref="_ra">
**<VODML><ROLE>src:source.SkyCoordinate.longitude</ROLE></VODML>**
</FILDEref>
<FIELDref ref="_dec">
**<VODML><ROLE>src:source.SkyCoordinate.latitude</ROLE></VODML>**
</FILDEref>
<GROUP ref="_icrs">\
**<VODML><ROLE>src:source.SkyCoordinate.frame</ROLE></VODML>**
</GROUP>
</GROUP>
</GROUP>
<FIELD name="designation" ID="_designation" .../>
<FIELD name="ra" ID="_ra" unit="deg" .../>
<FIELD name="dec" ID="_dec" unit="deg" .../>
<TR><TD>08120809-0206132</TD><TD>123.033734</TD><TD>-2.103671</TD></TR>
...
</TABLE>
~~~~~~

**DataType attribute as GROUP:**

As DataType-s are structured, their natural representation in a VOTable
is as a GROUP, whether used directly or indirectly. Hence if an
attribute is defined in a VO-DML data model as representing a DataType,
it is most naturally represented by a GROUP embedded in the GROUP of the
structured type owning the attribute.

The example below shows in red [TBD: no color in listings any more] a GROUP representing the attribute
‘position’ (identified by utype [TBD: utype!] src:source.Source.position) that has as
data type a SkyCoordinate, which itself consists of a ‘longitude’ and
‘latitude’ attribute (we defer discussing the reference to the next
section). Note their structure indicates *only* their relation to their
defining type, src:source.SkyCoordinate (though this, as discussed
above, is unimportant for the current approach which, apart from the
prefix, assigns no importance to the syntax of the utype identifiers in
VO-DML models).

~~~~~~
<TABLE>
<GROUP>
<VODML><TYPE>src:source.Source</TYPE></VODML>
<FIELDref ref="_designation">
<VODML><ROLE>vo-dml:ObjectType.ID</ROLE></VODML>
</FIELDref>
<FIELDref ref="_designation">
<VODML><ROLE>src:source.Source.name</ROLE></VODML>
</FIELDref>
<PARAM name=”type” value=”galaxy”>
<VALUES><OPTION value=”galaxy”/><OPTION
value=”star”/>...</VALUES>
<VODML><ROLE>src:source.Source.classification</ROLE></VODML>
</PARAM>
**<GROUP>**
**<VODML>**
**<ROLE>src:source.Source.position</ROLE>**
**<TYPE>src:source.SkyCoordinate</TYPE>**
**</VODML>**
**<FIELDref ref="_ra">**
**<VODML><ROLE>src:source.SkyCoordinate.longitude</ROLE></VODML>**
**</FIELDref>**
**<FIELDref ref="_dec">**
**<VODML><ROLE>src:source.SkyCoordinate.latitude</ROLE></VODML>**
**</FIELDref>**
**<GROUP ref="_icrs">**
**<VODML><ROLE>src:source.SkyCoordinate.frame</ROLE></VODML>**
**</GROUP>**
**</GROUP>**
</GROUP>
<FIELD name="designation" ID="_designation" .../>
<FIELD name="ra" ID="_ra" unit="deg" .../>
<FIELD name="dec" ID="_dec" unit="deg" .../>
<TR><TD>08120809-0206132</TD><TD>123.033734</TD><TD>-2.103671</TD></TR>
...
</TABLE>
<FIELD name="dec" ID="_dec" unit="deg" .../>
<TR><TD>08120809-0206132</TD><TD>123.033734</TD><TD>-2.103671</TD></TR>
...
</TABLE>
~~~~~~

In this example the utype of the child GROUP *only* indicates its role
in the definition of its parent GROUP, namely the attribute
src:source.Source.position. It does not indicate the type, which is
instead indicated by the PARAM with name “type”.

## Mapping Reference ##


\![](media/image8.png)

\![](media/image9.png)

### Referencing as ”GROUPref” to direct GROUP ###

The example below uses a GROUP+@ref to represent the reference from a
position object stored in a TABLE to a SkyCoordinateFrame stored in a
direct GROUP. Hence all rows in the table use the same frame and the
reference needs no structure.

~~~~~~
<GROUP **ID="_icrs"**>
<VODML><TYPE>src:source.SkyCoordinateFrame</TYPE></VODML>
<PARAM value="ICRS" ...>
<VODML><ROLE>src:source.SkyCoordinateFrame.name</ROLE></VODML>
</PARAM>
<PARAM value="J2000.0" ...>
<VODML><ROLE>src:source.SkyCoordinateFrame.equinox</ROLE></VODML>
</PARAM>
</GROUP>
<TABLE>
<GROUP>
<VODML><TYPE>src:source.Source</TYPE></VODML>
<FIELDref ref="_designation">
<VODML><ROLE>vo-dml:ObjectType.ID</ROLE></VODML>
</FIELDref>
<FIELDref ref="_designation">
<VODML><ROLE>src:source.Source.name</ROLE></VODML>
</FIELDref>
<GROUP>
<VODML>
<ROLE>src:source.Source.position</ROLE>
<TYPE>src:source.SkyCoordinate</TYPE>
</VODML>
<FIELDref ref="_ra" >
<VODML><ROLE>src:source.SkyCoordinate.longitude</ROLE></VODML>
</FIELDref>
<FIELDref ref="_dec">
<VODML><ROLE>src:source.SkyCoordinate.latitude</ROLE></VODML>
</FIELDref>
**<GROUP ref="_icrs">**
**<VODML><ROLE>src:source.SkyCoordinate.frame</ROLE></VODML>**
**</GROUP>**
</GROUP>
</GROUP>
<FIELD id="_ designation " name="parentId" datatype="char"/>
<FIELD id="_ra" name="ra" datatype="float"/>
<FIELD id="_dec" name="dec" datatype="float"/>
...
<DATA><TABLEDATA>
<TR><TD>08120809-0206132</TD><TD>123.034</TD><TD>-2.1037</TD>...</TR>
...
</TABLEDATA></DATA>
</TABLE>
~~~~~~

## Mapping Composition ##


A collection is a composition relation between a parent ObjectType and a
child, or *part*, ObjectType. The fact that objects can be stored in
different ways implies many different ways in which the relation may
need to be expressed. In XML serializations of a data model (such as
used in VO-URP and the Simulation Data Model) one may choose to have the
contained objects serialized *inside* the serialization of the parent.
It is possible to do so in VOTable as well using child GROUPs
representing a complete child object embedded in the GROUP representing
the parent object. This is natural for this relationship because a
collection element is really to be considered as a *part* of the parent
object.

This may be used to represent flattening of the parent-child relation.
One or more child objects may be stored together with the parent object
in the same row in a TABLE. Such a case is actually very common, for
example when interpreting tables in typical source catalogues. These
generally contain information of a source together with one or more
magnitudes. The latter can be seen as elements form a collection of
photometry points contained by the sources.

Hence a GROUP inside another GROUP MAY represent a collection. It can do
so in different modes that are illustrated by the following examples and
described in more detail in section \ref{sect0}.

\![](media/image10.png)\![](media/image11.png)

**Container and contained objects in same row in same TABLE:**

A very typical case is the following example, which shows a Source
object serialized in the same row as its luminosities. The Source data
model models luminosity measurements as a collection, which is flexible
and allows measurements from different sources to be combined in a
natural manner. But for a given catalogue such as SDSS or 2MASS, it is
known *a priori* how many magnitudes will be supplied and which bands
they will correspond to. Hence for a given catalogue the natural
representation is to simply add these as attributes to their model.
Interestingly enough, the approach proposed here is able to support this
mapping without any problem, even making a natural link to the actual
photometry filter used for the measurements.

~~~~~~
<TABLE>
<GROUP>
**<VODML><TYPE>src:source.Source</TYPE></VODML>**
<FIELDref ref="_designation">
<VODML><ROLE>src:source.Source.name</ROLE></VODML>
</FIELDref>
<GROUP>
**<VODML>**
**<ROLE>src:source.Source.position</ROLE>**
**<TYPE>src:source.SkyCoordinate</TYPE>**
**</VODML>**
<FIELDref ref="_ra">
**<VODML><ROLE>src:source.SkyCoordinate.longitude</ROLE></VODML>**
</FIELDref>
<FIELDref ref="_dec">
**<VODML><ROLE>src:source.SkyCoordinate.latitude</ROLE></VODML>**
</FIELDref>
<GROUP ref="_icrs">
**<VODML><ROLE>src:source.SkyCoordinate.frame</ROLE></VODML>**
</GROUP>
</GROUP>
<GROUP>
**<VODML>**
**<ROLE>src:source.Source.luminosity</ROLE>**
**<TYPE>src:source.LuminosityMeasurement</TYPE>**
**</VODML>**
<FIELDref ref="_magJ"">
**<VODML><ROLE>src:source.LuminosityMeasurement.value</ROLE></VODML>**
</FIELDref>
<FIELDref ref="_errJ"">
**<VODML><ROLE>src:source.LuminosityMeasurement.error</ROLE></VODML>**
</FIELDref>
<GROUP ref="2massJ"">
**<VODML><ROLE>src:source.LuminosityMeasurement.filter</ROLE></VODML>**
</GROUP>
</GROUP>
<GROUP>
**<VODML>**
**<ROLE>src:source.Source.luminosity</ROLE>**
**<TYPE>src:source.LuminosityMeasurement</TYPE>**
**</VODML>**
<FIELDref ref="_magK">
**<VODML><ROLE>src:source.LuminosityMeasurement.value</ROLE></VODML>**
</FIELDref>
<FIELDref ref="_errK"">
**<VODML><ROLE>src:source.LuminosityMeasurement.error</ROLE></VODML>**
</FIELDref>
<GROUP ref="2massK"">
**<VODML><ROLE>src:source.LuminosityMeasurement.filter</ROLE></VODML>**
</GROUP>
</GROUP>
<GROUP">
**<VODML>**
**<ROLE>src:source.Source.luminosity</ROLE>**
**<TYPE>src:source.LuminosityMeasurement</TYPE>**
**</VODML>**
<FIELDref ref="_magH">
**<VODML><ROLE>src:source.LuminosityMeasurement.value</ROLE></VODML>**
</FIELDref>
<FIELDref ref="_errH"">
**<VODML><ROLE>src:source.LuminosityMeasurement.error</ROLE></VODML>**
</FIELDref>
<GROUP ref="2massH"">
**<VODML><ROLE>src:source.LuminosityMeasurement.filter</ROLE></VODML>**
</GROUP>
</GROUP>
</GROUP>
<FIELD name="designation" ID="_designation" >
<DESCRIPTION>source designation formed from *sexigesimal*
coordinates</DESCRIPTION>
</FIELD>
<FIELD name="ra" ID="_ra" unit="deg">
<DESCRIPTION>right ascension (J2000 decimal
*deg*)</DESCRIPTION>
</FIELD>
<FIELD name="dec" ID="_dec" unit="deg">
<DESCRIPTION>declination (J2000 decimal *deg*)</DESCRIPTION>
</FIELD>
<FIELD name="magJ" ID="_magJ">
<DESCRIPTION>J magnitude</DESCRIPTION>
</FIELD>
<FIELD name="errJ" ID="_errJ" unit="deg">
<DESCRIPTION>J magnitude error</DESCRIPTION>
</FIELD>
FIELD name="magH" ID="_magH">
<DESCRIPTION>H magnitude</DESCRIPTION>
</FIELD>
<FIELD name="errH" ID="_errH" unit="deg">
<DESCRIPTION>H magnitude error</DESCRIPTION>
</FIELD>
FIELD name="magK" ID="_magK">
<DESCRIPTION>K magnitude</DESCRIPTION>
</FIELD>
<FIELD name="errK" ID="_errK" unit="deg">
<DESCRIPTION>K magnitude error</DESCRIPTION>
</FIELD>
<TR>
<TD>08120809-0206132</TD><TD>123.033734</TD><TD>-2.103671</TD><TD>23.2</TD><TD>.04</TD>
<TD>23.0</TD><TD>.03</TD>
<TD>23.5</TD><TD>.03</TD>
</TR>
...
</TABLE>
~~~~~~

**Container and collection all as direct instances in GROUPs:**

When not using tables at all in a mapping, the individual elements from
a collection can be directly represented inside the parent GROUP.

~~~~~~
<GROUP>
**<VODML><TYPE>src:source.Source</TYPE><VODML>**
<GROUP>
<VODML><ROLE>src:source.Source.luminosity</ROLE></VODML>
<GROUP>
**<VODML><TYPE>src:source.LuminosityMeasurement</TYPE></VODML>**
<PARAM value=”23.2” ...>
**<VODML><ROLE>src:source.LuminosityMeasurement.value</ROLE></VODML>**
</PARAM>
<PARAM value=”.04” ...>
**<VODML><ROLE>src:source.LuminosityMeasurement.error</ROLE></VODML>**
</PARAM>
<GROUP ref="2massJ">
**<VODML><ROLE>src:source.LuminosityMeasurement.filter</ROLE></VODML>**
</GROUP>
...
</GROUP>
~~~~~~

## Mapping value types ##


The examples above started from the assumption that the basis of a
serialization was an **`ObjectType`**. Value types only show up as
attributes. There may be some use cases however where one wishes to
indicate *only* that a certain column or set of column represent some
known value type. One reason may be that a standard, global data model
does not exist that defines **`ObjectType`**-s matching the one in one’s
serialization, but that one can identify some sub-components that could
be mapped to a **`DataType`** for example.

In fact the SourceDM used here is an example. It is a model for
*Source*-s. The IVOA does currently not have an accepted model for this
concept, though attempts in this direction have been made[^12]. This
implies many tables of interest in the VO can currently not formally
declare they store instances of a *Source*. However they could declare
they have columns that together correspond to a coordinate on the sky in
the STC model. This could lead to a VOTable fragment as the following,
which is a version of an example in 6.2, but with altered VO-DML
annotation, and removal of the GROUP representing the *Source*.

~~~~~~
<TABLE>
**<GROUP> **
**<VODML><TYPE>src:source.SkyCoordinate</TYPE></VODML>**
**<FIELDref ref="_ra">\
<VODML><ROLE>src:source.SkyCoordinate.longitude</ROLE></VODML>**
**</FIELDref>**
**<FIELDref ref="_dec">\
<VODML><ROLE>src:source.SkyCoordinate.latitude</ROLE></VODML>**
**</FIELDref>**
**<GROUP ref="_icrs">\
<VODML><ROLE>src:source.SkyCoordinate.frame"</ROLE></VODML>**
**</GROUP>**
**</GROUP>**
<FIELD name="designation" ID="_designation" .../>
<FIELD name="ra" ID="_ra" unit="deg" .../>
<FIELD name="dec" ID="_dec" unit="deg" .../>
<TR><TD>08120809-0206132</TD><TD>123.033734</TD><TD>-2.103671</TD></TR>
...
</TABLE>
~~~~~~

Tools that understand STC may be able to do something with this
annotation, even though they cannot know what role the coordinate plays.

Another example arises from a possible access protocol specification.
Say Simple Cone Search (SCS) would declare that the result of a request
MUST be a VOTable with a GROUP representing the actual request
consisting of a position and a search radius. It could insist the
position must be serialized using a GROUP representing an STC coordinate
following our data modeling serialization prescription. E.g. as in the
following example:

~~~~~~
<GROUP ID="_icrs">
<VODML><TYPE>src:source.SkyCoordinateFrame</TYPE></VODML>
<PARAM value="ICRS" ...>
**<VODML><ROLE>src:source.SkyCoordinateFrame.name</ROLE></VODML>**
</PARAM>
<PARAM value="J2000.0" ...>
**<VODML.ROLE>src:source.SkyCoordinateFrame.equinox</ROLE></VODML>**
</PAARAM>
</GROUP>
...
<GROUP name="SCS">
<INFO value="The SCS request "/>
<PARAM name="SR" datatype="float">
**<VODML><TYPE>ivoa:real</TYPE></VODML>**
<GROUP name="center">
**<VODML><TYPE>src:source.SkyCoordinate</TYPE></VODML>**
<INFO value="The center coordinate of the simple cone search"/>
<PARAM name="ra" value="123.00000" datatype="float">
**<VODML><ROLE>src:source.SkyCoordinate.longitude</ROLE></VODML>**
</PARAM>
<PARAM name="dec" value="-2.10000" datatype="float">
**<VODML><ROLE>src:source.SkyCoordinate.latitude</ROLE></VODML>**
</PARAM>
<GROUP ref="_icrs"">
**<VODML><ROLE>src:source.SkyCoordinate.frame</ROLE></VODML>**
</GROUP>
</GROUP>
</GROUP>
...
~~~~~~

Note, this example even assigns a VODML/TYPE to the search radius SR,
identifying it as the **`PrimitiveType`** *ivoa:real*. This shows that
also **`PrimitiveType`**-s and **`Enumerations`** can be used directly, i.e.
outside of a role they play.

Note furthermore that the GROUP named “center” does not have a
VODML/ROLE, only a VODML/TYPE, even though it is contained inside
another GROUP. This is legal as a VODML annotation because that parent
GROUP is *not* mapped to a data model. One could imagine the SCS
protocol defining a little standard data model, say *scs* to be used in
the serialization of its results. In that case also the parent group,
currently named “SCS’’, could have been declared to represent, say, an
*scs:SCSQuery* and *scs:SCSQuery.center* might be a possible VODML/ROLE
for the child GROUP as illustrated in the following example:

~~~~~~
<GROUP ID="_icrs">
**<VODML><TYPE>src:source.SkyCoordinateFrame</TYPE></VODML>**
<PARAM value="ICRS" ...>
**<VODML><ROLE>src:source.SkyCoordinateFrame.name</ROLE></VODML>**
</PARAM>
<PARAM value="J2000.0" ...>
**<VODML><ROLE>src:source.SkyCoordinateFrame.equinox</ROLE></VODML>**
</PARAM>
</GROUP>
...
<GROUP name="SCS">
**<VODML><TYPE>scs:SCSQuery</TYPE><VODML>**
<PARAM name="SR" datatype="float">
**<VODML><ROLE>scs:SCSQuery.SR</ROLE></VODML>**
</PARAM>
<GROUP name="center">
**<VODML>**
**<ROLE>scs:SCSQuery.center</ROLE>**
**<TYPE>src:source.SkyCoordinate</TYPE>**
**</VODML>**
<PARAM name="ra" value="123.00000" datatype="float">
**<VODML><ROLE>src:source.SkyCoordinate.longitude</ROLE></VODML>**
</PARAM>
<PARAM name="dec" value="-2.10000" datatype="float">
**<VODML><ROLE>src:source.SkyCoordinate.latitude</ROLE></VODML>**
</PARAM>
<GROUP ref="_icrs">
**<VODML><ROLE>src:source.SkyCoordinate.frame</ROLE></VODML>**
</GROUP>
</GROUP>
</GROUP>
...
~~~~~~

## Mapping Inheritance ##


We have introduced the Source Data Model to provide concrete mapping
examples. Let’s now assume that a Data Provider has some information
that pertains to astronomical sources, such as the redshift of the
source, and they want to serialize this information in their files. The
Source class in Source DM does not provide such a field. The solution is
for the Data Provider to *extend* the Source class in Source DM.

Notice that the Data Provider might as well decide to include the
information in a customized fashion, for example setting a particular
name for the redshift column. However, this customized annotation
requires readers to know the specifics of each custom file and their
annotation. Instead, by adopting a common framework and this mapping
language, the information can be provided to the user interactively and
consistently, even by model-agnostic clients.

It is important that naïve clients find the information about the parent
class without needing to parse anything but the VOTable. Advanced
applications must be able to provide the information about the child
class fields dynamically, while provider specific libraries can be
derived from standard libraries to implement new functions.

First of all, the Data Provider must include a VO-DML declaration
pointing to the description file and introducing a custom prefix:

~~~~~~
<GROUP name="Source">
<VODML><TYPE>vo-dml:Model</TYPE></VODML>
<PARAM name="url" datatype="char" arraysize="\*"
value="https://volute.googlecode.com/svn/trunk/projects/dm/vo-dml/models/sample/Sample.vo-dml.xml">
<VODML><ROLE>vo-dml:Model.url</ROLE></VODML>
</PARAM>
<PARAM value="src" name="prefix" datatype="char" arraysize="\*">
<VODML><ROLE>vo-dml:Model.name</ROLE></VODML>
</PARAM>
</GROUP>
**<GROUP name="ExtendedSource">**
**<VODML><TYPE>vo-dml:Model</TYPE></VODML>**
**<PARAM name="url" datatype="char" arraysize="\*"
value="https://volute.googlecode.com/svn/trunk/projects/dm/vo-dml/models/sample/ExtendedSource.vo-dml.xml">**
**<VODML><ROLE>vo-dml:Model.url</ROLE></VODML>**
**</PARAM>**
**<PARAM name="uri" datatype="char" arraysize="\*"
value="ivo://ivoa.net/std/ExtendedSourceDM" />**
**<PARAM utype="vo-dml:Model.prefix" value="xsrc" name="prefix"
datatype="char" arraysize="\*">**
**<VODML><ROLE>vo-dml:Model.name</ROLE></VODML>**
**</PARAM>**
**</GROUP>**
~~~~~~

In the VO-DML/XML document of the new model, the new field might have
the **`vodml-id`** ‘*source.Source.redshift*’, which translate to the
**`vodmlref`** ‘*xsrc:source.Source.redshift*’. This **`vodmlref`** can be
used to annotate instances of the extended class, like in the following
example:

~~~~~~
<TABLE>
<GROUP>
<VODML>
<TYPE>xsrc:source.Source</TYPE>
<TYPE>src:source.Source</TYPE>
</VODML>
<FIELDref ref="_designation" utype=" vo-dml:ObjectType.ID"/>
<FIELDref ref="_designation" utype="src:source.Source.name"/>
<FIELDref ref="_z" utype="xsrc:source.Source.redshift"/>
<PARAM name=”type” utype=”src:source.Source.classification”
value=”galaxy”>
<GROUP utype="src:source.Source.position">
<PARAM name=”type” utype=”vo-dml:Instance.type”
value=”src:source.SkyCoordinate” datatype=”char” arraysize=”\*”/>
<FIELDref ref="_ra" utype="src:source.SkyCoordinate.longitude"/>
<FIELDref ref="_dec" utype="src:source.SkyCoordinate.latitude"/>
<GROUP ref="_icrs" utype="src:source.SkyCoordinate.frame"/>
</GROUP>
</GROUP>
<FIELD name="designation" ID="_designation" .../>
<FIELD name="ra" ID="_ra" unit="deg" .../>
<FIELD name="dec" ID="_dec" unit="deg" .../>
<FIELD name=”z” ID=”_z” .../>
<TR><TD>08120809-0206132</TD><TD>123.033734</TD><TD>-2.103671</TD><TD>0.123</TD></TR>
...
</TABLE>
~~~~~~

TBD Notice that the GROUP representing the *Source* has two VODML/TYPE
elements, one referring to the subtype *xsrc:source.Source* and the
other to its super type, *src:source.Source*. This possibility is a
proposal on how to facilitate the work for so called *naïve clients*
tailored to specific models (see Appendix B). Such clients are written
only for a few specific models and only on the look-out for specific
**`vodmlrefs`** from those models. By providing all[^13] types in the
inheritance hierarchy above the actual type, such clients would find a
type they know and they will not be confused by the unknown *xsrc*
annotation.

Notice that the extending class inherits all of the parent’s attributes
**and their vodmlrefs**: this way a naïve client of the *src* model can
find all the elements it may understand, while a client of the child
class can also look for the *xsrc* elements added in the subtype
definition. The knowledge of the Data Model, either hardwired in the
reader or dynamically generated using the VO-DML XML description,
provides the client with the ability to look for the **`vodmlrefs`** it is
interested in.

## Other patterns \[INITIAL NOTES\] ##

### Calculated columns[^14] ###

Example from SDSS/PhotoObjAll

Has columns indicating the difference between the position of a source
defined in a particular band form the "global" position. A model
supporting multiple positions could be identified here, but only by
creating a kind of virtual, calculated column. We could try to come up
with a some kind of language for expressing such relations, maybe
related to parameter description language (PDL, TODO add ref). TBD how
easy is it to define and interpret these? Remember, goal is to create
model instance from the serialized version. PDL would define expressions
whose result is to produce the model instance from one or more columns.

### Views as derived tables

Alternatively, and especially useful and applicable for TAP, the mapping
is allowed to provide an explicit VIEW defined as an ADQL/SQL query on a
TABLE (or set of tables?) that includes such calculated columns. The
mapping could be defined against the metadata defined for the view.
\[TBD if we allow this, why not also allow

### More ???<span id="_Ref229724599" class="anchor"><span id="_Ref273893524" class="anchor"><span id="_Toc274072081" class="anchor"></span></span></span>

[^11]: TBD We may want to rename this type to avoid confusion and
    because one can argue that it should not be a type but an instance.

[^12]: See http://wiki.ivoa.net/twiki/bin/view/IVOA/IVAODMCatalogsWP.

[^13]: It is TBD in this proposal whether the whole type hierarchy
    should be replicated, or whether for example only one type is
    required per model.

[^14]: Thanks to Sebastien Derriere to emphasize this point.
