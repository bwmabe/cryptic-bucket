# Cryptic Bucket
Buckets of unknown knowledge! Wow!

## About

Just a fancy name for a shell script wrapper around creating and mounting LUKS containers.

I can never remember the process, so I thought I'd make a shell script that remembers it for me and publish it on the internet in case it helps someone else out.

### Some Details
* Keys
	- Created by read 2KB of random data from `/dev/urandom` and then base64 encoding it
	- The encoding is done to remove any and all non-text characters 
	- OpenSSL with `aes-256-cbc pbkdf2` encryption is used to encrypt the keys
	- Encryption only occurs *after* it is used to lock the container
	- Decryption only occurs immediately *before* it is used to unlock the container
	- **Keys are only ever decrypted in memory**
* Containers
	- Locked with keys generated from random data
	- Managed with LUKS and cryptsetup
	- Currently hardcoded to use the `ext4` filesystem for implementation simplicity

## Usage

Make a bucket:
```bash
cryptic-bucket create BUCKET_NAME 
```
**Note to self:** *Probably put something about choosing filesystem type here...*

Mount a bucket:
```bash
cryptic-bucket mount BUCKET_NAME MOUNTPOINT
```

Unmount a bucket:
```bash
cryptic-bucket unmount BUCKET_NAME
```

List your buckets:
```bash
cryptic-bucket list 
```

## Process

### Creation

It uses `dd` and `/dev/urandom` to generate some random data to use as a key; passes it to `gpg` and then encrypts the random data with a passphrase chosen by `gpg`. 

```
dd + /dev/urandom -> keyfile -+-> gpg -> encrpted_keyfile
                              |
                              + Bucket gets locked here!
```

*how do you even lock a bucket? gah, this is computers, it doesn't have to make sense*

### Mounting

You choose a bucket and where to mount it; then the script mounts the bucket. Mathmagical!

```
encrypted_keyfile -> gpg -> keyfile -> unlocked bucket, ready to be mounted!
```

### References

The main logic of the program is adapted and heavily modified from the algorithm in [this article that came up when I Googled how to make an encrypted container that I have to Google every time.](https://willhaley.com/blog/encrypted-file-container-disk-image-in-linux/)
