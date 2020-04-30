---
layout: default
title: Video/Image/Annotation Registration Example 1
parent: MBARI
nav_order: 2
---

# Video/Image/Annotation Registration Example 1

In this example, we have videos from MBARI's ROVs that have been ripped from video tape. These videos have had ML applied to generate images of interest, annotations, and localizations (bounding boxes). All of this data needs to be registered in M3/VARS using our existing data load paths and infrastructure.

## Registering the Videos

First, determine if the video has already been registered in VARS. The simplest way to do that is to use a sha512 checksum.

### Create a checksum

Here's an example from the command line:

```
❯ shasum -p -a 512 ~/Downloads/V4066_20170822T135738Z_h264.mp4 | cut -d " " -f1
d4db763ee0ce8d517f0b08bf09419ff9f948970aac17313e9c232deaf8f1abad11a74ce3b84910abcd6b0d296575a6a4f333151f932a1c90d830c3c3f7ab1d55
```

### Check if that checksum exists

Make an HTTP GET request to a [vampire-squid](https://github.com/mbari-media-management/vampire-squid) endpoint to see if the video already exists. The endpoint format is `http://vampiresquid/media/sha512/{sha512}`. Using the sha generated above and MBARI's servers, the request would be <http://m3.shore.mbari.org/vam/v1/media/sha512/d4db763ee0ce8d517f0b08bf09419ff9f948970aac17313e9c232deaf8f1abad11a74ce3b84910abcd6b0d296575a6a4f333151f932a1c90d830c3c3f7ab1d55> and would return:

```json
{
  "video_sequence_uuid": "d92156c3-7a6c-4961-b442-905b151af584",
  "video_reference_uuid": "1b7e9001-f2dc-4612-8354-d56ac571683d",
  "video_uuid": "b1701e19-0a2b-4387-8b26-db92d9af9909",
  "video_sequence_name": "Ventana 4066",
  "camera_id": "Ventana",
  "video_name": "Ventana 4066 20170822T135738Z",
  "uri": "http://m3.shore.mbari.org/videos/M3/mezzanine/Ventana/2017/08/4066/V4066_20170822T135738Z_h264.mp4",
  "start_timestamp": "2017-08-22T13:57:38Z",
  "duration_millis": 900420,
  "container": "video/mp4",
  "width": 1920,
  "height": 1080,
  "frame_rate": 0.0,
  "size_bytes": 2947194196,
  "sha512": "D4DB763EE0CE8D517F0B08BF09419FF9F948970AAC17313E9C232DEAF8F1ABAD11A74CE3B84910ABCD6B0D296575A6A4F333151F932A1C90D830C3C3F7AB1D55"
}
```

If no matching sha512 is found the endpoint will return a 404 code.

If the video is already registered ... great! Move on to the next step. If it's not registered you will need to do the following steps:

### Registering a video (if needed)

#### Step 1: Verify and set the media creation time

Find the correct start time of the media (e.g. the time that the first frame was recorded)

Next, See if that's already set in the media:

```
❯ ffprobe -v quiet V4066_20170822T135738Z_h264.mp4 -print_format json -show_entries stream=index,codec_type:stream_tags=creation_time:format_tags=creation_time
{
    "programs": [

    ],
    "streams": [
        {
            "index": 0,
            "codec_type": "video",
            "tags": {
                "creation_time": "2017-08-22T13:57:38.000000Z"
            }
        },
        {
            "index": 1,
            "codec_type": "audio",
            "tags": {
                "creation_time": "2017-08-22T13:57:38.000000Z"
            }
        }
    ],
    "format": {
        "tags": {
            "creation_time": "2017-08-22T13:57:38.000000Z"
        }
    }
}
```

Finally, if not set correctly, set the creation_time using ffmpeg. Here's and example: `ffmpeg -i V4066_20170822T135738Z_h264.mp4 -metadata creation_time="2029-05-02 22:01:04" -codec copy VID_trashme.mp4`. Note that will generate a new copy of the video. 

#### Step 2: Registering a video

Fortunatly, at MBARI, this is mostly done automatically. Place the video in the corresponding location on `titan:/M2`. For example, MP4s from V4066 would to into `M3/mezzanine/Ventana/2017/08/4066`, MOVs would go into the sister `master` directory. Video registration occurs automatically once a day.

To verify that you're video is registered, as well as to retrieve the `video_reference_uuid`, you can use the videofile name to look it up: <http://m3.shore.mbari.org/vam/v1/media/videoreference/filename/V4066_20170822T135738Z_h264.mp4>. This call will return a JSON array of data for all videos with the name `V4066_20170822T135738Z_h264.mp4` or an empty array if no matches exist.

## Registering new images, annotations, and localizations

### Image wrangling

The first step is to get the image moved to the correct location. One method is to manually copy the framegrabs to `atlas:/framegrabs` then map the directory location a corresponding web URL. Here's an example mapping:

- directory: `atlas:/framegrabs/Ventana/images/4066/00_34_04_15.png`
- URL: <http://search.mbari.org/ARCHIVE/frameGrabs/Ventana/images/4066/00_34_04_15.png>

Another method is to upload an image to [panoptes](https://github.com/mbari-media-management/panoptes). It will archive the image in an appropriate location and return the new URL to the image. The workflow is as follows:

__authenticate__

Request

```
POST http:/m3.shore.mbari.org/panoptes/v1/auth
Authorization: APIKEY <clientsecret>
```

Response

```
HTTP/1.1 200 OK
Connection: close
Date: Thu, 30 Apr 2020 21:02:19 GMT
Content-Type: application/json;charset=utf-8
Content-Encoding: gzip

{
  "token_type": "Bearer",
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzUxMiJ9.eyJpc3MiOiJodHRwOi8vd3d3Lm1iYXJpLm9yZyIsImV4cCI6MTU4ODM2NjkzOSwiaWF0IjoxNTg4MjgwNTM5fQ.-HgomFYdj6O3XYMYfr8ytNx6SmhwokktQ5N2w3ofFFqhx3CUKOipuZ0CQLZXBbJllPa2hqXhUnlT3MnnYpOGEg"
}
```

You will need to access token for the next steps. The access token won't expire for a while, so you can reuse it for multiple requests.

__upload image__

Now you can submit a file to panoptes. Here's a python example:

Request

```python
# Url is http://panoptesroot/v1/images/{Platform name}/{Dive}/{filename}
url = "http:/m3.shore.mbari.org/panoptes/v1/images/Ventana/4066/00_34_04_15.png"
files = {'file': open('00_34_04_15.png', 'rb')}
r = requests.post(url, files=files, headers={"Authorization": "Bearer eyJ0eXAiOiJKV1Qi...})
```

Response

```python
r.text
# '{"uri":"http://search.mbari.org/ARCHIVE/frameGrabs/Ventana/images/4066/00_34_04_15.png","cameraId":"Ventana","deploymentId":"4066","name":"00_34_04_15.png"}'
```

### Registering image/annotation/bounding box

The path of least resistance it to add your data is to build a JSON struct and submit to to [annosaurus](https://github.com/mbari-media-management/annosaurus). Here's an example:

Note: annosaurus normally assigns most UUID's for you. However, you can define them yourself. For loading images and linking them to your association, pre-generate the image_reference_uuids. The example below shows JSON where you've assigned these UUIDs.

```json
[
  {
    "concept": "Bathocyroe",
    "observer": "name of ML tool?",
    "video_reference_uuid": "a9acd66d-509b-43a9-9fbb-97dd91c0b63a",
    "elapsed_time_millis": 185493,
    "associations": [
      {
        "link_name": "bounding box",
        "link_value": "{\"x\": 659, \"y\": 810, \"width\": 234, \"height\": 394, \"image_reference_uuid\": \"ed6fbffc-0f1a-4254-bd99-700a09a9c6ad\"}",
        "to_concept": "self",
        "mime_type": "application/json",
      },
    ],
    "image_references": [
      {
        "description": "source image",
        "url": "http://search.mbari.org/ARCHIVE/frameGrabs/Doc%20Ricketts/images/0710/02_09_18_27.png",
        "height_pixels": 1920,
        "width_pixels": 1080,
        "format": "image/png",
        "uuid": "ed6fbffc-0f1a-4254-bd99-700a09a9c6ad"
      }
    ]
  },
  {
    "concept": "Grimpoteuthis",
    "observer": "expert-observer",
    "video_reference_uuid": "a9acd66d-509b-43a9-9fbb-97dd91c0b63a",
    "elapsed_time_millis": 185493,
    "associations": [
      {
        "link_name": "bounding box",
        "link_value": "{\"x\": 303, \"y\": 855, \"width\": 261, \"height\": 462, \"image_reference_uuid\": \"ed6fbffc-0f1a-4254-bd99-700a09a9c6ad\"}",
        "to_concept": "self",
        "mime_type": "application/json",
      },
      {
        "link_name": "comment",
        "link_value": "Somre additional explanation text",
        "to_concept": "self",
        "mime_type": "text/plain",
        "uuid": "3c52e0a6-da2d-43df-0c66-d321495aa81e"
      },
      {
        "link_name": "project",
        "link_value": "VAA",
        "to_concept": "self",
        "mime_type": "text/plain",
        "uuid": "7d9d3b23-b74f-44a7-0d66-d321495aa81e"
      }
    ]
  }
]
```

With your JSON in hand, get an access token:

Request

```
POST http://m3.shore.mbari.org/anno/v1/auth
Authorization: APIKEY <client secret>
```

Response

```
HTTP/1.1 200 OK
Connection: close
Date: Thu, 30 Apr 2020 21:02:19 GMT
Content-Type: application/json;charset=utf-8
Content-Encoding: gzip

{
  "token_type": "Bearer",
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzUxMiJ9.eyJpc3MiOiJodHRwOi8vd3d3Lm1iYXJpLm9yZyIsImV4cCI6MTU4ODM2NjkzOSwiaWF0IjoxNTg4MjgwNTM5fQ.-HgomFYdj6O3XYMYfr8ytNx6SmhwokktQ5N2w3ofFFqhx3CUKOipuZ0CQLZXBbJllPa2hqXhUnlT3MnnYpOGEg"
}
```

Then use the access token to submit your JSON:

```
POST http://m3.shore.mbari.org/anno/v1/annotations/bulk
Authorization: Bearer eyJ0eXAiOiJKV1Qi...
Content-Type: application/json

[
  {
    "concept": "Bathocyroe",
    "observer": "name of ML tool?",
    "video_reference_uuid": "a9acd66d-509b-43a9-9fbb-97dd91c0b63a",
    "elapsed_time_millis": 185493,
    "associations": [
      {
        "link_name": "bounding box",
        "link_value": "{\"x\": 659, \"y\": 810, \"width\": 234, \"height\": 394, \"image_reference_uuid\": \"ed6fbffc-0f1a-4254-bd99-700a09a9c6ad\"}",
        "to_concept": "self",
        "mime_type": "application/json",
      },
    ], [...]
```

