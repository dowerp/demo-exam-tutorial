[ISP]
    hostnamectl set-hostname isp.au-team.irpo
    iptables -t nat -A POSTROUTING -o ens3 -j MASQUERADE
    iptables-save >> /etc/sysconfig/iptables
    systemctl enable --now iptables

[HQ-RTR]
    enable
    configure terminal
    hostname HQ-RTR
    ip domain-name au-team.irpo
    ip route 0.0.0.0 0.0.0.0 172.16.1.1
    interface tunnel.0
    ip address 10.10.10.1/30
    ip tunnel 172.16.1.2 172.16.2.2 mode gre
    exit
    router ospf 0
    network 10.10.100.0/27 area 0
    network 10.10.200.0/28 area 0
    network 10.10.30.0/29 area 0
    network 10.10.10.0/30 area 0
    passive-interface default
    no passive-interface tunnel.0
    area 0 authentication
    exit
    interface tunnel.0
    ip ospf authentication message-digest
    ip ospf message-digest-key 1 md5 P@ssword
    exit
    interface isp
    ip nat outside
    exit
    interface vl100
    ip nat inside
    exit
    interface vl200
    ip nat inside
    exit
    interface vl999
    ip nat inside
    exit
    ip nat pool HQ 10.10.0.1-10.10.200.254
    ip nat source dynamic inside-to-outside pool HQ overload interface isp
    ntp timezone utc+3
    end
    write memory

[BR-RTR]
    enable
    configure terminal
    hostname BR-RTR
    ip domain-name au-team.irpo
    ip route 0.0.0.0 0.0.0.0 172.16.2.1
    interface tunnel.0
    ip address 10.10.10.2/30
    ip tunnel 172.16.2.2 172.16.1.2 mode gre
    exit
    router ospf 0
    network 10.20.10.0/30 area 0
    network 10.10.10.0/30 area 0
    passive-interface default
    no passive-interface tunnel.0
    no passive-interface fw
    area 0 authentication
    exit
    interface tunnel.0
    ip ospf authentication message-digest
    ip ospf message-digest-key 1 md5 P@ssword
    exit
    interface fw
    ip ospf authentication message-digest
    ip ospf message-digest-key 1 md5 P@ssword
    exit
    interface isp
    ip nat outside
    exit
    interface fw
    ip nat inside
    exit
    ip nat pool BR 10.0.0.1-10.254.254.254
    ip nat source dynamic inside-to-outside pool BR overload interface isp
    ntp timezone utc+3
    end
    write memory

[HQ-SW]
    hostnamectl set-hostname hq-sw.au-team.irpo

[HQ-SRV]
    hostnamectl set-hostname hq-srv.au-team.irpo
    echo "10.10.100.2/27" > /etc/net/ifaces/ens3/ipv4address
    echo "default via 10.10.100.1" > /etc/net/ifaces/ens3/ipv4route
    cat << 'EOF' > /etc/net/ifaces/ens3/resolv.conf
search au-team.irpo
nameserver 77.88.8.8
nameserver 127.0.0.1
EOF
    systemctl restart network
    sed -i 's/^#*WHEEL_USERS ALL=(ALL:ALL) NOPASSWD: ALL/WHEEL_USERS ALL=(ALL:ALL) NOPASSWD: ALL/' /etc/sudoers
    sed -i 's|^#*Banner .*|Banner /etc/openssh/banner|' /etc/openssh/sshd_config
    echo "Authorized access only" > /etc/openssh/banner
    systemctl restart sshd
    cat << 'EOF' > /etc/bind/options.conf
options {
    version "unknown";
    directory "/etc/bind/zone";
    listen-on { 10.10.100.2; 127.0.0.1; };
    listen-on-v6 { none; };
    forwarders { 77.88.8.8; };
    allow-query { any; };
};
EOF
    cat << 'EOF' > /etc/bind/zone/au-team.irpo.zone
$TTL 86400
@   IN  SOA hq-srv.au-team.irpo. root.au-team.irpo. (
            2024010101 3600 1800 604800 86400 )
    IN  NS  hq-srv.au-team.irpo.

hq-rtr  IN  A   172.16.1.2
br-rtr  IN  A   172.16.2.2
hq-srv  IN  A   10.10.100.2
hq-cli  IN  A   10.10.200.2
br-srv  IN  A   10.20.20.2
br-cli  IN  A   10.20.30.2
br-fw   IN  A   10.20.10.2
web     IN  A   172.16.1.1
docker  IN  A   172.16.2.1
EOF
    cat << 'EOF' > /etc/bind/zone/10.10.in-addr.arpa
$TTL 1D
@   IN  SOA au-team.irpo. root.au-team.irpo. (
            2025110600 12H 1H 1W 1H )
    IN  NS  au-team.irpo.

1.100   IN  PTR hq-rtr.au-team.irpo.
2.100   IN  PTR hq-srv.au-team.irpo.
2.200   IN  PTR hq-cli.au-team.irpo.
EOF
    systemctl enable --now bind.service

[HQ-CLI]
    hostnamectl set-hostname hq-cli.au-team.irpo

[BR-CLI]
    hostnamectl set-hostname br-cli.au-team.irpo
    mv /etc/net/ifaces/ens18 /etc/net/ifaces/ens3
    echo "10.20.30.2/29" > /etc/net/ifaces/ens3/ipv4address
    echo "default via 10.20.30.1" > /etc/net/ifaces/ens3/ipv4route
    cat << 'EOF' > /etc/net/ifaces/ens3/resolv.conf
search au-team.irpo
nameserver 10.10.100.2
EOF
    systemctl restart network

[BR-SRV]
    hostnamectl set-hostname br-srv.au-team.irpo
    echo "10.20.20.2/28" > /etc/net/ifaces/ens3/ipv4address
    echo "default via 10.20.20.1" > /etc/net/ifaces/ens3/ipv4route
    cat << 'EOF' > /etc/net/ifaces/ens3/resolv.conf
search au-team.irpo
nameserver 10.10.100.2
EOF
    systemctl restart network
    sed -i 's/^#*WHEEL_USERS ALL=(ALL:ALL) NOPASSWD: ALL/WHEEL_USERS ALL=(ALL:ALL) NOPASSWD: ALL/' /etc/sudoers
    sed -i 's|^#*Banner .*|Banner /etc/openssh/banner|' /etc/openssh/sshd_config
    echo "Authorized access only" > /etc/openssh/banner
    systemctl restart sshd

[BR-FW]
    hostnamectl set-hostname br-fw.au-team.irpo
