---
title: Rotate DX Certs
---

## Quick reference

At some point you may need to rotate certificates on your Data Exchange nodes. FireFly provides an API to update a node identity, but there are a few prerequisite steps to load a new certificate on the Data Exchange node itself. This guide will walk you through that process. For more information on different types of identities in FireFly, please see the [Reference page on Identities](../reference/identities.md).

> **NOTE**: This guide assumes that you are working in a local development environment that was set up with the [Getting Started Guide](../gettingstarted/index.md). For a production deployment, the exact process to accomplish each step may be different. For example, you may generate your certs with a CA, or in some other manner. But the high level steps remain the same.

The high level steps to the process (described in detail below) are:

- Generate new certs and keys
- Install new certs and keys on each Data Exchange filesystem
- Remove old certs from the `peer-certs` directory
- Restart each Data Exchange process
- `PATCH` the node identity using the FireFly API

## Generate new certs and keys

To generate a new cert, we're going to use a self signed certificate generated by `openssl`. This is how the FireFly CLI generated the original cert that was used when it created your stack.

For the first member of a FireFly stack you run:

```
openssl req -new -x509 -nodes -days 365 -subj /CN=dataexchange_0/O=member_0 -keyout key.pem -out cert.pem
```

For the second member:

```
openssl req -new -x509 -nodes -days 365 -subj /CN=dataexchange_1/O=member_1 -keyout key.pem -out cert.pem
```

> **NOTE**: If you perform these two commands in the same directory, the second one will overwrite the output of the first. It is advisable to run them in separate directories, or copy the cert and key to the Data Exchange file system (the next step below) before generating the next cert / key pair.

## Install the new certs on each Data Exchange File System

For a dev environment created with the FireFly CLI, the certificate and key will be located in the `/data` directory on the Data Exchange node's file system. You can use the `docker cp` command to copy the file to the correct location, then set the file ownership correctly.

```
docker cp cert.pem dev_dataexchange_0:/data/cert.pem
docker exec dev_dataexchange_0 chown root:root /data/cert.pem
```

> **NOTE**: If your environment is not called `dev` you may need to change the beginning of the container name in the Docker commands listed in this guide.

## Remove old certs from the `peer-certs` directory

To clear out the old certs from the first Data Exchange node run:

```
docker exec dev_dataexchange_0 sh -c "rm /data/peer-certs/*.pem"
```

To clear out the old certs from the second Data Exchange node run:

```
docker exec dev_dataexchange_1 sh -c "rm /data/peer-certs/*.pem"
```

## Restart each Data Exchange process

To restart your Data Exchange processes, run:

```
docker restart dev_dataexchange_0
```

```
docker restart dev_dataexchange_1
```

## `PATCH` the node identity using the FireFly API

The final step is to broadcast the new cert for each node, from the FireFly node that will be using that cert. You will need to lookup the UUID for the node identity in order to update it.

### Request

`GET` `http://localhost:5000/api/v1/namespaces/default/identities`

### Response

In the JSON response body, look for the node identity that belongs on this FireFly instance. Here is the node identity from an example stack:

