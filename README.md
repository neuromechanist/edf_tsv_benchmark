# edf_tsv_benchmark
Comparsion between tsv.gz and EDF timeseries data storage

**TLDR**: EDF is faster in both read and write by ~10X than the native functions for tsv.gz

**Case 1**: A sample HBN dataset with ~30 columns of eye-tracking raw data and its derivatives. Data has ~19k rows (at 120Hz, about 2.5 minutes). So, it is not super long or high-freq.
Method:
a) Read the data. Remove/adjust the headers, and non-compliant columns. the time column is noncompliant with EDF since it uses >8-byte integers. You can remove it or subtract the first time-element from the time column.

b) Create a concatenated version of the data to simulate longer (or higher-frequency datasets). I concatenated the data with itself 10 times. So there are 10 data matrices, each with 30 columns, but with n, n*2, n*3, …., n*10 rows.
 
c)  Save the data metrices using `edfwrite`, BIDS `tsvwrite`, and `writematrix`. Note the `writematrix` does not support the `tsv` name. So, the files should be saved in `txt`, then renamed and then zipped.

d) Read the data files using `edfread`, BIDS `tsvread`, and `readmatrix`.

Except for the BIDS functions, each process was done five times to ensure stability.

Results are: Write: 1.7 sec, 960 sec, and 15 sec respectively.
		      Read: 0.16 sec, 173.4 sec, and 4.30 sec respectively.

For case 1 which has extended column count (~30 columns):
Writing tsv.gz is 900X slower using BIDS `tsvwrite` and 15X slower using native `writematrix` than writing EDF using native `edfwrite`.
Reading tsv.gz is 1000X slower using BIDS `tsvread` , 25X slower using native `readmatrix` than reading EDF using native `edfread`.

**Case 2**: Picking only **seven** columns of the HBN sample and redo the tests:

Results are: Write: 0.47 sec, 250 sec, and 3.6 sec respectively.
		      Read: 0.15 sec, 41.7 sec, and 1.13 sec respectively.

**Conclusion:**
1. Reading and writing EDF files are considerably faster than tsv files.
2. EDF files also enforce extra metadata to ensure “minimum” data quality.
3. Current BIDS `tsvread` and `tsvwrite` functions [link](https://github.com/bids-standard/bids-matlab/tree/master/%2Bbids/%2Butil) (while convenient) are not optimized for large datasets
