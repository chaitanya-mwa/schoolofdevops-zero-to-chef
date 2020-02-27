# Chapter 12: Creating Configuration Profiles with Environments

In this lab we are going to use chef's environment primitive to,
  * Create different configuration profiles. Setup environment specific configurations.
  * Set cookbook versioning constraints. Define cookbook versions to be used in each environment. This way you could limit your experimentation with your own dev env without affecting rest of the environments.
  * Provide isolation in combination with search.  


### Making search more specific with environments

In the previous module, we learnt about search, and even incorporated it to auto discover app servers from load balancer. However with the current configurations, the search results will return list of all app servers, irrespective of the environment. We need to make this search specific to the node's environment.

- Lets update **myhaproxy** recipe accordingly
- Path: _cookbooks/myhaproxy/recipes/default.rb_

Update the line which uses search to,

```ruby
all_web_nodes = search("node", "role:app AND chef_environment:#{node.chef_environment}")
```

Also update the version in the metadata.

Path: _cookbooks/myhaproxy/metadata.rb_

```ruby
version '0.3.0'
```

Upload the cookbook

```console
knife cookbook upload myhaproxy                                                                                            
```

## Creating a prod environment

- Create a environment file
- Path: _sysfoo/environments/prod.rb_

```ruby
name "prod"
description "Production Environment"
cookbook "myhaproxy", "= 0.3.0"
default_attributes({
  "tomcat" => { "deploy"  => {
                 "url" => 'https://11-94848332-gh.circle-artifacts.com/0/tmp/circle-artifacts.6gxaMPh/sysfoo.war'}
              }
})
```

### Uploading and exploring environment

```
knife environment from file prod.rb   

knife environment show prod
```

### Bringing nodes into the environment

From node4 keep watching the haproxy.cfg

```console
watch -n 1 tail /etc/haproxy/haproxy.cfg
```

Lets bring in two nodes into prod env

```console
knife node environment set app1 prod
knife node environment set app2 prod                                                                                         
```

Keep watching the haproxy config. It should take the two nodes off as the load balancer still belongs to `_default env`.

Lets bring load balancer into the prod env

```console
knife node environment set lb prod
```

See as the LB limits itself to serve only to the app servers in its own environment.

You should also see two differnt versions of the applications running in the two environments vz **production** and **_default**.
