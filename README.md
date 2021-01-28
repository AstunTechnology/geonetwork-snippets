# Generic Data Quality Snippets for GeoNetwork

WIP repository of generic data quality snippets for GeoNetwork. It is expected that this will be expanded to include different completeness reports over time.

The following process is an outline for how to define a new data quality snippet, add it to the GeoNetwork index, and allow it to show in the report picker in the editor layout for your schema. 

The process of defining the correct xml for your snippet is left to you but see the example given in this repository as a starting point.

Note that you will need access to the files for both your schema and the core GeoNetwork code to create and apply a new snippet, as well as an administator account in the web interface. You may need to restart GeoNetwork for changes to be applied, but should not need to rebuild the source code.

## Process

### Define snippet

* Must be contained within a single root element (see example in the completeness folder)
* Must have all the namespaces used within the snippet defined in the root element

### Add to index

Add an element to the `index-fields\index-subtemplate.xsl` for your metadata profile, containing the following:

	<xsl:template mode="index" match="gmd:DQ_CompletenessCommission[count(ancestor::node()) =  1]">
	    <xsl:variable name="date"
	                  select="*/gmd:specification/*/gmd:date/*/gmd:date/gco:Date"/>
	    <Field name="_title"
	           string="{gmd:result/*/gmd:explanation/gco:CharacterString}{if ($date != '') then concat(' (', $date, ')') else ''}"
	           store="true" index="true"/>

	    <xsl:call-template name="subtemplate-common-fields"/>
	</xsl:template>

Where:

* The element being matched is the root element of your snippet
* The date element (if required) is the xpath to the sub-element representing the (eg) Citation Date for the specification
* The title element (shown in the Geonetwork sub-directory and element picker while editing) is the xpath to the sub-element you wish to use as the title of the snippet

### Add a translation

Add an entry in `geonetwork/catalog/locales/en-custom.json` as follows:

	"root element": "human-readable name"

For example:

	"gmd:DQ_CompletenessCommission": "Data Quality Reports (Completeness)"

You will probably need to restart GeoNetwork and re-index your records for these changes to be applied

### Potential changes to editor layout

* If you have a boolean element (eg for `gmd:pass`) and you wish to allow it to display as "not evaluated" as well as "true" or "false" then ensure that the field definition for `gco:boolean` in your profile's `layout/config-editor.xml` contains a generic enough xpath to encompass your element.
* Edit the `gmd:report` directive in the same file to include your root element in the `data-filter` declaration.
* To prevent validation errors you might want to update `update-fixed-info.xsl` to force any empty numeric elements to have a `nilReason` attribute.

See https://github.com/AstunTechnology/iso19139.gemini23/tree/3.10.x/src/main/plugin/iso19139.gemini23 for examples of this being applied.

### Add snippet to directory

* As an admin, naviagate to contribute -> manage directory and choose "create a template"
* Paste your xml into the box labelled "XML snippet"
* Choose a group if required, otherwise leave this blank
* Click the "Import directory entry" button


## Known errors

* Not having a single root element
* Not getting the correct xpath
