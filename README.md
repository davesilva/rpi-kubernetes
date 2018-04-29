# Raspberry Pi Kubernetes

Configuration for a Raspberry Pi Kubernetes cluster.

## Flashing the images

This step uses the [Hypriot flash](https://github.com/hypriot/flash) utility 
to flash the SD card images. We're using the Kubernetes-ready image for
HypriotOS found [here](https://github.com/Tombar/image-builder-rpi-k8s/releases).

### Master

    flash --userdata cloud-config-master.yml \
          --hostname kubernetes-master \
          https://github.com/Tombar/image-builder-rpi-k8s/releases/download/v1.7.1-k8s-ALPHA/rpi-k8s-v1.7.1-k8s-ALPHA.img.zip
 
### Workers

    flash --userdata cloud-config-worker.yml \
          --hostname kubernetes-worker1 \
          https://github.com/Tombar/image-builder-rpi-k8s/releases/download/v1.7.1-k8s-ALPHA/rpi-k8s-v1.7.1-k8s-ALPHA.img.zip

Substitute `kubernetes-worker1` with a distinct name for each node.

## Joining workers to the cluster

First, we need to get the `kubeadm` token from the master node.

    $ ssh kubernetes-master.local
    
    $ sudo kubeadm token list

Tokens expire after 24 hours, so if there aren't any we can
create a new one with `sudo kubeadm token create`.

For the next part, we need the token and either the IP address of
the master node or a FQDN that points to it (.local won't work).
Then just connect to each worker and use `kubeadm` to join the
cluster.

    $ ssh kubernetes-worker1.local

    $ sudo kubeadm join --token <TOKEN> kubernetes-master.<DOMAIN>:6443

## Other things

### Regenerating API certs

On the master node:

    # rm /etc/kubernetes/pki/apiserver.*

    # sudo kubeadm alpha phase certs all --apiserver-advertise-address=0.0.0.0 --apiserver-cert-extra-sans=<DOMAIN>

    $ docker rm -f `docker ps -q -f 'name=k8s_kube-apiserver*'`

    # systemctl restart kubelet

### Read the kubelet logs

    # journalctl -u kubelet -f
