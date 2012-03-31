graphsom
===============

graphsom dumps provides visualizations for metrics collected using
[folsom](https://github.com/boundary/folsom). Currently graphsom only
supports [Graphite](http://graphite.wikidot.com/) visualizations.
Support for multiple backends will be added soon. 

Note: [Graphite](http://graphite.wikidot.com/) is good, despite the
website being really ghetto. 

Status
------

* Supports all [folsom](https://github.com/boundary/folsom) metric
  types: `gauge`, `counter`, `meter`, `histogram` and `history`
* Supports all [folsom](https://github.com/boundary/folsom) VM metrics
  including `memory`,`statistics`, `process_info`, `system_info` and `port_info`
* Support for other visualization backends coming soon

Getting Started
--------------

Prerequisites 

* [Download](https://launchpad.net/graphite/+download),
  [Install](http://graphite.wikidot.com/installation) and run graphite
* [Download](https://github.com/downloads/basho/rebar/rebar) or [build](https://github.com/basho/rebar) rebar
* Clone [graphsom](https://github.com/techtraits/graphsom.git)

Setup [rebar](https://github.com/basho/rebar)

    cd graphsom
	chmod u+x rebar  
    
or make [rebar](https://github.com/basho/rebar) executable from source  

Get dependencies & compile 

    mkdir deps
    ./rebar get-deps
    ./rebar compile

Create a release

    mkdir rel
    cd rel
    ../rebar create-node nodeid=graphsom
    
At this point, the release folder `rel` should contain `reltool.config` (release
tool configuration file). We need to tell `reltool` where to
find `graphsom` and `folsom` applications. This can be done by
specifying `lib_dir` in `reltool.config` as follows:

    {lib_dirs, ["../../", "../deps/"]},
    
Since by default applications are not automatically added to the
release, we can add folsom in our release by adding the following line
in `reltool.config`:

    {app, stdlib, [{incl_cond, include}]},
    {app, kernel, [{incl_cond, include}]},
    {app, folsom, [{incl_cond, include}]},
    **{app, folsom, [{incl_cond, include}]},**
    {app, graphsom, [{incl_cond, include}]}
    
Generating a release (a self-containing deployable directory
containing all the applications in your node):

    ./rebar generate

Running graphsom node:

    ./rel/graphsom/bin/graphsom console

Alternatively, Makefile is provided for convenience. 

Note: Graphsom takes care of starting `folsom`.

**Configuration Parameters**

The configuration parameters for graphsom can be added to `rel/files/sys.config`. An example configuration:
    
    {graphsom, [
             {report_interval, 30000},     %% report interval (ms)
             {graphite_host, "techtraits.com"}, %% defaults to localhost
             {graphite_port, 2003},        %% defaults to 2003
             {graphite_prefix, "graphsom"}, %% prefix added to all metrics
             {vm_metrics, [memory, statistics]},   %% defaults to []
             {report_all_folsom_metrics, false}   %% defaults to false
            ]}
           
* `graphite_prefix` can be used to specify the prefix prepended to all
metric names. For example, "response_time" would be reported as
"graphsom.response_time". 

* `vm_metrics` can be used to report folsom VM metrics

* If `report_all_folsom_metrics` is set to `true`, graphsom reports all folsom
  metrics that exist at the time of reporting. Alternatively, graphsom
  offers a `register_folsom_metric` API for registering metrics that are to be reported to graphite.

**Usage**

Add some folsom metrics ([folsom API](https://github.com/boundary/folsom)):

    folsom_metrics:new_counter(metric_name_1).
    folsom_metrics:new_meter(metric_name_2).

Update values for the metrics ([folsom API](https://github.com/boundary/folsom)):

	folsom_metrics:notify({metric_name_1, {inc, 100}}).
    folsom_metrics:notify({metric_name_2, 300}).

Register folsom metric with `graphsom` for reporting:

    graphsom:register_folsom_metric(metric_name_1).

De-register folsom metric:

    graphsom:deregister_folsom_metric(metric_name_1).

Changing reporting parameters at runtime:

    graphsom:update_config(report_interval, 60000)

Tell graphsom to start reporting:

    graphsom:start_reporting().

To get the list of currently registered folsom metrics

    graphsom:registered_metrics().
    
**Check graphite for the values!!**

Authors 
------

Bilal Sheikh (<bilal@techtraits.com>)  
Usman Ismail (<usman@techtraits.com>)
