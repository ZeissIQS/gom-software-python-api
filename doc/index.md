---
myst:
   html_meta:
      "description": "Introduction to the Python API for extending GOM Inspect 2022 with Packages/Add-ons"
      "keywords": "Metrology, GOM Inspect, Python API, GOM API, Scripting, Packages, Add-ons, How-tos"
--- 
```{important}
You are being redirected to the new location. Please update your bookmarks accordingly.

If you are not redirected automatically, follow this [link](https://zeiss.github.io/gom-software-python-api/2022/).
```

# GOM Inspect Add-On Development Documentation

Welcome to the GOM Inspect Add-On development documentation. With Add-Ons, you will be able to customize and extend the functionality of your GOM Inspect software. 
You can include several template configurations from the software, as well as completely new workflows programmed in python.

```{note}
This documentation is still under development. Expect bookmarks to sub-sites to change.
```

```{important}
Creating add-ons is a rather advanced topic, so you should be familiar with the basic inspection concept of the GOM Software beforehand. New to GOM Inspect? This free course teaches you the basics:

[(eLearning) 780 Starter Training GOM Inspect](https://training.gom.com/home/LearningPath/7265)

Or, depending on your application, you might be interested in the specific starter trainings for [GOM Volume Inspect](https://training.gom.com/home/LearningPath/7280), [GOM Correlate](https://training.gom.com/home/LearningPath/7282), or [GOM Blade Inspect](https://training.gom.com/home/LearningPath/7281).

```

If you are new to add-ons, we recommend following our how-to guides to get you started.

```{eval-rst}
.. toctree::
   :maxdepth: 1
   :caption: How-to Guides

   howtos/environments_for_python_scripts/environments_for_python_scripts
   howtos/python_api_introduction/python_api_introduction
   howtos/python_api_introduction/script_dialogs_introduction
   howtos/python_api_introduction/creating_wizard_dialogs
   howtos/python_api_introduction/using_script_resources
   howtos/scripted_elements/scripted_elements_toc
   howtos/adding_workspaces_to_packages/adding_workspaces_to_packages
   howtos/using_vscode_editor/using_vscode_editor
   howtos/localization/localization
   howtos/testing_addons/testing_addons
```


If you already know how to create an add-on and now you are interested in python programming in GOM Inspect, take a look at our collection of Python examples.

```{eval-rst}
.. toctree::
   :maxdepth: 1
   :caption: Python API Examples
   :titlesonly:

   python_examples/index
   python_examples/data_interfaces
   python_examples/dialog_widgets
   python_examples/misc
   python_examples/script_icons
   python_examples/script_resources
   python_examples/scripted_actuals
   python_examples/scripted_checks
```

Available API functions are documented in the Specification.

```{eval-rst}
.. toctree::
   :maxdepth: 2
   :caption: Python API Specification

   python_api/python_api
   python_api/scripted_elements_api
   python_api/resource_api
```
