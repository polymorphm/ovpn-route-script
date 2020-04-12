README
======

``ovpn-route-script`` is an one-file utility to handling routes on ``OpenVPN``
client-side for solving most common problems with adding/removing routes. The
solved problems are:

* More **accurate** working with route table then stock ``OpenVPN`` client
  does: now we get minimum for side-effects of route table manipulations.
* Incoming routes from ``OpenVPN`` server to ``OpenVPN`` client (passed by
  ``push`` directive) now can be **filtered** on client side. The utility lets
  set **white list** and **black list** of incoming routes. If any incoming
  route has too big network size it will be **automaticly splitted** to
  several routes appropriate network sizes before filtering.
* White and black route-filter-lists can be set with multi-layer manner,
  therefore we are able to set a rule exclusion list, a exclusion of
  exclusion list, a exclusion of exclusion of exclusion list, etc.

Version
-------

Development version.

Example Of Using
--------------------

Example 1: Connection To Private VPN Without Expected Public Internet Gateway
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Example of simple ensuring and filtering routes incoming from VPN server::

   route_up_down_cmd='/usr/bin/python
           /home/regular-user/my_projects/ovpn-route-script.proj/ovpn-route-script/ovpn-route-script
           254
           30
           10.0.0.0/8
           172.16.0.0/12
           192.168.0.0/16
           169.254.0.0/16
           ^192.168.88.0/24
           ^192.168.50.0/24
           ";"'

   openvpn --config private-network.conf  \
           --route-noexec \
           --route-up "$route_up_down_cmd" \
           --route-pre-down "$route_up_down_cmd" \
           --script-security 2

In this example rules do next:

* VPN routes gets ``metric=30 table=main(254)`` for our routing table.
* Access through VPN is allowed to networks ``10.0.0.0/8``, ``172.16.0.0/12``,
  ``192.168.0.0/16``, ``169.254.0.0/16`` but no more. Other networks are
  forbidden (through this VPN), IPv6 networks are forbidden as well.
* But networks ``192.168.88.0/24``, ``192.168.50.0/24`` are forbidden
  (through this VPN) in spite of allowing access to
  ``192.168.0.0/16``.  This has more priority then the rule above.

Example 2: Connection To VPN With Public Internet Gateway
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

More advanced example, we are turning routes here to getting public Internet
access (IPv4 and IPv6) through VPN, but with filtering private networks::

   route_up_down_cmd='/usr/bin/python
           /home/regular-user/my_projects/ovpn-route-script.proj/ovpn-route-script/ovpn-route-script
           -r0.0.0.0/0
           -r2000::/3
           254
           30
           0.0.0.0/0
           ^10.0.0.0/8
           ^172.16.0.0/12
           ^192.168.0.0/16
           ^169.254.0.0/16
           ^224.0.0.0/4
           10.42.0.0/16
           2000::/3
           ";"'

   openvpn --config public-internet-access.conf  \
           --route-noexec \
           --route-up "$route_up_down_cmd" \
           --route-pre-down "$route_up_down_cmd" \
           --script-security 2

In this example rules do next:

* We add additional routes (options: ``-r0.0.0.0/0`` and ``-r2000::/3``)
  besides to routes incoming from VPN server by ``push``.
* VPN routes gets ``metric=30 table=main(254)`` for our routing table.
* All IPv4 networks are allowed to access through this VPN.
* But networks ``10.0.0.0/8``, ``172.16.0.0/12``, ``192.168.0.0/16``,
  ``169.254.0.0/16``, ``224.0.0.0/4`` are forbidden through this VPN.
* But network ``10.42.0.0/16`` are allowed though this VPN.
* The network ``2000::/3`` (it's public IPv6 Internet) is allowed as well.

Rule's priority increases from up to down.
