# Convert Dataset to MindRecord

 `Linux` `Ascend` `GPU` `CPU` `Data Preparation` `Intermediate` `Expert`

<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [Convert Dataset to MindRecord](#convert-dataset-to-mindrecord)
	- [Overview](#overview)
	- [Basic Concepts](#basic-concepts)
	- [Convert Dataset to MindRecord](#convert-dataset-to-mindrecord)
	- [Load MindRecord Dataset](#load-mindrecord-dataset)

<!-- /TOC -->

<a href="https://gitee.com/mindspore/docs/blob/master/tutorials/source_en/advanced_use/dataset_conversion.md" target="_blank"><img src="../_static/logo_source.png"></a>

## Overview

User can convert non-standard datasets and common datasets into the MindSpore data format, MindRecord, so that they can be easily loaded to MindSpore for training. In addition, the performance of MindSpore in some scenarios is optimized, which delivers better user experience when you use datasets in the MindSpore data format.

The MindSpore data format has the following features:
1. Unified storage and access of user data are implemented, simplifying training data loading.
2. Data is aggregated for storage, which can be efficiently read, managed and moved.
3. Data encoding and decoding are efficient and transparent to user.
4. The partition size is flexibly controlled to implement distributed training.

The MindSpore data format aims to normalize datasets of user to MindRecord, which can be further loaded through `MindDataset` and used in the training procedure.

![data_conversion_concept](./images/data_conversion_concept.png)

## Basic Concepts

A MindRecord file consists of data files and index files. Data files and index files do not support renaming for now.

- Data file

    Contains the file header, scalar data page and block data page for storing normalized training data. It is recommended that the size of a single MindRecord file does not exceed 20 GB. User can break up a large dataset and store the dataset into multiple MindRecord files.

- Index file

    Contains index information generated based on scalar data (such as image labels and image file names), used for convenient data fetching and storing statistical data about the dataset.

![mindrecord](./images/mindrecord.png)

Data file consists of the following key parts:

- File Header

    File header stores the file header size, scalar data page size, block data page size, schema, index fields, statistics, file partition information, and mapping between scalar data and block data. It is the metadata of the MindRecord file.

- Scalar data page

    Scalar data page is used to store integer, string and floating point data, such as the label of an image, file name of an image, and length, width of an image. Information suitable for scalar data are stored here.

- Block data page

    Block data pages are used to store data such as binary strings, NumPy arrays. Additional examples include converted python dictionaries generated from texts and binary image files.

## Convert Dataset to MindRecord

The following tutorial demonstrates how to convert image data and its annotations to MindRecord format. For details about MindSpore data format conversion, see the [MindSpore Data Format Conversion](https://www.mindspore.cn/api/en/master/programming_guide/dataset_conversion.html) section in the programming guide.

1. Import the `FileWriter` class for file writing.

    ```python
    from mindspore.mindrecord import FileWriter
    ```

2. Define a dataset schema which defines dataset fields and field types.

    ```python
    cv_schema_json = {"file_name": {"type": "string"}, "label": {"type": "int32"}, "data": {"type": "bytes"}}
    ```

    Schema mainly contains `name`, `type` and `shape`:
    - `name`: field names, consist of letters, digits and underscores.
    - `type`: field types, include int32, int64, float32, float64, string and bytes.
    - `shape`: [-1] for one-dimensional array, [m, n, ...] for higher dimensional array in which m and n represent the dimensions.  

    > - The type of a field with the `shape` attribute can only be int32, int64, float32, or float64.
    > - If the field has the `shape` attribute, only data in `numpy.ndarray` type can be transferred to the `write_raw_data` API.

3. Prepare the data sample list to be written based on the user-defined schema format. Binary data of the images is transferred below.

    ```python
    data = [{"file_name": "1.jpg", "label": 0, "data": b"\x10c\xb3w\xa8\xee$o&<q\x8c\x8e(\xa2\x90\x90\x96\xbc\xb1\x1e\xd4QER\x13?\xff\xd9"},
            {"file_name": "2.jpg", "label": 56, "data": b"\xe6\xda\xd1\xae\x07\xb8>\xd4\x00\xf8\x129\x15\xd9\xf2q\xc0\xa2\x91YFUO\x1dsE1\x1ep"},
            {"file_name": "3.jpg", "label": 99, "data": b"\xaf\xafU<\xb8|6\xbd}\xc1\x99[\xeaj+\x8f\x84\xd3\xcc\xa0,i\xbb\xb9-\xcdz\xecp{T\xb1\xdb"}]
    ```

4. Adding index fields can accelerate data loading. This step is optional.

    ```python
    indexes = ["file_name", "label"]
    ```

5. Create a `FileWriter` object, transfer the file name and number of slices, add the schema and index, call the `write_raw_data` API to write data, and call the `commit` API to generate a local data file.

    ```python    
    writer = FileWriter(file_name="test.mindrecord", shard_num=4)
    writer.add_schema(cv_schema_json, "test_schema")
    writer.add_index(indexes)
    writer.write_raw_data(data)
    writer.commit()
    ```

    This example will generate `test.mindrecord0`, `test.mindrecord0.db`, `test.mindrecord1`, `test.mindrecord1.db`, `test.mindrecord2`, `test.mindrecord2.db`, `test.mindrecord3`, `test.mindrecord3.db`, totally eight files, called MindRecord dataset. `test.mindrecord0` and `test.mindrecord0.db` are called one MindRecord file, in which `test.mindrecord0` is the data file and `test.mindrecord0.db` is the index file.

    **Interface Description:**
    - `write_raw_data`: write data to the memory.
    - `commit`: write the data in the memory to the disk.

6. For adding data to the existing data format file, call the `open_for_append` API to open the existing data file, call the `write_raw_data` API to write new data, and then call the `commit` API to generate a local data file.

    ```python
    writer = FileWriter.open_for_append("test.mindrecord0")
    writer.write_raw_data(data)
    writer.commit()
    ```

## Load MindRecord Dataset

Tutorials below briefly demonstrate how to load MindRecord dataset using `MindDataset`.

1. Import `MindDataset` for dataset loading.

    ```python
    import mindspore.dataset as ds
    ```

2. Use `MindDataset` to load MindRecord dataset.

    ```python
    data_set = ds.MindDataset(dataset_file="test.mindrecord0")     # Read full data set
    count = 0
    for item in data_set.create_dict_iterator(output_numpy=True):
        print("sample: {}".format(item))
        count += 1
    print("Got {} samples".format(count))
    ```