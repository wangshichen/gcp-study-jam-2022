# Quest 
Perform Foundational Infrastructure Tasks in Google Cloud

## Topics of the Quest
- Cloud Storage: Qwik Start - Cloud Console or CLI/SDK
- Cloud IAM: Qwik Start
- Cloud Monitoring: Qwik Start
- Cloud Functions: Qwik Start - Console or Command Line
- Google Cloud Pub/Sub: Qwik Start - Console or CLI or Python
- Perform Foundational Infrastructure Tasks in Google Cloud: Challenge Lab

## Challenge Lab 
### Tasks
- Create a bucket for storing the photographs.
- Create a Pub/Sub topic that will be used by a Cloud Function you create.
- Create a Cloud Function.
- Remove the previous cloud engineer’s access from the memories project.

### Notes
- Create all resources in the `us-east1` region and `us-east1-b` zone, unless otherwise directed.
- Use the project VPCs.
- Naming is normally team-resource, e.g. an instance could be named `kraken-webserver1`
- Allocate cost effective resource sizes. Projects are monitored and excessive resource use will result in the containing project's termination (and possibly yours), so beware. This is the guidance the monitoring team is willing to share; unless directed, use **f1-micro** for small Linux VMs and **n1-standard-1** for Windows or other applications such as Kubernetes nodes.

### step reference
#### Prerequisite
- activate cloud shell terminal and type `gcloud auth list` for authorization.
- `gcloud config set compute/zone us-east1-b` & `gcloud config set compute/region us-east1` set default region/zone.
- Download the sample image file `curl https://storage.googleapis.com/cloud-training/gsp315/map.jpg --output map.jpg`

#### Task 1 Create a bucket
`gsutil mb gs://[BUCKET_NAME]`

#### Task 2 Create a Pub/Sub topic
`gcloud pubsub topics create [TOPIC_NAME]`    gcloud pubsub topics create [TOPIC_NAME]

#### Task 3 Create the thumbnail Cloud Function
- `mkdir gcf_thumbnail`
- `cd gcf_thumbnail`
- `vim index.js`
- paste the following content and replace the text **REPLACE_WITH_YOUR_TOPIC** ID with the Topic Name created in task 2.
```
/* globals exports, require */
//jshint strict: false
//jshint esversion: 6
"use strict";
const crc32 = require("fast-crc32c");
const { Storage } = require('@google-cloud/storage');
const gcs = new Storage();
const { PubSub } = require('@google-cloud/pubsub');
const imagemagick = require("imagemagick-stream");
exports.thumbnail = (event, context) => {
  const fileName = event.name;
  const bucketName = event.bucket;
  const size = "64x64"
  const bucket = gcs.bucket(bucketName);
  const topicName = "REPLACE_WITH_YOUR_TOPIC ID";
  const pubsub = new PubSub();
  if ( fileName.search("64x64_thumbnail") == -1 ){
    // doesn't have a thumbnail, get the filename extension
    var filename_split = fileName.split('.');
    var filename_ext = filename_split[filename_split.length - 1];
    var filename_without_ext = fileName.substring(0, fileName.length - filename_ext.length );
    if (filename_ext.toLowerCase() == 'png' || filename_ext.toLowerCase() == 'jpg'){
      // only support png and jpg at this point
      console.log(`Processing Original: gs://${bucketName}/${fileName}`);
      const gcsObject = bucket.file(fileName);
      let newFilename = filename_without_ext + size + '_thumbnail.' + filename_ext;
      let gcsNewObject = bucket.file(newFilename);
      let srcStream = gcsObject.createReadStream();
      let dstStream = gcsNewObject.createWriteStream();
      let resize = imagemagick().resize(size).quality(90);
      srcStream.pipe(resize).pipe(dstStream);
      return new Promise((resolve, reject) => {
        dstStream
          .on("error", (err) => {
            console.log(`Error: ${err}`);
            reject(err);
          })
          .on("finish", () => {
            console.log(`Success: ${fileName} → ${newFilename}`);
              // set the content-type
              gcsNewObject.setMetadata(
              {
                contentType: 'image/'+ filename_ext.toLowerCase()
              }, function(err, apiResponse) {});
              pubsub
                .topic(topicName)
                .publisher()
                .publish(Buffer.from(newFilename))
                .then(messageId => {
                  console.log(`Message ${messageId} published.`);
                })
                .catch(err => {
                  console.error('ERROR:', err);
                });
          });
      });
    }
    else {
      console.log(`gs://${bucketName}/${fileName} is not an image I can handle`);
    }
  }
  else {
    console.log(`gs://${bucketName}/${fileName} already has a thumbnail`);
  }
};
```
- `vim package.json`
- paste the following content
```
{
  "name": "thumbnails",
  "version": "1.0.0",
  "description": "Create Thumbnail of uploaded image",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {
    "@google-cloud/pubsub": "^2.0.0",
    "@google-cloud/storage": "^5.0.0",
    "fast-crc32c": "1.0.4",
    "imagemagick-stream": "4.1.1"
  },
  "devDependencies": {},
  "engines": {
    "node": ">=4.3.2"
  }
}
```
- `gcloud functions deploy [FUNCTION_NAME] --runtime nodejs14 --entry-point thumbnail --trigger-resource [BUCKET_NAME] --trigger-event google.storage.object.finalize`
- `gsutil cp map.jpg gs://[BUCKET_NAME]`
#### Task 4 Remove the previous cloud engineer
- browse the IAM page in google cloud console
- remove the role of the user 2 
