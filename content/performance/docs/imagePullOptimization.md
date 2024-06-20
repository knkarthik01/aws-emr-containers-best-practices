
Below is the guide on how you can configure the image pull policy for the Spark and fluentd containers for the job submitter,
driver and executor pods of your EMR on EKS job. The 3 allowed image pull policies are “Always”, “IfNotPresent” and “Never”. If
you do not specify an image pull policy, the default policy of “Always” will be applied to your job. This feature is supported for all
EMR on EKS releases.
CONFIGURING IMAGE PULL POLICY FOR DRIVERS AND EXECUTORS
You can use the Spark configuration spark.kubernetes.container.image.pullPolicy to configure the image pull
policy for both the Spark and Fluentd containers on the driver and executor pods of your job. This configuration can be added in
the job driver or the configuration overrides command argument.
Sample command for using image pull policy with job-driver
cat >spark-job-driver-image-pull-policy.json << EOF
{
"name": "spark-job-driver-image-pull-policy.json",
"virtualClusterId": "<virtual-cluster-id>",
"executionRole
Arn": "<execution-role-arn>",
"releaseLabel": "emr-6.2.0-latest",
"job
Driver": {
"sparkSubmitJob
Driver": {
"entryPoint": "s3://<s3 prefix>/trip-count.py",
"sparkSubmitParameters":
"--conf spark.kubernetes.container.image.pullPolicy=IfNotPresent \
--conf spark.driver.cores=2 --conf spark.executor.memory=2G \
--conf spark.driver.memory=2G --conf spark.executor.cores=2"
}
},
"configurationOverrides": {
"monitoringConfiguration": {
"cloudWatchMonitoringConfiguration": {
"logGroupName": "/emr-containers/jobs",
"logStreamNamePrefix": "demo"
},
"s3MonitoringConfiguration": {
"logUri": "s3://joblogs"
}
}
}
}
EOF
aws emr-containers start-job-run --cli-input-json \
file://spark-job-driver-image-pull-policy.json
Sample command for using image pull policy with configuration-overrides
cat >spark-config-overrides-image-pull-policy.json << EOF
{
"name": "spark-config-overrides-image-pull-policy",
"virtualClusterId": "<virtual-cluster-id>",
"executionRole
Arn": "<execution-role-arn>",
"releaseLabel": "emr-6.2.0-latest",
"job
Driver": {
"sparkSubmitJob
Driver": {
"entryPoint": "s3://<s3 prefix>/trip-count.py",
"sparkSubmitParameters":
"--conf spark.driver.cores=2 --conf spark.executor.memory=2G \
--conf spark.driver.memory=2G --conf spark.executor.cores=2"
}
},
"configurationOverrides": {
"applicationConfiguration": [
{
"classification": "spark-defaults",
"properties": {
"spark.kubernetes.container.image.pullPolicy": "IfNotPresent"
}
}
],
"monitoringConfiguration": {
"cloudWatchMonitoringConfiguration": {
"logGroupName": "/emr-containers/jobs",
"logStreamNamePrefix": "demo"
},
"s3MonitoringConfiguration": {
"logUri": "s3://joblogs"
}
}
}
}
EOF
aws emr-containers start-job-run --cli-input-json \
file://spark-config-overrides-image-pull-policy.json
CONFIGURING IMAGE PULL POLICY FOR JOB SUBMITTER POD
You can use job submitter configuration to configure the image pull policy for the job-runner and Fluentd containers on the job
submitter pod
Sample command for using image pull policy for job submitter pod
cat >spark-image-pull-policy-job-submitter.json << EOF
{
"name": "spark-image-pull-policy-job-submitter",
"virtualClusterId": "<virtual-cluster-id>",
"executionRole
Arn": "<execution-role-arn>",
"releaseLabel": "emr-6.10.0-latest",
"job
Driver": {
"sparkSubmitJob
Driver": {
"entryPoint": "s3://<s3 prefix>/trip-count.py",
"sparkSubmitParameters":
"--conf spark.driver.cores=2 --conf spark.executor.memory=2G \
--conf spark.driver.memory=2G --conf spark.executor.cores=2"
}
},
"configurationOverrides": {
"applicationConfiguration": [
{
"classification": "emr-job-submitter",
"properties": {
"jobsubmitter.container.image.pullPolicy": "IfNotPresent"
}
}
],
"monitoringConfiguration": {
"cloudWatchMonitoringConfiguration": {
"logGroupName": "/emr-containers/jobs",
"logStreamNamePrefix": "demo"
},
"s3MonitoringConfiguration": {
"logUri": "s3://joblogs"
}
}
}
}
EOF
aws emr-containers start-job-run --cli-input-json \
file://spark-image-pull-policy-job-submitter.json
CONFIGURING IMAGE PULL POLICY FOR ALL PODS
You can use the spark.kubernetes.container.image.pullPolicy and
jobsubmitter.container.image.pullPolicy together to set the image pull policy for all Spark and Fluentd
containers in the job submitter, driver and executor pods.
Sample command for using image pull policy for all pods
cat >spark-image-pull-policy-all-pods.json << EOF
{
"name": "spark-image-pull-policy-all-pods",
"virtualClusterId": "<virtual-cluster-id>",
"executionRole
Arn": "<execution-role-arn>",
"releaseLabel": "emr-6.10.0-latest",
"job
Driver": {
"sparkSubmitJob
Driver": {
"entryPoint": "s3://<s3 prefix>/trip-count.py",
"sparkSubmitParameters":
"--conf spark.driver.cores=2 --conf spark.executor.memory=2G \
--conf spark.driver.memory=2G --conf spark.executor.cores=2"
}
},
"configurationOverrides": {
"applicationConfiguration": [
{
"classification": "spark-defaults",
"properties": {
"spark.kubernetes.container.image.pullPolicy": "IfNotPresent"
}
},
{
"classification": "emr-job-submitter",
"properties": {
"jobsubmitter.container.image.pullPolicy": "IfNotPresent"
}
}
],
"monitoringConfiguration": {
"cloudWatchMonitoringConfiguration": {
"logGroupName": "/emr-containers/jobs",
"logStreamNamePrefix": "demo"
},
"s3MonitoringConfiguration": {
"logUri": "s3://joblogs"
}
}
}
}
EOF
aws emr-containers start-job-run --cli-input-json \
file://spark-image-pull-policy-all-pods.json
VERIFYING THE IMAGE PULL POLICY ON THE PODS
You can verify the image pull policy that’s applied on the pods by describing the pod and looking at the image pull policy for each
of the containers. Use the below kubectl commands to see the image pull policy of a specific container
# Get the image pull policy of all the containers in a specific pod (job submitter,
# driver or executor) using the command
kubectl get pods <pod-name> -n <namespace>
-o jsonpath='{.spec.containers[*].imagePullPolicy}'
