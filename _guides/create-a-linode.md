---
# vim: tw=80
title: Creating a Linode
blurb: Starting from nothing and ending with a running server.
layout: guide
---

Creating a Linode requires you to be logged in. Before proceeding, make sure
you have gone through the [Testing with curl guide](/guides/curl-guide), since
you will need an authorization token to proceed.

Before you can create a Linode, you will need to choose a **datacenter**,
a **service plan**, and a **distribution**. The API has endpoints available to
help you do this.

## Selecting a datacenter

A datacenter is a physical location which can run Linodes. To retrieve a list
of available datacenters, you can use the /datacenters API endpoint. To make an
API call against this endpoint over curl, run the following command:

{% highlight bash %}
curl https://api.linode.com/v2/datacenters
{% endhighlight %}

Note that since the datacenter list is public information, you don't need to
send your authorization token (although it will still work if you do). The
above command will return a JSON object like the following:

{% highlight json %}
{
    "datacenters": [
        {
            "label": "Dallas, TX",
            "id": "dctr_2"
        },
        {
            "label": "Fremont, CA",
            "id": "dctr_3"
        },
        {
            "label": "Atlanta, GA",
            "id": "dctr_4"
        },
        {
            "label": "Newark, NJ",
            "id": "dctr_6"
        },
        {
            "label": "London, UK",
            "id": "dctr_7"
        },
        {
            "label": "Singapore, SG",
            "id": "dctr_9"
        },
        {
            "label": "Tokyo, JP",
            "id": "dctr_8"
        }
    ],
    "page": 1,
    "total_pages": 1,
    "total_results": 7
}
{% endhighlight %}

The datacenter list is pretty self-explanatory: there are 7 available
datacenters, and their geographical locations are provided in the `label`
field. The `id` field is a unique ID which you'll use to refer to the
datacenter you want to select. For this example, we'll go with the
"Newark, NJ" datacenter with ID "dctr_6".

## Selecting a service plan

Once you have a datacenter in mind, the next step is to choose a Linode
service plan. A service plan determines the resources available to your new
Linode (such as memory, storage space, and network transfer). Run the
following curl command to retrieve a list of available Linode plans:

{% highlight bash %}
curl https://api.linode.com/v2/services/linode
{% endhighlight %}

The above command will return a JSON object like the following:

{% highlight json %}
{
    "services": [
        {
            "id": "serv_112",
            "label": "Linode 1024",
            "vcpus": 1,
            "mbits_out": 125,
            "disk": 24,
            "hourly_price": 1,
            "service_type": "linode",
            "ram": 1024,
            "monthly_price": 1000,
            "transfer": 2000
        }
        /* and so on */
    ],
    "page": 1,
    "total_pages": 1,
    "total_results": 1
}
{% endhighlight %}

