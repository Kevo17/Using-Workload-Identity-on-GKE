<h1>Using Workload Identity on GKE</h1>


<h2>Description</h2>
Running workloads with their own unique service identity in GCP allows you to exercise the security principle of least privilege: granting only the granular permissions required by a workload, and limiting the blast radius should it become compromised. With GKE Workload Identity, Kubernetes service accounts can be mapped to GCP service accounts to enable service identity authorization for requests to Google APIs and other services. In this lab, we will create a secret Cloud Run service and then map a service account to allow a GKE workload to access it.
<br />

<h2>Environments Used </h2>

- <b>GCP</b>
- <b>Windows 11</b>
- <b>Linux</b>
  
<h2>Program walk-through:</h2>

<h3>Deploy the Secret Service to Cloud Run</h3>


From the GCP console, click the Activate Cloud Shell icon (>_) to the right of the top menu.<br />

When prompted, click CONTINUE.

In the Cloud Shell terminal, enable the required APIs for this lab:
```
gcloud services enable container.googleapis.com containerregistry.googleapis.com cloudbuild.googleapis.com run.googleapis.com
```

<br/>
 
<p align="center">
<img src="https://i.imgur.com/jjQxANw.png" height="80%" width="80%" alt="Disk Sanitization Step"/>
</p>

<br />
<br />

Clone the GitHub repo provided for this lab:
```
git clone https://github.com/linuxacademy/content-google-certified-pro-cloud-developer
```

<br/>
 
<p align="center">
<img src="https://i.imgur.com/UlGPfaY.png" height="80%" width="80%" alt="Disk Sanitization Step"/>
</p>

<br />
<br />

Change to the GitHub repo's gke-workload-identity/secret-service directory:
```
cd content-google-certified-pro-cloud-developer/gke-workload-identity/secret-service/
```

<br/>
 
<p align="center">
<img src="https://i.imgur.com/Zf74Kt4.png" height="80%" width="80%" alt="Disk Sanitization Step"/>
</p>

<br />
<br />

From the directory, build the Secret service container, replacing <YOUR_PROJECT_ID> with your own project ID (the yellow text shown in the terminal):
```
gcloud builds submit --tag gcr.io/<YOUR_PROJECT_ID>/secret-service
```
<br/>
 
<p align="center">
<img src="https://i.imgur.com/fLOqE9B.png" height="80%" width="80%" alt="Disk Sanitization Step"/>
</p>

<br />
<br />
Deploy the container to Cloud Run, again substituting your project ID:
```
gcloud run deploy secret-service --image gcr.io/<YOUR_PROJECT_ID>/secret-service --no-allow-unauthenticated --platform managed --region us-central1
```

<br/>
 
<p align="center">
<img src="https://i.imgur.com/Q42oJbl.png.png" height="80%" width="80%" alt="Disk Sanitization Step"/>
</p>

<br />
<br />

After the service is deployed, try accessing the Service URL provided in the terminal. You should receive a Forbidden error message, as you do not have permission to access the service without the correct identity.

<br/>
 
<p align="center">
<img src="https://i.imgur.com/BXIx1wq.png" height="80%" width="80%" alt="Disk Sanitization Step"/>
</p>

<br />
<br />

<h3>Create the GKE Cluster with Workload Identity</h3>

CREATE to create a new cluster.<br />
To the right of Standard: You manage your cluster click CONFIGURE.<br />
In the left sidebar menu, locate NODE POOLS, and select the default-pool.<br />
In the Size section, set the Number of nodes to 1.

<br/>
 
<p align="center">
<img src="https://i.imgur.com/CJGNQBd.png" height="80%" width="80%" alt="Disk Sanitization Step"/>
</p>

<br />
<br />

In the left sidebar menu, locate CLUSTER, and select Security.<br />
In the Security pane, check the box next to Enable Workload Identity.<br />
Note: Both Enable Workload Identity and Enable Shielded GKE Nodes should be selected.<br />

Click CREATE to create the cluster.


<br/>
 
<p align="center">
<img src="https://i.imgur.com/tOYFGJr.png" height="80%" width="80%" alt="Disk Sanitization Step"/>
</p>

<br />
<br />

After the cluster is created, use the three-dot Action menu to the right of the cluster details to select Connect.<br />
In the pop-up Connect to the cluster, below Command-line access, copy the command to your clipboard.<br />
Navigate back to the Cloud Shell terminal and paste the copied command.<br />
After the command populates in the Cloud Shell terminal, run it to set up kubectl.<br />

<br/>
 
<p align="center">
<img src="https://i.imgur.com/GHAtixy.png" height="80%" width="80%" alt="Disk Sanitization Step"/>
</p>

<br />
<br />

Navigate back to the Cloud Shell terminal and paste the copied command.<br />
After the command populates in the Cloud Shell terminal, run it to set up kubectl.


<br/>
 
<p align="center">
<img src="https://i.imgur.com/PnszTF5.png" height="80%" width="80%" alt="Disk Sanitization Step"/>
</p>

<br />
<br />

<h3>Configure Service Accounts and IAM</h3>

Create a service account for your Secret Agent workload:
```
gcloud iam service-accounts create secret-agent
```


<br/>
 
