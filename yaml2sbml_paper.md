# #Yaml2sbml ...

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
    
 - name: Lyman Spitzer, Jr. Fellow, Princeton University
   index: 1
   
 - name: Institution Name
   index: 2
   
 - name: Independent Researcher
   index: 3
date: 31 January 2021
bibliography: paper.bib
---

# Summary



# Statement of need


Ordinary Differential Equations (ODEs) are a vital tool in mechanisitic modelling of biological processes. ODE models span a wide range of scales and applications, from molecular to population level and arise e.g. as discretization of Partial Differential Equation models or moments of stochastic systems.

The Systems Biological Markup Language (SBML) is a widely adopted community standard for specifying mechanistic models of biological systems. SBML comes with a large number of software tools supporting ODE model simulation (TODO, cite COPASI, AMICI, d2d, dmod, the [biomodels.net](biomodels.net) data base contains more than 1000 published models in the SBML format.

Model parameters can be estimated from data via a maximum likelihood or maximum a posteriori estimator. Therefore the systems states need to be mapped to measured quantities by an observable function and a measurement noise model needs to be specified. The PEtab format was recently introduced to extend an SBML model by specifying observables, measurements, experimental conditions and parameters to be estimated in separate tab-separated value (TSV) files. [CITE TODO]. Currently 9 software toolboxes support PEtab as an input format.

Despite its broad usage, constructing an SBML model from scratch is often tedious: Due to its broad scope, SBML models have a lot of required and optional keys, that complicate the definition of a valid SBML. Further SBML can be considered neither human readable nor human writeable.

Therefore various approaches have been presented to facilitate model construction from text based input formats or in code [Cite libsbml, simleSBML, MOCASIN, Antimony, ScrumPy, ...]. However, these tools have a different focus to our approach, as they have an text based input format tightly tied to a specific programming language as MOCCASIN (translating MATLAB code into SBML), others have a text based input format, that is centered around chemical reactions and not around ODEs directly (Antimony, ScrumPy), or only offer a python based way of defining SBMLs, instead of a text-based way (simpleSBML). Neither of the aforementioned tools support PEtab or other formats for specifying parameter estimation problems.

Here we present a human readable and writeable format for ODE models based on yaml, that can be validated and translated into SBML and PEtab via the Python tool `yaml2sbml` or a command line interface (CLI). Furthermore `yaml2sbml` comes with a Python-based Model Editor, that allows to generate, import and export a YAML model within code and even completely circumvent the yaml input if desired.


# Tool and Format

## YAML Format

The yaml-based input format is organized in blocks for the different model components. The format allows for the following blocks:

* `odes:` Dene states, right hand side and initial values (as numeric values or parameters) as
* `parameters:` Dene parameters, allows to initialize them to values. Further keys allow to specify e.g. optimization bounds. These values are written in the PEtab parameter table.
* `time:` Allows to specify the time variable of the ODE.
* `assignments:` Assignments assign a variable dynamically to a value. They are incoded as assignment rules in the SBML model.
* `functions:` Allows to define functions, that can be used in other parts of the model.
* `observables:` Observables are only used, when generating a PEtab problem for generating an observable table. Do not have an eect on the resulting SBML.
* `conditions:` Conditions are only used, when generating a PEtab problem for generating an condition table. Do not have an eect on the resulting SBML.

## Python Tool for SBML/PEtab translation 

The python tool `yaml2sbml` allows to translate models, that are specified in the YAML input format to SBML and Petab via

```python
import yaml2sbml

# SBML translation
yaml2sbml.yaml2sbml(yaml_input_file, sbml_output_file)
# PEtab conversion
yaml2sbml.yaml2petab(yaml_input_file, PEtab_dir, model_name)
```

## Format Validation

Furthere `yaml2sbml` offers functionality to validate an input YAML via

```python
yaml2sbml.validate_yaml(yaml_file)
```

The validation is also performed before translation to SBML and PEtab within the translation routines. Further `libsbml`s SBML validator and PEtabs linter validate the files after tranlsation.

## Command Line Interface

The Command Line Interface offers SBML/PEtab conversion as well as format validation via the commands `yaml2sbml`, `yaml2petab` and `yaml2sbml_validate` respectively 

## Model Editor

The `yaml2sbml` Model Editor allows to construct YAML models in python e.g. via

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

The repository contains an extensive [format documentation](https://github.com/yaml2sbml-dev/yaml2sbml/blob/master/doc/format_documentation.md) and jupyter notebooks containing [various examples](https://github.com/yaml2sbml-dev/yaml2sbml/tree/master/doc/examples) covering all aspects of the project, on a range from didactic to real world applications. Code testing and continuous is performed via GitHub actions.

# Examples

`yaml2sbml`'s GitHub repository contains several jupyter notebooks for examples on a range of complexity. 

## Introductory Examples

Three notebooks use the Lotka-Volterra Equations as a small and well established example ODE to showcase the different functionalities of `yaml2sbml`: The format and the basic functionality of the python toolbox (Link: TODO), the CLI(Link: TODO) and the Model Editor(Link: TODO) (Figure 1A). The format features notebook (Link: TODO) showcases features of the format as time dependent or discontinuous right hand sides. (Figure 1B)

## Real-world applications

The Finite State Projection(FSP) apporximates the probability distribution of stochastic reaction networks by only considering a finite set of states. In the [FSP example](Link_to_example:TODO) `yaml2sbml`s Model Editor implements the FSP for a two-stage model of gene transmission as discussed e.g. in (Cite PNAS) (Figure 1C). This example shows how a large scale ODE containing hundreds of states can be implemented in less than 20 lines of code, by exploiting the structure of the problem.

The Sorensen model is a well established model of human Glucose-Insulin metabolism, describing the dynamics of Glucose and Insulin in different compartments of the body via a 22-dimensional ODE. The jupyter notebook presents an implementation of the Sorensen model in the YAML input format and use the Model Editor to extend the model to encode an intravenuous glucose injection. This shows how `yaml2sbml` can easily extend a preexisting ODE model to encode e.g. patient specific treatment. (Figure 1D) 

# Funding

