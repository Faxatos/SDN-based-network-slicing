
# SDN-based Network Slicing #
This repository showcases the implementation of network slicing in an SDN to enable the isolation of network resources. The goal of this example is to show that different requirements can be fulfilled on a shared physical infrastructure by using network slicing.
## Architecture Overview ##
The architecture employs a *multi-hop topology* for two emulation applications/tasks. The network comprises four hosts (h1, h2, h3, h4) and five switches (s1, s2, s3, s4, s5). See the picture below for visualization:

<p align="center">
    <img src="https://github.com/Faxatos/SDN-based-network-slicing/assets/57751761/e00aa388-aec2-49c6-8fd5-698426891557" alt="drawing" width="700"/>
</p>

This folder contains the following files:

1. network.py: Script to build a network with four hosts and five switches, bandwidth is 1Mbps and 10Mbps.

2. topology_slicing.py: Application to isolate the network topology into upper slice (h1 -> s1 -> s2 -> s5 -> s4 -> h3, 10Mbps) and lower slice (h2 -> s1 -> s3 -> s4 -> h4, 1Mbps).

4. service_slicing.py: Application to isolate the service traffics into video traffic (UDP port 9999) obtaining 10Mbps and non-video traffic (the remaining services) obtaining 1Mbps.

## Getting Started ##

### Prerequisites ###

Make sure to have installed:
  
  + [Vagrant](https://developer.hashicorp.com/vagrant/install) (>= v2.2.5)
  + [Virtualbox](https://www.virtualbox.org/wiki/Downloads) (>= v6.0)

Install then ComNetsEmu following [these instructions](https://stevelorenz.github.io/comnetsemu/installation.html#option-1-install-in-a-vagrant-managed-vm-highly-recommended). Additional documentation can be found  [here](https://github.com/stevelorenz/comnetsemu). ComNetsEmu is recommended to run in a VM with 2 vCPUs and 2GB RAM.

### Installation ###
To launch ComNetsEmu, navigate to the ComNetsEmu directory and execute the following command:

```sh
  vagrant up comnetsemu
  ```
Once the VM when is up and running (indicated by the ComNetsEmu banner on the screen), SSH into the VM:

```sh
  vagrant ssh comnetsemu
  ```
Move to the `./app` directory and clone the repository into a new local directory:
```sh
  git clone https://github.com/Faxatos/SDN-based-network-slicing.git
  ```
To halt the VM when done, utilize the following command:
```sh
  vagrant halt
  ```

### Usage ###

You can simply run the emulation with following commands in the new local directory.

1. Enable Ryu controller to load the application and to run in background:
    ```bash
    $ ryu-manager topology_slicing.py &
    ```
2. Start the network with Mininet:
    ```bash
    $ sudo python3 network.py
    ```
    
Please stop the running Ryu controller before starting a new Ryu controller. For example, type `htop` in the terminal to show all running processes, press the key `F4` to look for the process *ryu-manager*, then press the key `F9` to stop the process, with the key `F10` to quite `htop`.

## How to verify ##
There are three modes to verify the slice:

1. ping mode: verifying connectivity, e.g.
    ```bash
    mininet> pingall
    *** Ping: testing ping reachability
    h1 -> X h3 X 
    h2 -> X X h4 
    h3 -> h1 X X 
    h4 -> X h2 X 
    *** Results: 66% dropped (4/12 received)
    ```

2. iperf mode: verifying bandwidth, e.g.
    ```bash
    mininet> iperf h1 h3
    *** Iperf: testing TCP bandwidth between h1 and h3 
    *** Results: ['9.50 Mbits/sec', '12.4 Mbits/sec']
    mininet> iperf h2 h4
    *** Iperf: testing TCP bandwidth between h2 and h4 
    *** Results: ['958 Kbits/sec', '1.76 Mbits/sec']
    ```

3. client mode: verifying flows on each switch, e.g.
    ```bash
    mininet> sh ovs-ofctl dump-flows s1
    ```

## Task 2: Service Slicing ##
The network should be isolated into two slices for two services:
1. Video traffic slice:
    The video traffic (UDP port 9999) is able to gain max. 10Mbps bandwidth. The right of video traffic to use it is not affected by other traffics.
2. Other traffic slice:
    All traffics, which are not type UDP port 9999, can use any path of the network, but must not affect video traffic.

### How to Run ###
You can simple run the emulation applications with following commands in ./app/network_slicing/.

1. Enabling Ryu controller to load each application and to run background:
    ```bash
    $ ryu-manager service_slicing.py &
    ```
2. Starting the network with Mininet:
    ```bash
    $ sudo python3 network.py
    ```

Please stop the running Ryu controller before starting a new Ryu controller. For example, type `htop` in the terminal to show all running processes, press the key `F4` to look for the process *ryu-manager*, then press the key `F9` to stop the process, with the key `F10` to quite `htop`.

## How to Verify ##

There are three modes to verify the slice:

1. *ping mode*: verifying connectivity, e.g.
    ```bash
    mininet> pingall
    *** Ping: testing ping reachability
    h1 -> X h3 X 
    h2 -> X X h4 
    h3 -> h1 X X 
    h4 -> X h2 X 
    *** Results: 66% dropped (4/12 received)
    ```

2. *iperf mode*: verifying bandwidth, e.g.
    ```bash
    mininet> iperf h1 h3
    *** Iperf: testing TCP bandwidth between h1 and h3 
    *** Results: ['9.50 Mbits/sec', '9.98 Mbits/sec']
    mininet> iperf h2 h4
    *** Iperf: testing TCP bandwidth between h2 and h4 
    *** Results: ['958 Kbits/sec', '1.32 Mbits/sec']
    ```

3. *client mode*: verifying flows on each switch, e.g.
    ```bash
    mininet> sh ovs-ofctl dump-flows s1
    ```
## Service chain ## 
The  inclusion  of  the  s5  switch  serves  to  implement  a **service  chain**, allowing  the  addition  of  network  functionalities  (such  as  firewalls)  by  modifying  the  switch  flows. This  can  be  achieved  either  with code  or  through  the  CLI. For  simplicity, we'll  utilize  CLI  using  the  [ovs-ofctl](https://www.openvswitch.org/support/dist-docs/ovs-ofctl.8.txt) commands, compatible with any OpenFlow switch.

Here are some useful commands for deleting and adding switch flows:
 ```bash
sh ovs-ofctl del-flows <switch> in_port=<port>
sh ovs-ofctl del-flows <switch> <protocol>
sh ovs-ofctl add-flow <switch> in_port=<port>,priority=<priority>,actions=<action>
sh ovs-ofctl add-flow <switch> <protocol>,priority=<priority>,actions=<action>
 ```
 Here's an example of how this can be applied within the described architecture:
 ```bash
 sh ovs-ofctl add-flow s5 icmp,priority=2,actions=drop
  ```
  By adding this rule, all ICMP packets passing through s5 will be dropped. You can test this using the _ping_ command.
  
## Acknowledgments

This example is based on one of the [comnetsemu](https://git.comnets.net/public-repo/comnetsemu) applications.

Released under the MIT License.

## License

Distributed under the MIT License. See `LICENSE.txt` for more information.

