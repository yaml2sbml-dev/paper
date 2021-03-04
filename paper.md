# yaml2sbml: Human readable and writable specification of ODE models and its specification to SBML files

---

title: yaml2sbml: Human readable and writable specification of ODE models and its specification to SBML files

tags:

  - Python

  - SBML

  - Ordinary Differential Equation

  - YAML

  authors:

  - name: Jakob Vanhoefer
    orcid: 0000-0002-3451-1701
    affiliation: 1

  - name: Marta R. A. Matos
    orcid: 0000-0003-4288-1005
    affiliation: 2

  - name: Dilan Pathirana
    orcid: 0000-0001-7000-2659
    affiliation: 1

  - name: Yannik Schälte
    orcid: 0000-0003-1293-820X
    affiliation: "3, 4" # (Multiple affiliations must be quoted)

  - name: Jan Hasenauer
    orcid: 0000-0002-4935-3312
    affiliation: "1, 3, 4" # (Multiple affiliations must be quoted)

affiliations:


    - name: Faculty of Mathematics and Natural Sciences, University of Bonn, Bonn, Germany
    index: 1
    
    - name: The Novo Nordisk Foundation Center for Biosustainability, Technical University of Denmark, DK-2800 Kgs. Lyngby, Denmark
    
    index: 2
    
    - name: Institute of Computational Biology, Helmholtz Zentrum München, Neuherberg, Germany
    index: 3
    
    - name: Department of Mathematics, Technical University Munich, Garching, Germany
    index: 4

date: 05 March 2021

bibliography: bibliography.bib


---

# Summary

Ordinary differential equation (ODE)  models are used in various fields of natural sciences to describe dynamic processes. In the life sciences, they are mostly stored and exchanged using the the Systems Biological Markup Language (SBML),  a widely adopted community standard based on XML. The Parameter Estimation table (PEtab) format builds on SBML to describe parameter estimation problems. There exists a large number of software pipelines for simulating SBML models and performing parameter estimation for PEtab problems. Hence, specifying ODE models in SBML and parameter estimation problem in PEtab provides access to a broad spectrum of tools. However, SBML is considered to be neither human-readable nor human-writable. An easy-to-use approach to construct the SBML/PEtab models for a given ODE model will facilitate model generation. 

In this contribution, we present `yaml2sbml`, a Python tool for converting ODE models specified in an easy to read and write YAML file into SBML/PEtab.  `yaml2sbml` comes with a format validator for the input YAML, a command-line interface and a model editor to: 1) create an ODE model programmatically in Python which can then be written into SBML/PEtab or YAML and 2) edit an ODE model previously encoded in YAML. Several examples illustrate the use of `yaml2sbml` on realistic problems.

# Statement of need

Ordinary differential equations (ODEs) are a vital tool in mechanistic modelling of biological processes. ODE models span a wide range of scales and applications, from molecular to population level [@FroehlichKes2018, @RaimundezDud2020_arxiv] and arise e.g. as discretizations of Partial Differential Equation models [@FischerFie2019] or moments of stochastic systems [@Engblom2006].

