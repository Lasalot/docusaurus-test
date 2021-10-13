.. _ip-reputation:

IP reputation
#############

BitNinja is a disruptive technology so there are some concepts that are
important to understand in order to comprehend the way BitNinja works.

IP reputation is a very effective way of securing your server. Itâs
a database with information about various IPs in the world. BitNinja clients
use IP reputation information automatically on their servers to make security
decisions and to find out more about an IP address.

Every server with BitNinja can detect and defend a wide range of attacks.
The server can send gathered incident information to our central database. Based
on the type, timing, and amount of incidents an IP has in the database, it is
categorized into one of the following lists:

Not listed
==========

If there is no information about an IP address, or based on the latest
behavior the IP is not listed.

User Greylist
=============

In traditional IP reputation terminology, we differentiate black- and whitelists.
An IP can be trusted (whitelisted) or absolutely denied (blacklisted). This concept is
very inflexible and this is the cause of the bad reputation that IP reputation lists
have. If an IP is false-positively blacklisted, users will get angry at you
because they can't access the system they want to and they will be frustrated as
they are unable to do anything about it.

That's how the concept of greylisting was born. We wanted to represent a
list of IPs we think may be malicious but we are not completely sure of it yet.

The greylist contains suspicious IPs that the BitNinja client handles with special
care. BitNinja has different CAPTCHA modules for different protocols. The duty of
a CAPTCHA module is as follows:

#. Decide if the user is human or not
#. Inform the user about the fact that his/her IP has been greylisted
#. Provide a safe way for the user to delist his/her IP
#. Save any requests made by non-human parties, growing the knowledgebase about the IP and the sin list.
#. Honeypotting by pretending to be a vulnerable system so bots will try to connect

.. note:: Traffic from an IP can be malicious and innocent at the same time. Think of
    an infected computer. The requests the virus will make are malicious but the
    requests the user makes by using his/her browser are innocent. If we completely
    block the traffic, the user will not know about the infection. Greylisting
    solves this by informing the user of the greylisting and helps the innocent
    users resolve their IPs.

If there are suspicious incidents about an IP address, the IP can be
greylisted by some users. If an IP is user-greylisted, it means it
is only greylisted by some users, not all BitNinja users. When we have
enough information about an IP that is sending malicious requests, we move
it to the global greylist. If an IP is globally greylisted, it is greylisted
by all BitNinja servers.

You can check if an IP is listed by BitNinja using the web
front end https://admin.bitninja.io or using our CLI front end that is called bitninjacli.

You can check, add, and delete IPs from the greylist using both tools.

    .. sourcecode:: bash

        bitninjacli --greylist --check 1.2.3.4


To see if an IP is on any of these lists, you can use the following command:


    .. sourcecode:: bash

        for set in $(ipset -L -n); do echo $set; ipset test $set 1.2.3.4; done


.. note:: If one of your servers catches an attack and the IP is not yet
    greylisted, the IP will be user greylisted and this information will be distributed
    across all of your servers instantly.


Global greylist
===============

If there is enough evidence that an IP is suspicious, we move it to the global
greylist and distribute this information to every BitNinja protected server.

.. note::
    If there are no logs about a greylisted IP address it will be delisted:
     * Static IP addresses: 3 months without logs
     * Dynamic IP address: 7 days without logs


Global blacklists
=================
When an IP is globally greylisted and is still sending malicious requests, we identify
it as dangerous. Such IPs are moved to the global blacklist we maintain.
BitNinja clients will drop packets for IPs on the global blacklist.
The false-positive rate of the global blacklist is very low, as there are
many steps before we decide to blacklist an IP. Blacklisted IPs are moved back to
the greylist from time-to-time to check if the traffic is still malicious or
the system has been disinfected.



User-level black- and whitelists
================================
You can use BitNinja to maintain blacklists and whitelists across your servers.
The BitNinja Dashboard or our command-line interface, BitNinja CLI, can be used
to add/remove/check IPs against a userâs blacklist and whitelist.
If you blacklist an IP, all of your servers will DROP any packets from that IP.
You can add CIDRs to your blacklist or your whitelist via BitNinja Dashboard.
When you whitelist an IP it guarantees BitNinja won't interfere with that IP.

You can set an expiration date for each IP address or IP range or country after which they are
removed from the account-level blacklist or whitelist. 

It is also possible to specify a server on which you whish to blacklist or whitelist the given IP address.
This way the IP address will be blacklisted/whitelisted *only* on the specified server. 
You can also specify a server in case of the country blacklist or whitelist. 

