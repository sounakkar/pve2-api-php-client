This class provides the building blocks for someone wanting to use PHP to talk to Proxmox's API.
Relatively simple piece of code, just provides a get/put/post/delete abstraction layer as methods
on top of Proxmox's REST API, while also handling the Login Ticket headers required for authentication.

See http://pve.proxmox.com/wiki/Proxmox_VE_API for information about how this API works.
API spec available at https://pve.proxmox.com/pve-docs/api-viewer/index.html

## Requirements: ##

PHP 5/7 with cURL (including SSL/TLS) support.

## Usage: ##

Example - Return status array for each Proxmox Host in this cluster.

    require_once("./pve2-api-php-client/pve2_api.class.php");

    # You can try/catch exception handle the constructor here if you want.
    $pve2 = new PVE2_API("hostname", "username", "realm", "password");
    # realm above can be pve, pam or any other realm available.

    if ($pve2->login()) {
        foreach ($pve2->get_node_list() as $node_name) {
            print_r($pve2->get("/nodes/".$node_name."/status"));
        }
    } else {
        print("Login to Proxmox Host failed.\n");
        exit;
    }

Example - Create a new VM  on the first host in the cluster.

    require_once("./pve2-api-php-client/pve2_api.class.php");

    # You can try/catch exception handle the constructor here if you want.
    $pve2 = new PVE2_API("hostname", "username", "realm", "password");
    # realm above can be pve, pam or any other realm available.

    if ($pve2->login()) {

    $nextid = $pve2->get("/cluster/nextid");
	$nvx= array();
	$nvx['vmid']=$nextid;
	$nvx['cores']="1";
	$nvx['memory']="1024";
	$nvx['net0']="virtio,bridge=vmbr0,firewall=1,rate=12.5"; //example with firewall speed set to 100mbps
	$nvx['ide2']="local:iso/generic-pc-2.11.1b178.amd64.iso,media=cdrom";
	$nvx['onboot']="1"; //Start on boot
	$nvx['scsi0']="local-zfs:32,ssd=on"; //example with advance option ssd type
	$nvx['vga']="qxl,memory=128"; //for enable spice protocol with 128 mb memory
	$pve2->POST("/nodes/pve/qemu", $nvx);
		echo $nextvmid."-OK";
    } else {
        print("Login to Proxmox Host failed.\n");
        exit;
    }

Example - Modify DNS settings on an existing container on the first host.

    require_once("./pve2-api-php-client/pve2_api.class.php");

    # You can try/catch exception handle the constructor here if you want.
    $pve2 = new PVE2_API("hostname", "username", "realm", "password");
    # realm above can be pve, pam or any other realm available.

    if ($pve2->login()) {

        # Get first node name.
        $nodes = $pve2->get_node_list();
        $first_node = $nodes[0];
        unset($nodes);

        # Update container settings.
        $container_settings = array();
        $container_settings['nameserver'] = "4.2.2.2";

        # NOTE - replace XXXX with container ID.
        var_dump($pve2->put("/nodes/".$first_node."/openvz/XXXX/config", $container_settings));
    } else {
        print("Login to Proxmox Host failed.\n");
        exit;
    }

Example - Delete an existing container.

    require_once("./pve2-api-php-client/pve2_api.class.php");

    # You can try/catch exception handle the constructor here if you want.
    $pve2 = new PVE2_API("hostname", "username", "realm", "password");
    # realm above can be pve, pam or any other realm available.

    if ($pve2->login()) {
        # NOTE - replace XXXX with node short name, and YYYY with container ID.
        var_dump($pve2->delete("/nodes/XXXX/openvz/YYYY"));
    } else {
        print("Login to Proxmox Host failed.\n");
        exit;
    }

Licensed under the MIT License.
See LICENSE file.
