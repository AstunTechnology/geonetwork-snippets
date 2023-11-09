# Generic Snippets for GeoNetwork 4.2.x branch

WIP repository of generic snippets for GeoNetwork. It is expected that this will be expanded to include different types of snippet over time.

The following process is an outline for how to define a new snippet, add it to the GeoNetwork index, and allow it to show in the report picker in the editor layout for your schema.

**Caution, this is a complex process and requires some knowledge of xml and xpath. Take backups of any files you change!**

The process of defining the correct xml for your snippet is left to you but see the examples given in this repository as a starting point.

Note that you will need access to the files for both your schema and the core GeoNetwork code to create and apply a new snippet, as well as an administrator account in the web interface. You may need to restart GeoNetwork for changes to be applied, but should not need to rebuild the source code.

## Process

### Define snippet

* Must be contained within a single root element (see examples in this repository)
* Must have all the xml namespaces used within the snippet defined in the root element
* Must have a string element that will represent the title of the snippet in the index and the display label in the directory

### Add to index

Add an element to the `index-fields\index-subtemplate.xsl` for the core ISO19139 profile, containing something like the following:

	<xsl:template mode="index" match="gmd:DQ_ConceptualConsistency[count(ancestor::node()) = 1]">
        <xsl:variable name="title"
                  select="gmd:result/gmd:DQ_ConformanceResult/gmd:specification/gmd:CI_Citation/gmd:otherCitationDetails/(gco:CharacterString|gmx:Anchor)"/>
    		<resourceTitleObject type="object">
    			{"default": "<xsl:value-of select="gn-fn-index:json-escape($title)"/>"}
    		</resourceTitleObject>
    		<xsl:call-template name="subtemplate-common-fields"/>
    </xsl:template>

Where:

* The element being matched is the root element of your snippet (in this case `DQ_ConceptualConsistency`)
* `[count(ancestor::node()) = 1]` is mandatory
* The string element is an xpath statement to create an entry that will represent the title of the snippet in the index and the display label in the directory

Then ensure that the Gemini 2.3 `index-fields\index-subtemplate.xsl` contains a reference back to ISO19139 (it probably does already):

	<xsl:import href="../../iso19139/index-fields/index-subtemplate.xsl"/>

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
	        data-filter='{"root": "gmd:DQ_*"}'
	        data-insert-modes="text"
	        data-template-type="report"/>
	</for>

Where:

* **data-template-add-action** indicates whether the entry should also contain a standard button for adding a new element
* **data-search-action** indicates whether the search box is present
* **data-popup-action** indicates whether the add button is small and unlabeled or large with a label
* **data-filter='{"\_root": "gmd:DQ_*"}'** indicates the xpath of the child element, eg the one you want to add the snippet for. In this example a wild-card character has been used to allow any `gmd:DQ_` element to use the selector
* **data-insert-modes** indicates whether there should be an option to insert as "text", "x-link" or "" (both text and x-link)
* **data-template-type** indicates the type of template being searched for

#### Specific field configuration

This will be dependent on whether the element you're working with is explicitly defined in the tab layout or not.

If only the parent element is defined as a section and your element is **not** explicitly mentioned then you don't need to make any further changes, eg

	<section xpath="/gmd:MD_Metadata/gmd:dataQualityInfo" or="dataQualityInfo"
	       in="/gmd:MD_Metadata"/>

If your element **is** explicitly defined then you need to change the definition of it to include the angular directive, eg:

	<action type="add" btnLabel="addAccessConstraints" name="addAccessConstraints" or="resourceConstraints"
                  in="/gmd:MD_Metadata/gmd:identificationInfo/gmd:MD_DataIdentification" addDirective="data-gn-directory-entry-selector">

### Add snippet to directory

* As an admin, navigate to Contribute -> Manage directory and choose "Add New Entry" -> Create an entry from scratch
* Paste your xml into the box labeled "XML snippet"
* Choose a group to assign it to (this is mandatory)
* Click the "Import directory entry" button

## Common problems

* Not having a single root element in your snippet will cause an error when you try to save it in the directory manager
* Not defining the correct xpath for the `_title` element will mean that the title is blank when the snippet is indexed
