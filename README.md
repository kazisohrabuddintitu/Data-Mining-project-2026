# Data-Mining-project-2026

```text
project/
├── dataset/
│   ├── data.csv          # full dataset (not tracked, ~5 GB)
│   ├── data.parquet      # fast copy, created by the Task 1 notebook (not tracked)
│   └── data_500.csv      # first 500 rows, all columns
├── notebooks/
│   └── task1_data_understanding_preparation.ipynb
├── report/
│   ├── main.tex          # report source (LaTeX template)
│   ├── references.bib
│   └── images/
├── requirements.txt      # pip install -r requirements.txt
└── dm_-_project_2026_published.pdf   # project assignment
```


### How CSV stores the data

A CSV is plain text written row by row:

```text
ioCountry,postid,follower_count
ecuador,0bafe020,1528
ecuador,608fba51,1528
ecuador,514b2c38,35
```

### How Parquet stores the data

Parquet writes the same table column by column:

```text
[ecuador, ecuador, ecuador]        <- all ioCountry values
[0bafe020, 608fba51, 514b2c38]     <- all postid values
[1528, 1528, 35]                   <- all follower_count values
footer: schema (column names and types) + where each block starts
```

For example, `ioCountry` is one column of 8.7 million values, cut into 76 pieces on disk:

```text
row group 1:  ioCountry = [ecuador, ecuador, ..., ecuador]    ~115,000 values
row group 2:  ioCountry = [ecuador, ..., serbia, serbia, ...]  ~115,000 values
...
row group 76: ioCountry = [...]                                ~115,000 values
                                                     total = 8,714,356 values
```

`load(["ioCountry"])` reads these 76 pieces, joins them in order and returns a DataFrame of shape `(8714356, 1)`.

### Why 76 row groups

```python
read_options = pv.ReadOptions(block_size=64 << 20)
```

`64 << 20` is 64 × 2²⁰ bytes = 64 MB. pyarrow reads the CSV 64 MB of text at a time, and every batch is written with `writer.write_batch(batch)`, which makes one row group.

```text
5.03 GB / 64 MB = 5,030,000,000 / 67,108,864 ≈ 75.0
→ 75 full batches + 1 small last batch = 76 row groups
8,714,356 rows / 76 ≈ 115,000 rows per row group
```