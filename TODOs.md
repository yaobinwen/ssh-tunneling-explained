# TODOs

SSH tunneling is a big topic and I don't think I have (probably nor can I) cover all the aspects in one single article. I still have some topics I am interested in but haven't talked about yet. They are listed here.

## Explain option `-g` of ssh(1)

(I made some notes as follow but need to verify them when I have time and merge them into the main article.)

Suppose we have two hosts:

- Host A:
  - IP: 192.168.16.242
- Host B:
  - IP: 192.168.16.243
  - Username: `vagrant`
  - SSH server: at port 22
  - Has an Apache2 server running at the port 80.

When we do the local forwarding as follows in order to access the Apache2 service from Host A:

- `ssh -L 9001:localhost:80 -p 22 -l vagrant 192.168.16.243`:
  - By default, `localhost` is bound to the local port (`9001` in this case) so the command above is equivalent to `ssh -L localhost:9001:localhost:80 ...`.
  - `curl http://192.168.16.242:9001` fails.
  - `curl http://localhost:9001` succeeds.
- `ssh -L 192.168.16.242:9001:localhost:80 -p 22 -l vagrant 192.168.16.243`:
  - `curl http://192.168.16.242:9001` succeeds.
  - `curl http://localhost:9001` fails.
- `ssh -L *:9001:localhost:80 -p 22 -l vagrant 192.168.16.243`:
  - This binds all the network interfaces with the port `9001`.
  - `curl http://192.168.16.242:9001` succeeds.
  - `curl http://localhost:9001` succeeds.
- `ssh -g -L 9001:localhost:80 -p 22 -l vagrant 192.168.16.243`:
  - Equivalent to `ssh -L *:9001:localhost:80 ...`.
  - `curl http://192.168.16.242:9001` succeeds.
  - `curl http://localhost:9001` succeeds.
- `ssh -g -L localhost:9001:localhost:80 -p 22 -l vagrant 192.168.16.243`:
  - Although `-g` is used, `localhost:9001` overrides `-g` so only `localhost` is bound to the port `9001`. The effect of `-g` is thus cancelled.
  - `curl http://192.168.16.242:9001` fails.
  - `curl http://localhost:9001` succeeds.

## Explain option `-N` of ssh(1)

See ssh(1).

## Dynamic port forwarding

See [dynamic port forwarding](https://en.wikipedia.org/wiki/Port_forwarding#Dynamic_port_forwarding).

## Learn more about security implication of SSH tunnels.

## Read other articles about SSH tunneling

By reading the articles written by others, I can learn how to explain the topic better:

- [SSH Tunneling](https://www.ssh.com/academy/ssh/tunneling)
- [SSH Tunneling Explained](https://goteleport.com/blog/ssh-tunneling-explained/)
- [A Visual Guide to SSH Tunnels: Local and Remote Port Forwarding](https://iximiuz.com/en/posts/ssh-tunnels/)
- [SSH Local, Remote and Dynamic Port Forwarding - Explain it like I am five!](https://erev0s.com/blog/ssh-local-remote-and-dynamic-port-forwarding-explain-it-i-am-five/)
- [SSH Tunnel - Local, Remote and Dynamic Port Forwarding](https://blog.jakuba.net/ssh-tunnel---local-remote-and-dynamic-port-forwarding/)
