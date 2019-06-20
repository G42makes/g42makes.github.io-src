Title: AWS2DNS: Resolving DNS against an API
Date: 2019-06-19 10:00
Category: Python
Tags: python, AWS, DNS
Authors: Jared Young
Summary: Resolving your instance IDs via custom DNS.

## Introduction

Recently, two situations came together:

1. I discovered the `CanonicalDomains` option for my `~/.ssh/config` file
2. I was working with some tools in AWS that only presented EC2 Instance IDs when working on the console.

Now, I was just testing out a feature and it was going to live in automation later, but a (very) few times during my research I needed to get into an instance that was not returning the results I wanted and debug. Of course, I just looked them up, it takes only a moment, in the console. But, I realized it would be interesting to see if I could take advantage of the SSH config to allow me to access them by only the Instance ID on the command line.

This solution is not a good one for my need, I did this to see if I could. It's not polished, but it could be. That said, let's look at what I did.

## DNS Server

To start, I needed to write/modify a DNS server. My language of choice for stuff like this is Python, but I will admit I am no expert.

Initially, I was able to find a few examples of DNS servers that had been written from scratch, but they required me to parse the requests myself and forwarding those results I didn't generate was going to be fun too.

While researching, I found the [DNSLib](https://bitbucket.org/paulc/dnslib/src/default/) library for python. While it's marketed as providing DNS query assembly and processing, it also includes DNS servers that can be easily modified. In my case they had one that was designed to process certain requests internally and forward the rest to another server, the [intercept.py](https://bitbucket.org/paulc/dnslib/src/default/dnslib/intercept.py) example. This is perfect for my needs.

## Modifications

After spending some time understanding the flow, adding in some better handling for [keyboard interupts](https://github.com/G42makes/AWS2DNS/blob/46daa8225a7a2c4b479f7cfd3ce861a48ba420b1/aws2dns.py#L153-L161)(as I was testing on the command line), and learning how to create response records in a format that worked for the library, I was ready to add my features.

I stripped out the code related to passed in intercept domains and skipped(NXDOMAIN) domains. A format was decided(see below) to recognize the DNS requests and some [ugly code](https://github.com/G42makes/AWS2DNS/blob/46daa8225a7a2c4b479f7cfd3ce861a48ba420b1/aws2dns.py#L36-L72) was written to process the domain name into the parts I needed.

At this point, after much fun in testing, I could lookup a nice long DNS name and get the IP of that host from AWS directly.

## Private Domain Name

In order to recognize my requests(and forward all others), I needed a top level, and decided to include a second level, domain. In this case I chose `dns.aws` as I knew there would be no overlap on those queries. Additionally, I included 3 other sub levels for lookups before including the ID to actually query. This enabled me to request either public or private interface information, and either the IP or a DNS record of the interface.

Let's break down an example domain name to see what each part has:
```
i-01234567890.private.ip.ec2.dns.aws
```
We will work backwards here, as is tradition in the DNS.

1. `aws.dns`: our indicator that we should intercept and resolve this inside the server.
2. `ec2`: which resource type we are looking for. Right now this is the only supported option, but adding something like `alb` would be easy.
3. `ip`: what type of data we want back, `ip` gives a DNS A record, `cname` gives a DNS CNAME to the DNS entry.
4. `private`: which interface to return information for, `public` or `private`
5. `i-01234567890`: this is the Instance ID of the host we want to query. Nothing special here.

As you can see, there is room to extend this if you have a good need for it, right now it can just request the public or private interface information of an EC2 instance, but there is no reason we could not lookup the ip information for an RDS instance, or the address of a load balancer.

### Examples

1. `i-01234567890.private.ip.ec2.dns.aws`: this will return the private IP of the host with the id specified. The result is an A record(specified by the .ip. section).
2. `i-01234567890.private.cname.ec2.dns.aws`: Similar to the above, this returns the Private DNS entry for the host. The result is a CNAME, allowing resolution when on a system that can resolve the .internal addresses AWS uses.
3. `i-01234567890.public.ip.ec2.dns.aws`: Similar to the first entry, it returns an A record with the public IP of the host.
4. `i-01234567890.public.cname.ec2.dns.aws`: Finally you can get the public DNS entry via CNAME for the host.

With the public IP/DNS requests, if they are not configured you will get back a NXDOMAIN result(non-existing domain).

## SSH

The way I designed this to work for me, was to enable quick SSH into a host by instance ID, `ssh i-01234567890`. This required some reasonable changes to the `~/.ssh/config` file on my system, which are documented/exampled in the [sshconfig](https://github.com/G42makes/AWS2DNS/blob/master/sshconfig) file of the repo.

### DNS Configuration

SSH needed to talk to this DNS server, and since it can forward to your normal SSH server, I just set the DNS setting(on my Mac I used a new network Location) to localhost. All queries to `dns.aws` are resolved locally, the rest are just fowarded. Now my SSH config works perfectly.

##  Going Forward

For me, I mostly did this to see if I could. I had basically already gotten past the point where I needed this in my work. I decided to write it anyway, and post it online just to see if anyone wants to take it further. Pull Requests will happily be reviewed and ownership can be passed on if you really want to run with this.

For now, at least, I will not be making any real changes.
