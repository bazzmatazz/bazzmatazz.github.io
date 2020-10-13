---
layout: post
title:  "Cloud Storage Bucket Specific Access Using gsutil"
date:   2019-06-09 06:14:59 +0500
categories: 
---

Create a services account, and then:

- `gsutil iam ch serviceAccount:{service account name}:{role} gs://{bucket name}`

- `gsutil iam ch serviceAccount:secret@super.iam.gserviceaccount.com:objectAdmin gs://secretBucket`
