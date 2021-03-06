pymitm6
=======
                                           __                 ____    
                                        __/\ \__             /'___\   
             _____   __  __    ___ ___ /\_\ \ ,_\   ___ ___ /\ \__/   
            /\ '__`\/\ \/\ \ /' __` __`\/\ \ \ \/ /' __` __`\ \  _``\ 
            \ \ \L\ \ \ \_\ \/\ \/\ \/\ \ \ \ \ \_/\ \/\ \/\ \ \ \L\ \
             \ \ ,__/\/`____ \ \_\ \_\ \_\ \_\ \__\ \_\ \_\ \_\ \____/
              \ \ \/  `/___/> \/_/\/_/\/_/\/_/\/__/\/_/\/_/\/_/\/___/ 
               \ \_\     /\___/                                       
                \/_/     \/__/                                        



    Copyright (C) 2013 Karim Sudki [SCRT]

    Licensed under the GPL version 3.

    This application is an open source software for quick and easy implementation
    of IPv6 Man-In-The-Middle attack on IPv4 networks, known as "SLAAC attack".

    Translation between IPv6 and IPv4 networks is acheived using NAT64 algorithm.
    
    To allow packets to be modified, the infamous TUN interface has been used.

    Example of use :
     ______________             _______________________             _____________
    |              |           |                       |           |            |
    | Dual stacked |__IPV6 ____| ETH0 --- TUN --- ETH0 |__IPV4 ____|    IPV4    |
    |     Host     |    COMM   |                       |    COMM   |   Network  |
    |______________|           |_______________________|           |____________|

         TARGET                         HACKER                          WORLD


####[Features]###################################################################
    
    now()
    - Targets selection
    - Support custom DNS record from csv file, reload possible while running
    - Resolution and display of hosts MAC addresses for easy recognition
    - Terminal interface

    next()

    - Raguard evasion
    - Plugins integration
    - Graphical interface


####[Pre-requisites]#############################################################
    
    - Python 2.x
    - iproute2
    - iptables
    

####[Usage]######################################################################

    usage: pymitm6.py [-h] [-c DNS_FILE] [-int PHY_INT] [-dns4 DNS_V4]
                      [-dns6 DNS_V6] [-good GOOD_PREFIX] [-bad BAD_PREFIX]
                      [-pool IP_POOL] -tun TUN_NAME -u USER[--mktun]

    optional arguments:
      -h, --help         show this help message and exit
      -c DNS_FILE        DNS file (comma separated)
      -int PHY_INT       Physical Interface Name
      -dns4 DNS_V4       IPv4 DNS Server
      -dns6 DNS_V6       IPv6 DNS Proxy
      -good GOOD_PREFIX  GOOD prefix (CIDR)
      -bad BAD_PREFIX    BAD prefix (CIDR)
      -pool IP_POOL      IPv4 pool for NAT64 (CIDR)
      -tun TUN_NAME      TUN Interface Name
      -u USER            Running user
      --mktun            Create TUN interface


    0 - Flush previous configuration

    # ip link delete [TUN_NAME]
    # iptables -F
    # ip6tables -F

    1 - Create TUN interface :

    # pymitm6.py -u [USER] -tun [TUN_NAME] --mktun

    2 - Configure all interfaces :

    [PHY_INT]

    # ip addr add [DNS_V6]/[PREFIX_LEN] dev [PHY_INT]
    
    [TUN_NAME]

    # ip link set [TUN_NAME] up
    # ip addr add [BAD_PREFIX_IP] dev [TUN_NAME]
    # ip addr add [POOL_IP] dev [TUN_NAME]
    # ip route add [POOL] dev [TUN_NAME]
    # ip route add [BAD_PREFIX]/[PREFIX_LEN] dev [TUN_NAME]

    3 - Configure iptables :
    
    # iptables -t nat -A POSTROUTING -o [PHY_INT] -j MASQUERADE
    # ip6tables -A OUTPUT -p icmpv6 --icmpv6-type destination-unreachable -j DROP
    # ip6tables -A OUTPUT -p icmpv6 --icmpv6-type router-solicitation -j DROP

    4 - Configure forwading :

    # sysctl -w net.ipv4.conf.all.forwarding=1
    # sysctl -w net.ipv6.conf.all.forwarding=1

    5 - Run the program (example) :

    # pymitm6.py -bad 2001:6f8:608:ace::/64   \
                 -good 2001:6f8:608:fab::/64  \
                 -dns4 8.8.8.8                \
                 -dns6 2001:6f8:608:fab::2    \
                 -tun nat64                   \
                 -int eth0                    \
                 -pool 192.168.100.0/24       \
                 -u root

####[Script]#####################################################################

13.01.2016      Bash script automating above procedure
                1. configure variables
                2. chmod +x command.sh && sudo ./command.sh
