---
date: 2023-01-28
title: Apache Arrow and Parquet Files
subtitle: A Very Brief Primer
summary: Apache Arrow is a development platform for in-memory analytics. It contains a set of technologies that enable big data systems to process and move data fast. It specifies a standardized language-independent columnar memory format for flat and hierarchical data, organized for efficient analytic operations on modern hardware.
format: hugo
draft: false
slug: arrow-and-parquet
categories:
- TIL
---

## Apache Arrow

> Apache Arrow is a development platform for in-memory analytics. It contains a set of technologies that enable big data systems to process and move data fast. It specifies a standardized language-independent columnar memory format for flat and hierarchical data, organized for efficient analytic operations on modern hardware. [https://arrow.apache.org/](https://arrow.apache.org/docs/index.html)

``` python
import pyarrow as pa
```

## Arrow Data Types

**Type Metadata**: Describes the data types in arrays, schemas, record batches, and tables (*analog to regular data types in `R` or `python`*).

``` python
pa.int32()
```

    DataType(int32)

``` python
pa.string()
```

    DataType(string)

``` python
pa.timestamp('ms')
```

    TimestampType(timestamp[ms])

**Schemas**: Describes a named collection of data types, e.g., in record batches or tables (*analog to column names and data types in `data.frame()` or `pd.DataFrame()`*).

``` python
py_schema = pa.schema([('col1', pa.float64()), ('col2', pa.int32()), ('col3', pa.string()), ('col4', pa.binary())])
py_schema
```

    col1: double
    col2: int32
    col3: string
    col4: binary

**Arrays**: Instances of atomic, contiguous columnar data structures of a given data type (*analog to `c()` in `R` or `np.array()` in `python`*).

``` python
single = pa.array([1, 2, None, 3])
single
```

    <pyarrow.lib.Int64Array object at 0x00000000640113A0>
    [
      1,
      2,
      null,
      3
    ]

``` python
nested = pa.array([[], None, [1, 2], [None, 1]])
nested
```

    <pyarrow.lib.ListArray object at 0x0000000064011220>
    [
      [],
      null,
      [
        1,
        2
      ],
      [
        null,
        1
      ]
    ]

**Record Batches**: A collection of Arrays (of the same length) with a particular Schema.

``` python
batch = pa.RecordBatch.from_arrays(
  [pa.array([1, 2, 3, 4]),
   pa.array(['hel', 'lo', 'world', None]),
   pa.array([True, None, False, True])],
  ['col1', 'col2', 'col3']
)

batch.num_columns
```

    3

``` python
batch.schema
```

    col1: int64
    col2: string
    col3: bool

**Tables**: A tabular data structure in which each column consists of one or more arrays of the same type. (extension to the base Apache Arrow data types)

``` python
table = pa.Table.from_batches([batch])
table
```

    pyarrow.Table
    col1: int64
    col2: string
    col3: bool

**Datasets**: Provide functions to handle tabular, potentially larger than memory, and multi-file datasets.

``` python
from pathlib import Path

files = list(Path(DIR_CSV).glob('*.csv'))

print(
  f'Number of docs: {len(files)}\n\n',
  f'First 10 docs: {[f.name for f in files[:10]]}'
)
```

    Number of docs: 205465

     First 10 docs: ['1000041.csv', '1000091.csv', '1000191.csv', '1000351.csv', '100040.csv', '1000451.csv', '1000471.csv', '1000491.csv', '100051.csv', '1000591.csv']

``` python
from pyarrow import csv
import pyarrow.dataset as ds

opts = csv.ParseOptions(delimiter=";")
dataset = ds.dataset(files, format=ds.CsvFileFormat(parse_options=opts))
```

``` python
print(dataset.schema.to_string(show_field_metadata=True))
```

    : int64
    section: string
    speaker: string
    role: string
    text: string

``` python
dataset.to_table().to_pandas()
```

                   ...                                               text
    0           0  ...  Good morning, ladies and gentlemen, and welcom...
    1           1  ...  Thank you, operator, and good morning, everyon...
    2           2  ...  Thank you, John, and thank you all for joining...
    3           3  ...  Thank you, Armando, and good morning everyone....
    4           4  ...  Thanks, Keith. In conjunction with the financi...
    ...       ...  ...                                                ...
    13360899   98  ...                            Okay, great, thank you.
    13360900   99  ...  Ladies and gentlemen, we have reached the end ...
    13360901  100  ...  I'd like to thank everybody for joining us thi...
    13360902  101  ...  Thank you. That does conclude today's teleconf...
    13360903    0  ...  This is Laura Brown, Senior Vice President, Co...

    [13360904 rows x 5 columns]

In-memory filter

``` python
dataset \
  .to_table(filter=ds.field('role') == 'Firm') \
  .to_pandas()
```

                   section  ...  role                                               text
    0          1  scripted  ...  Firm  Thank you, operator, and good morning, everyon...
    1          2  scripted  ...  Firm  Thank you, John, and thank you all for joining...
    2          3  scripted  ...  Firm  Thank you, Armando, and good morning everyone....
    3          4  scripted  ...  Firm  Thanks, Keith. In conjunction with the financi...
    4          7       Q&A  ...  Firm  Let me take the first one, and it's around lau...
    ...      ...       ...  ...   ...                                                ...
    6577968   91       Q&A  ...  Firm  Not at the moment. The coal power plant we tal...
    6577969   95       Q&A  ...  Firm  Tom, it's actually not the cost of the hedge, ...
    6577970   97       Q&A  ...  Firm  Again yes, but consider the fact that their di...
    6577971  100       Q&A  ...  Firm  I'd like to thank everybody for joining us thi...
    6577972    0  scripted  ...  Firm  This is Laura Brown, Senior Vice President, Co...

    [6577973 rows x 5 columns]

``` python
dataset \
  .to_table(filter=(ds.field('role') == 'Firm') & (ds.field('section') == 'Q&A')) \
  .to_pandas()
```

                 section  ...  role                                               text
    0          7     Q&A  ...  Firm  Let me take the first one, and it's around lau...
    1          8     Q&A  ...  Firm  I think consistent with what we've said in the...
    2         11     Q&A  ...  Firm  I think as I addressed in the presentation, th...
    3         14     Q&A  ...  Firm  I think that, as I mentioned in the prepared r...
    4         17     Q&A  ...  Firm  Yeah, I think we would be in a position to lau...
    ...      ...     ...  ...   ...                                                ...
    5614213   89     Q&A  ...  Firm  We've been looking at a coal plant in part of ...
    5614214   91     Q&A  ...  Firm  Not at the moment. The coal power plant we tal...
    5614215   95     Q&A  ...  Firm  Tom, it's actually not the cost of the hedge, ...
    5614216   97     Q&A  ...  Firm  Again yes, but consider the fact that their di...
    5614217  100     Q&A  ...  Firm  I'd like to thank everybody for joining us thi...

    [5614218 rows x 5 columns]

Column selection

``` python
dataset \
  .to_table(columns=['section', 'speaker', 'text']) \
  .to_pandas()
```

               section  ...                                               text
    0         scripted  ...  Good morning, ladies and gentlemen, and welcom...
    1         scripted  ...  Thank you, operator, and good morning, everyon...
    2         scripted  ...  Thank you, John, and thank you all for joining...
    3         scripted  ...  Thank you, Armando, and good morning everyone....
    4         scripted  ...  Thanks, Keith. In conjunction with the financi...
    ...            ...  ...                                                ...
    13360899       Q&A  ...                            Okay, great, thank you.
    13360900       Q&A  ...  Ladies and gentlemen, we have reached the end ...
    13360901       Q&A  ...  I'd like to thank everybody for joining us thi...
    13360902       Q&A  ...  Thank you. That does conclude today's teleconf...
    13360903  scripted  ...  This is Laura Brown, Senior Vice President, Co...

    [13360904 rows x 3 columns]

Save to disk as partitioned Parquet files

*Note: The partitioning scheme is using `key=value` dir names, analog to Apache Hive.*

``` python
ds.write_dataset(
  dataset,
  "./partitioned",
  format="parquet",
  partitioning=ds.partitioning(pa.schema([("section", pa.string())]))
)
```

Read partitions from disk

``` python
dataset = ds.dataset("./partitioned", format="parquet")

i = 0
for record_batch in dataset.to_batches():
    col1 = record_batch.column('text')
    print(f'{col1._name}: {col1[0]} [...] {col1[-1]}\n')
    i += 1
    if i > 5:
      break
```

    text: Thank you. And our question comes from Nicolas Chialva with Itaú BBA. [...] That concludes today's conference. Thank you for your participation.

    text: And our first question comes from Robert Baird of Milleni Securities. [...] Okay. Thank you.

    text: Thank you. Your first question comes from William Bremer of Maxim Group. [...] Thank you.  That concludes today's conference.  You may now disconnect.

    text: Your first question comes from the line of John Reucassel from BMO Capital Markets. Your line is now open. [...] Ladies and gentlemen, this concludes today's conference call. You may now disconnect.

    text: Thank you. And we'll go first to Michael Schmidt at Leerink Swann. [...] Thank you very much operator, and thank you all this morning for joining us. We look forward to continuing to update you again in the coming months. Have a great day.

    text: And your first question comes from the line of Andy Yeung with Oppenheimer. [...] Thank you. This concludes today's conference call. You may now disconnect.

## Parquet Format

[Parquet](https://parquet.apache.org/) is an open source file format commonly employed by big data systems:

-   Parquet files are column-oriented which makes it a performant file format.
-   Parquet files are in a custom binary format which implements an efficient encoding and enables greater compression.
-   Parquet files are type-aware in the sense that they store metadata about column types.
-   Parquet files allow for natural partitioning of the data to improve the performance of preprocessing operations.  
    *Rule-of-thumb: Create partitions of sizes that range between 20MB and 2GB and choose partition variables that enable efficient filtering.*