<p align="center">
<img src="https://i.imgur.com/ze5sEk1.png" height="80%" width="80%" alt="Disk Sanitization Step"/>
</p>

<br />
<br />

Create an IAM policy to allow the Secret Agent service account to invoke the Secret Service in Cloud Run, replacing <YOUR_PROJECT_ID> with your own project ID:
```
gcloud run services add-iam-policy-binding secret-service --member='serviceAccount:secret-agent@<YOUR_PROJECT_ID>.iam.gserviceaccount.com' --role='roles/run.invoker' --platform managed --region us-central1
```


<br/>
 
<p align="center">
<img src="https://i.imgur.com/Om9cjcw.png" height="80%" width="80%" alt="Disk Sanitization Step"/>
</p>

<br />
<br />

Change into the gke-workload-identity/secret-agent directory:
```
cd ../secret-agent
```

<br/>
 
<p align="center">
<img src="https://i.imgur.com/UiIWKBF.png" height="80%" width="80%" alt="Disk Sanitization Step"/>
</p>

<br />
<br />

Create the Kubernetes service account for the Secret Agent workload:
```
kubectl create -f serviceaccount.yaml
```

<br/>
 
<p align="center">
<img src="https://i.imgur.com/LBhpdcZ.png" height="80%" width="80%" alt="Disk Sanitization Step"/>
</p>

<br />
<br />

Allow the Kubernetes service account to impersonate the Google service account by creating an IAM policy binding between them, again substituting your project ID in both required locations:
```
gcloud iam service-accounts add-iam-policy-binding --role roles/iam.workloadIdentityUser --member "serviceAccount:<YOUR_PROJECT_ID>.svc.id.goog[default/secret-agent]" secret-agent@<YOUR_PROJECT_ID>.iam.gserviceaccount.com
```

<br/>
 
<p align="center">
<img src="https://i.imgur.com/FMaDggL.png" height="80%" width="80%" alt="Disk Sanitization Step"/>
</p>

<br />
<br />

Annotate the Kubernetes service account to complete the mapping, again substituting your project ID:
```
kubectl annotate serviceaccount --namespace default secret-agent iam.gke.io/gcp-service-account=secret-agent@<YOUR_PROJECT_ID>.iam.gserviceaccount.com
```

<br/>
 
<p align="center">
<img src="https://i.imgur.com/Id2T9Aa.png" height="80%" width="80%" alt="Disk Sanitization Step"/>
</p>

<br />
<br />

<h3>Deploy the Secret Agent Workload to GKE</h3>

Create the PROJECT_ID environment variable for the workload:
```
export PROJECT_ID=$(gcloud config list project --format "value(core.project)")
```
<br/>
 
<p align="center">
<img src="https://i.imgur.com/EcsXMY4.png" height="80%" width="80%" alt="Disk Sanitization Step"/>
</p>

<br />
<br />

Create the SECRET_URL environment variable for the workload:
```
export SECRET_URL=$(gcloud run services describe secret-service --platform managed --region us-central1 --format 'value(status.url)')
```
<br/>
 
<p align="center">
<img src="https://i.imgur.com/eOeUI7A.png" height="80%" width="80%" alt="Disk Sanitization Step"/>
</p>

<br />
<br />

Confirm the PROJECT_ID variable was set correctly:
```
echo $PROJECT_ID
```
<br/>
 
<p align="center">
<img src="https://i.imgur.com/XWDca55.png" height="80%" width="80%" alt="Disk Sanitization Step"/>
</p>

<br />
<br />

Confirm the SECRET_URL variable was set correctly:
```
echo $SECRET_URL
```
<br/>
 
<p align="center">
<img src="https://i.imgur.com/Rp39Ws8.png" height="80%" width="80%" alt="Disk Sanitization Step"/>
</p>

<br />
<br />

From within the gke-workload-identity/secret-agent directory, create the agent Pod manifest:
```
envsubst < pod-template.yaml > agent-pod.yaml
```
<br/>
 
<p align="center">
<img src="https://i.imgur.com/wT4wKaI.png" height="80%" width="80%" alt="Disk Sanitization Step"/>
</p>

<br />
<br />

Build the Secret Agent container, replacing <YOUR_PROJECT_ID> with your own project ID:
```
gcloud builds submit --tag gcr.io/<YOUR_PROJECT_ID>/secret-agent
```
<br/>
 
<p align="center">
<img src="https://i.imgur.com/KZQAGCl.png" height="80%" width="80%" alt="Disk Sanitization Step"/>
</p>

<br />
<br />

Deploy the Secret Agent Pod to GKE:
```
kubectl apply -f agent-pod.yaml
```
<br/>
 
<p align="center">
<img src="https://i.imgur.com/IxxtHEp.png" height="80%" width="80%" alt="Disk Sanitization Step"/>
</p>

<br />
<br />

After the Pod is in a Running state, you should be able to observe that it can successfully connect to the Secret service Cloud Run service by viewing its logs:
```
kubectl logs secret-agent
```
<br/>
 
<p align="center">
<img src="https://i.imgur.com/w4niuyp.png" height="80%" width="80%" alt="Disk Sanitization Step"/>
</p>

<br />
<br />
