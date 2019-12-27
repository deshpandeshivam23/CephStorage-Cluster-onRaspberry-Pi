# Ceph-Storage-Cluster-On-RBPI

What is CEPH Storage ?

Ceph is open source software defined storage to provide highly scalable object-, block- and file-based storage under a unified system.

Ceph storage clusters are designed to run on commodity hardware, using an algorithm called CRUSH (Controlled Replication Under Scalable Hashing) to ensure data is evenly distributed across the cluster and that all cluster nodes can retrieve data quickly without any centralized bottlenecks

Ceph object storage is accessible through Amazon Simple Storage Service (S3) and OpenStack Swift Representational State Transfer (REST)-based application programming interfaces (APIs), and a native API for integration with software applications.

Ceph block storage makes use of a Ceph Block Device, which is a virtual disk that can be attached to bare-metal Linux-based servers or virtual machines. The Ceph Reliable Autonomic Distributed Object Store (RADOS) provides block storage capabilities, such as snapshots and replication. The Ceph RADOS Block Device is integrated to work as a back end with OpenStack Block Storage.

Ceph file storage makes use of the Portable Operating System Interface (POSIX)-compliant Ceph file system (CephFS) to store data in a Ceph Storage Cluster. CephFS uses the same clustered system as Ceph block storage and Ceph object storage.

  Reference : https://docs.ceph.com/docs/master/
    

#################################################################################
# Build CEPH-on-RBPI Steps :
#################################################################################

Step 1: Compile CEPH On RBPI Using Clang/LLVM
    
   Reference: https://louwrentius.com/compiling-ceph-on-the-raspberry-pi-3b-armhf-using-clangllvm.html
                      
Step 2: Build Cluster
 
        Accordind to Step 1 we consider that CEPH Complie for RBPI
        
        LAB Setup :
        3 * RBPI With Ceph Compile.
        
        The RBPI in this tutorial will use the following hostnames and IP addresses.
        
        hostname            IP address

          ceph-admon              192.168.15.10    # This Node is Used for admin and monitor
          Ceph-osd1               192.168.15.11    # This Node is Used for Object-Storage-Daemon 1 or Manager
          ceph-osd2               192.168.15.12    # This Node is Used for Object-Storage-Daemon 2
          
        
        Note: Manager node is requried for Luminious Platform. Please Refer CEPH Documentations.
        Note : This is for Simple Project Purpose Configuration. Because less Hardware 
        Reference : https://docs.ceph.com/docs/master/
        
        
Step 3 : Configure All Nodes
            
            Create Ceph User for all nodes and give them root privileges
      
      # useradd -d /home/cephuser -m cephuser
      # passwd cephuser
      
      # echo "cephuser ALL = (root) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/cephuser
      # chmod 0440 /etc/sudoers.d/cephuser
      # sed -i s'/Defaults requiretty/#Defaults requiretty'/g /etc/sudoers
      
     Configure Hosts File
     
     # vi /etc/hosts
     
          192.168.15.10   ceph-admon
          192.168.15.11   ceph-osd1
          192.168.15.12   ceph-osd2
          
          
Step 4 - Configure the SSH Server
          
        In this step, I will configure the ceph-admon node. The ceph-admon node is used for configuring the monitor node and the osd nodes. Login to the ceph-admon node and become the 'cephuser'.          
        
           # ssh root@ceph-admon
           # su - cephuser
           
           
The admon node is used for installing and configuring all cluster nodes, so the user on the ceph-admon node must have privileges to connect to all nodes without a password. We have to configure password-less SSH access for 'cephuser' on 'ceph-admon' node.

Generate the ssh keys for 'cephuser'.
        
        $ ssh-keygen
        
Next, create the configuration file for the ssh configuration.

        $ vim ~/.ssh/config
        
            Host ceph-admON
                    Hostname ceph-admon
                    User cephuser
 
            Host ceph-osd1
                    Hostname ceph-osd1
                    User cephuser

            Host ceph-osd2
                    Hostname ceph-osd2
                    User cephuser
            wq!