```json
...
    {
        "id": "20da74a2-d4e6-4eaf-8506-e7cd205d8254",
        "did": "did:firefly:node/node_2b9630",
        "type": "node",
        "parent": "41e93d92-d0da-4e5a-9cee-adf33f017a60",
        "namespace": "default",
        "name": "node_2b9630",
        "profile": {
            "cert": "-----BEGIN CERTIFICATE-----\nMIIC1DCCAbwCCQDa9x3wC7wepDANBgkqhkiG9w0BAQsFADAsMRcwFQYDVQQDDA5k\nYXRhZXhjaGFuZ2VfMDERMA8GA1UECgwIbWVtYmVyXzAwHhcNMjMwMjA2MTQwMTEy\nWhcNMjQwMjA2MTQwMTEyWjAsMRcwFQYDVQQDDA5kYXRhZXhjaGFuZ2VfMDERMA8G\nA1UECgwIbWVtYmVyXzAwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQDJ\nSgtJw99V7EynvqxWdJkeiUlOg3y+JtJlhxGC//JLp+4sYCtOMriULNf5ouImxniR\nO2vEd+LNdMuREN4oZdUHtJD4MM7lOFw/0ICNEPJ+oEoUTzOC0OK68sA+OCybeS2L\nmLBu4yvWDkpufR8bxBJfBGarTAFl36ao1Eoogn4m9gmVrX+V5SOKUhyhlHZFkZNb\ne0flwQmDMKg6qAbHf3j8cnrrZp26n68IGjwqySPFIRLFSz28zzMYtyzo4b9cF9NW\nGxusMHsExX5gzlTjNacGx8Tlzwjfolt23D+WHhZX/gekOsFiV78mVjgJanE2ls6D\n5ZlXi5iQSwm8dlmo9RxFAgMBAAEwDQYJKoZIhvcNAQELBQADggEBAAwr4aAvQnXG\nkO3xNO+7NGzbb/Nyck5udiQ3RmlZBEJSUsPCsWd4SBhH7LvgbT9ECuAEjgH+2Ip7\nusd8CROr3sTb9t+7Krk+ljgZirkjq4j/mIRlqHcBJeBtylOz2p0oPsitlI8Yea2D\nQ4/Xru6txUKNK+Yut3G9qvg/vm9TAwkNHSthzb26bI7s6lx9ZSuFbbG6mR+RQ+8A\nU4AX1DVo5QyTwSi1lp0+pKFEgtutmWGYn8oT/ya+OLzj+l7Ul4HE/mEAnvECtA7r\nOC8AEjC5T4gUsLt2IXW9a7lCgovjHjHIySQyqsdYBjkKSn5iw2LRovUWxT1GBvwH\nFkTvCpHhgko=\n-----END CERTIFICATE-----\n",
            "endpoint": "https://dataexchange_0:3001",
            "id": "member_0/node_2b9630"
        },
        "messages": {
            "claim": "95da690b-bb05-4873-9478-942f607f363a",
            "verification": null,
            "update": null
        },
        "created": "2023-02-06T14:02:50.874319382Z",
        "updated": "2023-02-06T14:02:50.874319382Z"
    },
...
```

Copy the UUID from the `id` field, and add that to the `PATCH` request. In this case it is `20da74a2-d4e6-4eaf-8506-e7cd205d8254`.

### Request

Now we will send the new certificate to FireFly. Put the contents of your `cert.pem` file in the `cert` field.

> **NOTE**: Usually the `cert.pem` file will contain line breaks which will not be handled correctly by JSON parsers. Be sure to replace those line breaks with `\n` so that the `cert` field is all on one line as shown below.

`PATCH` `http://localhost:5000/api/v1/namespaces/default/identities/20da74a2-d4e6-4eaf-8506-e7cd205d8254`

```json
{
  "profile": {
    "cert": "-----BEGIN CERTIFICATE-----\nMIIC1DCCAbwCCQDeKjPt3siRHzANBgkqhkiG9w0BAQsFADAsMRcwFQYDVQQDDA5k\nYXRhZXhjaGFuZ2VfMDERMA8GA1UECgwIbWVtYmVyXzAwHhcNMjMwMjA2MTYxNTU3\nWhcNMjQwMjA2MTYxNTU3WjAsMRcwFQYDVQQDDA5kYXRhZXhjaGFuZ2VfMDERMA8G\nA1UECgwIbWVtYmVyXzAwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQCy\nEJaqDskxhkPHmCqj5Mxq+9QX1ec19fulh9Zvp8dLA6bfeg4fdQ9Ha7APG6w/0K8S\nEaXOflSpXb0oKMe42amIqwvQaqTOA97HIe5R2HZxA1RWqXf+AueowWgI4crxr2M0\nZCiXHyiZKpB8nzO+bdO9AKeYnzbhCsO0gq4LPOgpPjYkHPKhabeMVZilZypDVOGk\nLU+ReQoVEZ+P+t0B/9v+5IQ2yyH41n5dh6lKv4mIaC1OBtLc+Pd6DtbRb7pijkgo\n+LyqSdl24RHhSgZcTtMQfoRIVzvMkhF5SiJczOC4R8hmt62jtWadO4D5ZtJ7N37/\noAG/7KJO4HbByVf4xOcDAgMBAAEwDQYJKoZIhvcNAQELBQADggEBAKWbQftV05Fc\niwVtZpyvP2l4BvKXvMOyg4GKcnBSZol7UwCNrjwYSjqgqyuedTSZXHNhGFxQbfAC\n94H25bDhWOfd7JH2D7E6RRe3eD9ouDnrt+de7JulsNsFK23IM4Nz5mRhRMVy/5p5\n9yrsdW+5MXKWgz9569TIjiciCf0JqB7iVPwRrQyz5gqOiPf81PlyaMDeaH9wXtra\n/1ZRipXiGiNroSPFrQjIVLKWdmnhWKWjFXsiijdSV/5E+8dBb3t//kEZ8UWfBrc4\nfYVuZ8SJtm2ZzBmit3HFatDlFTE8PanRf/UDALUp4p6YKJ8NE2T8g/uDE0ee1pnF\nIDsrC1GX7rs=\n-----END CERTIFICATE-----\n",
    "endpoint": "https://dataexchange_0:3001",
    "id": "member_0"
  }
}
```

