# Physiological and other continuous recordings

Physiological recordings such as cardiac and respiratory signals and other
continuous measures (such as parameters of a film or audio stimuli) MAY be
specified using two files:

1.  a [gzip](https://datatracker.ietf.org/doc/html/rfc1952)
    compressed TSV file with data (without header line)

1.  a JSON file for storing metadata fields (see below)

[Example datasets](https://github.com/bids-standard/bids-examples)
with physiological data have been formatted using this specification
and can be used for practical guidance when curating a new dataset:

-   [`7t_trt`](https://github.com/bids-standard/bids-examples/tree/master/7t_trt)
-   [`ds210`](https://github.com/bids-standard/bids-examples/tree/master/ds210)

Template:

```Text
sub-<label>/[ses-<label>/]
    <datatype>/
        <matches>[_recording-<label>]_physio.tsv.gz
        <matches>[_recording-<label>]_physio.json
        <matches>[_recording-<label>]_stim.tsv.gz
        <matches>[_recording-<label>]_stim.json
```

For the template directory name, `<datatype>` can correspond to any data
recording modality, for example `func`, `anat`, `dwi`, `meg`, `eeg`, `ieeg`,
or `beh`.

In the template filenames, the `<matches>` part corresponds to task filename
before the suffix.
For example for the file `sub-control01_task-nback_run-1_bold.nii.gz`,
`<matches>` would correspond to `sub-control01_task-nback_run-1`.

Note that when supplying a `*_<physio|stim>.tsv.gz` file, an accompanying
`*_<physio|stim>.json` MUST be supplied as well.

The [`recording-<label>`](../appendices/entities.md#recording)
entity MAY be used to distinguish between several recording files.
For example `sub-01_task-bart_recording-eyetracking_physio.tsv.gz` to contain
the eyetracking data in a certain sampling frequency, and
`sub-01_task-bart_recording-breathing_physio.tsv.gz` to contain respiratory
measurements in a different sampling frequency.

Physiological recordings (including eyetracking) SHOULD use the `_physio`
suffix, and signals related to the stimulus SHOULD use `_stim` suffix.

The following table specifies metadata fields for the `*_<physio|stim>.json` file.

<!-- This block generates a metadata table.
These tables are defined in
  src/schema/rules/sidecars
The definitions of the fields specified in these tables may be found in
  src/schema/objects/metadata.yaml
A guide for using macros can be found at
 https://github.com/bids-standard/bids-specification/blob/master/macros_doc.md
-->
{{ MACROS___make_sidecar_table(["continuous.Continuous", "continuous.Physio"]) }}

Additional metadata may be included as in
[any TSV file](../common-principles.md#tabular-files) to specify, for
example, the units of the recorded time series.
Please note that, in contrast to other TSV files in BIDS, the TSV files specified
for physiological and other continuous recordings *do not* include a header
line.
Instead the name of columns are specified in the JSON file (see `Columns` field).
This is to improve compatibility with existing software (for example, FSL, PNM)
as well as to make support for other file formats possible in the future.
As in any TSV file, column names MUST NOT be blank (that is, an empty string),
and MUST NOT be duplicated within a single JSON file describing a headerless
TSV file.

Example `*_physio.tsv.gz`:

<!-- This block generates a file tree.
A guide for using macros can be found at
 https://github.com/bids-standard/bids-specification/blob/master/macros_doc.md
-->
{{ MACROS___make_filetree_example(
   {
   "sub-control01": {
      "func": {
         "sub-control01_task-nback_physio.tsv.gz": "",
         },
      },
   }
) }}

(after decompression)

```Text
34    110    0
44    112    0
23    100    1
```

Example `*_physio.json`:

<!-- This block generates a file tree.
A guide for using macros can be found at
 https://github.com/bids-standard/bids-specification/blob/master/macros_doc.md
-->
{{ MACROS___make_filetree_example(
   {
   "sub-control01": {
      "func": {
         "sub-control01_task-nback_physio.json": "",
         },
      },
   }
) }}

```JSON
{
    "SamplingFrequency": 100.0,
    "StartTime": -22.345,
    "Columns": ["cardiac", "respiratory"],
    "Manufacturer": "Brain Research Equipment ltd.",
    "cardiac": {
        "Description": "continuous pulse measurement",
        "Units": "mV"
        },
    "respiratory": {
        "Description": "continuous measurements by respiration belt",
        "Units": "mV"
        }
}
```

Note how apart from the general metadata fields like `SamplingFrequency`, `StartTime`, `Columns`,
and `Manufacturer`,
each individual column in the TSV file may be documented as its own field in the JSON file
(identical to the practice in other TSV+JSON file pairs).
Here, only the `Description` and `Units` fields are shown, but you may use any other of the
[defined fields](../common-principles.md#tabular-files) such as `TermURL`, `LongName`, and so on.
In this example, the `"cardiac"` and `"respiratory"` time series are produced by devices from
the same manufacturer and follow the same sampling frequency.
To specify different sampling frequencies or manufacturers, the time series would have to be split
into separate files like `*_recording-cardiac_physio.<tsv.gz|json>` and `*_recording-respiratory_physio.<tsv.gz|json>`.

## Recommendations for specific use cases

To store pulse or breathing measurements, or the scanner trigger signal, the
following naming conventions SHOULD be used for the column names:

<!-- This block generates a columns table.
The definitions of these fields can be found in
  src/schema/rules/tabular_data/*.yaml
and a guide for using macros can be found at
 https://github.com/bids-standard/bids-specification/blob/master/macros_doc.md
-->
{{ MACROS___make_columns_table("physio.PhysioColumns") }}

For any other data to be specified in columns, the column names can be chosen
as deemed appropriate by the researcher.

Recordings with different sampling frequencies or starting times should be
stored in separate files
(and the [`recording-<label>`](../appendices/entities.md#recording)
entity MAY be used to distinguish these files).

If the same continuous recording has been used for all subjects (for example in
the case where they all watched the same movie), one file MAY be used and
placed in the root directory.
For example, `task-movie_stim.tsv.gz`

For motion parameters acquired from MRI scanner side motion correction, the
`_physio` suffix SHOULD be used.

For multi-echo data, a given `physio.tsv` file is applicable to all echos of
a particular run.
For example:

<!-- This block generates a file tree.
A guide for using macros can be found at
 https://github.com/bids-standard/bids-specification/blob/master/macros_doc.md
-->
{{ MACROS___make_filetree_example(
   {
   "sub-01": {
      "func": {
        "sub-01_task-cuedSGT_run-1_physio.tsv.gz": "",
        "sub-01_task-cuedSGT_run-1_echo-1_bold.nii.gz": "",
        "sub-01_task-cuedSGT_run-1_echo-2_bold.nii.gz": "",
        "sub-01_task-cuedSGT_run-1_echo-3_bold.nii.gz": "",
         },
      },
   }
) }}
