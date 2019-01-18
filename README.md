# ansible-role: Libvirt Manager

This Ansible role configures a host to run as a QEMU/KVM host using libvirt and then provisions new user-defined VMs.

<h2>Description</h2>

The role includes two sub-roles which can be used indepedently. The libvirt_host & virt_provisioner.
The role has been tested on the following Linux images:
<ul>
  <li>debian-8.10.0-amd64-netinst</li>
  <li>debian-9.5.0-amd64-netinst</li>
  <li>ubuntu-16.04.5-server-amd64</li>
  <li>ubuntu-18.04.1-live-server-amd64</li>
  <li>CentOS-7-x86_64-Minimal-1810</li>
 </ul>
 It is possible that the role can be run succesfully in other versions as well. Surely, the custom module decompress.py which is included in virt_provisioner role needs python v2.7. But this module is not core and can be excluded from the playbook, provided that no compressed unarchived image files are used as boot-images in VM definition file.

<h3>libvirt_host: Configure a Host to run libvirt</h3>
The libvirt_host takes a fresh installation of a Linux Machine and configures it to run as a libvirt host. The tasks performed are as follows:
<ol> 
  <li>Checks the distribution release and if it is "Ubuntu", it installs required Ubuntu repos, since the virt-install package is missing by default. </li>
  <li>Installs the required packages per detected distro, and starts libvirt daemon</li>
  <li>Configures the minimum networking required by libvirt per detected release. It creates a bridge interface and includes the default interface in the bridge.</li>
  <li>Restarts network services in order to apply the new network settings</li>
</ol>
The host now is ready to define VMs. It is time for the virt_provisioner.

<h3>Virt_provisioner: Define new VMs</h3>
The virt_provisioner does exactly what is sounds. In particular:
<ol>
  <li>Parses the VM definition file and sets various facts that are going to be used in the following tasks.</li>
  <li>Creates VM disk directories if they are not exist and moves any pre-existing VM disks in VM disk directory.</li>
  <li>Setup the boot image files. According to the boot-image type (url,file,path,compressed) it executes the necessary tasks so the boot-image file to be in the default or user-defined libvirt boot folder.</li>
  <li>Invokes the virt-install commands for each Defined VM. </li>
</ol>

There is an extra task included in the role's task folder which can be used indepedently for deleting vms. See the undefine.yml playbook file to see an example on how to use this task. The only requirement for the undefine_vm.yml task is to provide a vars dictionary in the form:        

    vms: 
      k1:
        name: 'VMname1' 
      k2:
        name: 'VMname2'
      ⋮
      kx:
        name: 'VMnamex'

By finishing the playbook, the nd result will be a libvirt host that runs all the vms which are defined in the vm definition file. The role has completed its job. If any installation needed from now on in each VM, it is out of the scope of this role. The user may connect with VNC to each VM and completes any required tasks.

<h2> How to use Libvirt Manager</h2>

If you want to setup a new VM host from scratch keep reading. If you already have a working VM host and just provision new VMs then you can skip to virt_provisioner.

<h3>Libvirt_host</h3>

Just add the following in the playbook in order to use the role. 
<br>roles:
<br>&nbsp;&nbsp;&#45; libvirt_host

The user may have to define/override the role's default variables to meet their needs. The list of the variables the user can set is:
<ul>
  <dt><b>vm_host_br_ip:</b></dt>            
    <dd><p>This is the IP address of the vm host bridge interface following network setup.  
      <br><i>Type: String</li>
      <br>Default: The detected IP of the target vm host</i></dd>
      
<dt><b>vm_host_br_netmask:</b> </dt>      
    <dd><p>The netmask of the vm host bridge interface. 
    <br><i>Type: String
    <br>Default: The detected netmask of the target vm host</i></dd>
    
<dt><b>vm_host_br_gw:</b>   </dt>         
    <dd><p>The gateway IP address of the vm host bridge interface.  
    <br><i>Type: String
    <br>Default: The detected GW IP address of the target vm host</i>    </dd>
    
<dt><b>vm_host_br_prefix_size:</b></dt>   
    <dd><p>The CIDR notation of the bridge interface netmask.   
    <br><i>Type: String    
    <br>Default: '24' </i></dd>
  
<dt><b>vm_host_br_dns: </b> </dt>         
    <dd><p> The DNS IP addresses list. This var must be declared as a list even if there is one item only. 
    <br><i>Type: List      
    <br>Default: ['ansible_default_ipv4.gateway', '1.1.1.1' ] </i>   </dd>
  
<dt><b>vm_host_br_name: </b></dt>    
    <dd><p> The name of the bridge interface, e.g. 'br0'.  
    <br><i>Type: String    
    <br>Default: 'br0'      </i>  </dd>
  
