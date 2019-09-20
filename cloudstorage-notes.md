# Cloud Storage Notes

We've been using github pages to host the entire set of content located in the images repo (under the /images folder of our site).

In order to switch over to googles cloud storage, we'll have to start running a separate step to deploy new files. Since we want all of these resources to be publicly available, the easiest way seems to be using gsutil to sync up the contents of the images repo.

## Refactoring everything

### images repo

1. move actual image files into images folder of repo
2. maybe rename repo to assets on github (not entirely important)
3. create deploy scripts to help with the workflow described below

### site (feedme-ng) repo

1. change the relative prefixes for locations of image and attribution files

### feedme-data

fix various scripts and explorer pages as needed

## Basic Workflow

One time processes (need to do for each environment)

* create the account
* turn on cloud storage
* set up the security rules to allow public read only access
* create/get name of bucket
* set the cors headers
* setup cloud function log archiver

Workflow for testing/deploying new static resources

1. push (new) files to stage with rsync
2. chmod the files to make them public
3. test changes
4. sync stage to prod (make sure to preserve acls)

## push the files

from the root of the images directory:

    gsutil -m rsync -a public-read images/*.jpg gs://feedme-stage.appspot.com/images/
    gsutil -m rsync -J -a public-read attributions/ gs://feedme-stage.appspot.com/attributions/
    gsutil rsync -a public-read meta/ gs://feedme-stage.appspot.com/meta/
    
## sync to prod

    gsutil -m rsync -a public-read gs://feedme-stage.appspot.com/images/ gs://feedme-jefferson.appspot.com/images/
    gsutil -m rsync -a public-read gs://feedme-stage.appspot.com/attributions/ gs://feedme-jefferson.appspot.com/attributions/
    gsutil -m rsync -a public-read gs://feedme-stage.appspot.com/meta/ gs://feedme-jefferson.appspot.com/meta/
    
    
## set cors on bucket

create a json file with appropriate cors headers -- something like:

```
[
    {
      "origin": ["https://feedme-stage.firebaseapp.com","https://feedme-stage.firebaseapp.com","https://www.feedmejefferson.com","http://www.feedmejefferson.com"],
      "responseHeader": ["Content-Type"],
      "method": ["GET"],
      "maxAgeSeconds": 10
    }
]
```

then run:

    gsutil cors set cors.json gs://feedme-stage.appspot.com
    
    
## Log archiving

This isn't entirely related to the image data specifically, but it will be using the same bucket in cloud storage, so we need to make sure that it doesn't interfere at all

[log into the console](https://console.cloud.google.com/functions/list?project=feedme-stage)

1. from the left nav menu (click the hamburger) select __Cloud Functions__
2. Click on the `appetite` function
3. Click on `View Logs` near the top of the page
4. Click on `Create Export`
5. On the right hand pane __Edit Export__ 
    1. name the sink `appetite-logs`
    2. select __Cloud Storage__ from the sink service drop down
    3. select the project main bucket from the Sink Destination drop down
    4. click `save sink` 