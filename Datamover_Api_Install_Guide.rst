**Install steps for Trilio Datamover Api (DmApi)**

This triliovault components need to be installed on all controller nodes. Here on-words we will be reffering this component as 'DmApi'.
Perform all steps starting from step-2 on all of your controller nodes of target OpenStack.

**Note**: *Perform following steps on all controller nodes(Starting from step-2).*

**1. Pre-requisites**

  i)You should have launched at-least one TrilioVault VM and this VM should have l3 connectivity with
  OpenStack compute, controller and horizon nodes.
  Get IP address of TrilioVault VM. For example, we assume it's 192.168.14.56. 

**2. Setup Trilio repository**

Clone the repository on controller node:

    git clone https://github.com/trilioData/triliovault-cfg-scripts.git
   
    cd triliovault-cfg-scripts/
   
  *If platform is RHEL/CentOs*
  Create /etc/yum.repos.d/trilio.repo file with following content.
  Make sure, you replace "192.168.14.56" with actual TrilioVault VM IP(if you have deployed 3 node tvault cluster, provide virtual ip) from your enviornment
  
      cp ansible/roles/ansible-datamover-api/templates/trilio.repo /etc/yum.repos.d/trilio.repo

  *If platform is Ubuntu*
  
      cp ansible/roles/ansible-datamover-api/templates/trilio.list /etc/apt/sources.list/trilio.list

**3. Install Trilio Datamover Api package**

   *If platform is RHEL/CentOS*
   
      yum makecache

      yum install dmapi
   
   *If platform is Ubuntu*
   
      apt-get update

      apt-get install dmapi
    
**4. Populate DmApi conf file (/etc/dmapi/dmapi.conf)**
You can either manually edit "/etc/dmapi/dmapi.conf" and fill all the configuration values OR
You can use our command line tool named 'populate-conf', to automatically populate all values.
This tool will be automatically installed with "dmapi" package[step-2].

Steps to use 'populate-conf' command line tool to populate dmapi.conf file:
 i) Create /tmp/datamover_url 
 
          cp ansible/roles/ansible-datamover-api/templates/datamover_url /tmp/datamover_url
    
    Edit this file /tmp/datamover_url and fill controller node fixed ip. This file will be used by populate-conf tool.
    
    **If you are not using ssl or ssl terminated at haproxy in OpenStack datamover_url file will look like following:**
    
      [DEFAULT]
    
      **dmapi_link_prefix = http://<controller_node_ip>:8784**
    
      dmapi_enabled_ssl_apis =
    
      [wsgi]
    
      ssl_cert_file = 
    
      ssl_key_file = 
    
    **In case of ssl enabled on dmapi endpoint, datamover_url file will look like following:**
    
      [DEFAULT]*
    
      **dmapi_link_prefix = https://<controller_node_ip>:13784**
    
      dmapi_enabled_ssl_apis = dmapi
    
      [wsgi]
    
      ssl_cert_file = sample_ssl_cert_file
    
      ssl_key_file = sample_ssl_key_file
      
  ii) Run 'populate-conf' command, it will populate necessary fields in /etc/dmapi/dmapi.conf. You can verify that.
      populate-conf

**5. Create dmapi log directory:**
        mkdir /var/log/dmapi
     
        chown -R nova:nova /var/log/dmapi
    
**6. Create service init file: /etc/systemd/system/tvault-datamover-api.service**

        cp conf-files/tvault-datamover-api.service /etc/systemd/system/   
    
**7. Start dmapi service**

        systemctl daemon-reload
    
        systemctl enable tvault-datamover-api.service
          
        systemctl restart tvault-datamover-api.service
    
**8. Verify Installation**

    i) Verify that dmapi service is started
    
          systemctl status tvault-datamover-api
          
    ii) Verify that no error appears in /var/log/dmapi/dmapi.log file
      
