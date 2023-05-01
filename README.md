  
  Enable essential APIs for Cloud Build
  gcloud services enable container.googleapis.com     cloudbuild.googleapis.com     sourcerepo.googleapis.com     artifactregistry.googleapis.com
  
  
  Create an artifact repo (This step is only required if you if your don't have a docker artifact registry)
  gcloud artifacts repositories create laravel-repository   --repository-format=docker   --location=us-central1


 Create a kubernetes cluster (This step is only required if you if your don't have a kubernetes cluster)
 gcloud container clusters create-auto laravel-gke  --region us-central1
 
 Download a sample app 
 Create Code repos (This step is only required if you if your don't have an app)
 git config --global user.email info@zaynsolutions.com
 git config --global user.name Adeel Shafqat
 gcloud source repos create laravel-app

  Download a sample app (This step is only required if you if your don't have an app)
  cd ~
  git clone https://github.com/adshafqat/laravel-app.git     laravel-app
  cd laravel-app/
  ls -lrt
  PROJECT_ID=$(gcloud config get-value project)
  
  Add this sample app to code repo we created earlier  (This step is only required if you if your don't have an app)
  git remote add google     "https://source.developers.google.com/p/${PROJECT_ID}/r/laravel-app"
  git add .
  git commit -m "Latest code"
  git push google master

  
  Manually test a build. This will create an image and store it in Artifactory. This step uses your Docker file
  
  COMMIT_ID="$(git rev-parse --short=7 HEAD)"
  gcloud builds submit --tag="us-central1-docker.pkg.dev/${PROJECT_ID}/laravel-repository/laravel-app:${COMMIT_ID}" .



  To deploy the application in your Kubernetes cluster, Cloud Build needs the Kubernetes Engine Developer Identity and Access Management Role.
  PROJECT_NUMBER="$(gcloud projects describe ${PROJECT_ID} --format='get(projectNumber)')"
  gcloud projects add-iam-policy-binding ${PROJECT_NUMBER}     --member=serviceAccount:${PROJECT_NUMBER}@cloudbuild.gserviceaccount.com     --role=roles/container.developer
  
  
  Grant the Source Repository Writer IAM role to the Cloud Build service account for the laravel-app repository. This step is required if you want cloud build to write in a repo. 
  In multiple environment setup this can be used for your cloud build to promote code from one repo to another

   PROJECT_NUMBER="$(gcloud projects describe ${PROJECT_ID} --format='get(projectNumber)')"

  cat >/tmp/hello-cloudbuild-env-policy.yaml <<EOF
bindings:
- members:
  - serviceAccount:${PROJECT_NUMBER}@cloudbuild.gserviceaccount.com
  role: roles/source.writer
EOF

gcloud source repos set-iam-policy     laravel-app /tmp/hello-cloudbuild-env-policy.yaml


Create a Trigger in cloud build. When someone changes something in the code. Build should automatically be triggered. This should be done through Cloud Build console

After creating the trigger. Just implement a small change to see if it triggers a build


cd ~/laravel-app
git add .
git commit -m "Trigger CD pipeline"
git push google master

Go to the build console and you will see a new build is triggered. If you login to kubernetes you can see a new deployed application

Connect to GKE cluster
gcloud container clusters get-credentials laravel-gke --region us-central1 --project devops2-385401

us-central1-docker.pkg.dev/devops2-385401/laravel-repository/laravel-app:d60a5ba



git branch production
git add .
git commit -m "code update"
git push google production
git branch -M master
git branch

https://cloud.google.com/kubernetes-engine/docs/tutorials/gitops-cloud-build

