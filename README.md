# macos-openvpn-server

Run an OpenVPN server on macOS so remote devices can connect through your Mac. This setup uses certificate authentication, UDP port 443, and PF firewall rules, with Tunnelblick configuration notes for macOS and support for clients such as OpenVPN on iOS.

## Features

- Configure an OpenVPN TLS server with Ed25519 certificates generated through Easy-RSA.
- Accept VPN connections on UDP port 443 from compatible OpenVPN clients.
- Apply PF firewall rules to traffic routed through the Mac.
- Configure Tunnelblick to keep the server connection active across macOS Fast User Switching.

## Installation

The reference configuration targets OpenVPN 2.5 and Easy-RSA 3. You need administrator access to configure forwarding and PF. If your Mac sits behind a router, forward UDP port 443 to it.

Install the dependencies with [Homebrew](https://brew.sh/):

```sh
brew install openvpn easy-rsa
```

Check the installed versions before adapting the configuration:

```sh
openvpn --version
easyrsa --version
```

Homebrew may install newer releases than the reference versions. Check their configuration and cryptographic compatibility before deployment.

Enable IP forwarding for the current session:

```sh
sudo sysctl -w net.inet.ip.forwarding=1
sudo sysctl -w net.inet6.ip6.forwarding=1
```

These commands do not establish persistence after reboot. The donor documents `/etc/sysctl.conf` for persistence on its tested macOS systems; verify that approach on your macOS release.

Before starting the server:

1. Create a certificate authority and issue separate server and client certificates with Easy-RSA. The reference setup uses Ed25519; confirm support in your server and client builds.
2. Prepare an OpenVPN server configuration with `proto udp`, `port 443`, and paths to your certificates and server private key.
3. Configure PF to permit the VPN connection and route client traffic. Add NAT rules if clients will access the internet through the Mac.
4. Prepare client profiles with the server address and the appropriate client credentials.

**Forwarding alone is not enough:** PF rules and OpenVPN routes must match your network interfaces and VPN subnet. Review existing PF rules before changing them, especially over a remote connection.

## Usage

Run the macos-openvpn-server configuration through OpenVPN. No separate `macos-openvpn-server` executable is documented by the donor project.

After you have created a complete server configuration, start it in the foreground:

```sh
sudo "$(brew --prefix openvpn)/sbin/openvpn" \
  --config /absolute/path/server.conf \
  --verb 3
```

Replace `/absolute/path/server.conf` with your configuration file. Connect from an OpenVPN client using its client profile, then check that the expected destinations are reachable through the VPN.

If you use [Tunnelblick](https://tunnelblick.net/) to manage the server connection, configure it to remain active across Fast User Switching. The donor reports routing problems when the connection does not persist.

Ad, tracker, and malware blocking require a separately configured filtering proxy or firewall policy. Starting OpenVPN does not enable those filters by itself.

## License

MIT. This project is based on [essandess/macos-openvpn-server](https://github.com/essandess/macos-openvpn-server), also licensed under MIT. Retain the original copyright and permission notice with any copied code or documentation, as required by that license.

