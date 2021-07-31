# Creating Dataproc Cluster with Custom VPC Tutorial


This demo will show how to build a Dataproc cluster with a custom VPC

Go to your cloud-shell in your Gcloud console and create your dataproc cluster


## Create the VPC network

```sh
export PROJECT="$(gcloud config get-value project)"

gcloud compute networks create demo-vpc \
    --subnet-mode=custom \
    --bgp-routing-mode=regional 
```

## Create the subnet


```sh
gcloud compute networks subnets create demo-vpcsub \
    --network=demo-vpc \
    --range=10.0.0.0/9 \
    --region=us-central1

```
## Create the cluster and attach the network created above.

```sh

gcloud dataproc clusters create exercise-cluster \
 --image-version 2.0-debian10 \
    --subnet "projects/$PROJECT/regions/us-central1/subnetworks/demo-vpcsub" \
    --region=us-central1 \
    --tags=datatag

```


## Exercise 1

Please can you check why the cluster is not working? 
Hint: Please navigate to the cloud console logging page to view cluster logs. 
You should see log entries with similar error messages below


```sh
2021-07-16T02:41:57Z Only 0 out of 2 minimum required datanodes running. I 
2021-07-16T02:41:57.264Z Starting scan to move intermediate done files I 
2021-07-16T02:41:58Z Only 0 out of 2 minimum required node managers running. I 
2021-07-16T02:41:58Z Only 0 out of 2 minimum required datanodes running. I 
2021-07-16T02:41:58.264Z Starting scan to move intermediate done files I 
2021-07-16T02:41:59Z Only 0 out of 2 minimum required node managers running. 

```

## Solution

Add the firewall rule to allow port 0-65535 to allow traffic between the VMs.

```sh
gcloud compute firewall-rules create "tcp-rule" --allow=tcp:0-65535 --target-tags=datatag \
    --source-ranges="10.0.0.0/9" --description="Narrowing TCP traffic" --network=demo-vpc


gcloud compute firewall-rules create "ssh-rule" --allow=tcp:22 --target-tags=datatag \
    --source-ranges="0.0.0.0/0" --description="allow ssh" --network=demo-vpc


```

## Clean up

```sh

gcloud dataproc clusters delete exercise-cluster --region=us-central1

gcloud compute firewall-rules delete tcp-rule

gcloud compute firewall-rules delete ssh-rule

gcloud compute networks subnets delete demo-vpcsub

gcloud compute networks delete demo-vpc 

```
