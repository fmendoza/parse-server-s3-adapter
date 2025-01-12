# Parse Server S3 File Adapter <!-- omit in toc -->

[![Build Status](https://github.com/parse-community/parse-server-s3-adapter/workflows/ci/badge.svg?branch=master)](https://github.com/parse-community/parse-server-s3-adapter/actions?query=workflow%3Aci+branch%3Amaster)
[![Snyk Badge](https://snyk.io/test/github/parse-community/parse-server-s3-adapter/badge.svg)](https://snyk.io/test/github/parse-community/parse-server-s3-adapter)
[![Coverage](https://img.shields.io/codecov/c/github/parse-community/parse-server-s3-adapter/master.svg)](https://codecov.io/github/parse-community/parse-server-s3-adapter?branch=master)
[![auto-release](https://img.shields.io/badge/%F0%9F%9A%80-auto--release-9e34eb.svg)](https://github.com/parse-community/parse-server-s3-adapter/releases)

[![npm latest version](https://img.shields.io/npm/v/@parse/s3-files-adapter.svg)](https://www.npmjs.com/package/@parse/s3-files-adapter)

---

The official AWS S3 file storage adapter for Parse Server. See [Parse Server S3 File Adapter Configuration](https://docs.parseplatform.org/parse-server/guide/#configuring-s3adapter) for more details. 

---

- [Installation](#installation)
- [AWS Credentials](#aws-credentials)
- [Deprecated Configuration](#deprecated-configuration)
- [Usage with Parse Server](#usage-with-parse-server)
    - [Parameters](#parameters)
    - [Using a config file](#using-a-config-file)
    - [using environment variables](#using-environment-variables)
    - [passing as an instance](#passing-as-an-instance)
    - [Usage with Digital Ocean Spaces](#usage-with-digital-ocean-spaces)
    - [Adding Metadata and Tags](#adding-metadata-and-tags)

# Installation

`npm install --save @parse/s3-files-adapter`

# AWS Credentials

⚠️ The ability to explicitly pass credentials to this adapter is deprecated and will be removed in a future release.

You may already be compatible with this change.  If you have not explicitly set an `accessKey` and `secretKey` and you have configured the environment variables `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`, then you're all set and this will continue to work as is.

If you explicitly configured the environment variables `S3_ACCESS_KEY`
`S3_SECRET_KEY`

*OR*

If you explicitly configured the `accessKey` and `secretKey` in your adapter configuration, then you'll need to...

For non AWS hosts:

*  Run `aws configure` in a terminal which will step you through configuring credentials for the AWS SDK and CLI

For an AWS host:

*  Ensure that the role that your host is running as has permissions for your s3 bucket

*Then*

*  remove the `accessKey` and `secretKey` from your configuration

If for some reason you really need to be able to set the key and secret explicitly, you can still do it using `s3overrides` as described below and setting `accessKeyId` and `secretAccessKey` in the `s3Overrides` object.

# Deprecated Configuration

Although it is not recommended, AWS credentials can be explicitly configured through an options
object, constructor string arguments or environment variables ([see below](#using-a-config-file)).
This option is provided for backward compatibility and will be removed in the forthcoming version 2.0 of this adapter.

The preferred method is to use the default AWS credentials pattern.  If no AWS credentials are explicitly configured, the AWS SDK will look for credentials in the standard locations used by all AWS SDKs and the AWS CLI. More info can be found in [the docs](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html#config-settings-and-precedence).  For more information on AWS best practices, see [IAM Best Practices User Guide](http://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html).

# Usage with Parse Server

### Parameters

*(This list is still incomplete and in the works, in the meantime find more descriptions in the chapters below.)*

| Parameter | Optional | Default value | Environment variable | Description |
|-----------|----------|---------------|----------------------|-------------|
| `fileAcl` | yes | `undefined` | S3_FILE_ACL | Sets the [Canned ACL](https://docs.aws.amazon.com/AmazonS3/latest/dev/acl-overview.html#canned-acl) of the file when storing it in the S3 bucket. Setting this parameter overrides the file ACL that would otherwise depend on the `directAccess` parameter. Setting the value `'none'` causes any ACL parameter to be removed that would otherwise be set. |

### Using a config file

```
{
  "appId": 'my_app_id',
  "masterKey": 'my_master_key',
  // other options
  "filesAdapter": {
    "module": "@parse/s3-files-adapter",
    "options": {
      "bucket": "my_bucket",
      // optional:
      "region": 'us-east-1', // default value
      "bucketPrefix": '', // default value
      "directAccess": false, // default value
      "fileAcl": null, // default value
      "baseUrl": null, // default value
      "baseUrlDirect": false, // default value
      "signatureVersion": 'v4', // default value
      "globalCacheControl": null, // default value. Or 'public, max-age=86400' for 24 hrs Cache-Control
      "ServerSideEncryption": 'AES256|aws:kms', //AES256 or aws:kms, or if you do not pass this, encryption won't be done
      "validateFilename": null, // Default to parse-server FilesAdapter::validateFilename.
      "generateKey": null // Will default to Parse.FilesController.preserveFileName
    }
  }
}
```
***Note*** By default Parse.FilesController.preserveFileName will prefix all filenames with a random hex code.   You will want to disable that if you enable it here or wish to use S3 "directories".

### using environment variables

Set your environment variables:

```
S3_BUCKET=bucketName
```

the following optional configuration can be set by environment variable too:

```
S3_SIGNATURE_VERSION=v4
```

And update your config / options

```
{
  "appId": 'my_app_id',
  "masterKey": 'my_master_key',
  // other options
  "filesAdapter": "@parse/s3-files-adapter"
}
```


### passing as an instance
```
var S3Adapter = require('@parse/s3-files-adapter');

var s3Adapter = new S3Adapter('accessKey',
                  'secretKey', bucket, {
                    region: 'us-east-1'
                    bucketPrefix: '',
                    directAccess: false,
                    baseUrl: 'http://images.example.com',
                    signatureVersion: 'v4',
                    globalCacheControl: 'public, max-age=86400',  // 24 hrs Cache-Control.
                    validateFilename: (filename) => {
                      if (filename.length > 1024) {
                         return 'Filename too long.';
                       }
                       return null; // Return null on success
                    },
                    generateKey: (filename) => {
                        return `${Date.now()}_${filename}`; // unique prefix for every filename
                    }
                  });

var api = new ParseServer({
	appId: 'my_app',
	masterKey: 'master_key',
	filesAdapter: s3adapter
})
```
**Note:** there are a few ways you can pass arguments:

```
S3Adapter("bucket")
S3Adapter("bucket", options)
S3Adapter("key", "secret", "bucket") -- Deprecated, see notice above
S3Adapter("key", "secret", "bucket", options) -- Deprecated, see notice above
S3Adapter(options) // where options must contain bucket.
S3Adapter(options, s3overrides)
```
If you use the last form, `s3overrides` are the parameters passed to [AWS.S3](http://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/S3.html#constructor-property).

In this form if you set `s3overrides.params`, you must set at least `s3overrides.params.Bucket`

or with an options hash

```
var S3Adapter = require('@parse/s3-files-adapter');

var s3Options = {
  "bucket": "my_bucket",
  // optional:
  "region": 'us-east-1', // default value
  "bucketPrefix": '', // default value
  "directAccess": false, // default value
  "baseUrl": null // default value
  "signatureVersion": 'v4', // default value
  "globalCacheControl": null, // default value. Or 'public, max-age=86400' for 24 hrs Cache-Control
  "validateFilename": () => null, // Anything goes!
  "generateKey": (filename) => filename,  // Ensure Parse.FilesController.preserveFileName is true!
}

var s3Adapter = new S3Adapter(s3Options);

var api = new ParseServer({
  appId: 'my_app',
  masterKey: 'master_key',
  filesAdapter: s3Adapter
})
```
### Usage with Digital Ocean Spaces

```
var S3Adapter = require("@parse/s3-files-adapter");
var AWS = require("aws-sdk");

//Configure Digital Ocean Spaces EndPoint
const spacesEndpoint = new AWS.Endpoint(process.env.SPACES_ENDPOINT);
var s3Options = {
  bucket: process.env.SPACES_BUCKET_NAME,
  baseUrl: process.env.SPACES_BASE_URL,
  region: process.env.SPACES_REGION,
  directAccess: true,
  globalCacheControl: "public, max-age=31536000",
  bucketPrefix: process.env.SPACES_BUCKET_PREFIX,
  s3overrides: {
    accessKeyId: process.env.SPACES_ACCESS_KEY,
    secretAccessKey: process.env.SPACES_SECRET_KEY,
    endpoint: spacesEndpoint
  }
};

var s3Adapter = new S3Adapter(s3Options);

var api = new ParseServer({
  databaseURI: process.env.DATABASE_URI || "mongodb://localhost:27017/dev",
  cloud: process.env.CLOUD_CODE_MAIN || __dirname + "/cloud/main.js",
  appId: process.env.APP_ID || "myAppId",
  masterKey: process.env.MASTER_KEY || "",
  serverURL: process.env.SERVER_URL || "http://localhost:1337/parse",
  logLevel: process.env.LOG_LEVEL || "info",
  allowClientClassCreation: false,
  filesAdapter: s3Adapter,
  }
});
```

### Adding Metadata and Tags

Use the optional options argument to add [Metadata](https://docs.aws.amazon.com/AmazonS3/latest/user-guide/add-object-metadata.html) and/or [Tags](https://docs.aws.amazon.com/AmazonS3/latest/user-guide/add-object-tags.html) to S3 objects

```


const S3Adapter = require('@parse/s3-files-adapter');

const s3Options = {}; // Add correct options
const s3Adapter = new S3Adapter(s3Options);

const filename = 'Fictional_Characters.txt';
const data = 'That\'s All Folks!';
const contentType = 'text/plain';
const tags = {
  createdBy: 'Elmer Fudd',
  owner: 'Popeye'
};
const metadata = {
  source: 'Mickey Mouse'
};
const options = { tags, metadata };
s3Adapter.createFile(filename, data, contentType, options);

```

**Note:** This adapter will **automatically** add the "x-amz-meta-" prefix to the beginning of metadata tags as stated in [S3 Documentation](https://docs.aws.amazon.com/AmazonS3/latest/user-guide/add-object-metadata.html).