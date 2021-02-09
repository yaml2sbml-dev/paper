# yaml2sbml: Human readable and writable specification of ODE models in SBML 

---

title: yaml2sbml: Human readable and writable specification of ODE models in SBML 

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

date: 31 January 2021

bibliography: bibliography.bib
    

---

# Summary

Ordinary differential equations (ODEs) are commonly used to model biological systems. These models are generally encoded using the Systems Biological Markup Language (SBML), a widely adopted community standard based on XML. Since the SBML format does not allow for the specification of experimental conditions which are key for parameter estimation, the PEtab format was created as a SBML extension to describe parameter estimation problems.  Most software available for constructing and/or simulating ODE models in systems biology support the SBML format, and a significant number also support PEtab. Thus, specifying an ODE model in SBML is crucial.
However, SBML is considered to be neither human-readable nor human-writable, therefore an easy-to-use approach to construct the SBML/PEtab models for a given ODE model will facilitate model generation. In this contribution we present `yaml2sbml`, a python tool to convert ODE models specified in an easy to read and write YAML format into SBML/PEtab. Furthermore, `yaml2sbml` comes with a validator for the YAML input file, a Command Line Interface and a Model Editor to: 1) create an ODE model programmatically in python which can then be written into SBML/PEtab or YAML and 2) edit an ODE model previously encoded in YAML. Several examples highlighting the usage of `yaml2sbml` on realistic problems are provided. 

# Statement of need

Ordinary differential equations (ODEs) are a vital tool in mechanistic modelling of biological processes. ODE models span a wide range of scales and applications, from molecular to population level [@RaimundezDud2020_arxiv] and arise e.g. as discretizations of Partial Differential Equation models [@FischerFie2019] or moments of stochastic systems [@Engblom2006].

The Systems Biological Markup Language (SBML) is a widely adopted community standard for specifying mechanistic models of biological systems [@HuckaFin2003]. A large number of software tools support the SBML format [@HoopsSah2006; @FroehlichWei2020; @RaueSte2015; @KaschekMad2019], and [biomodels.net](biomodels.net), the main database for SBML models, contains more than 1000 published models [@NovereBor2006].
> [name=Yannik Schälte] Not sure if biomodels is needed

Model parameters can be estimated from data via a maximum likelihood or maximum a posteriori estimator. To do this the states of the system need to be mapped to measured quantities by an observable function and a measurement noise model needs to be specified. The PEtab format was recently introduced to extend the SBML format by specifying observables, measurements, experimental conditions and parameters to be estimated in tab-separated value files  [@SchmiesterSch2020]. Currently 9 software toolboxes support PEtab as an input format, among them Copasi and [TODO pick another one?]
> [name=Marta] Assuming here that Copasi is actually one of them xD

Despite its broad usage, constructing an SBML model from scratch is often tedious, as SBML is considered to be neither human-readable nor human-writeable. Therefore various approaches have been introduced to facilitate model construction, either from text based input formats or programatically [@BornsteinKea2008; @CannistraMed2015; @GomezHuc2016; @SmithBer2009; @Poolman2006]. However, these tools have a different focus than our approach. The tool by @GomezHuc2016 is tightly tied to MATLAB, as it translates MATLAB code into SBML; @Poolman2006's uses a plain text based input format that is centered on chemical reactions instead of ODEs directly; @SmithBer2009 only offer a plain text based way of defining the ODEs, while @CannistraMed2015's way of defining the ODEs is Python based.
> [name=Yannik Schälte] I think it is crucial to sufficiently clarify the difference to other tools. Sounds good, but maybe one can try to make it even clearer. Also, maybe mention graphical model construction tools?

Here we present a human readable and writeable format based on YAML to specify ODE models. This YAML based format can be validated and translated into SBML and PEtab via the Python tool `yaml2sbml`. YAML is also a standard format, which makes it straightforward to validate. In contrast to the aforementioned tools, our YAML based format is not tied to a specific programming language and allows the user to specify ODE models directly. In addition, the user can also specify parameter estimation problems using our format, which is then converted to a PEtab file besides the SBML. 
The Python tool `yaml2sbml` can be used both programatically inside Python code and via a command line interface (CLI), which does not require the user to use Python. Finally, `yaml2sbml` comes with a YAML format validator which can also be used through a CLI, and a Python-based model editor that allows the user to: import a YAML model, edit it using Python, and export the model to either YAML or SBML/PEtab, or even completely circumvent the YAML input and create a model from scratch.
> [name=Yannik Schälte] Maybe add that YAML is a very simple data-serialization language and programming language independent? I.e. highlight how easy it is to get into it and it requires no big setup. I think simplicity is yaml2sbml's strongest feature.


# Tool and Format

## YAML Format

The YAML-based input format is organized in blocks for different model components. The format allows for the following blocks:

* `odes:` Define states, right hand side and initial values (as numeric values or parameters).
* `parameters:` Define parameters, allows to initialize them to values. Further optional keys specify e.g. optimization bounds. These values are written in the PEtab parameter table.
* `time:` Specify the time variable of the ODE.
* `assignments:` Assignments assign a variable dynamically to a value. They are encoded as parameter assignment rules in the SBML model.
* `functions:` Define functions, that can be used in other parts of the model.
* `observables:` Maps ODE states to measurements. Observables are only encoded in PEtabs observable table. They do not have an efect on the resulting SBML.
* `conditions:` Defines different experimental conditions. Conditions are only encoded in the PEtabs condition table. They do not have an efect on the resulting SBML.

