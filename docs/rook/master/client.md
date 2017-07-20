---
layout: docs
title: Rook Client
weight: 20
---

# Using Rook

The `rook` client tool can be used to manage your Rook cluster once it is running as well as manage block, file and object storage.  See the sections below for details on how to configure each type of storage.

If you don't yet have a Rook cluster running or for instructions to launch the `rook` client, refer to our [Quickstart Guides](README.md).

## Block Storage

1. Create a new volume image (10MB)

    ```bash
    rook block create --name test --size 10485760
    ```

1. Map the block volume and format it and mount it

    ```bash
    # If running in the rook client container, no need to run privileged
    rook block map --name test --format --mount /tmp/rook-volume

    # If running standalone, you may need to run privileged and take ownership of the folder
    sudo -E ./rook block map --name test --format --mount /tmp/rook-volume
    sudo chown $USER:$USER /tmp/rook-volume
    ```

1. Write and read a file

    ```bash
    echo "Hello Rook!" > /tmp/rook-volume/hello
    cat /tmp/rook-volume/hello
    ```

1. Cleanup

    ```bash
    # If running in the rook client container, no need to run privileged
    rook block unmap --mount /tmp/rook-volume

    # If running standalone, you may need to run privileged
    sudo -E ./rook block unmap --mount /tmp/rook-volume
    ```

## Shared File System
1. Create a shared file system

    ```bash
    rook filesystem create --name testFS
    ```

1. Verify the shared file system was created

   ```bash
   rook filesystem ls
   ```

1. Mount the shared file system from the cluster to your local machine

   ```bash
   # If running in the rook client container, no need to run privileged
   rook filesystem mount --name testFS --path /tmp/rookFS

   # If running standalone, you may need to run privileged and take ownership of the folder
   sudo -E ./rook filesystem mount --name testFS --path /tmp/rookFS
   sudo chown $USER:$USER /tmp/rookFS
   ```

1. Write and read a file to the shared file system

   ```bash
   echo "Hello Rook!" > /tmp/rookFS/hello
   cat /tmp/rookFS/hello
   ```

1. Unmount the shared file system (this does **not** delete the data from the cluster)

   ```bash
   # If running in the rook client container, no need to run privileged
   rook filesystem unmount --path /tmp/rookFS

   # If running standalone, you may need to run privileged
   sudo -E ./rook filesystem unmount --path /tmp/rookFS
   ```

1. Cleanup the shared file system from the cluster (this **does** delete the data from the cluster)

   ```
   rook filesystem delete --name testFS
   ```

## Object Storage
1. Create an object storage instance in the cluster

   ```bash
   rook object create
   ```

1. Create an object storage user

   ```bash
   rook object user create rook-user "my object store user"
   ```

### Consume the Object Storage
Use an S3 compatible client to create a bucket in the object store. If you are running in Kubernetes,
the s3cmd tool is included in the [Rook toolbox](toolbox.md) pod.

1. Get the connection information for accessing object storage

   ```bash
   eval $(rook object connection rook-user --format env-var)
   ```

1. Create a bucket in the object store

   ```bash
   s3cmd mb --no-ssl --host=${AWS_ENDPOINT} --host-bucket=  s3://rookbucket
   ```

1. List buckets in the object store

   ```bash
   rook object bucket list
   ```

1. Upload a file to the newly created bucket

   ```bash
   echo "Hello Rook!" > /tmp/rookObj
   s3cmd put /tmp/rookObj --no-ssl --host=${AWS_ENDPOINT} --host-bucket=  s3://rookbucket
   ```

1. Download and verify the file from the bucket

   ```bash
   s3cmd get s3://rookbucket/rookObj /tmp/rookObj-download --no-ssl --host=${AWS_ENDPOINT} --host-bucket=
   cat /tmp/rookObj-download
   ```
