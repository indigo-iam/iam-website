---
title: "Deployment with packages"
linkTitle: "Deployment with packages"
weight: 5
---

IAM can be deployed from packages on the RHEL 8 and 9 platforms.
Packages and repo files are hosted on the [INDIGO IAM package stable repository](https://repo.cloud.cnaf.infn.it/service/rest/repository/browse/indigo-iam-rpm-stable/).

{{% alert title="Warning" color="warning" %}}
We no longer maintain packages for the CENTOS 7 and Ubuntu platform.
{{% /alert %}}

## AlmaLinux 9

<!-- Since IAM v1.14.0

1. Install the INDIGO IAM release key:

  ```shell
  $ sudo rpm --import https://repo.cloud.cnaf.infn.it/repository/indigo-iam/CNAFSD.asc
  ```
-->

1. Install the repo files:

  ```shell
  $ sudo yum-config-manager --add-repo https://indigo-iam.github.io/repo/repofiles/rhel/indigoiam-stable-el9.repo
  ```

2. Install packages:

  ```shell
  $ sudo yum install -y iam-login-service
  ```

## IAM service configuration

The IAM service is configured via a configuration file named `iam-login-service`
which holds the settings for the environment variables that drive its
configuration (as described in the [configuration reference
section]({{< ref "/docs/reference/configuration" >}})).

The file is located in the following path:

```
/etc/sysconfig/iam-login-service
```
## Run the service

The IAM login service is managed by `systemd`.

To enable the service use the following command:

```shell
$ sudo systemctl enable iam-login-service
```

To start the service use the following command:

```shell
$ sudo systemctl start iam-login-service
```

To access the service logs, use the following command:

```shell
$ sudo journalctl -fu iam-login-service
```

### Deployment Tips
In headless servers, running `haveged` daemon is recommended to generate more entropy.
Before running the IAM login service, check the available entropy with:

```shell
$ cat /proc/sys/kernel/random/entropy_avail
```

If the obtained value is less than 1000, then `haveged` daemon is mandatory.

Install EPEL repository:

```shell
$ sudo yum install -y epel-release
```

Install Haveged:

```shell
$ sudo yum install -y haveged
```

Enable and run the `haveged` daemon with:

```shell
$ sudo systemctl enable haveged
$ sudo systemctl start haveged
```

[iam-pkg-repo]: https://indigo-iam.github.io/repo
