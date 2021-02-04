# # yaml2sbml: Human readable and writable specification of ODE models in SBML 

---
title: 'TODO'
tags:

  - Python

  - SBML

  - Ordinary differential Equation

  - YAML

  authors:

  - name: Jakob Vanhoefer
    orcid: TODO
    affiliation: TODO "1, 2" # (Multiple affiliations must be quoted)
    
  - name: Marta R. Matos
    orcid: TODO
    affiliation: TODO "1, 2" # (Multiple affiliations must be quoted)
    
  - name: Dilan Pathirana
    orcid: TODO
    affiliation: TODO "1, 2" # (Multiple affiliations must be quoted)
    
  - name: Yannik Schälte
    orcid: TODO
    affiliation: TODO "1, 2" # (Multiple affiliations must be quoted)
    
  - name: Jan Hasenauer
    orcid: TODO
    affiliation: TODO "1, 2" # (Multiple affiliations must be quoted)
affiliations:
    date: 31 January 2021
    bibliography: paper.bib
    
---

# Summary

Ordinary Differential Equations are a common model type throughout all natural sciences. The Systems Biological Markup Language (SBML) on the other hand is a widely adopted community standard for specifying dynamic models in the field of systems biology, that was recently extended to the PEtab format for describing parameter estimation problems. There exist a large number of software pipelines for simulating SBML models and performing parameter estimation for PEtab problems. Specifying an ODE model in SBML would grant direct access to these software pipelines. Since SBML is considered neither human-readable nor human-writable, an easy to use approach to construct SBML/PEtab models for a given ODE will facilitate model generation. In this contribution we present `yaml2sbml`,  a python tool for converting ODE models specified in an easy to read and write YAML file into SBML/PEtab. Further `yaml2sbml` comes with a validator for the input YAML, a Command Line Interface and a Model Editor to build an input YAML within python code. Several examples highlight the usage of `yaml2sbml` on realistic problems. 

# Statement of need


Ordinary Differential Equations (ODEs) are a vital tool in mechanisitic modelling of biological processes. ODE models span a wide range of scales and applications, from molecular to population level and arise e.g. as discretization of Partial Differential Equation models or moments of stochastic systems.

The Systems Biological Markup Language (SBML) is a widely adopted community standard for specifying mechanistic models of biological systems. SBML comes with a large number of software tools supporting ODE simulation (TODO, cite COPASI, AMICI, d2d, dmod, the [biomodels.net](biomodels.net) data base contains more than 1000 published models in the SBML format.

Model parameters can be estimated from data via a maximum likelihood or maximum a posteriori estimator. Therefore the systems states need to be mapped to measured quantities by an observable function and a measurement noise model needs to be specified. The PEtab format was recently introduced to extend an SBML model by specifying observables, measurements, experimental conditions and parameters to be estimated in tab-separated value files. [CITE TODO]. Currently 9 software toolboxes support PEtab as an input format.

Despite its broad usage, constructing an SBML model from scratch is often tedious and SBML can be considered neither human-readable nor human-writeable. Therefore various approaches to facilitate model construction from text based input formats or in code have been presented  [Cite libsbml, simleSBML, MOCASIN, Antimony, ScrumPy, ...]. However, these tools have a different focuses to our approach, as they have e.g. an text based input format tightly tied to a specific programming language (e.g. MOCASIN translating MATLAB code into SBML), others have a text based input format, that is centered around chemical reactions and not around ODEs directly (Antimony, ScrumPy), or only offer a Python based way of defining SBMLs, instead of a text-based way (simpleSBML). Neither of the aforementioned tools support PEtab or other formats for specifying parameter estimation problems.

Here we present a human readable and writeable format for ODE models based on YAML, that can be validated and translated into SBML and PEtab via the Python tool `yaml2sbml` or a command line interface (CLI). Furthermore `yaml2sbml` comes with a Python-based Model Editor, that allows to generate, import and export a YAML model within code and even completely circumvent the YAML input if desired.


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

Internally libsbml (TODO: Cite)  generates the resulting SBML.

## Format Validation

`yaml2sbml`  validates an input YAML  via

```python
yaml2sbml.validate_yaml(yaml_file)
```

The validation is also performed before translation to SBML and PEtab within the translation routines. Further `libsbml`s SBML validator and PEtabs linter check the SBML/PEtab files after tranlsation.

## Command Line Interface

`yaml2sbml` comes with a CLI, that offers SBML/PEtab conversion as well as format validation via the commands `yaml2sbml`, `yaml2petab` and `yaml2sbml_validate` respectively .

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

The Finite State Projection (FSP) apporximates the probability distribution of stochastic reaction networks by only considering a finite set of states. In the [FSP example](https://github.com/yaml2sbml-dev/yaml2sbml/blob/master/doc/examples/Finite_State_Projection/Finite_State_Projection.ipynb) `yaml2sbml`s Model Editor implements the FSP for a two-stage model of gene transmission as discussed e.g. in (Cite PNAS) (Figure 1C). This example shows how a large scale ODE containing hundreds of states can be implemented in less than 20 lines of code, by exploiting the structure of the problem.

The Sorensen model is a well established model of human Glucose-Insulin metabolism, describing the dynamics of Glucose and Insulin in different compartments by an ODE with 22 states. The [jupyter notebook](https://github.com/yaml2sbml-dev/yaml2sbml/blob/master/doc/examples/Sorensen/yaml2sbml_Sorensen.ipynb) presents an implementation of the Sorensen model in the YAML input format and use the Model Editor to extend the model to encode an intravenuous glucose injection.  `yaml2sbml` Model Editor extends a preexisting YAML model to encode  patient specific treatment (Figure 1D). 

![**Figure 1** Simulation results for different examples of models specified using `yaml2sbml`. A) [Lotka Volterra Equations](Link TODO) B) Discontinuous right hand side of the ODE C) Finite State Projection of a two-stage model of gene transmission D) Sorensen Model of human Glucose-Insulin metabolism.](/Users/jvanhoefer/Dokumente/PhD/Paper writing/yaml2sbml/yaml2sbml_MD/paper/Figure1.png)

# Funding

TBD

# References



