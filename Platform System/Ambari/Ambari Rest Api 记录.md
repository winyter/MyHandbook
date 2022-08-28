# Ambari Rest Api 记录

- 组件配置信息

  ```
  ambari-url/api/v1/clusters/[clustername]/configurations/service_config_versions?service_name.in(service1,service2,...)&is_current=true&fields=*
  ```

- 获取所有服务列表

  ```
  ambari-url/api/v1/clusters/[clustername]/services/
  ```

- 获取服务基本信息和组件列表

  ```
  ambari-url/api/v1/clusters/[clustername]/services/[server_name]
  ```

- 获取组件基础信息（包括组件 IP 分布）

  ```
  ambari-url/api/v1/clusters/[clustername]/services/[server_name]/components/[component_name]
  ```

- 获取集群节点列表（包括集群名称）

  ```
  ambari-url/api/v1/hosts
  ```

- 获取集群认证信息

  ```
  ambari-url/api/v1/clusters/[clustername]?fields=Clusters/security_type
  ```

  