## Python Tool for SBML/PEtab translation 

The Python tool `yaml2sbml` allows to translate models, that are specified in the YAML input format to SBML and PEtab via

```python
import yaml2sbml

# SBML translation
yaml2sbml.yaml2sbml(yaml_input_file, sbml_output_file)
# PEtab conversion
yaml2sbml.yaml2petab(yaml_input_file, PEtab_dir, model_name)
```

Internally libsbml [@BornsteinKea2008] generates the resulting SBML.

## Format Validation

`yaml2sbml`  validates an input YAML  via

```python
yaml2sbml.validate_yaml(yaml_file)
```

The validation is also performed before translation to SBML and PEtab within the translation routines. Further `libsbml`s SBML validator and PEtabs linter check the SBML/PEtab files after tranlsation.

## Command Line Interface

Aside its Python API, `yaml2sbml` comes with a CLI that offers SBML/PEtab conversion as well as format validation via the commands `yaml2sbml`, `yaml2petab`, and `yaml2sbml_validate`, respectively .

## Model Editor

The `yaml2sbml` Model Editor allows to construct YAML models within Python code e.g. via

```python
model = yaml2sbml.YamlModel()
model.add_ode(state_id='x', 
              right_hand_side='3*x', 
              initial_value=1)
```

The Model Editor provides similar functionality to add, delete or modify the different components of YAML models. Further the Model Editor allows to import, modify and export models to YAML, SBML and PEtab (hence potentially circumventing YAML completely).

## Availability and Code Development

`yaml2sbml` is developed and published under the MIT license on [GitHub](https://github.com/yaml2sbml-dev/yaml2sbml). The tool can be `pip` installed via

```shell
pip install yaml2sbml
```

The repository contains an extensive [format documentation](https://github.com/yaml2sbml-dev/yaml2sbml/blob/master/doc/format_documentation.md), further documentation is hosted on [read th docs](https://yaml2sbml.readthedocs.io/en/latest/).  Jupyter notebooks contain [various examples](https://github.com/yaml2sbml-dev/yaml2sbml/tree/master/doc/examples) covering all aspects of the project. Code testing and continuous integration is performed via GitHub actions.

# Examples

`yaml2sbml`'s GitHub repository contains several jupyter notebooks for examples on a range of complexity. 

## Introductory Examples

Three notebooks use the Lotka-Volterra Equations as a small and well established example ODE to showcase the different functionalities of `yaml2sbml`: The format and the basic functionality of the [Python toolbox](https://github.com/yaml2sbml-dev/yaml2sbml/blob/master/doc/examples/Lotka_Volterra/Lotka_Volterra_python/Lotka_Volterra.ipynb), the [CLI](https://github.com/yaml2sbml-dev/yaml2sbml/blob/master/doc/examples/Lotka_Volterra/Lotka_Volterra_CLI/Lotka_Volterra_CLI.ipynb) and the [Model Editor](https://github.com/yaml2sbml-dev/yaml2sbml/blob/master/doc/examples/Lotka_Volterra/Lotka_Volterra_CLI/Lotka_Volterra_CLI.ipynb) (Figure 1A). The [format features](https://github.com/yaml2sbml-dev/yaml2sbml/blob/master/doc/examples/Format_Features/Format_Features.ipynb) notebook showcases features of the format as time dependent or discontinuous right hand sides. (Figure 1B)

## Real-world applications

The Finite State Projection (FSP) apporximates the probability distribution of stochastic reaction networks by only considering a finite set of states [@MunskyKha2006]. In the [FSP example](https://github.com/yaml2sbml-dev/yaml2sbml/blob/master/doc/examples/Finite_State_Projection/Finite_State_Projection.ipynb) `yaml2sbml`s Model Editor implements the FSP for a two-stage model of gene transmission as discussed e.g. in [@ShahrezaeiSwa2008] (Figure 1C). This example shows how `yaml2sbml` allows to implement a large scale ODE containing hundreds of states in less than 20 lines of code, by exploiting the structure of the problem.

The Sorensen model is a well established model of human Glucose-Insulin metabolism, describing the dynamics of Glucose and Insulin in different compartments by an ODE with 22 states [@Sorensen1985]. The [jupyter notebook](https://github.com/yaml2sbml-dev/yaml2sbml/blob/master/doc/examples/Sorensen/yaml2sbml_Sorensen.ipynb) presents an implementation of the Sorensen model in the YAML input format and use the Model Editor to extend the model to encode an intravenuous glucose injection.  `yaml2sbml` Model Editor extends a preexisting YAML model to encode  patient specific treatment (Figure 1D). 

![**Figure 1** Simulation results for different examples of models specified using `yaml2sbml`. A) Lotka Volterra Equations B) Discontinuous right hand side of the ODE C) Finite State Projection of a two-stage model of gene transmission D) Sorensen Model of human Glucose-Insulin metabolism.](Figure1.png)


# Funding

TBD
M.R.A.M. was funded by the Novo Nordisk Foundation (NNF10CC1016517 and NNF14OC0009473).

# References