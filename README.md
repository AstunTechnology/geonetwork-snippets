# Generic Snippets for GeoNetwork

WIP repository of generic snippets for GeoNetwork. It is expected that this will be expanded to include different types of snippet over time.

The following process is an outline for how to define a new snippet, add it to the GeoNetwork index, and allow it to show in the report picker in the editor layout for your schema. 

**Caution, this is a complex process and requires some knowledge of xml and xpath. Take backups of any files you change!**

The process of defining the correct xml for your snippet is left to you but see the examples given in this repository as a starting point.

Note that you will need access to the files for both your schema and the core GeoNetwork code to create and apply a new snippet, as well as an administator account in the web interface. You may need to restart GeoNetwork for changes to be applied, but should not need to rebuild the source code.

## Process

### Define snippet

* Must be contained within a single root element (see examples in this repository)
* Must have all the namespaces used within the snippet defined in the root element

### Add to index

Add an element to the `index-fields\index-subtemplate.xsl` for your metadata profile, containing something like the following:

	<xsl:template mode="index" match="gmd:DQ_CompletenessCommission[count(ancestor::node()) =  1]">
	    <Field name="_title"
	           string="{gmd:result/*/gmd:explanation/gco:CharacterString}"
	           store="true" index="true"/>

	    <xsl:call-template name="subtemplate-common-fields"/>
	</xsl:template>


Where:

* The element being matched is the root element of your snippet (in this case `gmd:DQ_CompletenessCommission`)
* `[count(ancestor::node()) =  1]` is mandatory
* The string element is an xpath statement to create an entry that will represent the title of the snippet in the index and the display label in the directory

Then add the same entry above into `index-fields\index-subtemplate.xsl` for the core ISO19139 metadata profile, containing the following, making the same substitutions as above. This ensures that the entry is present and displays correctly in both your metadata profile editor interface and the main directory management page.


### Add a translation

Add an entry in `geonetwork/catalog/locales/en-custom.json` as follows:

	"root element": "human-readable name"

For example:

	"gmd:DQ_CompletenessCommission": "Data Quality Reports (Completeness)"

You will probably need to restart GeoNetwork and re-index your records for these changes to be applied

### Changes to editor layout

Finally you need to change the editor layout (`layout/config-editor.xml`) for the metadata schema where you want to use the snippets so that the element uses the angular directive (data-gn-directory-entry-selector) rather than a default text element.

#### Form field type configuration

In the `<fields>` section, define a new entry for the parent element of the one you are editing. In other words, for (eg) `gmd:DQ_Completeness` you need to define an entry for `gmd:report`:

	<for name="gmd:report" addDirective="data-gn-directory-entry-selector">
	      <directiveAttributes
	        data-template-add-action="true"
	        data-search-action="true"
	        data-popup-action="true"
	        data-filter='{"_root": "gmd:DQ_*"}'
	        data-insert-modes="text"
	        data-template-type="report"/>
	</for>

Where:

* **data-template-add-action** indicates whether the entry should also contain a standard button for adding a new element
* **data-search-action** indicates whether the search box is present
* **data-popup-action** indicates whether the add button is small and unlabelled or large with a label
* **data-filter='{"\_root": "gmd:DQ_*"}'** indicates the xpath of the child element, eg the one you want to add the snippet for. In this example a wildcard character has been used to allow any `gmd:DQ_` element to use the selector
* **data-insert-modes** indicates whether there should be an option to insert as "text", "x-link" or "" (both text and x-link)
* **data-template-type** indicates the type of template being searched for

#### Specific field configuration

This will be dependent on whether the element you're working with is explicitly defined in the tab layout or not.

If only the parent element is defined as a section and your element is **not** explicitly mentioned then you don't need to make any further changes, eg

	<section xpath="/gmd:MD_Metadata/gmd:dataQualityInfo" or="dataQualityInfo"
	       in="/gmd:MD_Metadata"/>

If your element **is** explicitly defined then you need to change the definition of it to include the angular directive. eg:

	<action type="add" btnLabel="addAccessConstraints" name="addAccessConstraints" or="resourceConstraints"
                  in="/gmd:MD_Metadata/gmd:identificationInfo/gmd:MD_DataIdentification" addDirective="data-gn-directory-entry-selector">

See https://github.com/AstunTechnology/iso19139.gemini23/tree/3.10.x/src/main/plugin/iso19139.gemini23/layout/config-editor.xml for examples of this being applied.

### Add snippet to directory

* As an admin, naviagate to contribute -> manage directory and choose "create a template"
* Paste your xml into the box labelled "XML snippet"
* Choose a group to assign it to (this is mandatory)
* Click the "Import directory entry" button


## Known errors

* Not having a single root element
* Not getting the correct xpath
