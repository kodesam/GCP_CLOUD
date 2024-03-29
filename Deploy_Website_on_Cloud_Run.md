Deploy Your Website on Cloud Run

Clone Source Repository
Since you are deploying an existing website, you just need to clone the source, so you can focus on creating Docker images and deploying to Cloud Run.

Run the following commands to clone the git repo to your Cloud Shell instance and change to the appropriate directory. You will also install the NodeJS dependencies so you can test the application before deploying:

```

git clone https://github.com/googlecodelabs/monolith-to-microservices.git

```
```
cd ~/monolith-to-microservices
./setup.sh

```
Test your application by running the following command to start the web server:

```
cd ~/monolith-to-microservices/monolith
npm start

```
Output:

Monolith listening on port 8080!
Preview your application by clicking the web preview icon and selecting Preview on port 8080.

Cloud Build will compress the files from the directory and move them to a Cloud Storage bucket. The build process will then take all the files from the bucket and use the Dockerfile, which is present in the same directory, to run the Docker build process. Since you specified the --tag flag with the host as gcr.io for the Docker image, the resulting Docker image will be pushed to Container Registry.

First run the following command to enable the Cloud Build API:
```
gcloud services enable cloudbuild.googleapis.com
```

After the API is enabled, run the following command to start the build process:

```
gcloud builds submit --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/monolith:1.0.0 .

```
Command-Line
You will deploy the image that was built earlier, and choose the managed version of Cloud Run by specifying --platform managed.

First you need to enable the Cloud Run API. Run the following command to enable it:
```

gcloud services enable run.googleapis.com

```
Run the following command to deploy your application:

```
gcloud run deploy --image=gcr.io/${GOOGLE_CLOUD_PROJECT}/monolith:1.0.0 --platform managed

```
Accept the default suggested service name (it will be "monolith") by pressing Enter.

Specify which region you'd like to run in. Type the number for the region closest to you.

Verify deployment
To verify the deployment was created successfully, run the following command:
```
gcloud run services list

```
It may take a few moments for the pod status to be Running.

Type "1" to choose the first option: [1] Cloud Run (fully managed)

Output:

SERVICE   REGION    URL  LAST DEPLOYED BY          LAST DEPLOYED AT
✔  monolith  us-east1 <your url>  <your email>  2019-09-16T21:07:38.267Z
Create new revision with lower concurrency
Deploy your application again, but this time adjust one of the parameters.

By default, a Cloud Run application will have a concurrency value of 80, meaning that each container instance will serve up to 80 requests at a time. This is a big departure from the Functions-as-a-Service model, where one instance handles one request at a time.

Re-deploy the same container image with a concurrency value of 1 (just for testing), and see what happens:

```
gcloud run deploy --image=gcr.io/${GOOGLE_CLOUD_PROJECT}/monolith:1.0.0 --platform managed --concurrency 1
    
```
Answer the subsequent questions just as you did the first time. Once the command is successful, check the Console to see the result.

To see the details, from the Navigation menu, click on _Cloud Run, then click on the monolith service :

Next, restore the original concurrency without re-deploying.

You could set the concurrency value back to the default of "80", or you could just set the value to "0", which will remove any concurrency restrictions and set it to the default max (which happens to be 80).

Run the following command from Cloud Shell to update the current revision:
    
```

gcloud run deploy --image=gcr.io/${GOOGLE_CLOUD_PROJECT}/monolith:1.0.0 --platform managed --concurrency 80

```    
Answer the subsequent questions just as you have before.
    

You will notice that another revision has been created, that traffic has now been redirected, and that the concurrency is back up to 80

Make Changes To The Website
Scenario: Your marketing team has asked you to change the homepage for your site. They think it should be more informative of who your company is and what you actually sell.

Task: You will add some text to the homepage to make the marketing team happy! It looks like one of our developers already created the changes with the file name index.js.new. You can just copy this file to index.js and your changes should be reflected. Follow the instructions below to make the appropriate changes.

Run the following commands to copy the updated file to the correct file name and then print its contents to verify the changes:
    
```

cd ~/monolith-to-microservices/react-app/src/pages/Home
mv index.js.new index.js
cat ~/monolith-to-microservices/react-app/src/pages/Home/index.js
    
```
The resulting code should look like this:

