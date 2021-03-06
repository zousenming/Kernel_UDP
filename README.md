# Kernel_UDP
Two kernel modules that implement UDP client - server communication. <br>
Tested using Ubuntu 17.4 kernel version 4.10.0-26-generic. <br>

Once running, the client send HELLO to server, that answers OK.

## Files
`k_udp/udp_client.c`: kernel module for client <br>
`k_udp/udp_server.c`: kernel module for server <br>
`k_udp/kernel_udp.c` : internal implementation of udp functions needed by both server an client <br>
`k_udp/include/kernel_udp.h` : header file containing all methods implemented by `kernel_udp.c` <br>
`user_client.c`: a c user application to test that server receives message from any udp client. <br> <br>

If you want to test only the server, you can also use `netcat -u [ipaddress] [port]` to send message (to install it, run `apt-get install netcat`). <br>
If you want to test only the client, you can also use `netcat -l -u [ipaddress] [port]` to listen for message (it will receive HELLO as soon the client executes).

## Usage
1. Compile ( `make` in this folder)
2. Load server with `sudo insmod udp_server.ko` (see [Parameters](#parameters) for optional parameters)
3. Load client with `sudo insmod udp_client.ko` (see [Parameters](#parameters) for optional parameters)
4. Observe in /var/log/kern.log the message passing:
```
kernel: [ 7155.409235] Server: Initialized
kernel: [ 7155.409384] Server: Thread running [udp_server_start]
kernel: [ 7155.409390] Server: Created socket [udp_server_listen]
kernel: [ 7155.409411] Server: Socket is bind to 127.0.0.4 [udp_server_listen]
kernel: [ 7161.140902] Client: Initialized
kernel: [ 7161.140954] Client: Thread running [udp_server_start]
kernel: [ 7161.140969] Client: Created socket [udp_server_listen]
kernel: [ 7161.141355] Client: Socket is bind to 127.0.0.1 [udp_server_listen]
kernel: [ 7161.141444] Client: Sent message to 127.0.0.4 : 3000, size 6 [udp_server_send]
kernel: [ 7161.141456] Server: Received message from 127.0.0.1 : 3001 , size 6 [udp_server_receive]
kernel: [ 7161.141460] Server: Got HELLO
kernel: [ 7161.141490] Client: Received message from 127.0.0.4 : 3000 , size 3 [udp_server_receive]
kernel: [ 7161.141493] Server: Sent message to 127.0.0.1 : 3001, size 3 [udp_server_send]
kernel: [ 7161.141495] Client: Got OK
kernel: [ 7161.141496] Client: All done, terminating client [connection_handler]
kernel: [ 7162.702058] Client: Terminated thread [network_server_exit]
kernel: [ 7162.702060] Client: Module unloaded [network_server_exit]
kernel: [ 7164.674459] Server: Terminated thread [network_server_exit]
```
<b>Tip:</b> If you want to have a dynamic view of the kern.log file, use the command `tail -f /var/log/kern.log` or `dmesg -wH`

By default, server is bind to address 127.0.0.4, port 3000, while the client is bind to 127.0.0.1 random free port (chosen by system).

## Parameters:
On client, the server ip, server ip and port can be specified as module parameters
when loading the module: <br> `sudo insmod udp_client.ko myip=123,12,1,0 myport=2900 destip=123,12,1,2 destport=3000`

On server, the server ip and port can be specified as module parameters
when loading the module: <br> `sudo insmod udp_server.ko myip=123,12,1,2 port=3000`

You can also type `modinfo module_name.ko` in the terminal to get these informations.
<b> Note:</b> Use the commas and not the dot when inserting the ip addresses.

## Speedtest

Inside `k_udp/include/kernel_udp.h`  there is a macro called SPEED_TEST. If set to 1, the udp server will just listen for message and count how many per second are received, printing the total amount received before unloading. The udp client will just send messages, counting how many per second are sent and how many are sent in total.<br> This can be used to see the maximum load of packets the system can take.

## run.sh
To make the loading / unloading of modules easier, I also created a script `run.sh`. This script calls `make`, `insmod`,
wait a command to terminate and call `rmmod`.

1. Give appropriate executing permission using `chmod`
2. Run it `./run module_name [other optionals parameters]`
3. Once unloaded the module, it will wait for `enter` or `ctrl-c` to either continue and unload the module or terminate.

If the module is already running and this script is called, the module is unloaded first, recompiled, and loaded again.
```
emanuele@emanuele-MacBookPro:~/Desktop/Kernel_Modules/Kernel_UDP$ ./run.sh udp_client
Module Successfully complied
Successfully loaded Module
Press enter to remove the module or Ctrl+C to exit...
Closing
Successfuly unloaded Module
```

## Disclaimer
These are my first kernel modules, if you experience bugs, or simply want to improve it, any suggestion is appreciated. <br>

In case the module blocks (impossible to unload it, `lsmod` returns `Used by -1`, whole screen freezes),
the only thing you can do is force restart the computer (hold power button, or unplug it). If you manage to shut it down
normally, but the interface still freeze and it does not shut down (ubuntu logo with moving dots), do a force shut down.<br>
On restart, the kernel will be restored and module unloaded. <br>
I am not responsible of any use you do or any damage it might create.
