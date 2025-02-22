# Module 8: Using compliance reports

**Goal:** Use global reports to satisfy compliance requirementes.

## Steps

1. Use `Compliance Reports` view to see all generated reports.

    >We have deployed a few compliance reports in one of the first labs and by this time a few reports should have been already generated. 
    ```bash
    kubectl get globalreport                                                          
    NAME                         CREATED AT
    cluster-inventory            2021-09-30T23:58:05Z
    cluster-policy-audit         2021-09-30T23:58:05Z
    daily-cis-results            2021-09-30T23:58:04Z
    cluster-network-access   2021-09-30T23:58:05Z
    ```

    >If you don't see any reports, you can manually kick off report generation task. Follow the steps below if you need to do so.

    Calico provides `GlobalReport` resource to offer [Compliance reports](https://docs.tigera.io/v3.9/compliance/overview) capability. There are several types of reports that you can configure:

    - CIS benchmarks
    - Inventory
    - Network access
    - Policy audit

    <br>

    A compliance report could be configured to include only specific endpoints leveraging endpoint labels and selectors. Each report has the `schedule` field that determines how often the report is going to be generated and sets the timeframe for the data to be included into the report.

    Compliance reports organize data in a CSV format which can be downloaded and moved to a long term data storage to meet compliance requirements.
    
    <br>

    ![compliance report](../img/compliance-report.png)

2. Generate a reports at any time to specify a different start/end time.
   
   a. Review and apply the yaml file for the managed cluster.

    Instructions below for a Managed cluster only. Follow [configuration documentation](https://docs.tigera.io/compliance/overview#run-reports) to configure compliance jobs for management and standalone clusters.

   ```bash
   vi demo/40-compliance-reports/compliance-reporter-pod.yaml
   ```

   b. We need to substitute the Cluster Name in the YAML file with the variable `CALICOCLUSTERNAME` we configured in Module 1. This enables compliance jobs to target the correct index in Elastic Search
	```bash
	sed -i "s/\$CALICOCLUSTERNAME/${CALICOCLUSTERNAME}/g" ./demo/40-compliance-reports/compliance-reporter-pod.yaml
	```
	For other variations/shells the following syntax may be required

	```bash
	sed -i "" "s/\$CALICOCLUSTERNAME/${CALICOCLUSTERNAME}/g" ./demo/40-compliance-reports/compliance-reporter-pod.yaml
	```

    c. Now apply the compliance job YAML
	```bash
	kubectl apply -f demo/40-compliance-reports/compliance-reporter-pod.yaml
	```

    Once the `run-reporter` job finished, you should be able to see this report in manager UI and download the csv file. 

3. Reports are generated 30 minutes after the end of the report as [documented](https://docs.tigera.io/v3.9/compliance/overview#change-the-default-report-generation-time). As the compliance reports deployed in the [manifests](https://github.com/tigera-solutions/calicocloud-aks-workshop/tree/main/demo/40-compliance-reports) are scheduled to run every 10 minutes the generation of reports will take between 30-40 mins depending when the manifests were deloyed.
<br>

<br>



[Next -> Module 9](../modules/using-alerts.md)