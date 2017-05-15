VO-DML XML schema [NORMATIVE] {#sec:schema}
=============================

In order to keep the VOTable standard as stable as possible and to better separate the
concerns of the different documents, the elements used for mapping data model instances are
defined in a standalone schema, that is then imported by the VOTable 1.4 schema itself.

Additions to VOTable
--------------------

The new VOTable 1.4 VOTable schema [@votable] simply imports the mapping schema through
`xs:import`:

~~~~ xml
<xs:import namespace="http://www.ivoa.net/xml/VODML_Mapping/v1.0"
               schemaLocation="VODML-mapping.xsd"/>
~~~~

And then defines a new element **`VODML`** as a direct child of **`VOTABLE`** (line 5):

~~~~ { .xml .numberLines }
	<!-- VOTable is the root element -->
	<xs:element name="VOTABLE">
		<xs:complexType>
			<xs:sequence>
				<xs:element name="VODML" type="vodml:VODML" minOccurs="0" />
				<xs:element name="DESCRIPTION" type="anyTEXT" minOccurs="0" />
				<xs:element name="DEFINITIONS" type="Definitions" minOccurs="0" />
				<!-- Deprecated -->
				<xs:choice minOccurs="0" maxOccurs="unbounded">
					<xs:element name="COOSYS" type="CoordinateSystem" />
					<!-- Deprecated in V1.2 -->
					<xs:element name="GROUP" type="Group" />
					<xs:element name="PARAM" type="Param" />
					<xs:element name="INFO" type="Info" minOccurs="0" maxOccurs="unbounded" />
				</xs:choice>
				<xs:element name="RESOURCE" type="Resource" minOccurs="1" maxOccurs="unbounded" />
				<xs:element name="INFO" type="Info" minOccurs="0" maxOccurs="unbounded" />
			</xs:sequence>
			<xs:attribute name="ID" type="xs:ID" />
			<xs:attribute name="version">
				<xs:simpleType>
					<xs:restriction base="xs:NMTOKEN">
						<xs:enumeration value="1.3" />
					</xs:restriction>
				</xs:simpleType>
			</xs:attribute>
		</xs:complexType>
	</xs:element>
~~~~

VODML-mapping shema
-------------------

~~~~ include
file: data/VODML-mapping.xsd
classes:
 - xml
 - numberLines
 - resizable
caption: VODML additions to the VOTable schema.
id: lst:mapping
~~~~

References
==========