The Systems Biological Markup Language (SBML) is a widely adopted community standard for specifying biological reaction networks [@HuckaFin2003]. [sbml.org](http://sbml.org/SBML_Software_Guide/SBML_Software_Summary#cat_12) lists more than 100 software tools that accept SBML as their input format for dynamic model simulation, among them [@HoopsSah2006; @FroehlichWei2020; @RaueSte2015; @KaschekMad2019]. The [biomodels.net](biomodels.net) data base contains more than 1,000 published models in the SBML format [@NovereBor2006].

Model parameters can be estimated from data by formulating a likelihood function. Therefore, the  system states must be mapped to measured quantities by observable functions, and a measurement noise model must be specified. The PEtab format was recently introduced to complement SBML by tab-separated value files specifying observables, measurements, experimental conditions, and estimated parameters [@SchmiesterSch2020]. Currently 9 software toolboxes support PEtab as an input format, among them COPASI, D2D and AMICI/pyPESTO.

Despite its broad usage, constructing an SBML model from scratch is often tedious, as SBML is considered neither human-readable nor -writeable. In particular for persons from other research fields who might be interested in using the broad spectrum of available tools, it is demanding. Therefore, various approaches to facilitate model construction from text-based input formats or in code have been presented [@BornsteinKea2008; @CannistraMed2015; @GomezHuc2016; @SmithBer2009; @Poolman2006]. The tool by [@GomezHuc2016] offers a text based input format, that is tightly tied to MATLAB, as the tool translates MATLAB code into SBML. Other tools have a text based input format that is centered around chemical reactions and not around ODEs directly [e.g. @Poolman2006], or only offer a text-based [@SmithBer2009] or only a Python-based way of defining SBML models [@CannistraMed2015], but not both at the same time interchangeably. Also, none of the aforementioned tools support PEtab or other formats for specifying parameter estimation problems.

Here, we present a human-readable and -writeable format for ODE models based on YAML that can be validated and translated to SBML and PEtab via the Python tool `yaml2sbml` or a command line interface (CLI). Building the input format on YAML allows the input to be easily parsed and validated, while keeping the simplicity of a text-based input. Furthermore, `yaml2sbml` comes with a format validator and a Python-based model editor that allows to generate, import and export a YAML model within code.

# Tool and Format

## YAML Format

The YAML-based input format is organized in blocks for different model components. The allowed blocks are:

* `odes` define states, right hand sides and initial values (as numeric values or parameters).
* `parameters` define parameters, allows to initialize them to values. Further optional keys specify, e.g., optimization bounds. These values are written to the PEtab parameter table.
* `time` specifies the time variable of the ODE.
* `assignments` assign a variable dynamically to a value. These are encoded as parameter assignment rules in the SBML file.
* `functions` define functions that can be used in other model parts.
* `observables` map ODE states to measurements. Observables are (optionally) encoded as parameter assignments in the SBML file or the PEtab observable table.
* `conditions` define different experimental setups, e.g. specific inputs. Conditions are only encoded in the PEtab condition table. These do not affect the resulting SBML file.

## Python Tool for SBML/PEtab translation

The Python tool `yaml2sbml` allows to translate models specified in the YAML input format to SBML and PEtab via

```python
import yaml2sbml

# SBML translation
yaml2sbml.yaml2sbml(yaml_input_file, sbml_output_file)
# PEtab conversion
yaml2sbml.yaml2petab(yaml_input_file, PEtab_dir, model_name)
```

Internally `libsbml` [@BornsteinKea2008] generates the resulting SBML.

## Format Validation

`yaml2sbml` validates an input YAML via

```python
yaml2sbml.validate_yaml(yaml_file)
```

The validation is performed internally before translating the input YAML to SBML and PEtab. The SBML validator in `libsbml` and the validator in the PEtabs library check the SBML and PEtab files obtained after translation.

## Command Line Interface

Aside its Python API, `yaml2sbml` comes with a CLI offering SBML/PEtab conversion as well as format validation via the commands `yaml2sbml`, `yaml2petab`, and `yaml2sbml_validate`, respectively.

## Model Editor

The `yaml2sbml` model editor allows the user to construct YAML models within Python code e.g. via

```python
model = yaml2sbml.YamlModel()
model.add_ode(state_id='x',
              right_hand_side='3*x',
              initial_value=1)
```

The model editor provides functionality to programmatically add, delete, or modify components. Further, the model editor allows to import, modify and export models to YAML, SBML and PEtab.

## Availability and Code Development

`yaml2sbml` is developed and published under the MIT license on [GitHub](https://github.com/yaml2sbml-dev/yaml2sbml). The tool can be installed from `PyPI` via

```shell
pip install yaml2sbml
```

The repository contains an extensive [format documentation](https://yaml2sbml.readthedocs.io/en/latest/format_specification.html), further documentation is hosted on [readthedocs](https://yaml2sbml.readthedocs.io/en/latest/). Jupyter Notebooks presented below contain [various examples](https://github.com/yaml2sbml-dev/yaml2sbml/tree/master/doc/examples) covering all aspects of the tool. Code testing and continuous integration is performed via GitHub Actions.

# Examples

The functionality of `yaml2sbml` is illustrated using multiple notebooks.

## Introductory Examples

Three notebooks use the Lotka-Volterra equations [ref] - a small and well-established example - to introduce the different functionalities of `yaml2sbml`: The format and the basic functionality of the [Python toolbox](https://github.com/yaml2sbml-dev/yaml2sbml/blob/master/doc/examples/Lotka_Volterra/Lotka_Volterra_python/Lotka_Volterra.ipynb), the [CLI](https://github.com/yaml2sbml-dev/yaml2sbml/blob/master/doc/examples/Lotka_Volterra/Lotka_Volterra_CLI/Lotka_Volterra_CLI.ipynb) and the [model editor](https://github.com/yaml2sbml-dev/yaml2sbml/blob/master/doc/examples/Lotka_Volterra/Lotka_Volterra_CLI/Lotka_Volterra_CLI.ipynb) (Figure 1A). The [format features](https://github.com/yaml2sbml-dev/yaml2sbml/blob/master/doc/examples/Format_Features/Format_Features.ipynb) notebook showcases features of the format as time dependent or discontinuous right hand sides. (Figure 1B)

##  Applications Examples

The introductory examples are complemented by two more comprehensive examples, which do not fit in the classical reaction network formulation for which SBML is intended.

The first application example considers the approximation of the Chemical Master Equation [ref], a stochastic model used for (bio-)chemical processes. Specifically, we consider the Finite State Projection (FSP), which truncates the usually infinite state space, yielding a finite-dimensional ODE [@MunskyKha2006]. The [FSP example](https://github.com/yaml2sbml-dev/yaml2sbml/blob/master/doc/examples/Finite_State_Projection/Finite_State_Projection.ipynb) implements the FSP for a two-stage model of gene transmission as discussed e.g. in [@ShahrezaeiSwa2008] (Figure 1C). This example shows how `yaml2sbml` allows to implement a large scale ODE containing hundreds of states in less than 20 lines of code, by exploiting the problem structure.

The second application example considers a pharmacological process. We implement the Sorensen model, a well-established model of human Glucose-Insulin metabolism, describing the dynamics of glucose and insulin in different compartments by an ODE with 22 state variables [@Sorensen1985]. The [Jupyter Notebook](https://github.com/yaml2sbml-dev/yaml2sbml/blob/master/doc/examples/Sorensen/yaml2sbml_Sorensen.ipynb) presents an implementation of the Sorensen model in the YAML input format and uses the model editor to extend the preexisting YAML model to encode a patient specific treatment (Figure 1D).

![**Figure 1** Simulation results for different models specified using `yaml2sbml`. A) Lotka Volterra equation B) Discontinuous right hand side of the ODE C) Finite State Projection of a two-stage model of gene transmission D) Sorensen Model of human Glucose-Insulin metabolism.](Figure1.png)

# Acknowledgement

The authors thank Elba Raimúndez Álvarez for her help designing the logo.

# Funding

TBD
M.R.A.M. was funded by the Novo Nordisk Foundation (NNF10CC1016517 and NNF14OC0009473).

# References
