# b2_fuse - FUSE for Backblaze B2
 
### Version: 1.3 

#### This is fresh release based upon B2 Command line tool's API for B2. 

#### Warning this software may contain bugs, be careful of using it with important data.
#### Please report bugs, use-case issues and feature requests through the Github issue tracker



### Basic setup:

Requires YAML and FUSE for python to work (this is not the same as "python-fuse" package). 

Install YAML for python as follows: 

```
sudo apt-get install python-yaml
```

Install FUSE for python as follows: 

```
sudo apt-get install python-pip
sudo pip install fusepy
```

Install B2 Comming Line Tool for python as follows: 

```
sudo pip install b2
```

On Python 2.7 use this instead:
```
sudo python -m pip install fusepy
```

An example config ("config.yaml"):

```
accountId: <youraccountid>
applicationKey: <yourapplicationid>
bucketId: <yourbucketid>
```

In order to use the FUSE driver as an interface to the online service B2 run:

```
python b2fuse.py <mountpoint>
```

Full usage info:


```
usage: b2fuse.py [-h] [--enable_hashfiles] [--use_disk]
                 [--account_id ACCOUNT_ID] [--application_key APPLICATION_KEY]
                 [--bucket_id BUCKET_ID] [--memory_limit MEMORY_LIMIT]
                 [--temp_folder TEMP_FOLDER]
                 [--config_filename CONFIG_FILENAME]
                 mountpoint

positional arguments:
  mountpoint            Mountpoint for the B2 bucket

optional arguments:
  -h, --help            show this help message and exit
  --enable_hashfiles
  --use_disk
  --account_id ACCOUNT_ID
                        Account ID for your B2 account (overrides config)
  --application_key APPLICATION_KEY
                        Application key for your account (overrides config)
  --bucket_id BUCKET_ID
                        Bucket ID for the bucket to mount (overrides config)
  --memory_limit MEMORY_LIMIT
                        Memory limit
  --temp_folder TEMP_FOLDER
                        Temporary file folder
  --config_filename CONFIG_FILENAME
                        Config file

```

Usage notes:

* Can be used as a regular filesystem, but should not (high latency)
* Files are cached in memory. If you write or read very large files this may cause issues (you are limited by available ram)
* Neither permissions or timestamps are supported by B2. B2_fuse ignores any requests to set permissions.
* Filesystem contains ".sha1" files, these are undeletable and contain the hash of the file without the postfix. This feature can be disabled by setting variable "enable_hashfiles" to False.
* For optimal performance and throughput, you should store a few large files. Small files suffer from latency issues due to the way B2 API is implemented. Large files will allow you to saturate your internet connection.

### Application specific notes:

####Using RSync with B2 Fuse

Since there is no support for updating file times or permissions in a bucket, rsync must be told to ignore both when synching folders (sync will be based on checksum meaning files have to be downloaded to compare).

```
rsync -avzh --no-perms --no-owner --no-group dir1/ dir2/ 
```

Option "--inplace" may also be useful. RSync creates a temporary file when syncing, this option will make RSync update the file inplace instead (Effectively twice as fast syncing).

####Using unison to synchronize against mounted folder

Again, we ignore permissions as these are not applicable.

```
unison dir1/ dir2/ -auto  -perms 0  -batch
```

#### Using encfs to overlay a locally encrypted filesystem onto the bucket

Install encfs (apt-get install encfs)

```
encfs <bucket_mountpoint> <encrypted_filesystem>
```


### Known issues:

* Concurrent access from multiple client will lead to inconsistent results
* Small files give low read/write performance (due to high latency)



License: MIT license


