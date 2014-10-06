# Puppet Enterprise 3-split stack

## Overview

This is a Puppet Enterprise 3.3 standard "split-install" stack, consisting of
three base servers:

1. Primary master/CA
2. PuppetDB/PostgreSQL
3. Console

There is also support for additional "compile masters"

| Hostname       | Description                      |
| -------------- | -------------------------------- |
| puppetmaster1  | Primary master and CA            |
| puppetdb1      | PuppetDB and PostgreSQL          |
| puppetconsole1 | Enterprise Console               |
| puppetmaster2  | Additional "compile-only" master |


## Installation Procedure

### 1. Install the Primary Master/CA

Use the provided answer file to install Puppet Enterprise.  For example:

```shell
puppet-enterprise-installer -A ../../answers/ca.txt
```

Edit `/etc/puppetlabs/puppet/autosign.conf` and add the following entries:

* The full certificate name of the PuppetDB server (e.g. puppetdb01.vagrant.vm)
* The full certificate name of the Console server (e.g. puppetconsole01.vagrant.vm)

### 2. Install the PuppetDB/PostgreSQL server

Use the provided answer file to install Puppet Enterprise.  For example:

```shell
puppet-enterprise-installer -A ../../answers/puppetdb.txt
```

#### Using alternate names (CNAMEs)

If you want to use a generic CNAME (non-specific address) to communicate with
PuppetDB services, you'll have to perform the following procedure.  During the
master's installation, for instance, if you configured it to use a generic
CNAME to communicate with the PuppetDB server.

The reason: the PE installer will use a single certificate name (defaulting to
the fqdn) for the PuppetDB role.  It does not ask if you would like alternate
names.  For security, the Puppet CA will not autosign certificates that contain
alternate names.  This would prevent a certificate that contained them from
being autosigned during installation, which would cause an installation failure.

To use alternative names, you must do this post-install.


Edit `/etc/puppetlabs/puppet/puppet.conf` and add the following under the
`[main]` section:

```
dns_alt_names = puppetdb01,puppetdb.vagrant.vm,puppetdb
```

Basically, you want to add the generic CNAMEs as alternate certificate names for
the PuppetDB server.  You cannot do this during installation.

On the CA server, clean the existing certificate for the PuppetDB server.  For
example:

```shell
puppet cert clean puppetdb01.vagrant.vm
```

Remove the existing SSL data on the PuppetDB server and run the Puppet agent
to create a new CSR (certificate signing request) with the altnames.

```shell
rm -rf /etc/puppetlabs/puppet/ssl
puppet agent -t
```

Now go sign the certificate on the CA, allowing the alt names:

```shell
puppet cert sign --allow-dns-alt-names puppetdb01.vagrant.vm
```

Then run the agent on the PuppetDB server again to retrieve the signed certificate:

```shell
puppet agent -t
```

Re-generate the internal PuppetDB certificates on the PuppetDB server:

```shell
/opt/puppet/sbin/puppetdb ssl-setup -f
```

Restart the `pe-puppetdb` service:

```shell
service pe-puppetdb restart
```

### Install the Console server

Use the provided answer file to install Puppet Enterprise.  For example:

```shell
puppet-enterprise-installer -A ../../answers/console.txt
```

