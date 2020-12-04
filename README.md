Carvel customizations for TKG extensions.

The code in this repository demonstrates how to add a syslog output to the tkg-extensions logging module for TKG 1.2.

NOTE: This method is NOT supported by VMware GSS and you use this at your own risk.

To use it:
  1. Build a docker image.
  
    ```sh
    docker build -t yourrepo/tkg-extensions-1.2-fluentbit-syslog:v0.1 .
    ```
  
  2. Tag and push to a container registry of your choice(make sure this registry is accessible to your target cluster).
  
    ```sh
    docker tag yourrepo/tkg-extensions-1.2-fluentbit-syslog:v0.1 yourregistry/yourrepo/tkg-extensions-1.2-fluentbit-syslog:v0.1
    docker push yourregistry/yourrepo/tkg-extensions-1.2-fluentbit-syslog:v0.1
    ```
  
  3. Modify the `extensions/logging/fluent-bit/fluent-bit-extension.yaml` file in the tkg-extensions tarball.
  
    ```sh
    sed -i '.bak' -e 's|registry.tkg.vmware.run/tkg-extensions-templates:v1.2.0_vmware.1|yourregistry/yourrepo/tkg-extensions-1.2-fluentbit-syslog:v0.1|' fluent-bit-extension.yaml
    ```
  
  4. Install TMC's extension manager

    ```sh
    kubectl apply -f ../../tmc-extension-manager.yaml
    ```

  5. Install kapp-controller

    ```sh
    kubectl apply -f ../../kapp-controller.yaml
    ```

  6. Create fluent-bit namespace

    ```sh
    kubectl apply -f namespace-role.yaml
    ```
    
  7. Create fluent-bit data values for syslog
  
    ```sh
    mkdir syslog
    cat < EOF > syslog/fluent-bit-data-values.yaml
    #@data/values
    #@overlay/match-child-defaults missing_ok=True
    ---
    logging:
      image:
        repository: registry.tkg.vmware.run
    tkg:
      instance_name: "<your value here>"
      cluster_name: "<your value here>"
    fluent_bit:
      output_plugin: "syslog"
      syslog:
        host: "<destination syslog server ip or fqdn>"
        port: "<destination syslog port"
        mode: "<tcp or udp>"
        format: "rfc5424"
        maxsize: "2048"
        hostname_key: "namespace_name"
        appname_key:  "pod_name"
        procid_key:   "container_name"
        message_key:  "message"
        sd_key:       "dockerid=docker_id"
    EOF
    kubectl create secret generic fluent-bit-data-values --from-file=values.yaml=syslog/fluent-bit-data-values.yaml -n tanzu-system-logging
    ```
    
  8. Deploy fluent-bit extension

    ```sh
    kubectl apply -f fluent-bit-extension.yaml
    ```

  9. Retrieve status of an extension

      ```sh
      kubectl get extension fluent-bit -n tanzu-system-logging
      kubectl get app fluent-bit -n tanzu-system-logging
      ```

     Fluent Bit app status should change to `Reconcile Succeeded` once fluent-bit is deployed successfully

     View detailed status

     ```sh
     kubectl get app fluent-bit -n tanzu-system-logging -o yaml
     ```
