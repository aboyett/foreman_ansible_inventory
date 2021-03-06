foreman_ansible_inventory
=========================

This script can be used as an ansible dynamic inventory[1].
The connection parameters are set up via a configuration
file *foreman.ini* that resides in the same directory as the inventory
script.

## Variables and Parameters

The data returned from Foreman for each host is stored in a foreman
hash so they're available as *host_vars* along with the parameters
of the host and it's hostgroups:

     "foo.example.com": {
        "foreman": {
          "architecture_id": 1,
          "architecture_name": "x86_64",
          "build": false,
          "build_status": 0,
          "build_status_label": "Installed",
          "capabilities": [
            "build",
            "image"
          ],
          "compute_profile_id": 4,
          "hostgroup_name": "webtier/myapp",
          "id": 70,
          "image_name": "debian8.1",
          ...
          "uuid": "50197c10-5ebb-b5cf-b384-a1e203e19e77"
        },
        "foreman_params": {
          "testparam1": "foobar",
          "testparam2": "small",
          ...
        }

and could therefore be used in ansible like:

    - debug: msg="From Foreman host {{ foreman['uuid'] }}"

Which yields

    TASK [test_foreman : debug] ****************************************************
    ok: [foo.example.com] => {
    "msg": "From Foreman host 50190bd1-052a-a34a-3c9c-df37a39550bf"
    }


## Automatic ansible groups

The hostgroup, location and organization of each host is created as
ansible group with a foreman_<grouptype> prefix, all lowercase and
problematic parameters removed. So e.g. the foreman hostgroup

    myapp / webtier / datacenter1

would turn into the ansible group:

    foreman_hostgroup_myapp_webtier_datacenter1

Furthermore ansible groups can be created on the fly using the
*group_patterns* variable in *foreman.ini* so that you can build up
hierarchies using parameters on the hostgroup and host variables.

Lets assume you have a host that is built using this nested hostgroup:

    myapp / webtier / datacenter1

and each of the hostgroups defines a parameters respectively:

    myapp: app_param = myapp
    webtier: tier_param = webtier
    datacenter1: dc_param = datacenter1

The host is also in a subnet called "mysubnet" and provisioned via an image
then *group_patterns* like:

    [ansible]
    group_patterns = ["{app_param}-{tier_param}-{dc_param}",
                      "{app_param}-{tier_param}",
                      "{app_param}",
                      "{subnet_name}-{provision_method}"]

would put the host into the additional ansible groups:

    - myapp-webtier-datacenter1
    - myapp-webtier
    - myapp
    - mysubnet-image

by recursively resolving the hostgroups, getting the parameter keys
and values and doing a Python *string.format()* like replacement on
it.

[1]: http://docs.ansible.com/intro_dynamic_inventory.html
