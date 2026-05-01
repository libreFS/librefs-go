# librefs-go — libreFS S3 Go SDK

[![Apache V2 License](https://img.shields.io/badge/license-Apache%20V2-blue.svg)](LICENSE)

Go client SDK for [libreFS](https://github.com/libreFS/libreFS) and any Amazon S3-compatible object storage. Forked from `minio/minio-go`.

For a complete list of APIs and examples, see the [godoc documentation](https://pkg.go.dev/github.com/libreFS/librefs-go/v7).

These examples assume a working [Go development environment](https://golang.org/doc/install) and the [`lc` CLI tool](https://github.com/libreFS/librefs-cli).

Install
-------

From your project directory:

```sh
go get github.com/libreFS/librefs-go/v7
```

Initialize a Client Object
--------------------------

| Parameter        | Description                                                |
|------------------|------------------------------------------------------------|
| `endpoint`       | URL to object storage service.                             |
| `minio.Options`  | All the options such as credentials, custom transport etc. |

```go
package main

import (
	"log"

	"github.com/libreFS/librefs-go/v7"
	"github.com/libreFS/librefs-go/v7/pkg/credentials"
)

func main() {
	endpoint := "localhost:9000"
	accessKeyID := "YOUR-ACCESSKEYID"
	secretAccessKey := "YOUR-SECRETKEY"
	useSSL := false

	// Initialize S3 client object.
	s3Client, err := minio.New(endpoint, &minio.Options{
		Creds:  credentials.NewStaticV4(accessKeyID, secretAccessKey, ""),
		Secure: useSSL,
	})
	if err != nil {
		log.Fatalln(err)
	}

	log.Printf("%#v\n", s3Client) // s3Client is now set up
}
```

Example - File Uploader
-----------------------

This sample code connects to a libreFS server, creates a bucket, and uploads a file.

### FileUploader.go

```go
package main

import (
	"context"
	"log"

	"github.com/libreFS/librefs-go/v7"
	"github.com/libreFS/librefs-go/v7/pkg/credentials"
)

func main() {
	ctx := context.Background()
	endpoint := "localhost:9000"
	accessKeyID := "YOUR-ACCESSKEYID"
	secretAccessKey := "YOUR-SECRETKEY"
	useSSL := false

	// Initialize S3 client object.
	s3Client, err := minio.New(endpoint, &minio.Options{
		Creds:  credentials.NewStaticV4(accessKeyID, secretAccessKey, ""),
		Secure: useSSL,
	})
	if err != nil {
		log.Fatalln(err)
	}

	bucketName := "testbucket"
	location := "us-east-1"

	err = s3Client.MakeBucket(ctx, bucketName, minio.MakeBucketOptions{Region: location})
	if err != nil {
		exists, errBucketExists := s3Client.BucketExists(ctx, bucketName)
		if errBucketExists == nil && exists {
			log.Printf("We already own %s\n", bucketName)
		} else {
			log.Fatalln(err)
		}
	} else {
		log.Printf("Successfully created %s\n", bucketName)
	}

	objectName := "testdata"
	filePath := "/tmp/testdata"
	contentType := "application/octet-stream"

	info, err := s3Client.FPutObject(ctx, bucketName, objectName, filePath, minio.PutObjectOptions{ContentType: contentType})
	if err != nil {
		log.Fatalln(err)
	}

	log.Printf("Successfully uploaded %s of size %d\n", objectName, info.Size)
}
```

**1. Create a test file:**

```sh
dd if=/dev/urandom of=/tmp/testdata bs=2048 count=10
```

**2. Run FileUploader:**

```sh
go mod init example/FileUploader
go get github.com/libreFS/librefs-go/v7
go get github.com/libreFS/librefs-go/v7/pkg/credentials
go run FileUploader.go
```

**3. Verify with `lc ls`:**

```sh
lc ls myserver/testbucket
```

API Reference
-------------

Full API reference: [pkg.go.dev/github.com/libreFS/librefs-go/v7](https://pkg.go.dev/github.com/libreFS/librefs-go/v7)

### Bucket Operations

- [`MakeBucket`](https://pkg.go.dev/github.com/libreFS/librefs-go/v7#Client.MakeBucket)
- [`ListBuckets`](https://pkg.go.dev/github.com/libreFS/librefs-go/v7#Client.ListBuckets)
- [`BucketExists`](https://pkg.go.dev/github.com/libreFS/librefs-go/v7#Client.BucketExists)
- [`RemoveBucket`](https://pkg.go.dev/github.com/libreFS/librefs-go/v7#Client.RemoveBucket)
- [`ListObjects`](https://pkg.go.dev/github.com/libreFS/librefs-go/v7#Client.ListObjects)
- [`ListIncompleteUploads`](https://pkg.go.dev/github.com/libreFS/librefs-go/v7#Client.ListIncompleteUploads)

### Bucket Policy Operations

- [`SetBucketPolicy`](https://pkg.go.dev/github.com/libreFS/librefs-go/v7#Client.SetBucketPolicy)
- [`GetBucketPolicy`](https://pkg.go.dev/github.com/libreFS/librefs-go/v7#Client.GetBucketPolicy)

### Bucket Notification Operations

- [`SetBucketNotification`](https://pkg.go.dev/github.com/libreFS/librefs-go/v7#Client.SetBucketNotification)
- [`GetBucketNotification`](https://pkg.go.dev/github.com/libreFS/librefs-go/v7#Client.GetBucketNotification)
- [`RemoveAllBucketNotification`](https://pkg.go.dev/github.com/libreFS/librefs-go/v7#Client.RemoveAllBucketNotification)
- [`ListenBucketNotification`](https://pkg.go.dev/github.com/libreFS/librefs-go/v7#Client.ListenBucketNotification)
- [`ListenNotification`](https://pkg.go.dev/github.com/libreFS/librefs-go/v7#Client.ListenNotification)

### File Object Operations

- [`FPutObject`](https://pkg.go.dev/github.com/libreFS/librefs-go/v7#Client.FPutObject)
- [`FGetObject`](https://pkg.go.dev/github.com/libreFS/librefs-go/v7#Client.FGetObject)

### Object Operations

- [`GetObject`](https://pkg.go.dev/github.com/libreFS/librefs-go/v7#Client.GetObject)
- [`PutObject`](https://pkg.go.dev/github.com/libreFS/librefs-go/v7#Client.PutObject)
- [`StatObject`](https://pkg.go.dev/github.com/libreFS/librefs-go/v7#Client.StatObject)
- [`CopyObject`](https://pkg.go.dev/github.com/libreFS/librefs-go/v7#Client.CopyObject)
- [`RemoveObject`](https://pkg.go.dev/github.com/libreFS/librefs-go/v7#Client.RemoveObject)
- [`RemoveObjects`](https://pkg.go.dev/github.com/libreFS/librefs-go/v7#Client.RemoveObjects)
- [`SelectObjectContent`](https://pkg.go.dev/github.com/libreFS/librefs-go/v7#Client.SelectObjectContent)

### Presigned Operations

- [`PresignedGetObject`](https://pkg.go.dev/github.com/libreFS/librefs-go/v7#Client.PresignedGetObject)
- [`PresignedPutObject`](https://pkg.go.dev/github.com/libreFS/librefs-go/v7#Client.PresignedPutObject)
- [`PresignedHeadObject`](https://pkg.go.dev/github.com/libreFS/librefs-go/v7#Client.PresignedHeadObject)
- [`PresignedPostPolicy`](https://pkg.go.dev/github.com/libreFS/librefs-go/v7#Client.PresignedPostPolicy)

### Client Settings

- [`SetAppInfo`](https://pkg.go.dev/github.com/libreFS/librefs-go/v7#Client.SetAppInfo)
- [`TraceOn`](https://pkg.go.dev/github.com/libreFS/librefs-go/v7#Client.TraceOn)
- [`TraceOff`](https://pkg.go.dev/github.com/libreFS/librefs-go/v7#Client.TraceOff)

Full Examples
-------------

See the [examples/](examples/) directory.

Contribute
----------

See [CONTRIBUTING.md](CONTRIBUTING.md).

License
-------

Distributed under the [Apache License, Version 2.0](https://www.apache.org/licenses/LICENSE-2.0) — see [LICENSE](LICENSE) and [NOTICE](NOTICE).