<dt><b>vm_host_search_domain: </b></dt>   
    <dd><p> The list containing domain names of your organization. 
    <br><i>Type: List      
    <br>Default:   </i>  </dd>
  
<dt><b>vm_host_br_if: </b></dt>           
    <dd><p> The interface name that will be connected to the bridge, e.g 'eth0'   
    <br><i>Type: String    
    <br>Default: Ansible Detected Default Interface      </i>  </dd>
  
<dt><b>vm_host_br_network: </b></dt>       
    <dd><p> The network address of the bridge ip address.   
    <br><i>Type: String    
    <br>Default: Calculated automatically from the bridge IP address using regex.  </i> </dd>
  
<dt><b>vm_host_br_broadcast:</b> </dt>     
    <dd><p> The broadcast address of the bridge ip address.  
    <br><i>Type: String    
    <br>Default: Calculated automatically from the bridge IP address using regex.  </i></dd>
  
<dt><b>vm_host_repos_ubuntu: </b></dt>    
    <dd><p> A list of repositories for Ubuntu.    
    <br><i>Type: List      
    <br>Default: Default universe archive repos. </i></dd>  
  
<dt><b>packages:</b> </dt>                
    <dd><p>A dictionary containing the necessary packages for each linux release.    
    <br><i>Type: Dict      
    <br>Default: Packages for Debian, Ubuntu and CentOS.</i></dd>
</ul>

For more information on default values, please check defaults/main.yml file. It is advised to not make any changes there but to the vars/main.yml since the latter takes precedence.

If the user do not specify any variables, the role will execute succefully according to the target's detected network settings.

<h3>Virt_provisioner</h3>

This is the part where new VMs are created based on the VM definition file. The file is located in vars/main.yml and the VMs are defined as key,value pairs of the 'vms' variable.

The commited version contains an example of how this file is structured. The variables that the user can override are the following:

<ul>
  <dt><b>libvirt_boot_image_folder:</b></dt>
  <dd><p>The folder path where the libvirt boot images will stored. These are also libvirt default.
    <br><i>Type: String
    <br>Default: '/var/lib/libvirt/boot/'</i></dd>
  
  <dt><b>libvirt_vm_disk_folder</b></dt>
  <dd><p>The folder path where the libvirt vm disks will stored. These are also libvirt default.
    <br><i>Type: String
    <br>Default: 'var/lib/libvirt/images/'</i></dd>
    
  <dt><b>vms:</b></dt>
  <dd><p>The VM definition dictionary. The values of the dict items are the VMs to be provisioned.
  <br><i>Type: Dict
    <br>Default: </i></dd> 
 </ul>

<h3>Special Notes:</h3>

Apart from disks,cdrom and networks all the following keys that can be used in a vm declaration are the same as the virt-install options and arguments. A user can include any option in the form of "option: 'argument". In case of options that take no argumemnts the user may set the key with an empty string as a value. 

<b>Disks:</b>

This key is a list variable that the user declares to the vm definition and corresponds to the --disk option of the virt-install utility. The disks variable must be declared as a list even in the case of just one item. During the play the set_facts tasks will parse the variable and for each item in this list, it will construct the <b>--disk "{{item}}" </b>arguments that will pass to the resulted virt-install command.

The user can define new disks by adding the size option on the disk declaration, as per virt-install manual but they can also use existing disks by ommiting the size option. <b>In this case the user must add the existing disk or a link to the existing disk in the virt_provisioner/files folder</b>.

<b>cdrom:</b>

This key can accept different argument types so that's why is different from virt-install's. For user convinience the key can accept the following types as values:
- urls
- filenames
- paths

If the argument is a url, then the role will donwload the file, it will decompress it if is an unarchived compressed iso and it will put it in the boot images folder, ready to be used during VM installation.

If the argument is just a filename depicting that a local iso is present, such as boot-image.iso, then it will copy the file from the virt_provisioner/files folder to the boot-images folder to be used during VM installation. Warning: the user must have a copy or a link to the actual file in the files folder.

If the argument is a path, it will look for the defined file in that path on the target machine host. This assumes that the file is present on the vm host and it will get that path as a cdrom argument during run time. Beware in that case the file's permission. The safest way to use the boot image iso is to include it in the libvirt's default boot image directory.

<b>networks:</b>

This list contains the network definition arguments for the VM. Since, the vms variable is a dictionary and a dictionary cannot have the same name for multiple keys, in the case of networks, a user must declare them as item of that list. During the runtime the role will add the <b>--network "{{item}}"</b> to the virt-install command.

For more details on how to provision new VMs, check the virt_provisioner/vars/main.yml file where actual working examples are defined, since these were the exact that I used in my lab. If you run now the roles from the begining, without touching any variable you should get two running VMs, provided that you have 6GB of RAM or more. 
