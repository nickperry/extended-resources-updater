# extended-resources-updater
A simple mechanism to monitor the presence or absence of devices and update corresponding Kubernetes extended resources.

## Background
One of the workloads I run on my 3 node Intel NUC based Kubernetes cluster at home is a home automation system. The home automation software I use (Home Assistant) uses a couple of USB transceivers to communicate with different types of home automation sensors, switches, locks, etc.

I have a requirement to be able move these transceivers from one node to another and have the Home Assistant workload automatically move to the new node where I've plugged in the devices. This allows me to take a node down, for example to reinstall the OS on it, without giving it too much thought. This helps make it viable to play with my cluster to a reasonable degree whilst also running a fairly critical workload on it.

The foundation for achieving this goal is a mechanism by which to advertise to the Kubernetes scheduler which node has the USB device resources. Kubernetes' extended resources feature gives us a reasonable way of doing this. For further reading on extended resources, see https://kubernetes.io/docs/tasks/administer-cluster/extended-resource-node/.

## Demonstation
I made a short video demonstration (on YouTube):

[![Watch the video](https://img.youtube.com/vi/DIFoaZmHIuU/hqdefault.jpg)](https://youtu.be/DIFoaZmHIuU)

## Implementation Details

The repo contains three manifests.

extended-resources-updater-cm.yml provides a config map with a shell script which indefinitely:
1) Tests the presence of two character devices at /dev/ttyACM0 and /dev/ttyUSB0.
2) Retains the result from the last iteration.
3) If the state has changed since the last iteration, uses curl with a service account token to call the Kubernetes API and patch extended resources associated with the character devices.
4) Sleeps for 10 seconds.

The script in the config map can easily be modified for different vendor and device names and different device file paths.

extended-resources-updater-rbac.yml provides:
1) A service account.
2) A cluster role with solely the ability to patch node status.
3) A role binding to bind the service account and cluster role.

extended-resources-updater.yml provides a DaemonSet to run the updater script on every node in the cluster. It:
1) Mounts the updater script from the config map at /extended-resources-updater.sh
2) Mounts the node's /dev filesystem read-only.
3) Sets an environment variable MY_NODE_NAME used by the script, based on the pod's spec.nodeName.
4) Executes the script using sh.

## Making use of the Extended Resources
See this page of the official Kubernetes documentation:
https://kubernetes.io/docs/tasks/configure-pod-container/extended-resource/
