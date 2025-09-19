
# 📘 File Extractor Node

## **Description**
This node is used to read and return file's content.

- **input_map** is required and expects variables like:
  - `file1` = `variables.files.file1`
  - `file2` = `variables.files.file2`
- **output_variable_path** is optional, but required if you want to use file's content in the next node.  
- The result of this node will be content of each file, that can be accessed as `output_variable.{key}` where **keys** are provided input variables.

---

## Example

We have 2 files in our flow (document1, document2), let's say both of them are plain text files. To pass them inside extractor node, we can create inputs like **`file1` = `variables.files.document1`** and **`file2` = `variables.files.document2`**. Now we set **`output_variable_path`** as **`variables.files_content`**. After execution of extractor node we will be able to access each file's content from **`variables.files_content.file1`** and **`variables.files_content.file2`** accordingly.

---

## Supported File Extensions

- .txt
- .csv
- .pdf
- .json
- .docs/.docx

Any other file can be processed as well, but the proper content extraction is not guranteed.
