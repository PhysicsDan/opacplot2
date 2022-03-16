# opacplot2

[![Build Status](https://travis-ci.org/flash-center/opacplot2.svg?branch=master)](https://travis-ci.org/flash-center/opacplot2)
[![Docs](https://readthedocs.org/projects/opacplot2/badge/?version=latest)](http://opacplot2.readthedocs.io/en/latest/)

Python package for manipulating Equation of State (EoS) and Opacity data.

Opacplot2 includes command-line tools to make EoS tables of various formats
available to FLASH. `opac-convert` converts EoS/opacity tables like SESAME,
Propaceos and TOPS into a FLASH-readable IONMIX (.cn4) format. `opac-error` generates plots comparing the
contents of two EoS tables for the same material. It is particularly useful for checking
the consistency of conversions made with `opac-convert`.

### Current Status

| | EoS | Opacity| opac-convert | opac-error |
|:-------|-----|-----|--------|--------|
| SESAME |:heavy_check_mark:|| :heavy_check_mark: | :heavy_check_mark: | 
| MULTI&#8224; ||:heavy_check_mark:| :heavy_check_mark: | |
| Propaceos* |:heavy_check_mark:|| :heavy_check_mark: | :heavy_check_mark:  | 
| TOPS&#8225; |:heavy_check_mark:|:heavy_check_mark:| :heavy_check_mark: || 


*<sub>The Propaceos reader is only distributed with a Propaceos license.<sub>

&#8224;<sub>Only opacity parsers are available for MULTI.<sub>

&#8225;<sub>Only average free electron number and average square free electron number in EoS<sub>

### Dependencies

`opacplot2`'s dependencies include:

* numpy 
* six 
* tables 
* matplotlib 
* scipy
* periodictable
* hedp (https://github.com/luli/hedp)
* interpolation (optional for fast interpolation)
* numba (optional for fast planck integral calculation)
* beautifulsoup4 (optional for `tops_html2txt`)

They can be installed as follows:

```shell
pip install numpy six tables matplotlib scipy periodictable
pip install git+https://github.com/luli/hedp
```

For OS X users, it is advised to use Anaconda Python to install `matplotlib` in order to avoid framework errors:

```shell
conda install matplotlib
```


### Installation 

This module requires Python 2.7 or 3.5. The latest version can be installed with

```shell
pip install git+https://github.com/flash-center/opacplot2
```

If you have the Propaceos Python reader, in order to include it in the 
installation, you must install `opacplot2` as follows:

```shell
git clone https://github.com/flash-center/opacplot2
cp /path/to/opg_propaceos.py opacplot2/opacplot2/
cd opacplot2
python setup.py install
```

### Documentation & Wiki

Full documentation can be found [here](http://opacplot2.readthedocs.io/en/latest/intro.html).

Example output from `opac-convert` may be found in the [wiki](https://github.com/flash-center/opacplot2/wiki).

---

### Warning About the SESAME Database

Many SESAME EoS files have multiple materials. To ensure conversion accuracy,
it is important that the SESAME file must have only one material. In order to aid in the process
of extracting one material's table from an entire SESAME document, included in the command line tools
of `opacplot2` is `sesame-extract`, which will extract a single material table from a SESAME
document based on the material ID.

<a name="opac-convert"></a>
# opac-convert

Command line tool for converting EoS Table formats into the IONMIX format
that comes with `opacplot2`.

Supported input file formats:

* Propaceos (not distributed)
* SESAME (.ses)
* MULTI (.opp, .opr, .opz, .eps)
* TOPS (.html, .tops)

The only supported output format is IONMIX.

### Usage

```bash
opac-convert [options] myfile.ext
```

`opac-convert` will attempt to read your file extension and convert it to
IONMIX accordingly.
If it is unable to read the extension, you can use the input flag to specify
your filetype.
Some files need additional information to write to IONMIX,
such as atomic numbers. These you must specify with the command line options
shown below.

### Options

| Option | Action |
|:-------|--------|
|-i, --input| Specify the input filetype (`propaceos`, `sesame`, `multi`)|
|--Znum| Comma separated list of atomic numbers.|
|--Xfracs| Comma separated list of element fractions.|
|--outname| Specify the output filename.|
|--log| Comma separated list of logarithmic data.|
|--tabnum| SESAME table number (defaults to last).|

### Example

To specify the files atomic numbers, one may use `--Znum` with a comma separated
list of integers. If more than one atomic number is given, 
one must also specify the element fractions with `--Xfracs`.
For example, take a SESAME table for CH named `myfile.ses`:

```bash
opac-convert --Znum 1,6 --Xfracs .5,.5 myfile.ses
```

This will convert `myfile.ses` to an IONMIX file named `myfile.cn4`.

### Logarithmic Data 

If you would like to take the log of the data before you write it to the IONMIX
file, use `--log` with a comma separated list of the data keys as shown below.
Each key specified will be written to IONMIX after the base 10 logarithm has
been applied.

| Data | key |
|------|-----|
|Ion number density|`idens`|
|Temperatures|`temps`|
|Average ionization|`Zf_DT`|
|Ion pressure|`Pi_DT`|
|Electron pressure|`Pec_DT`|
|Ion internal energy|`Ui_DT`|
|Electron internal energy|`Uec_DT`|
|Opacity bounds|`groups`|
|Rosseland mean opacity|`opr_mg`|
|absorption Planck mean opacity|`opp_mg`|
|emission Planck mean opacity|`emp_mg`|


For example, in order to specify that the emission Planck Mean Opacity be written
logarithmically:

```bash
opac-convert --log emp_mg my-file.ext
```

### Troubleshooting

#### Invalid Literal for `int()`

The `--log` flag may be used to fix the following error: 

```bash
ValueError: invalid literal for int() with base 10
```

This error arises when the exponent for the data is more than 2 digits long, 
which IONMIX does not support. 
What that usually means is that the data was originally
stored logarithmically and must be written back to IONMIX as logarithmic data.

---

<a name="opac-error"></a>
# opac-error

Command line tool for comparing two EoS table data. Unlike `opac-convert`, this tool will only
compare equation of state data (not opacity). It is particularly useful in checking the consistency of `opac-convert` by comparing
the original file with the converted IONMIX output.

Supported input file formats:

* Propaceos (not distributed, contact jtlaune at uchicago dot edu.)
* SESAME (.ses)
* IONMIX

The output will consist of an error report with maximum absolute % error RMS % error.
All % errors are calculated with respect to the first file listed.
The user may also opt to create % error plots for the EoS data stored in the file
(ion/electron pressure and energy and average ionization).
These consist of three plots with resolutions of 10%, 1%, and .01%.

### Usage

```bash
opac-error [options] myfile_1.ext myfile_2.ext
```

Much like `opac-convert`, this tool will first attempt to read the extensions from your EoS tables
in order to open them up. However, some file types require additional information, as in 
`opac-convert`, which can be specified in the `[options]`. The options have many of the same
names as `opac-convert` but suffixed by `_1` for file 1 or `_2` for file 2.

Then, `opac-error` will create error reports for the following data:

* Average ionization
* Electron Ppessure
* Ion pressure
* Electron energy
* Ion energy

Included in the error report, we have

* Root mean squared % error
* Maximum absolute % error

If the `--plot` flag is called, `opac-convert` will also make error plots and save them as images
to the current directory.

### Options

| Option | Action |
|:-------|--------|
|--filetypes| Comma separated list of file types.|
|--mpi_#| Mass per ion (**in grams**) for IONMIX file 1 or 2.|
|--Znum_#| Comma separated list of atomic numbers for file 1 or 2.|
|--Xfrac_#| Comma separated list of number fractions for file 1 or 2.|
|--filters_#| `dens_filter, temp_filter` for file 1 or 2.|
|--tabnum_#| SESAME table number for file 1 or 2.|
|--plot| Create % error plots for data.|
|--writelog| Write log file with % errors for data.|
|--lin_grid| Plot using linear axes.|

### Example

See the [wiki](https://github.com/flash-center/opacplot2/wiki).

### How It Works

First, `opac-convert` takes a conservative intersection of the two dens/temp grids from each file.
Then it linearly interpolates the data from both files onto the intersection dens/temp grids.
 Using the interpolated data, it is able to create an error report.


<a name="sesame-extract"></a>
# sesame-extract

### Usage

Check syntax possibilities:

```bash
python sesame_extract.py -h
```

Extract water (SESAME element 7150 - see the publicly available PDF) information
from the 160 MB SESAME ASCII file, and save it into "water.ses":

```bash
python sesame_extract.py -o "./water.ses" "$HOME/SESAME/sesame-ec/sesame_ascii" 7150
```
