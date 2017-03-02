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

In particular, [@sec:norm-vodml;@sec:norm-model;@sec:norm-relations] describe
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

[^15]: E.g. when interpreting an `<INSTANCE>` as a certain **`ObjectType`**, if one of
 its children is not annotated or identifies a child element that is
 not available on the type, this is an error. For each pattern there
 is a set of rules that, if broken, are annotation errors. (**TODO** we
 better strive to make these comprehensive. OL thinks this is too strict of a default rule.
 We should probably be more lenient by default and add strictness where required.
 A lot is already mandated by the new schema.)

The `VODML` element { #sec:norm-vodml }
-------------------

### Example ###

~~~~ include
file: instances/models.votable.xml
classes:
  - xml
caption: A Caption
id: lst:code
----
~~~~

### Schema Constraints ###

### Models Declaration: `MODELS` ###

### Global Instances: `GLOBALS` ###

### Tabular Instances: `TEMPLATES` ###

### Instance Annotation: `INSTANCE` ###

Model Declaration: MODEL { #sec:norm-model }
--------------------------

Relations { #sec:norm-relations }
---------

### Attributes: `INSTANCE` ###

### Attributes: `LITERAL` ###

### Attributes: `CONSTANT` ###

### Template Attributes: `COLUMN` ###

### Containers: `CONTAINER` ###

### Containers: `PRIMARYKEY` ###

### Containers: `FOREIGNKEY` ###

### References: `REFERENCE` ###

### References: `IDREF` ###

### References: `REMOTEREFERENCE` ###

### References: `FOREIGNKEY` ###

### Compositions: `COMPOSITION` ###

### Compositions: `INSTANCE` ###

### Compositions: `EXTINTANCES` ###

Representing Types { #sec:norm-types }
------------------

### **`PrimitiveType`** ###

### **`Enumeration`** and **`EnumerationLiteral`** ###

### **`ObjectType`** ###

### **`DataType`** ###

### Type Extensions ###

### Quantities ###
