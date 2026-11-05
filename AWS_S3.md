# Amazon S3 with Python — learner guide

This guide shows how to use Python to list files in Amazon S3, read JSON and CSV data, and upload data. We used **PyCharm on a Mac**, **Boto3** to communicate with AWS, and **pandas** to work with tables.

The final example follows our tutor’s method: write a DataFrame into an **in-memory text buffer**, then upload its contents using `put_object()`. No local CSV file is needed.

## What is S3?

Amazon Simple Storage Service (S3) stores data in the cloud. Common uses include keeping datasets, images, backups and application files [1].

| S3 term | Example from our work |
| --- | --- |
| Bucket — a container for objects | `data-eng-resources` |
| Object — stored file contents and metadata | The happiness CSV file |
| Key — the object’s complete name within its bucket | `python/happiness-2019.csv` |
| Prefix — the beginning of a key used to group or filter objects | `python/` |

For these examples, `python/` and `Data606/` are parts of object keys. They look like folders in the console, but in a general purpose bucket they are prefixes, not local folders.

S3 stores the data; Python on our Mac reads or writes it. We do not need an EC2 instance or Flask for this exercise.

## Before starting

- Have access to the training bucket and permission for the actions you need.
- Use the Python interpreter from your PyCharm project’s virtual environment.
- Keep AWS keys in your local AWS credentials file, outside your code and Git repository.
- Use your own output key so you do not overwrite another learner’s work.

The bucket and source keys below match our training example. Replace them if you use a different bucket. Listing buckets, listing objects, reading objects and uploading objects require their respective permissions; one successful action does not grant all the others.

## 1. Set up the AWS files — Mac Terminal

Boto3 can read the standard AWS configuration files in your home folder [2]. Create or enter the correct directory:

```bash
mkdir -p ~/.aws
cd ~/.aws
pwd
```

The path should end with `/.aws`. The dot is part of the folder name.

Create the files if needed:

```bash
touch credentials
touch config
```

`touch` creates an empty file if it does not exist. If it already exists, its contents are preserved. The following examples assume you are setting up the default profile; preserve any other profiles already in your files.

Open the config file:

```bash
nano config
```

Enter this **inside nano**:

```ini
[default]
region = eu-central-1
```

Save with **Control + O**, press **Enter**, then close with **Control + X**.

Open the credentials file:

```bash
nano credentials
```

Enter your valid keys locally, replacing these placeholders:

```ini
[default]
aws_access_key_id = YOUR_ACCESS_KEY
aws_secret_access_key = YOUR_SECRET_KEY
```

Save and close using the same steps. Temporary credentials also require the supplied `aws_session_token`. Never paste real credentials into a public README or screenshot. Replace exposed keys before using them again.

Restrict file access:

```bash
chmod 700 ~/.aws
chmod 600 ~/.aws/config ~/.aws/credentials
```

The credential lines are file contents, not commands to type at the terminal prompt.

## 2. Prepare Python — PyCharm

If the packages are not already installed, run this in the PyCharm Terminal using the project’s activated virtual environment:

```bash
python -m pip install boto3 pandas
```

Create `AWS_S3.py` and add:

```python
import boto3
import io
import json
import pandas as pd
from pprint import pprint as pp

s3_client = boto3.client('s3')
bucket_name = "data-eng-resources"
```

Boto3 obtains credentials from its configured sources, including `~/.aws/credentials`. It signs requests and AWS checks permissions before allowing an operation. The bucket does not need to be public.

Run each section below with this setup above it. These are separate practice examples; you do not need to run every upload each time you run the script.

## 3. List buckets

```python
bucket_list = s3_client.list_buckets()
pp(bucket_list)
```

This requests a list of buckets for the account. `pp()` formats the response for easier reading. The response also contains request information; it is not just a list of names.

Our original script also created `s3_resource = boto3.resource('s3')`. A resource offers an object-based interface, but we did not use it in these examples, so it is not required here.

## 4. List objects in the training bucket

```python
bucket_contents = s3_client.list_objects_v2(
    Bucket=bucket_name,
    Prefix='python/'
)

for obj in bucket_contents.get('Contents', []):
    print(obj['Key'])
```

`Bucket` selects the container; `Prefix` limits results to names beginning with `python/`. Printing each key helps us copy the exact name for a later read. `.get('Contents', [])` handles a response with no matching objects.

One `list_objects_v2()` response returns at most 1,000 keys. Larger listings require pagination; this example inspects one page.

## 5. Read a JSON file