### Response

```json
{
  "id": "20da74a2-d4e6-4eaf-8506-e7cd205d8254",
  "did": "did:firefly:node/node_2b9630",
  "type": "node",
  "parent": "41e93d92-d0da-4e5a-9cee-adf33f017a60",
  "namespace": "default",
  "name": "node_2b9630",
  "profile": {
    "cert": "-----BEGIN CERTIFICATE-----\nMIIC1DCCAbwCCQDeKjPt3siRHzANBgkqhkiG9w0BAQsFADAsMRcwFQYDVQQDDA5k\nYXRhZXhjaGFuZ2VfMDERMA8GA1UECgwIbWVtYmVyXzAwHhcNMjMwMjA2MTYxNTU3\nWhcNMjQwMjA2MTYxNTU3WjAsMRcwFQYDVQQDDA5kYXRhZXhjaGFuZ2VfMDERMA8G\nA1UECgwIbWVtYmVyXzAwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQCy\nEJaqDskxhkPHmCqj5Mxq+9QX1ec19fulh9Zvp8dLA6bfeg4fdQ9Ha7APG6w/0K8S\nEaXOflSpXb0oKMe42amIqwvQaqTOA97HIe5R2HZxA1RWqXf+AueowWgI4crxr2M0\nZCiXHyiZKpB8nzO+bdO9AKeYnzbhCsO0gq4LPOgpPjYkHPKhabeMVZilZypDVOGk\nLU+ReQoVEZ+P+t0B/9v+5IQ2yyH41n5dh6lKv4mIaC1OBtLc+Pd6DtbRb7pijkgo\n+LyqSdl24RHhSgZcTtMQfoRIVzvMkhF5SiJczOC4R8hmt62jtWadO4D5ZtJ7N37/\noAG/7KJO4HbByVf4xOcDAgMBAAEwDQYJKoZIhvcNAQELBQADggEBAKWbQftV05Fc\niwVtZpyvP2l4BvKXvMOyg4GKcnBSZol7UwCNrjwYSjqgqyuedTSZXHNhGFxQbfAC\n94H25bDhWOfd7JH2D7E6RRe3eD9ouDnrt+de7JulsNsFK23IM4Nz5mRhRMVy/5p5\n9yrsdW+5MXKWgz9569TIjiciCf0JqB7iVPwRrQyz5gqOiPf81PlyaMDeaH9wXtra\n/1ZRipXiGiNroSPFrQjIVLKWdmnhWKWjFXsiijdSV/5E+8dBb3t//kEZ8UWfBrc4\nfYVuZ8SJtm2ZzBmit3HFatDlFTE8PanRf/UDALUp4p6YKJ8NE2T8g/uDE0ee1pnF\nIDsrC1GX7rs=\n-----END CERTIFICATE-----\n",
    "endpoint": "https://dataexchange_0:3001",
    "id": "member_0"
  },
  "messages": {
    "claim": "95da690b-bb05-4873-9478-942f607f363a",
    "verification": null,
    "update": "5782cd7c-7643-4d7f-811b-02765a7aaec5"
  },
  "created": "2023-02-06T14:02:50.874319382Z",
  "updated": "2023-02-06T14:02:50.874319382Z"
}
```

Repeat these requests for the second member/node running on port `5001`. After that you should be back up and running with your new certs, and you should be able to send private messages again.
