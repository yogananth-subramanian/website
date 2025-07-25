---
title: Adding scenarios via plugin api
description:
weight: 3
categories: [New scenarios, Placeholders]
tags: [docs]
---

# Scenario Plugin API:

This API enables seamless integration of Scenario Plugins for Krkn. Plugins are automatically 
detected and loaded by the plugin loader, provided they extend the `AbstractPluginScenario` 
abstract class, implement the required methods, and adhere to the specified [naming conventions](#naming-conventions).

## Plugin folder:

The plugin loader automatically loads plugins found in the `krkn/scenario_plugins` directory, 
relative to the Krkn root folder. Each plugin must reside in its own directory and can consist 
of one or more Python files. The entry point for each plugin is a Python class that extends the 
[AbstractPluginScenario](https://github.com/krkn-chaos/krkn/blob/main/krkn/scenario_plugins/abstract_scenario_plugin.py) abstract class and implements its required methods.

## `__init__` file
For the plugin to be properly found by the plugin api, there needs to be a init file in the base folder 

For example: [__init__.py](https://github.com/krkn-chaos/krkn/blob/main/krkn/tests/__init__.py)

## `AbstractPluginScenario` abstract class:

This [abstract class](https://github.com/krkn-chaos/krkn/blob/main/krkn/scenario_plugins/abstract_scenario_plugin.py) defines the contract between the plugin and krkn.
It consists of two methods:
- `run(...)`
- `get_scenario_type()`

Most IDEs can automatically suggest and implement the abstract methods defined in `AbstractPluginScenario`:
![pycharm](scenario_plugin_pycharm.gif)
_(IntelliJ PyCharm)_

### `run(...)`

```python
    def run(
        self,
        run_uuid: str,
        scenario: str,
        krkn_config: dict[str, any],
        lib_telemetry: KrknTelemetryOpenshift,
        scenario_telemetry: ScenarioTelemetry,
    ) -> int:

```

This method represents the entry point of the plugin and the first method 
that will be executed.
#### Parameters:

- `run_uuid`:
  - the uuid of the chaos run generated by krkn for every single run.
- `scenario`:
  - the config file of the scenario that is currently executed
- `krkn_config`:
  - the full dictionary representation of the `config.yaml`
- `lib_telemetry`
  - it is a composite object of all the [krkn-lib](https://krkn-chaos.github.io/krkn-lib-docs/modules.html) objects and methods needed by a krkn plugin to run.
- `scenario_telemetry`
  - the `ScenarioTelemetry` object of the scenario that is currently executed
 
### Return value:
Returns 0 if the scenario succeeds and 1 if it fails.
{{< notice type="warning" >}} All the exception must be handled __inside__ the run method and not propagated. {{< /notice >}}

### `get_scenario_types()`:

```python    def get_scenario_types(self) -> list[str]:```

Indicates the scenario types specified in the `config.yaml`. For the plugin to be properly
loaded, recognized and executed, it must be implemented and must return one or more
strings matching `scenario_type` strings set in the config.

{{< notice type="danger" >}}Multiple strings can map to a *single*  `ScenarioPlugin` but the same string cannot map to different plugins, an exception will be thrown for scenario_type redefinition. {{< /notice >}}

{{< notice type="info" >}}The `scenario_type` strings must be unique across all plugins; otherwise, an exception will be thrown. {{< /notice >}}

## Naming conventions:
A key requirement for developing a plugin that will be properly loaded 
by the plugin loader is following the established naming conventions. 
These conventions are enforced to maintain a uniform and readable codebase, 
making it easier to onboard new developers from the community.

### plugin folder:
- the plugin folder must be placed in the `krkn/scenario_plugin` folder starting from the krkn root folder
- the plugin folder __cannot__ contain the words
  - `plugin`
  - `scenario`
### plugin file name and class name:
- the plugin file containing the main plugin class must be named in _snake case_ and must have the suffix `_scenario_plugin`: 
  - `example_scenario_plugin.py`
- the main plugin class must named in _capital camel case_ and must have the suffix `ScenarioPlugin` : 
  - `ExampleScenarioPlugin`
- the file name must match the class name in the respective syntax:
  - `example_scenario_plugin.py` -> `ExampleScenarioPlugin`

### scenario type:
- the scenario type __must__ be unique between all the scenarios.

### logging:
If your new scenario does not adhere to the naming conventions, an error log will be generated in the Krkn standard output,
providing details about the issue:

```commandline
2024-10-03 18:06:31,136 [INFO] 📣 `ScenarioPluginFactory`: types from config.yaml mapped to respective classes for execution:
2024-10-03 18:06:31,136 [INFO]   ✅ type: application_outages_scenarios ➡️ `ApplicationOutageScenarioPlugin` 
2024-10-03 18:06:31,136 [INFO]   ✅ types: [hog_scenarios, arcaflow_scenario] ➡️ `ArcaflowScenarioPlugin` 
2024-10-03 18:06:31,136 [INFO]   ✅ type: container_scenarios ➡️ `ContainerScenarioPlugin` 
2024-10-03 18:06:31,136 [INFO]   ✅ type: managedcluster_scenarios ➡️ `ManagedClusterScenarioPlugin` 
2024-10-03 18:06:31,137 [INFO]   ✅ types: [pod_disruption_scenarios, pod_network_scenario, vmware_node_scenarios, ibmcloud_node_scenarios] ➡️ `NativeScenarioPlugin` 
2024-10-03 18:06:31,137 [INFO]   ✅ type: network_chaos_scenarios ➡️ `NetworkChaosScenarioPlugin` 
2024-10-03 18:06:31,137 [INFO]   ✅ type: node_scenarios ➡️ `NodeActionsScenarioPlugin` 
2024-10-03 18:06:31,137 [INFO]   ✅ type: pvc_scenarios ➡️ `PvcScenarioPlugin` 
2024-10-03 18:06:31,137 [INFO]   ✅ type: service_disruption_scenarios ➡️ `ServiceDisruptionScenarioPlugin` 
2024-10-03 18:06:31,137 [INFO]   ✅ type: service_hijacking_scenarios ➡️ `ServiceHijackingScenarioPlugin` 
2024-10-03 18:06:31,137 [INFO]   ✅ type: cluster_shut_down_scenarios ➡️ `ShutDownScenarioPlugin` 
2024-10-03 18:06:31,137 [INFO]   ✅ type: syn_flood_scenarios ➡️ `SynFloodScenarioPlugin` 
2024-10-03 18:06:31,137 [INFO]   ✅ type: time_scenarios ➡️ `TimeActionsScenarioPlugin` 
2024-10-03 18:06:31,137 [INFO]   ✅ type: zone_outages_scenarios ➡️ `ZoneOutageScenarioPlugin`

2024-09-18 14:48:41,735 [INFO] Failed to load Scenario Plugins:

2024-09-18 14:48:41,735 [ERROR] ⛔ Class: ExamplePluginScenario Module: krkn.scenario_plugins.example.example_scenario_plugin
2024-09-18 14:48:41,735 [ERROR] ⚠️ scenario plugin class name must start with a capital letter, end with `ScenarioPlugin`, and cannot be just `ScenarioPlugin`.
```

{{< notice type="info" >}}If you're trying to understand how the scenario types in the config.yaml are mapped to their corresponding plugins, this log will guide you! Each scenario plugin class mentioned can be found in the `krkn/scenario_plugin` folder simply convert the camel case notation and remove the ScenarioPlugin suffix from the class name e.g `ShutDownScenarioPlugin` class can be found in the `krkn/scenario_plugin/shut_down` folder.{{< /notice >}}

## ExampleScenarioPlugin
The [ExampleScenarioPlugin](https://github.com/krkn-chaos/krkn/blob/main/krkn/tests/test_classes/example_scenario_plugin.py) class included in the tests folder can be used as a scaffolding for new plugins and it is considered
part of the documentation.

For any questions or further guidance, feel free to reach out to us on the 
[Kubernetes workspace](https://kubernetes.slack.com/) in the `#krkn` channel. 
We’re happy to assist. Now, __release the Krkn!__
