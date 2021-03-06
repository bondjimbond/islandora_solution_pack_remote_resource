# Islandora Remote Resource Batch

Islandora batch module for ingesting objects managed by the Islandora Remote Resouce Solution Pack (i.e., of content model 'islandora:sp_remote_resource'). Can ingest objects from either an uploaded zip archive or a directory containing datastream files.
 
## Requirements

* [Islandora Batch](https://github.com/Islandora/islandora_batch)
* [Islandora Solr Search](https://github.com/Islandora/islandora_solr_search)
* [Islandora Remote Resource Solution Pack](https://github.com/mjordan/islandora_solution_pack_remote_resource)

## Usage

After enabling this module, collections that contain objects of the remote resource content model will have a "Remote Resource Batch" link in their "Overview" subtab, which allows users to upload a zip archive containing files arranged as described below in the "Preparing your content files for ingesting" section.

You can also run the following drush command to ingest objects:

`drush --user=admin islandora_remote_resource_batch_preprocess --target=/path/to/datastream/files --namespace=foo --parent=islandora:mycollection`

Then, to perform the ingest:

`drush --user=admin islandora_batch_ingest`

## Preparing your content files for ingesting

This batch module uses filename patterns to identify the files that are intended for specific datastreams. All of the files you are ingesting should go in the same directory, and for each object you want to ingest, you must have at least a file representing its OJB datastream. All other files are optional.

Content for batch ingestion can be prepared in a variety of ways, but harvesting it via OAI-PMH is a good strategy. Documentation on how to use the [Move to Islandora Kit](https://github.com/MarcusBarnes/mik) to harvest content via OAI-PMH that is ready to ingest into Islandora is available in the [Islandora Remote Resouce Tools](https://github.com/mjordan/islandora_remote_resource_batch_tools).

### OBJ datastreams

Text file with the extension `.txt`. This file contains the URL of the remote resource, e.g.:

```
http://example.com/some/path
```

To avoid duplication, prior to adding new objects to the batch ingest queue, this module queries Solr to determine if an object already exists that points to the value of the remote URL.

### TN datastreams

OBJ file base name with the double extension `.TN.ext` where `.ext` is one of '.jpg', '.jpeg', '.png', or '.gif'.

### DC datastreams

OBJ file base name with the double extension `.DC.xml`. The DC XML should contain the same namespaces as the native Fedora DC datastream XML files. Note that you should include only DC or MODS input files, not both.

### MODS datastreams

OBJ file base name with the double extension `.MODS.xml`. Note that you should include only DC or MODS input files, not both.


### Additional, arbitrary datastreams 

Objects managed by the Remote Resource Solution Pack only require an OBJ datastream that contains the URL of the remote resource. MODS, DC, and TN datastreams are optional. If they are noto present on ingest, Islandora will generate default datastreams.

For local purposes, you may want your objects to have datastreams in addition to a MODS, DC, or TN. For example, you may want to ingest OCR datastreams so the full text is indexed. To ingest additional datastreams, add files you want to create the datastreams from using the following file naming convention: for a base text file with the name somefile.txt, a datastream with the datastream ID 'JPEG' will be created from a file with the name `somefile.JPEG.jpg`.

The datastream's mimetype will be derived from its extension, in the example above, `.jpg`. Its label will be 'JPEG datastream'. The DSID and extension have no relationship, they just happend to be consistent in this example.

### Example input directories

Two text files, which will create two objects. The thumbnail and MODS datastreams for the objects will be set to defaults:

```
.
├── foo.txt
└── bar.txt
```

Two text files, which will create two objects. The thumbnail and MODS datastreams for the objects will be created from the file with TN and MODS in their filenames:

```
.
├── foo.txt
├── foo.MODS.xml
├── foo.TN.jpg
├── bar.txt
├── bar.MODS.xml
└── bar.TN.png
```

Three text files, which will create three objects. The object created from `foo.txt` will have its MODS datastream created from the `foo.MODS.xml` and its thumbnail created from defaults; the object created from `bar.txt` will have its TN datastream created from `bar.TN.png` and its MODS datastream created from defaults.

```
.
├── foo.txt
├── foo.MODS.xml
├── bar.txt
├── bar.TN.png
├── baz.txt
```

Same as the previous example, but for the object created from foo.txt, an additional datastream with the DSID 'SOMETHING' will be ingested from the file `foo.SOMETHING.png`.

```
.
├── foo.txt
├── foo.MODS.xml
├── foo.SOMETHING.png
├── bar.txt
├── bar.TN.png
├── baz.txt
```

### Using a zip file

Zip files used for ingesting objects via the GUI have the same structure as the directories described above. The zip file must not contain any subdirectories; all datastream files must be direct children of the top of the zip.

## Ingesting from a CSV file

This batch module does not support ingests from CSV files natively, but [Islandora Remote Resouce Tools](https://github.com/mjordan/islandora_remote_resource_batch_tools) offers a utility for converting data in a CSV file into ingest packages.

## Syncing updated datastreams

Copies of TN and MODS (and any optional datastreams) harvested from remote resrource objects may become out of sync with their remote originals. This batch loader provides a command to update datastreams harvested from the remote resource that have changed. To use it, pass in the directory that contains the datastream files as the value of the `--target` option:

`drush -u 1 islandora_remote_resource_batch_sync --target=/path/to/datastream/files`

The URL in the .txt file is used to query Islandora's Solr index for the corresponding PID, which is then used to update the corresponding object. To avoid needlessly replacing datastream files, this command generates a checksum for the existing datastream content and for the content of the new file, and only replaces the old with the new content if the checksums differ.

Datastream files should be prepared in the same way (and named in the same way) as they are for ingestion, as described above. OBJ files are not updated, only TN, MODS, and other datastreams, but the OBJ datastream file must be present as it is used to check the existence of the corresponding object. Datastream files for new objects and for existing objects can be located in the same directory; `islandora_remote_resource_batch_preprocess` skips objects that already exsit, and `islandora_remote_resource_batch_sync` only updates existing objects.

## Maintainer

* [Mark Jordan](https://github.com/mjordan)

## Development and feedback

Bug reports, use cases and suggestions are welcome, as are pull requests. See the CONTRIBUTING.md in [Islandora Remote Resource Solution Pack](https://github.com/mjordan/islandora_solution_pack_remote_resource) for more information.

## License

* [GPLv3](http://www.gnu.org/licenses/gpl-3.0.txt)