```python
s3_object = s3_client.get_object(
    Bucket=bucket_name,
    Key='python/chatbot-intent.json'
)

data = s3_object['Body'].read()
dict_data = json.loads(data)
pp(dict_data)
```

`get_object()` requests the specified file. `Body.read()` reads the returned contents as bytes. `json.loads()` converts JSON into Python data, such as a dictionary or list [3, 4].

Printing `s3_object` shows the response details and a body-stream object, rather than the parsed JSON itself. Read the body to get the contents. Once the stream is consumed, request the object again if you need to read it a second time.

**Our result:** the JSON read worked after correcting a typo in the key and adding `import json`.

## 6. Read a CSV into a DataFrame

```python
s3_object = s3_client.get_object(
    Bucket=bucket_name,
    Key='python/happiness-2019.csv'
)

df = pd.read_csv(s3_object['Body'])
print(df)
```

pandas reads the returned stream into a DataFrame: a table with rows and columns [5]. There is no need to download the CSV to your Mac first. Do not call `.read()` on this body before passing it to pandas, because that would consume the stream.

## 7. Upload a dictionary — two methods

Start with the example dictionary:

```python
dict_to_upload = {'name': 'Sabahi', 'course': 'Data 606'}
```

### Method A: save a local JSON file, then upload it

```python
with open('data.json', 'w', encoding='utf-8') as jsonfile:
    json.dump(dict_to_upload, jsonfile)

s3_client.upload_file(
    Filename='data.json',
    Bucket=bucket_name,
    Key='Data606/Test/Sabahi.json'
)
```

`json.dump()` writes JSON into a file. `upload_file()` uploads that existing file from your computer. The local file is created in the script’s working directory.

### Method B: upload JSON directly with `put_object()`

```python
s3_client.put_object(
    Body=json.dumps(dict_to_upload),
    Bucket=bucket_name,
    Key='Data606/ObjectTest/Sabahi.json',
    ContentType='application/json'
)
```

`json.dumps()` creates JSON text in memory. `put_object()` uploads the contents supplied in `Body` [4, 6]. This method does not create a local JSON file.

## 8. Upload a DataFrame with an in-memory text buffer

This is our tutor’s example, adapted to use our own output name. It is a complete standalone example:

```python
import boto3
import pandas as pd
import io

s3_client = boto3.client('s3')
bucket_name = "data-eng-resources"

# Create a small example DataFrame
df = pd.DataFrame([
    [1, 2, 3, 4, 5],
    [10, 20, 30, 40, 50]
])

# Write the DataFrame into a text buffer in memory
str_buffer = io.StringIO()
df.to_csv(str_buffer)

# Upload the CSV text directly to S3
response = s3_client.put_object(
    Body=str_buffer.getvalue(),
    Bucket=bucket_name,
    Key='Data606/df_test/Sabahi.csv'
)

print("Upload status:", response['ResponseMetadata']['HTTPStatusCode'])
```

The final status print is added to make success visible. Expected output after a successful upload:

```text
Upload status: 200
```

| Step | What it does and why |
| --- | --- |
| `pd.DataFrame(...)` | Creates a small table to practise uploading. |
| `io.StringIO()` | Creates a temporary text buffer in memory [7]. |
| `df.to_csv(str_buffer)` | Writes the table as CSV text into the buffer [8]. |
| `str_buffer.getvalue()` | Gets all the CSV text from the buffer. No rewind is needed for `getvalue()`. |
| `Body` | Supplies the contents that S3 will store. We do not pass the DataFrame itself. |
| `Bucket` | Selects the existing destination bucket. |
| `Key` | Gives the new object its complete name. |

Like the tutor’s code, `to_csv()` includes column headings and row numbers by default. To omit the extra row-number column, use:

```python
df.to_csv(str_buffer, index=False)
```

Use a fresh buffer when rewriting the CSV. To upload the happiness DataFrame from Step 6 instead, skip the small `pd.DataFrame(...)` creation and use the existing `df`.

The buffer method keeps the CSV in memory. It is convenient for this small exercise; the buffer must fit in the available memory. Running an upload again with the same key replaces that object’s current contents; versioning may retain previous versions.

## 9. Check the uploaded CSV

In the AWS console, open **S3 → data-eng-resources**, then find:

```text
Data606/df_test/Sabahi.csv
```

Or read it back in Python after the upload:

```python
uploaded_object = s3_client.get_object(
    Bucket=bucket_name,
    Key='Data606/df_test/Sabahi.csv'
)

uploaded_df = pd.read_csv(uploaded_object['Body'], index_col=0)
print(uploaded_df)
```

