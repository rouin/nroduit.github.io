---
title: Build Plug-ins
keywords: [ "plugins", "weasis plugins", "dicom viewer", "free dicom viewer", "open source dicom viewer", "weasis dicom viewer",  "multi-platform dicom viewer", "dicom", "pacs", "pacs viewer" ]
description: How to build new Weasis plug-ins and how they can be incorporated to the distributions
weight: 30
---

## <center>How to build and install a plug-in</center>

This page describes how to build new Weasis plug-ins and how they can be incorporated to the distributions, see also this [page](../../../getting-started/guidelines) for the IDE configuration.

### List of plug-ins types

- Media viewer or editor (main central panel that implements `ViewerPlugin` or `ImageViewerPlugin` and the factory implements `SeriesViewerFactory`)
- Toolbar associated to a viewer (implements `Toolbar`)
- Tool associated to a viewer (right panel that implements `DockableTool`)
- Data explorer (data model implements `DataExplorerModel` and data view implements `DataExplorerView`, and the factory implements `DataExplorerViewFactory`)
- Import data into an explorer (ex. `ImportDicom` and the factory implements `DicomImportFactory`)
- Export data into an explorer (ex. `ExportDicom` and the factory implements `DicomExportFactory`)
- DICOM editor or viewer for special modalities (`DicomSpecialElementFactory` and `SeriesViewerFactory`), see weasis-dicom-sr
- Media codec (implements `Codec`)
- Perferences (implements `PreferencesPageFactory`)
- UI aggregator. This is the application main user interface bundle. The maven artifact of this bundle must be defined in config.properties (ex. weasis.main.ui=weasis-base-ui)

{{% notice tip %}}
See the [Weasis Architecture](../../architecture) to understand the plug-in hierarchy.
{{% /notice %}}


### Build plug-ins from Maven archetype

1. From the root directory of an archetype execute: mvn install
2. Generate a sample project by executing the following command: mvn archetype:generate -DarchetypeCatalog=local
3. Select the archetype:
    - weasis-plugin-base-viewer-archetype (example of a toolbar and a tool for the non DICOM viewer)
    - weasis-plugin-dicom-viewer-archetype (example of a toolbar and a tool for the DICOM viewer)

{{% notice note %}}
From Eclipse: File > New > Maven Project and Search for weasis archetype in catalog filter.
{{% /notice %}}

### Install plug-ins

#### For weasis-portable distribution

The file "/weasis/conf/ext-config.properties" must be adapted and plug-ins must be placed in the directory "/weasis/plugins".

Example with [weasis-isowriter](http://github.com/nroduit/weasis-isowriter):

-   Add in /weasis/conf/ext-config.properties:

    ``` bash
    felix.auto.start.85=${weasis.codebase.url}/plugins/weasis-isowriter-2.6.1.jar
    ```
{{% notice tip %}}
For not modifying the current ext-config.properties create a new file and add to the launcher the following VM argument:
    `-Dfelix.extended.config.properties="file:///your_plugin_path/myplugin-config.properties"`
{{% /notice %}}

- Place the file "weasis-isowriter-2.6.1.jar" in the directory "/weasis/plugins"

#### For the WEB distribution

Build a new war file containing the plug-ins and the ext-config.properties file.

- Build "weasis-ext.war" with the following structure:
    ```
    weasis-ext/
    ├── conf/
    │   ├── ext-config.properties
    ├── WEB-INF/
    │   ├── web.xml
    ├── plugin1.jar
    └── plugin2.jar
    ```

- In /weasis-ext/conf/ext-config.properties, add the plug-ins references:

    ``` bash
    felix.auto.start.85= \
     ${weasis.codebase.ext.url}/plugin1.jar \
     ${weasis.codebase.ext.url}/plugin2.jar
    ```
{{% notice note %}}
Using `${weasis.codebase.ext.url}` allows to keep the base URL abstract, so moving the package to another server won’t be a problem. Otherwise absolute URLs must be used. The default value of `${weasis.codebase.ext.url}` is `${weasis.codebase.url}-ext`.
{{% /notice %}}

- weasis-ext is the default web context when launching Weasis, using another web context requires to modify the property weasis.ext.url, it can be done by:

    - changin the property in jnlp template in <a target="_blank" href="https://github.com/nroduit/weasis-pacs-connector#configuration-of-weasis-pacs-connector">weasis-pacs-connector configuration</a>.

    ``` bash
     weasis.ext.url=${server.base.url}/weasis-newext
    ```
{{% notice note %}}
It is aslo possible to add the code base for plugins (cdb-ext) directly in the URL: `http://localhost:8080/weasis-pacs-connector/viewer?patientID=9702672&cdb-ext=http://localhost:8080/plugins/weasis-ext`
{{% /notice %}}


{{% notice tip %}}
**For debugging  a specific configuration**: add to the launcher the following VM argument:
`-Dfelix.extended.config.properties="http://server:port/weasis-ext/conf/ext-config.properties`
{{% /notice %}}




[Example](https://github.com/nroduit/weasis-plugins-war-builder) that make a package of [weasis-isowriter](http://github.com/nroduit/weasis-isowriter) plugin:

- Build "weasis-ext.war":

    ```
    weasis-ext/
    ├── conf/
    │   ├── ext-config.properties
    ├── WEB-INF/
    │   ├── web.xml
    └── weasis-isowriter-2.0.3.jar
    ```

### Build OSGI services

All the plug-in type described in the list above are OSGI services that are registered and aggregated in the GUI. Building the plug-in from the Maven archetype will configure automatically the OSGI service. For adding new OSGI services, here is the procedure:

1. Create a class implementing one of the plug-in type and add at least the annotations *@Component* and *@Service*, for instance:
﻿
    ``` java
    @Component(immediate = false)
    @Service
    public class SamplePrefFactory implements PreferencesPageFactory {
    ﻿...
    }
    ```
{{% notice tip %}}
For more information about annotations see the <a target="_blank" href="http://felix.apache.org/documentation/subprojects/apache-felix-maven-scr-plugin/scr-annotations.html">Apache Felix SCR Annotations</a>.
{{% /notice %}}

2. Add in pom.xml of the plug-in the maven-scr-plugin (to generate xml file from the Java annotations) and the property for loading any xml file in maven-bundle-plugin:
﻿
    ``` xml
    <build>
    <plugins>
     <plugin>
        <groupId>org.apache.felix</groupId>
        <artifactId>maven-scr-plugin</artifactId>
     </plugin>
     <plugin>
        <groupId>org.apache.felix</groupId>
        <artifactId>maven-bundle-plugin</artifactId>
     </plugin>
    ...
    ```