Change the permission of the config file.
        
            chmod 644 ~/.ssh/config
 Now add the SSH key to all nodes with the ssh-copy-id command
 
        $ ssh-keyscan ceph-admon ceph-osd1 ceph-osd2  >> ~/.ssh/known_hosts
        $ ssh-copy-id ceph-admon
        $ ssh-copy-id ceph-osd1
        $ ssh-copy-id ceph-osd2
        
        
Step 5 - Configure the Ceph OSD Nodes

 we have 2 OSD nodes and each node has two partitions.

    /dev/sda for the root partition.
    /dev/sdb is an empty partition ------  32 GB in my case. ( I am Using Pen Drive )

We will use /dev/sdb for the Ceph disk. From the ceph-admon node, login to all OSD nodes and format the /dev/sdb partition with XFS.

Format the /dev/sdb partition with XFS filesystem and with a GPT partition table by using the parted command. On all OSD Nodes.

       $ sudo parted -s /dev/sdb mklabel gpt mkpart primary xfs 0% 100%
       $ sudo mkfs.xfs /dev/sdb -f
       $ sudo blkid -o value -s TYPE /dev/sdb
       
 Step 6 - Build the Ceph Cluster
 
 Note : We compile the ceph according to step 1. so, we have ceph install on all nodes.
 
        Create the new cluster directory.
        
        $ mkdir ceph-cluster
        $ cd ceph-cluster/

Next, create a new cluster configuration with the 'ceph-deploy' command, define the monitor node to be 'admon'.        
     
        $ ceph-deploy new admon
        
 The command will generate the Ceph cluster configuration file 'ceph.conf' in the ceph-cluster directory.
 
        $ ls
        $ vi ceph.conf
   Under [global] block, paste configuration below.
   
            # Your network address
                public network = 192.168.15.0/24
                osd pool default size = 2
                
 Now install Ceph on all other nodes from the ceph-admon node. This can be done with a single command.
 
        $ ceph-deploy install ceph-admon ceph-osd1 ceph-osd2 
        $ ceph-deploy mon create-initial
        $ ceph-deploy gatherkeys admon
        
Adding OSDS to the Cluster

When Ceph has been installed on all nodes, then we can add the OSD daemons to the cluster. OSD Daemons will create their data and journal partition on the disk /dev/sdb.

Check that the /dev/sdb partition is available on all OSD nodes.

        $ ceph-deploy disk list ceph-osd1 ceph-osd2
        You will see the /dev/sdb disk with XFS format.

Next, delete the /dev/sdb partition tables on all nodes with the zap option.

        $ ceph-deploy disk zap ceph-osd1:/dev/sdb ceph-osd2:/dev/sdb 
        The command will delete all data on /dev/sdb on the Ceph OSD nodes.
        
  Now prepare and activate all OSDS nodes. Make sure there are no errors in the results.
  
        $ ceph-deploy osd prepare ceph-osd1:/dev/sdb ceph-osd2:/dev/sdb 
        $ ceph-deploy osd activate ceph-osd1:/dev/sdb1 ceph-osd2:/dev/sdb1
        
  Next, deploy the management-key to all associated nodes.
  
         $ ceph-deploy admin ceph-admon ceph-osd1 ceph-osd2
         
 Change the permission of the key file by running the command below on all nodes.
 
         $ sudo chmod 644 /etc/ceph/ceph.client.admin.keyring
         
         Ceph Cluster On RBPI has Created

Step 7 - Testing the Ceph setup

          $ sudo ceph -s
          $ sudo ceph health
          
            Make sure Ceph health is OK
            
            
References: 
          
          https://docs.ceph.com/docs/master/start/intro/  
          https://access.redhat.com/documentation/en-us/red_hat_ceph_storage/3/html/installation_guide_for_ubuntu   /what_is_red_hat_ceph_storage
          https://bryanapperson.com/blog/the-definitive-guide-ceph-cluster-on-raspberry-pi/
          https://github.com/CyberHippo/Ceph-Pi