`index_col=0` restores the row-number column saved by the tutor’s default `to_csv()` call. If you uploaded with `index=False`, omit `index_col=0` when reading it back.

**Outcome:** Python can retrieve S3 data for analysis and send table data back to S3. The buffer example demonstrates **DataFrame → CSV text in memory → S3 object**. Confirm the upload using its response and the stored file; no upload result is assumed by this guide.

## Troubleshooting

| Problem | Cause and fix |
| --- | --- |
| `cd: no such file or directory: .aws` | `aws` and `.aws` are different names. Use `mkdir -p ~/.aws` and `cd ~/.aws`. |
| `command not found: aws_access_key_id` | Credential text was typed at the terminal prompt. Put it inside `~/.aws/credentials` using nano. |
| `NoCredentialsError` | Check the credentials file’s location, `[default]` heading and entries. Never print the secret to debug it. |
| Invalid or expired credentials | Use valid replacement credentials; temporary credentials also need their session token. |
| `AccessDenied` | AWS recognised the request but did not permit the action. Check permissions with the tutor or administrator. |
| `NoSuchKey` | The full key must match exactly. We corrected `chatbot-intnet.json` to `chatbot-intent.json`. List keys to check spelling and capitalisation. |
| `NameError: name 'json' is not defined` | Add `import json` at the top and rerun the whole script. |
| `NameError: name 'bucket_name' is not defined` | Define `bucket_name` before use. In our script it had accidentally been commented out with `#`. |
| `NameError` for `pd` or `io` | Add `import pandas as pd` or `import io` before using it. |
| `ModuleNotFoundError` for Boto3 or pandas | Install the package into the same virtual environment used by PyCharm to run the script. |
| Extra `Unnamed: 0` column on reading a CSV | The pandas index was saved. Use `index=False` when exporting, or `index_col=0` when restoring that index. |
| An empty body on a second read | The response stream was already consumed. Call `get_object()` again. |

## Key definitions

| Term | Simple meaning |
| --- | --- |
| AWS | Amazon’s cloud services platform. |
| S3 | AWS storage service used here to keep files as objects in buckets. |
| Boto3 | Python’s AWS SDK: the library we use to send AWS requests. |
| Client | An interface providing operations such as `get_object()` and `put_object()`. |
| Credentials | Access information used to authenticate requests. Permissions determine which actions are allowed. |
| Region | An AWS geographical area; we configured Frankfurt, `eu-central-1`. |
| JSON | A text format used to exchange structured data. |
| CSV | A text format for tables, usually separating fields with commas. |
| DataFrame | A pandas table with rows and columns. |
| `Body` | The contents of an S3 object being retrieved or uploaded. |
| Stream | Data read progressively; reading a response body moves its position forward. |
| Buffer / `StringIO` | A file-like place in memory that holds text temporarily. |
| Serialise | Convert Python data into a format that can be stored or transferred. |
| `json.load()` / `json.loads()` | Read JSON from a file-like object / parse JSON text or bytes. |
| `json.dump()` / `json.dumps()` | Write JSON to a file-like object / return JSON text. |
| `pp()` | A shorter name for `pprint()`, which displays Python data neatly. |

## References

1. AWS. [What is Amazon S3?](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html)
2. AWS. [Boto3 Quickstart: installation and configuration](https://docs.aws.amazon.com/boto3/latest/guide/quickstart.html)
3. AWS. [Boto3 S3 `get_object()`](https://docs.aws.amazon.com/boto3/latest/reference/services/s3/client/get_object.html)
4. Python. [JSON encoding and decoding](https://docs.python.org/3/library/json.html)
5. pandas. [`read_csv()`](https://pandas.pydata.org/docs/reference/api/pandas.read_csv.html)
6. AWS. [Boto3 S3 `put_object()`](https://docs.aws.amazon.com/boto3/latest/reference/services/s3/client/put_object.html)
7. Python. [`io.StringIO`](https://docs.python.org/3/library/io.html#io.StringIO)
8. pandas. [`DataFrame.to_csv()`](https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.to_csv.html)
9. AWS. [`list_objects_v2()`](https://docs.aws.amazon.com/boto3/latest/reference/services/s3/client/list_objects_v2.html) and [`upload_file()`](https://docs.aws.amazon.com/boto3/latest/reference/services/s3/client/upload_file.html)

Prepared from the S3 training examples and troubleshooting on 5 November 2026.
