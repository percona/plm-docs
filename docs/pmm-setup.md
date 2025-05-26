# Set up observability with Percona Monitoring and Management

{{pml.full_name}} is a tool that enables seamless data replication between MongoDB clusters. PMM provides visibility into key performance indicators including the number of processed events, data transfer sizes, document counts, and batch processing times. This comprehensive monitoring helps you optimize replication performance and quickly identify any potential issues during the replication process.

PMM is the server-client solution. The PMM Client collects the metrics and sends them to the PMM Server that displays them on dashboards in a user-friendly way.

PMM Server and PMM Client are installed separately. 

## Prerequisites

1. You must have PMM server up and running. You can run PMM Server as a Docker image, install it a virtual appliance, or on an AWS instance. Follow the [Quickstart quide](https://docs.percona.com/percona-monitoring-and-management/3/quickstart/quickstart.html) to start PMM Server in a Docker container. For other installation options, see [PMM Documentation](https://docs.percona.com/percona-monitoring-and-management/3/install-pmm/install-pmm-server/index.html)
2. Ensure PMM server and PML can communicate with each other over the network.

## Install PMM Client

You must install PMM Client on the same instance where {{pml.full_name}} is running. 

Configure monitoring for PML. Add PML as an external service to PMM server:

```{.bash data-prompt="$"}
$ pmm-admin add external --service-name=PML_test --listen-port=2242 --metrics-path=metrics --scheme=http
```

??? example "Expected output"
 
    ```{.text .no-copy}
    External Service added.
    Service ID  : 0b3460d9-4173-4ff8-adcd-105883a4ef56
    Service name: PML_test
    Group       : external
    ```

Now PMM Client collects the metrics for PML.

## Create a dashboard 

To view PML metrics, configure a dashboard in PMM Server. Here's how:

1. Log in to PMM Server
2. From the main menu select **Dashboards** and then **New**
3. Select **Import a dashboard**
4. 

