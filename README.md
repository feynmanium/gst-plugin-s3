# gst-plugin-s3

[![Build Status](https://travis-ci.org/ford-prefect/gst-plugin-s3.svg?branch=master)](https://travis-ci.org/ford-prefect/gst-plugin-s3)

This is a [GStreamer](https://gstreamer.freedesktop.org/) plugin to interact
with the [Amazon Simple Storage Service (S3)](https://aws.amazon.com/s3/).

Currently, a simple source element exists. The eventual plan is to also add a
sink, to allow writing out objects directly to S3.

## AWS Credentials

AWS credentials are picked up using the mechanism that
[rusoto's ChainProvider](http://rusoto.github.io/rusoto/rusoto/struct.ChainProvider.html)
uses. At the moment, that is:

 1. Environment variables: `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`
 2. AWS credentials file. Usually located at ~/.aws/credentials.
 3. IAM instance profile. Will only work if running on an EC2 instance with an instance profile/role.

An example credentials file might look like:

```ini
[default]
aws_access_key_id = ...
aws_secret_access_key = ...
```

## s3src

Reads from a given S3 (region, bucket, object, version?) tuple. The version may
be omitted, in which case the default behaviour of fetching the latest version
applies.

```
$ gst-launch-1.0 \
    s3src uri=s3://ap-south-1/my-bucket/my-object-key/which-can-have-slashes?version=my-optional-version !
    filesink name=my-object.out
```

### TODO

A bunch of things need work:

 * The default blocksize is 4 kB, so we're making tiny `GET` requests to read
   the data. This can be mitigated with a 64 kB blocksize (`blocksize=65536` in
   the above command-line example). The proper fix for this is tracked in the
   [rusoto issue for streaming support](https://github.com/rusoto/rusoto/issues/481)

 * Fetching by vesion hasn't been tested properly yet

 * We need to add support for `create()` rather than fill in the `Source` trait
   (this is in [gst-plugin-rs](https://github.com/sdroege/gst-plugin-rs). Using
   `fill()` as we current do means we're doing an extra memcpy.
