# ELK 6

Elasticsearch, Logstash & Kibana 6 deployed as a single node or scalable cluster.

WARN: To avoid vm.max_map_count errors you could set "Update host sysctl" to true. Then param vm.max_map_count will be update to 262144 if it's less in your hosts.


## Deployment Types

The catalog provides three deployment types utilizing the "Licensed Deployment" and "Enable Cluster" deployment flags.

* Non-clustered / Unsecured
* Non-clustered / Secured with RBAC
* Clustered / Unsecured
* Clustered / Secured with RBAC

Role-based Access Control (RBAC) security is enabled by setting the *Licensed Deployment* flag to true.  When enabled, ELK will utilize the the following user accounts:

* *elastic* - Elastic admin
* *kibana* - Kibana admin (monitoring, management)
* *logstash_system* - Logstash user for monitoring and accessing Elastic license
* *logstash_internal* - User Logstash uses to write to Elasticsearch (via output section of logsash config)
* *logstash_user* - Kibana user that can read logstash indices

Passwords for these accounts are set within the catalog.  By default, the password is set to the username (above).

RBAC security is only enabled with a valid Elastic Platinum license.  On first deployment, the flag *Apply license and Set Passwords* should be set to true (default).  When set, ELK will be deployed with  *xpack-configs* service that waits for Elasticsearch to be up and running, and then installs the license and configures the accounts above.  

When deploying ELK to an environment with existing data (i.e. elasticsearch data volume), *Apply License and Set Passwords* flag may be set to false.  However, make sure the passwords in the catalog are set those configured in the existing dataset or the deployment will not run.  You can reset user passwords by setting the flag to trye as long as the *elastic* user credentials are correct.  If you lose the password for the *elastic* user, your data may be lost.  Seek help from Elastic Support for potential workarounds.

Without clustering, ELK is deployed with a single instance of each service.  This deployment may be used for testing.  Production or higher-level environments should only use cluster deployments, as they can survive host failures.  

To enable clustering, simply set the *Enable Cluster* flag to true.  When set, ELK is deployed as follows:

* 10 instances of Elasticsearch are deployed (3 master nodes, 3 data nodes and 4 coordinating nodes)
* 2 Kibana instances load balanced
* 2 Logstash instances load balanced

In clustered mode, communication between Elasticsearch nodes are encrypted using generic self-signed certificates and moderate certifcation authentication.  This is requried by Elastic in order to utilize their platinum licenses.  

### SSL & TSL

Setting up SSL/TSL across all points of the ELK stack for our Rancher environment where services can be scaled at will is complex endeavor that involves self-signed certificates, container scripts and a coordinated auto restart workflow.  Due to this complexity and fact that ELk will be deployed in a locked-down Rancher environment, we felt that SSL/TSL was overkill and decided to not enable it.

The one area where SSL/TSL is probably warranted is at Kibana level as Kibana is exposed to CityNet.  This will most likely be added in the near future along with LDAP authentication.

## Notes

* Elasticsearch data is stored in docker volumes.  Default setting is local driver.  Local driver persists volumes to default docker volume directory (e.g. /var/lib/docker/volume).
* Cluster components are deployed to hosts with the following key:value labels:
  * elk:data - Elasticsearch master and data nodes
  * elk:other - All other nodes (master, client, kibana)
  * elk:public-lb - Load balancers exposed to external users (Kibana and Logstash clients)
* Anti-affinity scheduling rules keep like nodes off of same host
* Elastic data nodes and logstash nodes are the most resource intensive.  They should be deployed on hosts with the most RAM and CPU.
* Elastic master nodes are light-weight and should be able to run using minimal memory