Each entry in the services list represents an available Linode service plan.
You can review the details of each plan, such as the number of Virtual CPUs,
memory, disk space, monthly price, etc. For explanations of each field, see the
[complete service object reference](/reference/#object-service). Once you have
selected the service plan you want to launch, note its ID and continue to
the next step.

## Selecting a distribution

Now you need to choose a Linux distribution to deploy to your new Linode. Just
like selecting a service and a datacenter, issue a call to the API, this time
for a list of available distributions:

{% highlight bash %}
curl https://api.linode.com/v2/distributions
{% endhighlight %}

This will provide you with a list of distributions like the following:

{% highlight json %}
{
    "distributions": [
        {
            "id": "dist_140",
            "label": "Debian 8.1",
            "vendor": "Debian",
            "x64": true,
            "recommended": true,
            "experimental": false,
            "minimum_image_size": 900,
            "created": "2015-04-27T16:26:41"
        }
        /* and so on */
    ],
    "page": 1,
    "total_pages": 1,
    "total_results": 1
}
{% endhighlight %}

For detailed information about each field, see the
[complete distribution reference](/reference/#object-distribution).
For this example, we'll go with Debian 8.1.

## Creating your new Linode

Now that you've selected a datacenter, service plan, and distribution, you're
ready to launch a new Linode! The API calls above were unauthenticated
because they return only publicly visible information. However, launching a
Linode is tied to your account so this call must be authenticated.

You will need to substitute your authorization token in the command below
before running it (replace ```$TOKEN```). If you don't yet have an
authorization token, read through the
[Testing with curl guide](/guides/curl-guide) before proceeding.

You should also set a root password for the new Linode (replace
```$root_pass```).

As you can see, the datacenter, service plan, and Linux distribution are all
specified in the JSON POST data and can be changed as needed to deploy Linodes
to different locations and with different characteristics. Customize the
following curl command and run it when you're ready to deploy:

{% highlight bash %}
curl -X POST https://api.linode.com/v2/linodes \
-d '{"service": "serv_112","datacenter": "dctr_7","source": "dist_140","root_pass": "$root_pass"}' \
-H "Authorization: token $TOKEN" -H "Content-type: application/json"
{% endhighlight %}

If all was successful, you should get a response object detailing the newly
created Linode like the following:

{% highlight json %}
{
    "linode": {
        "id": "lnde_1",
        "total_transfer": 2000,
        "distribution": null,
        "label": "linode1",
        "status": "being_created",
        "group": "",
        "ssh_command": "ssh root@172.28.4.12",
        "lish_command": "ssh -t vagrant@lish-vagrant.linode.com linode1",
        "updated": "2015-12-07T18:03:28",
        "created": "2015-12-07T18:03:28",
        "datacenter": {
            "id": "dctr_7",
            "label": "Vagrant"
        },
        "ip_addresses": {
            "private": {
                "ipv4": [],
                "link-local": "fe80::f03c:91ff:fe96:469d"
            },
            "public": {
                "ipv4": ["172.28.4.12"],
                "ipv6": "2a01:7e00::f03c:91ff:fe96:469d",
                "failover": []
            }
        },
        "alerts": {
            "transfer_in": {
                "threshold": 5,
                "enabled": true
            },
            "transfer_quota": {
                "threshold": 80,
                "enabled": true
            },
            "transfer_out": {
                "threshold": 5,
                "enabled": true
            },
            "io": {
                "threshold": 5000,
                "enabled": true
            },
            "cpu": {
                "threshold": 90,
                "enabled": true
            }
        }
    }
}
{% endhighlight %}

The above response contains lots of details about the newly created Linode,
including IP addresses, alerts, datacenter information, and meta-data like
the date and time it was created. For information on all of the returned fields,
please see the [Linode object reference](/reference#object-linodes). Take note
of the ```id``` field, as you will need it for the next step.

## Booting a Linode

Before you can use your new Linode, you will need to boot it. Take the ```id```
returned by the previous API call and substitute it for ```$linode_id``` in
the following curl command. Also remember to replace ```$TOKEN``` with
your authorization token as in the previous API call.

{% highlight bash %}
curl -X POST https://api.linode.com/v2/linodes/$linode_id/boot \
-H "Authorization: token $TOKEN"
{% endhighlight %}

You should receive a response like the following, detailing the boot job you
just issued:

{% highlight json %}
{
    "action": "linode.boot",
    "id": "ljob_84",
    "label": "System Boot - My Debian 8.1 Disk Profile",
    "entered": "2015-12-07T22:09:48",
    "started": null,
    "finished": null,
    "success": null,
    "message": null,
    "duration": null
}
{% endhighlight %}

Most fields are ```null``` since the job hasn't finished yet. To check on the
progress, you can poll the status of your Linode with the following command
(make sure to replace ```$linode_id``` and ```$TOKEN```):

{% highlight bash %}
curl https://api.linode.com/v2/linodes/$linode_id \
-H "Authorization: token $TOKEN"
{% endhighlight %}

Once the boot job has finished, the Linode's status should change to "running":

{% highlight json %}
{
    "linode": {
        "id": "lnde_1",
        "status": "running",
        "ssh_command": "ssh root@172.28.4.12",
        /* and so on */
    }
}
{% endhighlight %}

# SSH into your Linode

Run the SSH command from the Linode response object and enter the root password
you set in the create Linode step:

{% highlight bash %}
ssh root@$public_ip
{% endhighlight %}

Congratulations! You have now successfully launched a Linode through the API!