.. image:: /imgs/modules/IPreputation_TTL_server.png
    :target: ../_images/IPreputation_TTL_server.png

.. note:: Whitelisting an IP will only bypass BitNinja iptables chains. There
    can be other iptables rules blocking the traffic. Whitelisting is not
    equal to universal ACCEPT, only to bypass BitNinja.

.. note:: Whitelisted addresses are processed prior to blacklisted ones starting from BitNinja version 1.16.16.

Essential list
==============
Essential list provides protection against the most dangerous IPs. These IPs are often used by the most
agressive hackers all around the world. When an IP generates more than 5000 malicious requests,
BitNinja places it on this list. The essential list is part of our basic IP reputation package,
so it's available for every BitNinja user.

Country blocking
================
You can add whole countries to your blacklist or whitelist. The country blocking (or whitelisting) feature of BitNinja
uses publicly available zone files to add the selected countries' IP ranges to the user blacklist or user whitelist.
The zone files can be found at http://www.ipdeny.com/ipblocks.

You can add countries on the Dashboard in the Firewall > Blacklist > Country menu.

.. note::
    Here you can specify if you want to blacklist a country only on one server or if you want to block a country until a specific date. 

.. image:: /imgs/concepts/Coutry_block_ttl_server.png
     :target: ../_images/Coutry_block_ttl_server.png

If you have ipsetv6 or ipsetv4 and add a country or remove it to the blacklist or whitelist,
the ipsets will be reloaded on your server(s).

If you don't have ipset, BitNinja uses iptables rules and simulated ipset. When you add or remove a country,
the iptables rules will be reloaded.

For now, if you have more than one server and add a country to your blacklist (or whitelist),
the selected IP ranges will be blacklisted (or whitelisted) on all your servers.

.. note:: Blocking an entire country might interfere with your extensions and plugins.

Searching in the BitNinja IP reputation database
================================================

You can search the IP history using the BitNinja Dashboard at
https://admin.bitninja.io.

.. image:: /imgs/concepts/ip_search.png

The search field is on the top of the page. You can type an IP and 
choose the IP option to search in the BitNinja IP reputation database.


Railgun server configuration
============================

Many of our customers who use Cloudflare are not able to see the proper visitor IPs in the logs, only
the railgun server IPs. This is due to the fact that the requests are not directly coming from
Cloudflare so mod_cloudflare will not restore the IPs of the visitor. That is why we would like to ask
you to configure mod_cloudflare the following way:

#. Open your Apache configuration ( if you do not know where to find it, ask your hosting provider for assitance )
#. At the very end, add:

    .. sourcecode:: bash

        CloudFlareRemoteIPTrustedProxy railgun_address
#. If you have more servers to add, it should look like this:

    .. sourcecode:: bash

        CloudFlareRemoteIPTrustedProxy xxx.xxx.xxx.xxx xxx.xxx.xxx.xxx

Following this configuration, you will be able to see the adequate logs on our Dashboard.

If you need any further assistance, do not hesitate to contact our support team: info@bitninja.io


Crawlers/Good Bots
==================

Itâs possible that BitNinja identifies internet crawlers as malicious bots due to their configuration
and may greylist them due to security reasons.

Our management decided it is the responsibility of our customers (the hosting providers) and the end-user to agree on what monitoring solutions, crawlers and bots do they allow.
As it is always a risk of some level to whitelist IP, let alone on a global level. We do not want to bear the responsibility of deciding what bots and crawlers we do whitelist. 
As mentioned above the decision of what bots and crawlers to allow is the hosting providers' and the end-users' therefore please contact the hosting providers and or the end-users with the whitelisting request.

If that is not possible we can disable the abuse email sending from the IP addresses used by the service. 
To do that please contact us at info@bitninja.io. 

IPv6 limitation
===============

For the time being BitNinja does not support IPv6. If you are not using IPv6 it's recommended to disable it.
If BitNinja caught an IPv4 address, the attacker can still attack through IPv6.
To disable IPv6 edit the file: :file:`/etc/sysctl.conf`

    .. sourcecode:: bash

        $ sudo gedit /etc/sysctl.conf

And fill in the following lines at the end of that file

    .. sourcecode:: bash

        # IPv6 disabled
        net.ipv6.conf.all.disable_ipv6 = 1
        net.ipv6.conf.default.disable_ipv6 = 1
        net.ipv6.conf.lo.disable_ipv6 = 1

Save the file and close it. Restart :code:`sysctl` with

    .. sourcecode:: bash

        $ sudo sysctl -p

BitNinja will support IPv6 in the future.