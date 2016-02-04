******************
Storage Controller
******************

 .. todo:: UNDER CONSTRUCTION

 .. todo:: UNDER CONSTRUCTION

 .. todo:: UNDER CONSTRUCTION

Binary files such as images are not normally stored inside a regular database.
Instead they need to be stored in a 3rd party file storage service such as Amazon S3
or Dropbox. Agile Toolkit supports multiple 3rd party storages:

- ``setStorage('Filestore', $dir);`` - store files on a filesystem
- ``setStorage('S3', $bucket);`` - store files in Amazon S3 (and Cloudfront)
- ``setStorage('Dropbox', ...);`` - store files in Dropbox

Agile Toolkit contains several UI widgets that allow you to operate with files:
upload, work with images, categorize or associate files with other data. Those
widgets can work with any 3rd party storage system which implements a proper
storage controller.

Usage Example
=============

The most basic way to use storage controller is to associate it with ``Model_File``
model::

    $f = $this->add('Model_File');
    $f->setStorage('S3');

    $f->upload($my_file_path);

The above code will upload the file located at ``$my_file_path`` and store it as
object inside Amazon S3. The other information about the file such as filename,
type, size and check-sum will be stored in your regular database called ``my-files``.

Once uploaded, the ``Model_File`` will represent your actual file inside Agile
Toolkit, you can search for files directly by name, type or through associations.


Components of File Storage System
=================================

There are 3 parts at play when you operate with the files.

1. File Model - Either instance of Model_File or descendant.
2. Controller_Storage - Link between model and the actuall implementation.
3. 3rd Party add-on. Typically a minimalistic SDK supplied by storage engine vendor.

The #3 is optional and use of existing 3rd party class would greatly simplify
implementation of Controller_Storage.

Model_File implementation
=========================


.. php:class:: Model_File

    Describes a physical file and it's properties.

The model is defining the following fields:

+------------------------+-------------------------------------------------+--------------------------------------------------------+
| Field                  | Type                                            | Value                                                  |
+========================+=================================================+========================================================+
| location               | String containing URI                           | s3://s3.amazonws.com/bucket123/folder/filename.jpeg    |
+------------------------+-------------------------------------------------+--------------------------------------------------------+
| url                    | Public URL (http or https) if file is available | http://bucket123.s3.amazonaws.com/folder/filename.jpeg |
+------------------------+-------------------------------------------------+--------------------------------------------------------+
| storage                | Type of the storage used                        | S3, Filestore, Dropbox, etc                            |
+------------------------+-------------------------------------------------+--------------------------------------------------------+
| status                 | Describes status of file upload progress        | draft, uploaded, verified, etc                         |
+------------------------+-------------------------------------------------+--------------------------------------------------------+
| meta.original_filename | Filename before it was uploaded                 | filename.jpeg                                          |
+------------------------+-------------------------------------------------+--------------------------------------------------------+
| meta.size              | Size in bytes                                   | 389283                                                 |
+------------------------+-------------------------------------------------+--------------------------------------------------------+
| meta.md5               | Check-sum as per md5_sum()                      | mm                                                     |
+------------------------+-------------------------------------------------+--------------------------------------------------------+
| meta.extension         | Determined best suited extension for the file   | jpg                                                    |
+------------------------+-------------------------------------------------+--------------------------------------------------------+
| meta.mime              | Mime type of the file                           | image/jpeg                                             |
+------------------------+-------------------------------------------------+--------------------------------------------------------+
| meta.is_image          | Is true when the file is an image               | true                                                   |
+------------------------+-------------------------------------------------+--------------------------------------------------------+
| meta.width             | Width in pixels, if fileis image                | 400                                                       |
+------------------------+-------------------------------------------------+--------------------------------------------------------+
| meta.height            | Height in pixels, if file is an image           | 250                                                       |
+------------------------+-------------------------------------------------+--------------------------------------------------------+

To access meta properties use::

    $file->ref('meta')['size'];

or to perform search by a meta-property::

    $file->addCondition('meta.size', '>', 1000000);

.. note:: Sub-model conditions are only supported for Databases that are capable
    of storing and indexing hierarchic data, e.g. Clusterpoint, Mongo.


.. php:method:: import($file)

    Import and store specified file.

.. php:method:: verify()

    Will download file from public URL and verify md5.

.. php:method:: getFile()

    Retrive file, store locally in a temproary location and return full path.



.. php:class:: Controller_Storage

    Controller for abstraction of 3rd party file storage systems. If you need
    to store files with another file storage service, you might need to create
    your own controller. The following methods will have to be implemented.

.. php:method:: put($model, $source_file)

    Store specified file and update $model fields (location, storage, url)

.. php:method:: get($model)

    Load model corresponding to a specified model and return filename.

.. php:method:: delete($model)

    Delete file object corresponding to the model.