```
/*
Copyright 2019 Google LLC
Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at
    https://www.apache.org/licenses/LICENSE-2.0
Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
*/
import React from "react";
import { makeStyles } from "@material-ui/core/styles";
import Paper from "@material-ui/core/Paper";
import Typography from "@material-ui/core/Typography";
const useStyles = makeStyles(theme => ({
  root: {
    flexGrow: 1
  },
  paper: {
    width: "800px",
    margin: "0 auto",
    padding: theme.spacing(3, 2)
  }
}));
export default function Home() {
  const classes = useStyles();
  return (
    <div className={classes.root}>
      <Paper className={classes.paper}>
        <Typography variant="h5">
          Fancy Fashion &amp; Style Online
        </Typography>
        <br />
        <Typography variant="body1">
          Tired of mainstream fashion ideas, popular trends and societal norms?
          This line of lifestyle products will help you catch up with the Fancy trend and express your personal style.
          Start shopping Fancy items now!
        </Typography>
      </Paper>
    </div>
  );
}

```
You updated the React components, but you need to build the React app to generate the static files.

Run the following command to build the React app and copy it into the monolith public directory:
```
    
cd ~/monolith-to-microservices/react-app
npm run build:monolith
    
```
Now that the code is updated, rebuild the Docker container and publish it to Container Registry. You can use the same command as before, except this time you will update the version label.

Run the following command to trigger a new Cloud Build with an updated image version of 2.0.0:

```
cd ~/monolith-to-microservices/monolith

```
Optional

#Feel free to test your application
    
```
npm start
gcloud builds submit --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/monolith:2.0.0 .
    
```
In the next section you will use this image to update your application with zero downtime.

Cloud Run treats each deployment as a new Revision which will first be brought online, then have traffic redirected to it. By default the latest revision will be assigned 100% of the inbound traffic for a service. It is possible to use "Routes" to allocate different percentages of traffic to different revisions within a service.

Follow the instructions below to update your website.

From Cloud Shell, re-deploy the service to update the image to a new version with the following command:
    
```

gcloud run deploy --image=gcr.io/${GOOGLE_CLOUD_PROJECT}/monolith:2.0.0 --platform managed

```
verify Deployment
Validate that your deployment updated by running the following command:

```

gcloud run services describe monolith --platform managed
    
```
Type in the number for the region you've been using.

Output:

✔ Service monolith in region us-east1
https://monolith-k3wa45c54a-ue.a.run.app
Traffic:
  100%               LATEST (currently monolith-00003-ten)
Last updated on 2020-02-20T20:26:56.049Z by student-03-e32055b0be3a@qwiklabs.net:
  Revision monolith-00003-ten
  Image:             gcr.io/qwiklabs-gcp-03-b1646832ec74/monolith:2.0.0
  Port:              8080
  Memory:            256M
  CPU:               1000m
  Concurrency:       1
  Timeout:           900s
  
  To verify the changes, navigate to the external URL of the Cloud Run service, refresh the page, and notice that the application title has been updated.

Run the following command to list the services and view the IP address:

```
gcloud beta run services list

```
Cleanup
When you end this lab all of the resources you used will be destroyed. It's good to know how to delete resources so you can do so in your own environment.

Run the following to delete Container Registry images:


# Delete the container image for version 1.0.0 of our monolith

```    
gcloud container images delete gcr.io/${GOOGLE_CLOUD_PROJECT}/monolith:1.0.0 --quiet
    
```
# Delete the container image for version 2.0.0 of our monolith
    
```
gcloud container images delete gcr.io/${GOOGLE_CLOUD_PROJECT}/monolith:2.0.0 --quiet
    
```
Run the following to delete Cloud Build artifacts from Cloud Storage:

# The following command will take all source archives from all builds and delete them from cloud storage
# Run this command to print all sources:

```    
gcloud builds list | awk 'NR > 1 {print $4}'
gcloud builds list | awk 'NR > 1 {print $4}' | while read line; do gsutil rm $line; done
    
```
Finally, delete Cloud Run service:
    
```

gcloud beta run services delete monolith --platform managed
    
```
